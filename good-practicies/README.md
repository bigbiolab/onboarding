Table of Content
---

- [Reproducible Research](#reproducible-research)
  - [Reproducibility from Bench to Code: *B2C*](#reproducibility-from-bench-to-code-b2c)
- [Project Management](#project-management)
  - [Setting Up R Environment with `pak`](#setting-up-r-environment-with-pak)
  - [Work in Self-Contained Projects](#work-in-self-contained-projects)
  - [Avoid the `setw()` Nightmare](#avoid-the-setw-nightmare)
- [Coding Practices](#coding-practices)
  - [Write Code for People, Not Computers](#write-code-for-people-not-computers)
  - [Always Use Version Control](#always-use-version-control)
  - [Error Handling](#error-handling)
- [Data Management](#data-management)
  - [Data Integrity](#data-integrity)
  - [Metadata and Study Design are Important](#metadata-and-study-design-are-important)
- [Research Collaborations](#research-collaborations)
- [Record Keeping](#record-keeping)
- [Lab Preferred Approaches](#lab-preferred-approaches)
  - [Statistical and General Data Analysis](#statistical-and-general-data-analysis)
  - [Version Control](#version-control)
- [References and Further Reading](#references-and-further-reading)

---

This TOC can help users navigate through the sections of your README more easily. Let me know if you need any changes!

# Reproducible Research

## Reproducibility from bench to code: *B2C*

-   **All research should be reproducible from a technical standpoint:**
    -   If two runs of the same code produce different results, these are not valid results.
-   **Never edit files using a GUI:**
    -   GUI edits (like in Excel) can introduce human error due to the lack of traceability.
    -   If provided with Excel or csv files which require editing or transforming, write code to do this.
        1.  You will know exactly what you have done, and so will other researchers
        2.  The chances of you having to repeat the editing with revised data are high, so write the code once
-   **Reproducible doesn't mean correct**
    -   Reproducible code with errors will simply reproduce the errors. Make sure your code is analytically sound.

> Note: Some of the guidelines here are specific for RStudio, please feel free to add the appropiate guideline if you work on a different IDE

# Project Management

## Setting Up R Environment with [`pak`](https://github.com/r-lib/pak)

All our bioinformatics projects use `pak` as the package manager for R. `pak` is a modern, fast, and user-friendly alternative to traditional package management tools, making it much easier to set up and maintain R environments for bioinformatics analysis.

### Why We Use `pak` for Bioinformatics

`pak` is the preferred package manager in our lab for several important reasons:

-   **Speed**: `pak` is significantly faster than traditional methods, downloading and installing packages in parallel.
-   **Simplicity**: Much easier to use than `renv` - no complex lock files or environment restoration steps.
-   **Comprehensive Support**: Seamlessly installs packages from:
    -   CRAN (standard R packages)
    -   Bioconductor (bioinformatics-specific packages)
    -   GitHub (development versions and custom packages)
    -   Local files
-   **Smart Dependency Resolution**: Automatically handles complex package dependencies common in bioinformatics workflows.
-   **Better Error Messages**: Clear, actionable error messages that help you fix problems quickly.

### Installing `pak` for the First Time

#### Step 1: Install `pak` in R

Open RStudio or R console and run:

``` r
install.packages("pak")
```

That's it! `pak` is now ready to use.

#### Step 2: Verify Installation

Check that `pak` is installed correctly:

``` r
library(pak)
pak::pak_sitrep()
```

This will show you the status of your `pak` installation and configuration.

### Using `pak` for Bioinformatics Packages

#### Installing CRAN Packages

For standard R packages from CRAN:

``` r
# Install a single package
pak::pkg_install("tidyverse")

# Install multiple packages at once
pak::pkg_install(c("ggplot2", "dplyr", "readr", "data.table"))
```

#### Installing Bioconductor Packages

For bioinformatics-specific packages from Bioconductor:

``` r
# Install Bioconductor packages (pak handles this automatically!)
pak::pkg_install("DESeq2")
pak::pkg_install("limma")

# Install multiple Bioconductor packages
pak::pkg_install(c(
  "GenomicRanges",
  "SummarizedExperiment",
  "biomaRt"
))
```

#### Installing Packages from GitHub

For development versions or lab-specific packages:

``` r
# Install from GitHub repository
pak::pkg_install("hadley/ggplot2")

# Install a specific version or branch
pak::pkg_install("user/package@v1.2.3")
pak::pkg_install("user/package@dev-branch")
```

### Common Bioinformatics Setup Workflow

Here's a typical workflow for setting up R for a bioinformatics project:

#### 1. Create a New R Project

-   In RStudio: `File -> New Project -> New Directory -> New Project`
-   Name your project (e.g., "rna-seq-analysis")
-   Choose a location in your working directory

#### 2. Install Core Bioinformatics Packages

Create a setup script (`setup.R`) in your project:

``` r
# Install pak if not already installed
if (!requireNamespace("pak", quietly = TRUE)) {
  install.packages("pak")
}

# Core data manipulation packages
pak::pkg_install(c(
  "tidyverse",      # Data manipulation and visualization
  "data.table",     # Fast data processing
  "readxl"          # Reading Excel files
))

# Bioconductor packages for genomics
pak::pkg_install(c(
  "DESeq2",              # Differential expression analysis
  "edgeR",               # RNA-seq analysis
  "limma",               # Linear models for microarray/RNA-seq
  "GenomicRanges",       # Genomic interval operations
  "rtracklayer",         # Import/export genomic data
  "biomaRt",             # Access Ensembl database
  "clusterProfiler",     # Functional enrichment analysis
  "ComplexHeatmap"       # Advanced heatmaps
))

# Additional visualization packages
pak::pkg_install(c(
  "pheatmap",       # Pretty heatmaps
  "ggrepel",        # Better text labels in plots
  "patchwork"       # Combine multiple plots
))
```

#### 3. Run Your Setup Script

In the R console:

``` r
source("setup.R")
```

`pak` will install all packages in parallel, making the process much faster than traditional methods.

### Updating Packages

Keep your packages up to date:

``` r
# Update all packages
pak::pkg_install("?all")

# Update specific packages
pak::pkg_install("tidyverse", upgrade = TRUE)
```

### Troubleshooting Common Issues

#### Problem: Package installation fails

**Solution**: Check the error message from `pak`. It usually provides clear guidance. Common issues:

``` r
# If you need system libraries, pak will tell you which ones
# On Ubuntu/Debian, you might need:
# sudo apt-get install libcurl4-openssl-dev libssl-dev libxml2-dev

# On macOS with Homebrew:
# brew install curl openssl libxml2
```

#### Problem: Package conflicts

**Solution**: `pak` automatically resolves most conflicts. If issues persist:

``` r
# Remove the problematic package and reinstall
remove.packages("package_name")
pak::pkg_install("package_name")
```

#### Problem: Need a specific package version

**Solution**: Use the `@` syntax:

``` r
# Install a specific version
pak::pkg_install("package_name@1.2.3")
```

### Documenting Your Environment

While `pak` doesn't use lock files like `renv`, it's still important to document your environment:

#### Create a `packages.R` file in your project:

``` r
# packages.R - List of required packages for this project
# Run this file to install all dependencies

required_packages <- c(
  # Data manipulation
  "tidyverse@2.0.0",
  "data.table",

  # Bioconductor packages
  "Bioconductor/DESeq2@1.42.0",
  "Bioconductor/clusterProfiler",

  # Visualization
  "ggplot2",
  "ComplexHeatmap"
)

# Install all packages
pak::pkg_install(required_packages)
```

This allows collaborators to quickly set up the same environment by running:

``` r
source("packages.R")
```

### Best Practices for Lab Members

1.  **Start every new project with a fresh R session**: `Session -> Restart R`
2.  **Document required packages** in a `packages.R` file at the start of your analysis
3.  **Use specific version numbers** for critical packages in publications (e.g., `DESeq2@1.42.0`)
4.  **Update packages regularly**, but test your code after updates
5.  **Share your `packages.R` file** with collaborators via GitHub
6.  **Keep pak updated**: Run `pak::pkg_install("pak")` monthly

### Quick Reference Card

``` r
# Install pak
install.packages("pak")

# Install CRAN package
pak::pkg_install("package_name")

# Install Bioconductor package
pak::pkg_install("Bioconductor/package_name")

# Install from GitHub
pak::pkg_install("username/repo")

# Install multiple packages
pak::pkg_install(c("pkg1", "pkg2", "Bioconductor/pkg3"))

# Update all packages
pak::pkg_install("?all")

# Check pak status
pak::pak_sitrep()
```

### Additional Resources

-   [pak GitHub Repository](https://github.com/r-lib/pak)
-   [pak Documentation](https://pak.r-lib.org/)
-   [Bioconductor Installation Guide](https://bioconductor.org/install/)

## Work in self-contained projects

-   All the files required to run your analysis must be contained within the same folder.
-   Avoid reading a file from a different location in your computer or on the web.
-   I strongly recommend the following folder structure

``` r
project/
- README.md   # Project description written in Markdown
- R/          # Where the R scripts live
- input/      # Data files
- output/     # Results: plots, tables..
```

## Avoid the `setw()` nightmare

-   `setwd()` sets the working directory of your R session, meaning all file paths are based on that location. Hardcoding file paths can create issues when the project is moved.
-   This means that you'll have to update `setw()` *every* time (if) you move your folder
-   This is unproductive and a bad working experience in collaborative projects
-   Use [R Studio Project](https://intro2r.com/rsprojs.html) to improve portability and reproducibility
-   I recommend [Project-oriented workflow](https://www.tidyverse.org/blog/2017/12/workflow-vs-script/) by Jenny Bryan

# Coding Practices

## Write code for people, not computers

-   **Variable names must be easy to interpret:**
    -   e.g. a list of genomes should be called `genomeList` not `g`
    -   Don't call objects `results` and `results2`. Include information in the name.
    -   Typing extra characters is easy using auto-complete
-   **Comment your code extensively:**
    -   You will have to revisit most analyses in the future. Write comments to remind yourself of what you've done.
    -   Collaborators with less coding experience than you will often need to read your code. Communication is much easier if they can understand what you've actually done.
-   **Write intelligent comments**
    -   Comments such as `# Load all data` could be better written as `# The data is spread across 183 csv files in the directory /myData. Load all into a single object`
    -   Too much information in a comment never broke a program
-   **Be stylistically consistent**
    -   This just makes your code easier to follow for everyone

## Always Use Version Control

-   **Version control is not just for package or software development.**
    -   Use version control for *every analysis*!
    -   Only use private repositories if required
-   **Be consistent across an organisation**
    -   *Git* is the version control methodology used by the Bioinformatics Hub
-   **Make incremental changes**
    -   Recompile or rerun regularly to minimize coding errors.
    -   *Stage after a successful run*: After you've confirmed that a specific change works save that change in version control (such as Git).
-   **Write informative commit messages**
    -   This makes it easier to find old code
-   **Use a remote code repository**
    -   Push your code daily. This provides an up-to-date copy in the case of disk failure
    -   This also enables collaborators to contribute

## Error Handling

-   **Include Checks:**
    -   These can include simple things like the presence of a file with the correct name, to verification of the output of a function. Another useful (and necessary) check is data structure and summary checks to ensure you are working with the intended data.
-   **Error Handling:**
    -   You will make mistakes! Often.
    -   Write code to fail quickly and clearly on errors
    -   Write clear error messages

# Data Management

## Data Integrity

-   **Keep a copy of all source data**
    -   Utilize storage which is backed up. Please review our Data BackUp guidelines in the On Boarding documentation
-   **Make the source data read-only**
    -   This prevent accidental deletion or editing as a result of poorly written, or copied scripts can easily do this
    -   This setting can easily be temporarily changed as required

## Metadata and Study Design are important

-   **Always begin an analysis by describing the data, motivation and the intended path to the results.**
-   Linking the publication that explains the experimental design is acceptable
-   **Where possible, make raw data available**
    -   At the very least, key collaborators should know where to find it.

# Research Collaborations

-   **Always confirm with the lead researcher (Lead Student or PI) before collaborating** (or adding new collaborators)
-   **Agree on the key positions of authors in advance**
    -   Any significant contribution should be rewarded with co-authorship

# Record Keeping

-   A lab book is a **requirement** for both in-lab and in-silico experiments to ensure completeness in research documentation.
-   **Never use a GUI**
    -   Ever
    -   But, if for some kind *Meteor hits the Earth* situation you had to edit your table in Excel or similar (guilty of charge), please always keep the raw data as exported/generated in a dedicated folder. And please, for the love of code, comment appropriately in your markdowm/quarto so your future self (and the rest of human kind) can understand what's going on. Something as simple as `fsom_01` to `fsom01` can cost you hours of debugging (Learned this the hard way).
-   **Analytic code must be version controlled**
    -   Version control is the practice of tracking and managing changes to code over time. GitHub is the preferred platform for version control.
    -   This provides digitally verified copies of the analysis with dates recorded
    -   Old approaches which need to be reinstated can be accessed
-   **Keep a lab book**
    -   Despite working mainly *in silico*, records of new approaches should be written and signed off in a lab book
    -   Keep full details of thought processes and ideas for new analytic approaches
    -   These are essential to legal proceedings (e.g. prosecuting a patent)

# Lab Preferred Approaches

## Statistical And General Data Analysis

-   This is to be perfomed using scripting languages such as `R` or `Python`
-   Exporting data and loading in Prism is acceptable (not preferred). Also, let's be honest, if you've read so far down this document is because you are a so motivated that you can definitely perform all the statistical analysis within your code. You've got this!
-   Excel is not acceptable. Please review our Training Folder: `Statistical Analysis Resources` and `Area Analysis` where you'll find templates to perform your analysis in [R](https://www.r-project.org/), [Jamovi](https://www.jamovi.org/) and [Prism](https://www.graphpad.com/features)

## Version Control

-   Git is the only acceptable version control system
-   https://github.com is the preferred repository
-   Analysis stored in a private repository must have a **minimum of two users** with read access or higher

# References and Further Reading

-   [A how-to guide for code-sharing in biology](https://arxiv.org/pdf/2401.03068)
-   [Productive R Worflow](https://www.productive-r-workflow.com/)
-   [Best Practices for Bioinformatics Research](https://github.com/UofABioinformaticsHub/BestPractices/blob/master/README.md)
-   [Coding Best Practices](https://biomadeira.github.io/2023-01-11-coding-best-practices)
