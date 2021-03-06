---
title: "Projects"
output: md_document
permalink: /projects/
questions:
  - "How do I create a package in R?"
  - "What can go in an R package?"
  - "How are R packages distributed?"
  - "What data formats can be used in an R package?"
  - "How should I document an R package?"
objectives:
  - "Describe the three things an R package can contain."
  - "Explain how R code in a package is distributed and one implication of this."
  - "Explain the purpose of the `DESCRIPTION`, `NAMESPACE` and `.Rbuildignore` files in an R project."
  - "Explain what should be put in the `R`, `data`, `man`, and `tests` directories of an R project."
  - "Describe and use specially-formatted comments with roxygen2 to document a package."
  - "Use `@export` and `@import` directives correctly in roxygen2 documentation."
  - "Add a dataset to an R package."
  - "Use functions from external libraries inside a package in a hygienic way."
  - "Rewrite references to bare column names to satisfy R's packaging checks."
  - "Correctly document the package as a whole and the datasets it contains."
keypoints:
  - "An R package can contain code, data, and documentation."
  - "R code is distributed as compiled bytecode in packages, not as source."
  - "R packages are almost always distributed through CRAN, the Comprehensive R Archive Network."
  - "Most of a project's metadata goes in a file called `DESCRIPTION`."
  - "Metadata related to imports and exports goes in a file called `NAMESPACE`."
  - "Add patterns to a file called `.Rbuildignore` to ignore files or directories when building a project."
  - "All source code for a package must go in the `R` sub-directory."
  - "`library` calls in a package's source code will *not* be executed as the package is loaded after distribution."
  - "Data can be included in a package by putting it in the `data` sub-directory."
  - "Data must be in `.rda` format in order to be loaded as part of a package."
  - "Data in other formats can be put in the `inst/extdata` directory, and will be installed when the package is installed."
  - "Add comments starting with `#'` to an R file to document functions."
  - "Use roxygen2 to extract these comments to create manual pages in the `man` directory."
  - "Use `@export` directives in roxygen2 comment blocks to make functions visible outside a package."
  - "Add required libraries to the `Imports` section of the `DESCRIPTION` file to indicate that your package depends on them."
  - "Use `package::function` to access externally-defined functions inside a package."
  - "Alternatively, add `@import` directives to roxygen2 comment blocks to make external functions available inside the package."
  - "Import `.data` from `rlang` and use `.data$column` to refer to columns instead of using bare column names."
  - "Create a file called <code>R/<em>package</em>.R</code> and document `NULL` to document the package as a whole."
  - "Create a file called <code>R/<em>dataset</em>.R</code> and document the string <code>'<em>dataset</em>'</code> to document a dataset."
---



