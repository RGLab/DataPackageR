# preprocessData
An R package to allow `R CMD preprocessData packagename` to run some preprocessing (i.e. data tidying) on raw data by running code in `packagename/data-raw` and generate standardied data sets or objects in `packagename/data`.


## Why?

You work in R. Say you have raw data that you'd like to clean up, preprocess, and standardize in order to share it with others or use in your own projects. You could do this with a series of shell scripts, and R script, a `knitr` markdown document, or any other number of ways to generate some `.csv` files or R `data.frames` or `data.tables` or other complex R objects that represent your data. You might then save these into the `/data` directory of your new R package, build and distribute. 

But what if your raw data change? Say you gather or receive more data? Because your data are in an R package, you get versioning for free. So, just process your data again and update the `.rda` files in `/data` and you're good to go.

But where did you put that processing code? Maybe you were clever and it's under version control somewhere. Or maybe you weren't so clever, and it's in some directory, now you need to find it.. but is this the version of the code that generated the data in that package? Ideally, the code and the data would live together. This is the general idea behind the whole *literate programming* paradigm.

R package support this during **build** by running code in **vignettes**, but that doesn't quite fit what we want to do here, as the vignettes may rely on some of the data they are supposed to generate, and so forth, and do you really need to generate a 50-page document detailing how the data was cleaned each time the package is built? Ideally, we'd like to separete out the data preprocessing from the build stage, but still have it be an integral part of the R package. 

## How it works

That's where `preprocessData` comes in. It enables the `R CMD preprocessData` command. Based upon `BiocCheck` developed by the BioConductor team, it installs a script (requires admin rights) into `$R_HOME/bin` that enables `R CMD preprocessData packagename` which does the following (based on ideas by Hadley Wickham and Yihui Xie):

- Looks for all `.R` files under the directory `packagename/data-raw` and sources them.

These `.R` files should read some raw data from a source (could be a url, or from `/inst/extdata`, tidy the data, standardize it, and save standardized `data.table`, `data.frame` or other complex `R objects` into `/data`. 

- The user writes `roxygen` documentation for each data set and places it in `/R`.

- `R CMD build packagename` generates `packagename_x.y.z.tar.gz` with documented and standardized data available via the usual `data()` mechanism. Unless `data-raw` is placed in `.Rbuildignore` the code to generate the data sets is distributed with the package.


## Benefits

Using the pacakge build system, the data are versioned, and the code to generate the data is also versioned. There's less room for error in letting data and related analysis to get out of sync, improving reproducibility. As an example, an analysis based on a data set could appear as a `vignette` or series of `vignettes` in the package. Alternately, analysis code can explicitly test the installed *version* of a data package.

Unit tests to verify the integrity of the data can be built into `/tests` and run automatically during build. 

## Ongoing work

While tools like `packrat` and others aim to do the above by fixing the version of all packages used for a project, this can be a bit of overkill, particularly if all consumers of a data set are using the same environment. Recording the `sessionInfo()` may be sufficient in many cases (since we are only concerned with data tidying). Some new tools from Robert Gentleman's group could enable users to store and verify the versions of packages used for preprocessing data sets between package builds. 

Documentation could be embedded in the `.R` files under `/data-raw`, which could be parsed out by `processData()` and placed in `/R` then `roxygenized`, eliminating the need for this additional step.


