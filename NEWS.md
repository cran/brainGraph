# brainGraph 1.0.0

2017-04-10

First *major* release; Fifth CRAN version

## Bug fix
* `plot_perm_diffs` previously didn't work with a low number of permutations, but now will work with any number
* `sim.rand.graph.par` previously didn't work with graphs lacking a `degree` vertex attribute
* Fixed problem with `plot_brainGraph_GUI` when plotting in the sagittal view for neighborhood graphs

## Major changes
* Multiple functions now run significantly faster after I updated the code to be more efficient
* `permute.group.auc` has been removed, and now `permute.group` accepts multiple densities and returns the same results. It can still take a single density for the old behavior
* The `lobe` and `network` vertex attributes are now *character* vectors
* `NBS` now handles more complex designs and contrasts through `brainGraph_GLM_design` and `brainGraph_GLM_fit`. The function arguments are different from previous versions
* `SPM` has been removed and is replaced by `brainGraph_GLM`
* Added atlas `craddock200` (with coordinates from `DPABI/DPARSF`)

## New functions
* `brainGraph_GLM`: replaces `SPM` and allows for more complex designs and contrasts
* `brainGraph_GLM_design`: function that creates a design matrix from a `data.table`
* `brainGraph_GLM_fit`: function that calculates the statistics from a design matrix and response vector
* `create_mats`: replaces `dti_create_mats` and adds functionality for resting-state fMRI data; also can create matrices that will have a desired graph density
* `gateway_coeff`: calculate the *gateway coefficient* (Vargas & Wahl, 2014)
* `plot_brainGraph_multi`: function to write a PNG file of 3-panel brain graphs

