# pathfindR 1.4.0

## Major Changes
- Replaced most occurrences of "pathway" to "term". This was adapted because "term" reflects the utility of the package better. The enrichment and clustering approaches work with any kind of gene set data (be it pathway gene sets, gene ontology gene sets, motif gene sets etc.) Accordingly:
  - `DESCRIPTION` was updated
  - The functions `annotate_pathway_DEGs()`, `calculate_pw_scores()`, `cluster_pathways()`, `fuzzy_pw_clustering()`, `hierarchical_pw_clustering()`, `visualize_pw_interactions()` and `visualize_pws()` were renamed to 
  `annotate_term_DEGs()`, `score_terms()`, `cluster_enriched_terms()`, `fuzzy_term_clustering()`, `hierarchical_term_clustering()`, `visualize_term_interactions()` and `visualize_terms()` respectively
  - The Rmd template file for the report `enriched_pathways.Rmd` was renamed to `enriched_terms.Rmd`
  - All the Rmd template files for the report were updated
  - Documentation of each function was updated accordingly
- Added the visualization function `term_gene_graph()`, which creates a graph of enriched terms - involved genes
- Made changes in `enrichment()` and `enrichment_analyses()` to get enrichment results faster
- Added the function `fetch_gene_set()` for obtaining gene set data more easily
- Terms in gene sets can now be filtered according to the number of genes a term contains (controlled by `min_gset_size`, `max_gset_size` in `fetch_gene_set()` and `run_pathfindR()`) 
- Added the argument `gaCrossover` during active subnetwork search which controls the probability of a crossover in GA (default = 1, i.e. always perform crossover)
- Added unit tests using `testthat`
- Updated all gene sets data
- Updated all RA example data
- The vignettes were updated
- Updated all PIN data
- Improved speed of kappa matrix calculation (`create_kappa_matrix()`)
- Added vignette for non-Homo-sapiens organisms
- Added Mus musculus (mmu) data:
  - `mmu_kegg_genes` & `mmu_kegg_descriptions`: mmu KEGG gene sets data
  - mmu STRING PIN
  - `myeloma_input` & `myeloma_output`: example mmu input and output data
- Added the STRING PIN (combined score >= 400)
- The argument `sig_gene_thr` in subnetwork filtering via `filterActiveSnws()` now serves the threshold proportion of significant genes in the active subnetwork. e.g., if there are 100 significant genes and `sig_gene_thr = 0.03`, subnetwork that contain at least 3 (100 x 0.03) significant genes will be accepted for further analysis
- Removed `pathview` dependency by implementing colored pathway diagram visualization function using `KEGGREST` and `KEGGgraph`