Mistakes were made in [the previous tutorial](../cleanup/).
It would be [hubris](../glossary/#hubris) to believe that we will not make more as we continue to clean this data.
What will guide us safely through these dark caverns and back into the light of day?

The answer is testing.
We must test our assumptions, test our code, test our very *being* if we are to advance.
Luckily for us,
R provides tools for this purpose not unlike those available in Python.
In order to use them,
we must first venture into the greater realm of packaging in R.

## Packaging

Unlike Python,
with its confusing plethora of packaging tools,
R has one way to do it.
Before converting our project into a package,
we will explore what a package should contain.

-   The text file `DESCRIPTION` (with no suffix) holds most of the package's metadata,
    including a description of what it,
    who wrote it,
    and what other packages it requires to run.
    We will edit its contents as we go along.
-   `NAMESPACE`,
    whose name also has no extension, contains the names of everything exported from the package
    (i.e., everything that is visible to the outside world).
    As we will see,
    we should leave its management in the hands of RStudio.
-   Just as `.gitignore` tells Git what files in a project to ignore,
    `.Rbuildignore` tells RStudio which files it doesn't need to worry about
    when building a package from source.
-   All of the R source for our package must go in a directory called `R`;
    sub-directories are not allowed.
-   As you would expect from its name,
    the `data` directory contains all the data in our package.
    In order for it to be loadable as part of the package,
    the data must be saved in `.rda` format.
    We can use R's function `save` to do this
    (and use `load` in our code to restore it),
    but a better choice is to load the `usethis` library
    and call `usethis::use_data(object, overwrite = TRUE)`.
-   Manual pages go in the `man` directory.
    The bad news is that they have to be in a sort-of-LaTeX format
    that is only a bit less obscure than the runes inscribed on the ancient dagger
    your colleague brought back from her latest archeological dig.
    The good news is,
    we can embed comments in our source code and use a tool called roxygen2
    to extract them and format them as required.
-   The `tests` directory holds the package's unit tests.
    It should contain files with names like <code>test_<em>some_feature</em>.R</code>,
    which should in turn contain functions named <code>test_<em>something_specific</em></code>.
    We'll have a closer look at these later.

In order to understand the rest of what follows,
it's important to understand that R packages are distributed as compiled bytecode,
*not* as source code (which is how Python does it).
When a package is built,
R loads and checks the code,
then saves the corresponding instructions.
Our R files should therefore define functions,
not run commands immediately,
because if they do the latter,
those commands will be executed every time the script loads,
which is probably not what users will want.

As a side effect,
this means that if a package uses `load(something)`,
then that `load` command is executed *while the package is being compiled*,
and *not* while the compiled package is being loaded by a user after distribution.
Thus,
this simple and rather pointless "package":


```r
library(stringr)

sr <- function(text, pattern, replacement) {
  str_replace(text, pattern, replacement)
}
```

probably won't work when it's loaded by a user,
because `stringr` may not be in memory on the user's machine at the time `str_replace` is called.

How then can our packages use libraries?
The safest way is to use [fully-qualified names](../glossary/fully-qualified-name)
such as `stringr::str_replace`
every time we call a function defined somewhere outside our package.
We will explore other options below.

And while we're hedging the statements we have already made:

1.  Data that *isn't* meant to be loaded directly into are should go in `inst/extdata`.
    The first part of the directory name, `inst`, is short for "install":
    when the package is installed,
    everything in this directory is bumped up a level and put in the installation directory.
    Thus,
    the installation directory will get a sub-directory called `extdata` (for "external data"),
    and that can hold whatever we want.
2.  We should always put `LazyData: TRUE` in `DESCRIPTION`
    so that datasets are only loaded on demand.

## Creating a Package

We cannot turn this tutorial into an R package because we're building it as a website, not as a package.
Instead, we will create an R package called `unicefdata` to hold cleaned-up copies of
[some HIV/AIDS data][unicef-hiv] and [maternal health data][unicef-maternal] from UNICEF.

The first step is to run RStudio's project creation wizard.
We will create `unicefdata` directly under our home directory,
make it a Git repository,
and turn on Packrat (a package manager for R):

![RStudio Project Creation Wizard](../files/rstudio-project-creation-wizard.png)

Before doing our first commit to version control,
we remove `R/hello.R` and `man/hello.Rd`
(which the project wizard helpfully provides as starting points),
and add `README.md`, `LICENSE.md`, `CONDUCT.md`, and `CITATION.md`
to describe the project as a whole,
its license,
the contributor code of conduct,
and how we want the project cited.
These files are nothing to do with R per se
(and as we'll see below, R isn't entirely happy having them here with these names),
but every project should have all four of these *somewhere*.

After committing all of this to version control,
we copy the data tidying script we wrote previous into `R/tidy_datasets.R`.
For reference,
this is what the file looks like at this point:


```r
library(tidyverse)

# Constants.
raw_filename <- "inst/extdata/infant_hiv.csv"
tidy_filename <- "/tmp/infant_hiv.csv"
first_year <- 2009
last_year <- 2017
num_rows <- 192

# Get and clean percentages.
raw <- read_csv(raw_filename, skip = 2, na = c("-"))
raw$ISO3[raw$Countries == "Kosovo"] <- "UNK"
sliced <- slice(raw, 1:num_rows)
countries <- sliced$ISO3
percents <- sliced %>%
  select(-ISO3, -Countries) %>%
  map_dfr(str_replace, pattern = ">?(\\d+)%", replacement = "\\1") %>%
  map_dfr(function(col) as.numeric(col) / 100)

# Separate three-column chunks and add countries and years.
num_years <- (last_year - first_year) + 1
chunks <- vector("list", num_years)
for (year in 1:num_years) {
  start = 3 * (year - 1) + 1
  chunks[[year]] <- select(percents, start:(start + 2)) %>%
    rename(estimate = 1, hi = 2, lo = 3) %>%
    mutate(country = countries,
           year = rep((first_year + year) - 1, num_rows)) %>%
    select(country, year, everything())
}

# Combine chunks and order by country and year.
tidy <- bind_rows(chunks) %>%
  arrange(country, year)

# Check.
write_csv(tidy, tidy_filename)
```

We're going to need to wrap this up as a function
so that these commands aren't executed while the library loads,
and we should probably also allow the user to specify the locations of the input and output files---we'll come back
and do all of this later.

First,
though,
let's edit `DESCRIPTION` and:

1.  change the `Title`, `Author`, `Maintainer`, and `Description`;
2.  change the `License` to [`MIT`][mit-license] (see [Choose a License][choose-a-license] for other options); and
3.  go to the `Build` tab in RStudio and run 'Check` to see if our package meets CRAN's standards.

A note on this:
[CRAN][cran] is the Comprehensive R Archive Network.
Like the [Python Package Index][pypi],
it is the place to go to find the packages you need.
CRAN's rules are famously strict,
which ensures that packages run for everyone,
but which also makes package development a little more onerous than it might be.

For example,
when we run `Check`,
we get this:

```
* checking top-level files ... NOTE
Non-standard files/directories found at top level:
  'CITATION.md' 'CONDUCT.md' 'LICENSE.md'
...
Found the following CITATION file in a non-standard place:
  CITATION.md
Most likely 'inst/CITATION' should be used instead.
```

After a bit of searching online,
we rearrange these files as follows:

1.  `LICENSE.md` becomes `LICENSE` with no extension (but still in the root directory).
2.  The `DESCRIPTION` entry for the license is updated to `License: MIT + file LICENSE` (spelled exactly that way).
3.  We add lines to `.Rbuildignore` to tell R to ignore `CITATION.md` and `CONDUCT.md`.
    We could instead move `CITATION.md` to `inst/CITATION`
    so that it will be copied into the root of the installation directory on users' machines,
    but a lot of people expect to find the citation description in the root directory of the original project.
    We could also duplicate the file,
    and once the package is mature enough to deploy,
    that might be the best answer.

These changes fix the warnings about non-standard files in non-standard places,
but we are far from done---we also have this to deal with:

```
checking for missing documentation entries ... WARNING
Undocumented code objects:
  'tidy_infant_hiv'
All user-level objects in a package should have documentation entries.
```

Fair enough---we want people to know what our package is for and how to use it,
so a little documentation seems like a fair request.
For this,
we turn to Hadley Wickham's *[R Packages][wickham-packages]*
and Karl Broman's "[R package primer][broman-packages]"
for advice on writing roxygen2 documentation.
We then return to our source file and wrap our existing code with this:


```r
#' Tidy up the infant HIV data set.
#'
#' @param src_path path to source file
#'
#' @return a tibble of tidy data
#'
#' @export

tidy_infant_hiv <- function(src_path) {
  # ...all the code from before...
  # Return the final tidy dataset.
  tidy
}
```

roxygen2 processes comment lines that start with `#'` (hash followed by single quote).
Putting a comment block right before a function associates that documentation with that function;
here,
we are saying that:

-   the function has a single parameter called `src_path`;
-   it returns a tibble of tidy data; and
-   we want it exported (i.e., we want it to be visible outside the package).

Our function is now documented,
but when we run `Check`,
we still get a warning.
After a bit more searching and experimentation,
we discover that we need to load the `devtools` package
and run `devtools::document()` to regenerate documentation:
it isn't done automatically.
When we do this,
we get good news and bad news:

```
Updating unicefdata documentation
Loading unicefdata
First time using roxygen2. Upgrading automatically...
Updating roxygen version in /Users/gvwilson/unicefdata/DESCRIPTION
Warning: The existing 'NAMESPACE' file was not generated by roxygen2, and will not be overwritten.
Writing tidy_infant_hiv.Rd
```

Ah---the tutorials did warn us about this.
We need to delete `NAMESPACE` and re-run `devtools:document()`
in order to create this file:

```
# Generated by roxygen2: do not edit by hand

export(tidy_infant_hiv)
```

The comment at the start tells roxygen2 it can overwrite the file,
and reminds us that we shouldn't edit it by hand.
The `export(tidy_infant_hiv)` directive is what we really want:
as you might guess,
it tells the package builder which function to make visible.

After doing this,
we go into "Build...More...Configure build tools" and check "Generate documentation with Roxygen".
Running the build again gives us:

```
tidy_infant_hiv: no visible global function definition for 'read_csv'
tidy_infant_hiv: no visible global function definition for 'slice'
tidy_infant_hiv: no visible global function definition for '%>%'
tidy_infant_hiv: no visible global function definition for 'select'
...several more...
tidy_infant_hiv: no visible global function definition for 'bind_rows'
tidy_infant_hiv: no visible global function definition for 'arrange'
Undefined global functions or variables:
  %>% Countries ISO3 arrange bind_rows country everything map_dfr
  mutate read_csv rename select slice str_replace year
```

This is the package loading problem mentioned earlier:
since our compiled-and-distributable package will only contain the bytecodes for its own functions,
direct calls to functions from other libraries won't work after the package is installed.
We will demonstrate two ways to fix this.
First,
we can add this to the roxygen2 comment block for our function:


```r
#' @import dplyr
#' @importFrom magrittr %>%
```

which tells the package builder that we want all of the functions in these two packages available.
Second (and more properly) we can change various call to use their package prefix, e.g.:

```
  percents <- sliced %>%
    select(ISO3, Countries) %>%
    purrr::map_dfr(stringr::str_replace, pattern = ">?(\\d+)%", replacement = "\\1") %>%
    purrr::map_dfr(function(col) as.numeric(col) / 100)
```

This changes the error to one that is slightly more confusing:

```
* checking package dependencies ... ERROR
Namespace dependencies not required: 'dplyr' 'magrittr'

See section 'The DESCRIPTION file' in the 'Writing R Extensions'
manual.
* DONE
Status: 1 ERROR
checking package dependencies ... ERROR
Namespace dependencies not required: 'dplyr' 'magrittr'
```

More searching,
more experimentation,
and finally we add this to the `DESCRIPTION` file:

```
Imports:
    readr (>= 1.1.0),
    dplyr (>= 0.7.0),
    magrittr (>= 1.5.0),
    purrr (>= 0.2.0),
    rlang (>= 0.3.0),
    stringr (>= 1.3.0)
```

The `Imports` field in `DESCRIPTION` actually has nothing to do with importing functions;
it just ensures that those packages are installed when this package is.
As for the version numbers in parentheses,
we got those by running `packageVersion("readr")` and similar commands inside RStudio
and then rounding off.

All right: are we done now?
No, we are not:

```
checking R code for possible problems ... NOTE
tidy_infant_hiv: no visible binding for global variable 'ISO3'
tidy_infant_hiv: no visible binding for global variable 'Countries'
tidy_infant_hiv: no visible binding for global variable 'country'
tidy_infant_hiv: no visible binding for global variable 'year'
```

This is annoying but understandable.
When the package builder is checking our code,
it has no idea what columns are going to be in our data frames,
so it has no way to know if `ISO3` or `Countries` will cause a problem.
However,
this is just a `NOTE`, not an `ERROR`,
so we can try running "Build...Install and Restart"
to build our package,
re-start our R session (so that memory is clean),
and load our newly-created package,
and then run `tidy_infant_hiv("inst/extdata/infant_hiv.csv")`.
This produces:

```
Error in read_csv(src_path, skip = 2, na = c("-")) : 
  could not find function "read_csv"
```

After calling upon the names of Those Who Shall Not Be Named
and making a fresh cup of tea,
we re-read our code and realize that we forgot to rename `read_csv` to `readr::read_csv`.
Fixing this doesn't fix the problem with column names, though;
to do that,
we add this to the roxygen2 comment block:

```
#' @importFrom rlang .data
```

and then modify the calls that use naked column names to be:

```
    select(-.data$ISO3, -.data$Countries) %>%
    ...
    select(.data$country, .data$year, everything())
    ...
    arrange(.data$country, .data$year)
```

What is this `.data` that we have invoked?
Typing `?rlang::.data` gives us the answer:
it is a pronoun that allows us to be explicit when we refer to an object inside the data.
Adding this---i.e.,
being explicity that `country` is a column of `.data` rather than an undefined variable---finally
(finally)
gives us a clean build.

But we are not done,
because we are never *truly* done,
any more than we are ever truly safe.
We still need to add our cleaned-up data to our package
and document the package as a whole.
There are three steps to this.

First, we load and clean the data, storing the cleaned tibble in a variable called `infant_hiv`,
then load the `usethis` package and called `usethis::use_data(infant_hiv)`
to store the tibble in `data/infant_hiv.rda`.
(We could just call `save` with the appropriate parameters,
but `usethis` is a useful set of tools for creating and managing R projects,
and in retrospect,
we should have started using it earlier.)
Note: we *must* save the data as `.rda`, not as (for example) `.rds` or `.csv`;
only `.rda` will be automatically loaded as part of the project.

Second,
we create a file called `R/infant_hiv.R` to hold documentation about the dataset
and put this in it:


```r
#' Tidied infant HIV data.
#'
#' This tidy data is derived from the `infant_hiv.csv` file, which in turn is
#' derived from an Excel spreadsheet provided by UNICEF - see the README.md file
#' in the raw data directory for details.
#'
#' @format A data frame
#' \describe{
#'   \item{country}{Country reporting (ISO3 code)}
#'   \item{year}{Year reported}
#'   \item{estimate}{Best estimate of rate (may be NA)}
#'   \item{hi}{High end of estimate range (may be NA)}
#'   \item{lo}{Low end of estimate range (may be NA)}
#' }
"infant_hiv"
```

Everything except the last line is a roxygen2 comment block
that describes the data in plain language,
then uses some tags and directives to document its format and fields.
(Note that we have also documented our data in `inst/extdata/README.md`,
but that focuses on the format and meaning of the raw data,
not the cleaned-up version.)

The last line is the string `"infant_hiv"`,
i.e.,
the name of the dataset.
We will create one placeholder R file like this for each of our datasets,
and each will have that dataset's name as the thing being documented.

We use a similar trick to document the package as a whole:
we create a file `R/unicefdata.R`
(i.e., a file with exactly the same name as the package)
and put this in it:


```r
#' Clean up and share some data from UNICEF on infant HIV rates and maternal mortality.
#'
#' @author Greg Wilson, \email{greg.wilson@rstudio.com}
#' @docType package
#' @name unicefdata
NULL
```

That's right:
to document the entire package,
we document `NULL`,
which is one of the few times R uses call-by-value.
(That's a fairly clumsy joke,
but honestly,
who among us is at our best at times like these?)

{% include links.md %}