## Minor changes
* Renamed `graph.efficiency` to `efficiency` (the old function name is still accessible)
* Renamed `set.brainGraph.attributes` to `set_brainGraph_attr`
* Renamed `part.coeff` to `part_coeff`
* All of the `rich.` functions have been renamed. The period/point/dot in each of those functions is replaced by the *underscore*. So, `rich.club.norm` is now `rich_club_norm`, etc.
* Renamed `color.vertices` and `color.edges` to `set_vertex_color` and `set_edge_color`
* Renamed `graph.contract.brain` to `contract_brainGraph`
* Renamed `graph_neighborhood_multiple` to `make_ego_brainGraph` (so it is a similar name to *igraph*'s function `make_ego_graph`)
* Renamed `write.brainnet` to `write_brainnet`

----
# brainGraph 0.72.0

2016-10-10

Fourth CRAN version

## Bug fix
* `sim.rand.graph.clust` previously returned a list; now it correctly returns an
    `igraph` graph object
* `aop` and `loo`: regional contributions were calculated incorrectly (without
    an absolute value)
* `rich.club.norm`: changed the p-value calculation again; this shouldn't affect
    many results, particularly if N=1,000 (random graphs)
* `NBS`:
  * the `t.stat` edge attribute was, under certain situations, incorrectly
    assigning the values; this has been fixed in the latest version
  * fixed bug when permutations didn't result in any connected components
  * fixed bug w/ data randomization; the bug didn't seem to affect the results
* `SPM`:
  * the permutation p-values were previously incorrect; has been fixed
  * added an argument to remove `NA` values
* `vec.transform`: fixed bug which occurred when the input vector is the same
    number repeated (i.e., when `range(x) = 0`)

## Major changes
* `dti_create_mats`: new function argument `algo` can be used to specify either 'probabilistic' or 'deterministic'. In the case of the latter, when dividing streamline count by ROI size, you can supply absolute streamline counts with the `mat.thresh` argument.
* Changed instances of `.parallel` to `use.parallel`; also, added it as an
    argument to `set.brainGraph.attributes` to control all of the functions that
    it calls; also added the argument to `part.coeff` and
    `within_module_deg_z_score`
* Added atlases `aal2.94`, `aal2.120`, and `dosenbach160`
* `plot_brainGraph`: can now specify the orientation plane, hemisphere to plot,
    showing a legend, and a character string of logical expressions for plotting
    subgraphs (previously was in `plot_brainGraph_list`)

## New functions
* `auc_diff`: calculates the area-under-the-curve across densities for two
    groups
* `cor.diff.test`: calculates the significance of the difference between
    correlation coefficients
* `permute.group.auc`: does permutation testing across all densities, and
    returns the permutation distributions for the difference in AUC between two
    groups
* `rich.club.attrs`: give a graph attributes based on rich-club analysis

## Minor changes
* Removed the `x`, `y`, and `z` columns from the atlas data files; now only the
    MNI coordinates are used. This should simplify adding a personal atlas to use
    with the package
* Added a column, `name.full` to some of the atlas data files
* `NBS`:
  * New edge attribute `p`, the p-value for that specific connection
  * Returns the `p.init` value for record-keeping
* `brainGraph_init`: can now provide a `covars` data table if you want to subset
    certain variables yourself, or if the file is named differently from
    `covars.csv`
* `plot_brainGraph`: can now manually specify a subtitle;
* `plot_brainGraph_gui`:
  * Option for specifying maximum values for edge widths
* `plot_corr_mat`: color cells based on weighted community or network
* `plot_global`:
  * legend position is now "bottom" by default
  * can specify `xvar` to be either "density" or "threshold"; if the latter, the
    x-axis is reversed
  * If data has a `Study.ID` column, the `ggplot2` function `stat_smooth` is used
    and the statistic is based on a generalized additive model
* `plot_perm_diffs`: added argument `auc` for using the area-under-the-curve
    across densities
* `plot_rich_norm`:
  * Added argument `fdr` to choose whether or not to use FDR-adjusted p-values
  * Should work for more than 2 groups
  * Now works with multi-subject data; collapses by *Group* and plots the group mean
* `plot_vertex_measures`: can facet by different variables (e.g., lobe, community, network, etc.)
* `set.brainGraph.attributes`:
  * calculate graph `strength`, which is the mean of vertex strength (weighted networks)
  * Invert edge weights for distance-based measures
* `write.brainnet`:
  * Now allows for writing weighted adjacency matrices, using the `edge.wt`
    function argument
  * Can color vertices by multiple variables

---
# brainGraph 0.62.0

2016-04-22

Third CRAN version

## Bug fix
* `rich.club.norm` had a bug in calculating the p-values. If you have already
    gone through the process of creating random graphs and the object `phi.norm`,
    you can fix with the following code: (add another loop if you have
    single-subject graphs, e.g. DTI data)

```
for (i in seq_along(groups)) {
  for (j in seq_along(densities)) {
    max.deg <- max(V(g[[i]][[j]])$degree)
    phi.norm[[i]][[j]]$p <- sapply(seq_len(max.deg), function(x)
        sum(phi.norm[[i]][[j]]$phi.rand[, x] >= phi.norm[[i]][[j]]$phi.orig[x]) / N)
  }
}
```

where `N` is the number of random graphs generated.
* `dti_create_mats`: there was a bug when *sub.thresh* equals 0; it would take
    matrix entries, even if they were below the *mat.thresh* values. This has
    been fixed. Argument checking has also been added.

## Major changes
* Now requires the package `RcppEigen` for fast linear model calculations;
    resulted in major speed improvements
* Now requires the package `permute` for the `NBS` function
* `group.graph.diffs`:
  * Uses the function `fastLmPure` from `RcppEigen` for speed/efficiency
  * Can specify multiple alternative hypotheses
  * Linear model specification is more limited now, though (on *TODO* list)
* Added data table for the `destrieux.scgm` atlas

## New functions
* `SPM`: new function that replaces and improves upon both `group.graph.diffs`
    and `permute.vertex`
* `NBS`: implements the network-based statistic
* `analysis_random_graphs`: perform all the steps for getting *small-world*
    parameters and normalized *rich-club* coefficients and p-values
* `plot_global`: create a line plot across all densities of global graph
    measures in the same figure
* `vertex_spatial_dist`: calculates the mean edge distance for all edges of a
    given vertex

## Minor changes
* `dti_create_mats`: changed a few arguments
* `edge_spatial_dist`: re-named from `spatial.dist`
* `group.graph.diffs`: returns a graph w/ spatial coord's for plotting
* `plot_brainGraph_list`:
  * You can now specify a condition for removing vertices (e.g. `hemi == "R"`
    will keep only right hemisphere vertices; includes complex logical
    expressions (i.e., with multiple '&' and '|' conditions)
  * Vertex sizing and coloring is a bit more flexible
* New vertex attribute `Lp` (average path length for each vertex)
* `plot_brainGraph_gui`:
  * Added a checkbox for displaying a color legend or not
  * Can color vertices by weighted community membership
  * Added an *Other* option for adjusting edge widths by a custom attribute
  * More options for adjusting vertex sizes when the graph is weighted
  * Made the GUI window more compact to fit lower screen resolutions
* `plot_rich_norm`:
  * New argument `facet.by` to group the plots by either "density" (default) or
    "threshold" (for multi-subject, e.g. DTI data)
* `set.brainGraph.attributes`: New calculations for weighted graphs:
  * *Modularity* and community membership
  * *Participation coefficient* and *within-module degree z-score*
  * Vertex-level *transitivity*
  * Vertex-level *shortest path lengths*


---
# brainGraph 0.55.0

2015-12-24

Second CRAN version

## New functions
* `aop` and `loo` calculate measures of *individual contribution* (see Reference
    within the function help)
  * Now requires the package `ade4`
* `plot_boot`: new function based on the removed plotting code from `boot_global`
* `plot_rich_norm`: function to plot normalized rich club coefficient curves

## Minor changes
* `boot_global`:
  * added an OS check to get multicore functionality on Windows
  * removed the code that created some plots
  * updated to work with the newer version of `corr.matrix`
* `brainGraph_init`:
  * does a better job of dealing with subcortical gray matter data
  * now also returns the "tidied" dataset
* `corr.matrix`:
  * was basically reverted back for speed purposes
  * minor syntax change
* `count_interlobar` no longer takes `atlas.dt` as an argument
* `dti_create_mats` now accepts argument `P` for "number of samples"
* `edge_asymmetry` now works on Windows (changed from *mclapply* to *foreach*)
* `get.resid`:
  * got a complete overhaul; now works with *data.table* syntax
  * now returns *data.table* of residuals with a *Study.ID* column
  * fixed minor bug when `use.mean=FALSE` but *covars* has columns
    *mean.lh* and/or *mean.rh*; fixed minor bug w/ RH residual calculation
  * fixed bug when `use.mean=TRUE` (syntax error for RH vertices)
* `graph.efficiency`: now works on Windows (changed from *mclapply* to *foreach*)
* `part.coeff`: has a workaround to work on Windows
* `permute.group`:
  * updated to work with new version of `corr.matrix`
  * no longer takes `atlas.dt` as an argument
* `vertex_attr_dt` is now essentially a wrapper for `igraph`'s function
    `as_data_frame`

* Exported `plot_perm_diffs`
* Added argument checking for most functions

---
# brainGraph 0.48.0

2015-12-08

Initial CRAN acceptance
