## Project setup in Rstudio

Previously, we've [set up a project directory and unpacked the data](intro) we're going to be using into it. We also [converted the files](msconvert) into a format we can work with. This project directory will be the basis of an Rproject that we'll be using in RStudio. To set up an Rproject, open up Rstudio, and select File > New Project. In the window that pops up, select 'Existing Directory', and go to the project directory that you created (click 'Open' when you've navigated into the correct directory).

Within the project, we can set up as many files/scripts as we need. To keep a record of what we've done, we'll save any commands that we're running as R scripts or as R markdown files.

Create a new R script file (File > New File > R Script). Save it as '00-PackageSetup.R' or similar in the project directory. Splitting code into multiple files can be useful as some parts of the will be run once, but other parts may go thorugh multiple rounds of changes and runs. Numbering the files helps to provide a logical order of scripts, but make sure to also have a short descriptive name.

In our new file, we can set up the packages that we'll use. Packages extend R's functionality, and we'll be using some that are specific to handling mass spectrometry data. Specifically, we'll set up MSnbase via [Bioconductor](https://www.bioconductor.org/about/) (a specialist repository for packages that extend functionality to handle biological data). From the default repositories we'll install MALDIquant and MALDIquantForeign

```r
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install(version = "3.10")
BiocManager::install("MSnbase")
install.packages(c('MALDIquant','MALDIquantForeign'))
```

Save the code, and run it (either select all code and press <kdb>CTRL</kbd>+<kbd>ENTER</kbd> or select all code and click 'Run' at the top left of the code pane). You may encounter a number of questions, including where you want to create a library (this library is for all packages in all R sessions, and is not just part of this project - the default is fine, don't set it to your project directory). The defaults are usually fine, but use your best judgement. Avoid using code that needs to be compiled (if asked). Warnings about missing rtools can be ignored.

Check the output for errors (installation had a non-zero exit status). If there were errors, read what they are and check for obvious solutions (typos? no network connection? other library not installed?)

If all is well, create a new R markdown file. HTML output is fine. Title and author are your choice. You may get a message asking whether you want to install recommended/required packages - this is something you want to set up.

If you're not familiar with R markdown, have a read through the example code in the new R markdown file you just created to see what markdown can do. Knit the file to see how code is formatted in the output documents.

## Next steps

Next, we take a [look at the raw data and perform some initial processing](sumR).
