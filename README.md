
<!-- README.md is generated from README.Rmd. Please edit that file -->

# <img src="inst/extdata/logo.png" align="left" height=150/> pathfindR: An R Package for Enrichment Analysis Utilizing Active Subnetworks

<!-- badges: start -->

[![Travis-CI Build
Status](https://travis-ci.org/egeulgen/pathfindR.svg?branch=master)](https://travis-ci.org/egeulgen/pathfindR)
[![Codecov test
coverage](https://codecov.io/gh/egeulgen/pathfindR/branch/master/graph/badge.svg)](https://codecov.io/gh/egeulgen/pathfindR)
[![CRAN
version](http://www.r-pkg.org/badges/version-ago/pathfindR)](https://cran.r-project.org/package=pathfindR)
[![CRAN total
downloads](https://cranlogs.r-pkg.org/badges/grand-total/pathfindR)](https://cran.r-project.org/package=pathfindR)
[![Lifecycle:
maturing](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)
[![License:
MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
<!-- badges: end -->

## Overview

`pathfindR` is a tool for enrichment analysis via active subnetworks.
The package also offers functionalities to cluster the enriched terms
and identify representative terms in each cluster, to score the enriched
terms per sample and to visualize analysis results.

The functionalities of pathfindR is described in detail in *Ulgen E,
Ozisik O, Sezerman OU. 2019. pathfindR: An R Package for Comprehensive
Identification of Enriched Pathways in Omics Data Through Active
Subnetworks. Front. Genet. <https://doi.org/10.3389/fgene.2019.00858>*

See [the pathfindR wiki](https://github.com/egeulgen/pathfindR/wiki) for
detailed documentation.

## Installation

You can install the released version of pathfindR from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("pathfindR")
```

And the development version from [GitHub](https://github.com/) with:

``` r
install.packages("devtools") # if you have not installed "devtools"
devtools::install_github("egeulgen/pathfindR")
```

> **IMPORTANT NOTE** For the active subnetwork search component to work,
> the user must have [Java](https://www.java.com/en/download/manual.jsp)
> installed and path/to/java must be in the PATH environment variable.

We also have docker images available on Docker Hub:

``` bash
# pull image for latest release
docker pull egeulgen/pathfindr:latest

# pull image for specific version (e.g. 1.3.0)
docker pull egeulgen/pathfindr:1.3.0

# pull image for development version
docker pull egeulgen/pathfindr:dev
```

See the [wiki
page](https://github.com/egeulgen/pathfindR/wiki/Installation) for more
details.

## Enrichment Analysis with pathfindR

![pathfindR Enrichment Workflow](./vignettes/pathfindr.png?raw=true
"pathfindr Enrichment Workflow")

This workflow takes in a data frame consisting of “gene symbols”,
“change values” (optional) and “associated p values”:

| Gene\_symbol | logFC  | FDR\_p  |
| :----------- | :----: | :-----: |
| FAM110A      | \-0.69 | 3.4e-06 |
| RNASE2       |  1.35  | 1.0e-05 |
| S100A8       |  1.54  | 3.5e-05 |
| S100A9       |  1.03  | 2.3e-04 |

After input testing, any gene symbol that is not in the chosen
protein-protein interaction network (PIN) is converted to an alias
symbol if there is an alias that is in the PIN. After mapping the input
genes with the associated p values onto the PIN, active subnetwork
search is performed. The resulting active subnetworks are then filtered
based on their scores and the number of significant genes they contain.

> An active subnetwork can be defined as a group of interconnected genes
> in a protein-protein interaction network (PIN) that predominantly
> consists of significantly altered genes. In other words, active
> subnetworks define distinct disease-associated sets of interacting
> genes, whether discovered through the original analysis or discovered
> because of being in interaction with a significant gene.

These filtered list of active subnetworks are then used for enrichment
analyses, i.e. using the genes in each of the active subnetworks, the
significantly enriched terms (pathways/gene sets) are identified.
Enriched terms with adjusted p values larger than the given threshold
are discarded and the lowest adjusted p value (over all active
subnetworks) for each term is kept. This process of `active subnetwork
search + enrichment analyses` is repeated for a selected number of
iterations, performed in parallel. Over all iterations, the lowest and
the highest adjusted-p values, as well as number of occurrences over all
iterations are reported for each significantly enriched term.

This workflow can be run using the function `run_pathfindR()`:

``` r
library(pathfindR)
output_df <- run_pathfindR(input_df)
```

This wrapper function performs the active-subnetwork-oriented enrichment
analysis and returns a data frame of enriched terms (as well as
visualization of enriched terms and an HTML report):

| ID       | Term\_Description         | Fold\_Enrichment | occurrence | lowest\_p | highest\_p | Up\_regulated         | Down\_regulated |
| :------- | :------------------------ | :--------------: | :--------: | :-------: | :--------: | :-------------------- | :-------------- |
| hsa03040 | Spliceosome               |      3.0975      |     10     |  2.6e-10  |  3.7e-09   | NDUFA1, NDUFB3, UQCRQ | SNRPB, SF3B2    |
| hsa05012 | Parkinson disease         |      2.3188      |     10     |  2.7e-08  |  2.7e-08   | COX7C                 | UBE2G1, VDAC1   |
| hsa00190 | Oxidative phosphorylation |      2.5240      |     10     |  2.9e-08  |  2.9e-08   | DDIT3, NDUFA1         | ATP6V0E2        |

<img src="inst/extdata/example_output-1.png" width="100%" />

Some useful arguments are:

``` r
# change the output directory
output_df <- run_pathfindR(input_df, output_dir = "/top/secret/results")

# change the gene sets used for analysis (default = "KEGG")
output_df <- run_pathfindR(input_df, gene_sets = "GO-MF")

# change the PIN for active subnetwork search (default = Biogrid)
output_df <- run_pathfindR(input_df, pin_name_path = "IntAct")
# or use an external PIN of your choice
output_df <- run_pathfindR(input_df, pin_name_path = "/path/to/myPIN.sif")

# change the number of iterations (default = 10)
output_df <- run_pathfindR(input_df, iterations = 25) 

# report the non-significant active subnetwork genes (for later analyses)
output_df <- run_pathfindR(input_df, list_active_snw_genes = TRUE)
```

The available PINs are “Biogrid”, “STRING”, “GeneMania”, “IntAct”,
“KEGG” and “mmu\_STRING”. The available gene sets are “KEGG”,
“Reactome”, “BioCarta”, “GO-All”, “GO-BP”, “GO-CC”, “GO-MF”, and
“mmu\_KEGG”. You also use a custom PIN (see `?return_pin_path`) or a
custom gene set (see `?fetch_gene_set`)

See the [wiki
page](https://github.com/egeulgen/pathfindR/wiki/Enrichment%20Documentation)
for more details.

## Clustering of the Enriched Terms

![Enriched Terms Clustering
Workflow](./vignettes/term_clustering.png?raw=true
"Enriched Terms Clustering Workflow") The wrapper function for this
workflow is `cluster_enriched_terms()`.

This workflow first calculates the pairwise kappa statistics between the
enriched terms. The function then performs hierarchical clustering (by
default), automatically determines the optimal number of clusters by
maximizing the average silhouette width and returns a data frame with
cluster assignments.

``` r
# default settings
clustered_df <- cluster_enriched_terms(output_df)

# display the heatmap of hierarchical clustering
clustered_df <- cluster_enriched_terms(output_df, plot_hmap = TRUE)

# display the dendrogram and automatically-determined clusters
clustered_df <- cluster_enriched_terms(output_df, plot_dend = TRUE)

# change agglomeration method (default = "average") for hierarchical clustering
clustered_df <- cluster_enriched_terms(output_df, clu_method = "centroid")
```

Alternatively, the `fuzzy` clustering method (as described in Huang DW,
Sherman BT, Tan Q, et al. The DAVID Gene Functional Classification Tool:
a novel biological module-centric algorithm to functionally analyze
large gene lists. Genome Biol. 2007;8(9):R183.) can be used:

``` r
clustered_df_fuzzy <- cluster_enriched_terms(output_df, method = "fuzzy")
```

See the [wiki
page](https://github.com/egeulgen/pathfindR/wiki/Clustering%20Documentation)
for more details.

## Term-Gene Graph Visualization

The function `term_gene_graph()` (adapted from the Gene-Concept network
visualization by the R package `enrichplot`) can be utilized to
visualize which significant genes are involved in the enriched terms.
The function creates the term-gene graph, displaying the connections
between genes and biological terms (enriched pathways or gene sets).
This allows for the investigation of multiple terms to which significant
genes are related. The graph also enables determination of the degree of
overlap between the enriched terms by identifying shared and/or distinct
significant genes.

![Term-Gene Graph](./vignettes/term_gene.png?raw=true "Term-Gene Graph")

For more details, see the [wiki
page](https://github.com/egeulgen/pathfindR/wiki/Term-Gene-Graph).

## Per Sample Enriched Term Scores

![Agglomerated Scores for all Enriched Terms per
Sample](./vignettes/score_hmap.png?raw=true "Scoring per Sample")

The function `score_terms()` can be used to calculate the agglomerated z
score of each enriched term per sample. This allows the user to
individually examine the scores and infer how a term is overall altered
(activated or repressed) in a given sample or a group of samples.

See the [wiki
page](https://github.com/egeulgen/pathfindR/wiki/Enriched-Term-Scoring)
for more details.