## Minor changes and bug fixes
- In `hierarchical_term_clustering()`, redefined the distance measure as `1 - kappa statistic`
- Fixed minor issue in `cluster_graph_vis()` (during the calculations for additional node colors)
- Removed title from graph visualization of hierarchical clustering in `cluster_graph_vis()`
- In `active_snw_search()`, unnecessary warnings during active subnetwork search were removed
- Fixed minor issue in `enrichment_chart()`, supplying fuzzy clustered results no longer raises an error
- Added new checks in `input_testing()` and `input_processing()` to ensure that both the initial input data frame and the processed input data frame for active subnetwork search contain at least 2 genes (to fix the corner case encountered in issue #17)
- Fixed minor issue in `enrichment_chart()`, ensuring that bubble sizes displayed in the legend (proportional to # of DEGs) are integers
- In `enrichment_chart()`, added the arguments `num_bubbles` (default is 4) to control number of bubbles displayed in the legend and `even_breaks` (default is `TRUE`) to indicate if even increments of breaks are required
- Updated the logo
- Minor fix in `term_gene_graph()` (create the igraph object as an undirected graph for better auto layout)
- Minor fix in `visualize_term_interactions()`. The legend no longer displays "Non-input Active Snw. Genes" if they were not provided
- The argument `human_genes` in `run_pathfindR()` and `input_processing()` was renamed as `convert2alias`
- The gene symbols in the input data frame, the PIN and the gene sets are now turned into uppercase (for obtaining the best overlap)
- Added the argument `top_terms` to `enrichment_chart()`, controlling the number top enriched terms to plot (default is 10)
- Other minor bug/error fixes

***

# pathfindR 1.3.0

## Major Changes
- Separated the steps of the function `run_pathfindR` into individual functions: `active_snw_search`, `enrichment_analyses`, `summarize_enrichment_results`, `annotate_pathway_DEGs`, `visualize_pws`.
- renamed the function `pathmap` as `visualize_hsa_KEGG`, updated the function to produce different visualizations for inputs with binary change values (ordered) and no change values (the `input_processing` function, assigns a change value of 100 to all).
- Created new the visualization function `visualize_pw_interactions`, which creates PNG files visualizing the interactions (in the selected PIN) of genes involved in the given pathways.
- Added new vignette, describing the step-by-step execution of the pathfindR workflow
- Changed clustering metric to kappa statistic, created the new clustering related functions `create_kappa_matrix`, `hierarchical_pw_clustering`, `fuzzy_pw_clustering` and `cluster_pathways`.
- Implemented the new function `cluster_graph_vis` for visualizing graph diagrams of clustering results.

## Minor changes and bug fixes
- Fixed the bug where the arguments `score_quan_thr` and `sig_gene_thr` for `run_pathfindR` were not being utilized.
- in `run_pathfindR`, added message at the end of run, reporting the number enriched pathways.
- the function `run_pathfindR` now creates a variable `org_dir` that is the "path/to/original/working/directory". `org_dir` is used in multiple functions to return to the original working directory if anything fails. This changes the previous behavior where if a function stopped with an error the directory was changed to "..", i.e. the parent directory. This change was adapted so that the user is returned to the original working directory if they supply a recursive output folder (`output_dir`, e.g. "./ALL_RESULTS/RESULT_A"). 
- in `input_processing`, added the argument `human_genes` to only perform alias symbol conversion when human gene symbols are provided. - Updated the Rmd files used to create the report HTML files
- Added the data for `GO-All`, all annotations in the GO database (BP+MF+CC)
- Updated the vignette `pathfindR - An R Package for Pathway Enrichment Analysis Utilizing Active Subnetworks` to reflect the new functionalities.

***

# pathfindR 1.2.3
## Minor changes and bug fixes
- in the function `plot_scores`, added the argument `label_cases` to indicate whether or not to label the cases in the pathway scoring heatmap plot. Also added the argument `case_control_titles` which allows the user to change the default "Case" and "Control" headers. Also added the arguments `low` and `high` used to change the low and high end colors of the scoring color gradient.
- in the function `plot_scores`, reversed the color gradient to match the coloring scheme used by pathview (i.e. red for positive values, green for negative values)
- minor change in `parseActiveSnwSearch`, replaced `score_thr` by `score_quan_thr`. This was done so that the scoring filter for active subnetworks could be performed based on the distribution of the current active subnetworks and not using a constant empirical score value threshold.
- minor change in `parseActiveSnwSearch`, increased `sig_gene_thr` from 2 to 10 as we observed in most of the cases, this resulted in faster runs with comparable results.
- in `choose_clusters`, added the argument `p_val_threshold` to be used as p value threshold for filtering the enriched pathways prior to clustering.

***

# pathfindR 1.2.2

## Major Changes
- fixed issue related to the package `pathview`.
## Minor changes and bug fixes
- in the function `choose_clusters`, added option to use pathway names instead of pathway ids when visualizing the clustering dendrogram and heatmap.

***

# pathfindR 1.2.1

## Major Changes
- Added the option to specify a custom gene set when using `run_pathfindR`. For this, the `gene_sets` argument should be set to "Custom" and `custom_genes` and `custom_pathways` should be provided.

## Minor changes and bug fixes
- fixed minor bug in `calculate_pw_scores` where if there was one DEG, subsetting the experiment matrix failed
- added if condition to check if there were DEGs in `calculate_pw_scores`. If there is none, the pathway is skipped.
- in `calculate_pw_scores`, if `cases` are provided, the pathways are reordered before plotting the heat map and returning the matrix according to their activity in `cases`. This way, "up" pathways are grouped together, same for "down" pathways.
- in `calculate_pwd`, if a pathway has perfect overlap with other pathways, change the correlation value with 1 instead of NA.
- in `choose_clusters`, if `result_df` has less than 3 pathways, do not perform clustering.
- `run_pathfindR` checks whether the output directory (`output_dir`) already exists and if it exists, now appends "(1)" to `output_dir` and displays a warning message. This was implemented to prevent writing over existing results.
- in run `run_pathfindR`, recursive creation for the output directory (`output_dir`) is now supported.
- in run `run_pathfindR`, if no pathways are found, the function returns an empty data frame instead of raising an error.

***

# pathfindR 1.2

## Major Changes
- Implemented the (per subject) pathway scoring function `calculate_pw_scores` and the function to plot the heatmap of pathway scores per subject `plot_scores`.

- Added the `auto` parameter to `choose_clusters`. When `auto == TRUE` (default), the function chooses the optimal number of clusters `k` automatically, as the value which maximizes the average silhouette width. It then returns a data frame with the cluster assignments and the representative/member statuses of each pathway.

- Added the `Fold_Enrichment` column to the resulting data frame of `enrichment`, and as a corollary to the resulting data frame of `run_pathfindR`.

- Added the option `bubble` to plot a bubble chart displaying the enrichment results in `run_pathfindR` using the helper function `enrichment_chart`. To plot the bubble chart set `bubble = TRUE` in `run_pathfindR` or use `enrichment_chart(your_result_df)`. 

## Minor changes and bug fixes
- Add the parameter `silent_option` to `run_pathfindR`. When `silent_option == TRUE` (default), the console outputs during active subnetwork search are printed to a file named "console_out.txt". If `silent_option == FALSE`, the output is printed on the screen. Default was set to `TRUE` because multiple console outputs are simultaneously printed when running in parallel.

- Added the `list_active_snw_genes` parameter to `run_pathfindR`. When `list_active_snw_genes == TRUE`, the function adds the column `non_DEG_Active_Snw_Genes`, which reports the non-DEG active subnetwork genes for the active subnetwork which was enriched for the given pathway with the lowest p value.

- Added the data `RA_clustered`, which is the example output of the clustering workflow.

- In the function, `run_pathfindR` added the option to specify the argument `output_dir` which specifies the directory to be created under the current working directory for storing the result HTML files. `output_dir` is "pathfindR_Results" by default.

- `run_pathfindR` now checks whether the output directory (`output_dir`) already exists and if it exists, stops and displays an error message. This was implemented to prevent writing over existing results.

- `genes_table.html` now contains a second table displaying the input gene symbols for which there were no interactions in the PIN.

***

# pathfindR 1.1

## Major changes
- Added the `gene_sets` option in `run_pathfindR` to chose between different gene sets. Available gene sets are `KEGG`, `Reactome`, `BioCarta` and Gene Ontology gene sets (`GO-BP`, `GO-CC` and `GO-MF`).
- `cluster_pathways` automatically recognizes the ID type and chooses the gene sets accordingly.

## Minor changes and bug fixes
- Fixed issue regarding p values < 1e-13. No active subnetworks were found when there were p values < 1e-13. These are now changed to 1e-13 in the function `input_processing`.
- In `input_processing`, genes for which no interactions are found in the PIN are now removed before active subnetwork search
- Duplicated gene symbols no longer raise an error. If there are duplicated symbols, the lowest p value is chosen for each gene symbol in the function `input_processing`.
- To prevent the formation of nested folders, by default and on errors, the function `run_pathfindR` returns to the user's working directory.
- Citation information are now provided for our BioRxiv pre-print.
