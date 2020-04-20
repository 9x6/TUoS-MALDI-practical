## Project setup in Rstudio

Previously, we've [set up a project directory and unpacked the data](intro) we're going to be using into it. We also [converted the files](msconvert) into a format we can work with. This project directory will be the basis of an Rproject that we'll be using in RStudio. To set up an Rproject, open up Rstudio, and select File > New Project. In the window that pops up, select 'Existing Directory', and go to the project directory that you created (click 'Open' when you've navigated into the correct directory).

Within the project, we can set up as many files/scripts as we need. To keep a record of what we've done, we'll save any commands that we're running as R scripts or as R markdown files.

Create a new R script file (File > New File > R Script). Save it as '00-PackageSetup.R' or similar in the project directory. Splitting code into multiple files can be useful as some parts of the will be run once, but other parts may go through multiple rounds of changes and runs. Numbering the files helps to provide a logical order of scripts, but make sure to also have a short descriptive name.

In our new file, we can set up the packages that we'll use. Packages extend R's functionality, and we'll be using some that are specific to handling mass spectrometry data. Specifically, we'll set up MSnbase via [Bioconductor](https://www.bioconductor.org/about/) (a specialist repository for packages that extend functionality to handle biological data). From the default repositories we'll install MALDIquant and MALDIquantForeign:

```r
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install(version = "3.10")
BiocManager::install("MSnbase")
install.packages(c('MALDIquant','MALDIquantForeign'))
```

Save the code in your R Script, and run it (either select all code and press <kbd>CTRL</kbd>+<kbd>ENTER</kbd> or select all code and click 'Run' at the top left of the code pane). You may encounter a number of questions, including where you want to create a library (this library is for all packages in all R sessions, and is not just part of this project - the default is fine, don't set it to your project directory). The defaults are usually fine, but use your best judgement. Avoid using code that needs to be compiled (if asked). Warnings about missing rtools can be ignored.

Check the output for errors (installation had a non-zero exit status). If there were errors, read what they are and check for obvious solutions (typos? no network connection? other library not installed?)

## Rmarkdown and code blocks in this tutorial

If the installation of your software worked out, see if R markdown is working. Create a new R _markdown_ file. When you're asked questions: HTML output is fine. Title and author are your choice. You may get a message asking whether you want to install recommended/required packages - please do so.

If you're not familiar with R markdown, have a read through the example code in the new R markdown file you just created to see what markdown can do. Knit the file to see how code is formatted in the output documents to get a better understanding. If you want to play around with markdown and see live changes, there are some online tools available (for example [cryptpad code](https://cryptpad.fr/code)). There is also a [reference guide for R markdown](https://rstudio.com/wp-content/uploads/2015/03/rmarkdown-reference.pdf).

R Markdown allows you to keep your code, output and notes together. It is strongly suggested to include some text that describes what each code block does and what you take from the output (if visible output is generated). Code blocks in R Markdown look like this:
````
```{r}
plot(iris)
```
````
A new block can be started either by typing the opening line (``​```{r}``) and then closing it by typing the closing line (``​```​``) or by clicking insert > R in on the menu directly above the code pane. In this tutorial the code blocks are shown without the opening and closing lines, so the above block is normally shown as:
```r
plot(iris)
```
Below the code pane there is a console pane, which is minimised as a bar by default. You can bring it up by double clicking it, or using the resize buttons on the right in the bar. You can run code in here directly, but it is better practice to save all your code in Markdown (or R script in some cases). Nevertheless, the occasional quick calculation that isn't core to the code (e.g. `sqrt(1024)`) can be run this way.

Through the tutorial there a questions to help you think about the steps that are being taken. To enable you to work at your own pace, and to be more independent of an audio/video conferencing, the answers are usually included in the tutorial. To avoid cases where you automatically read the answer before you've had a chance to think, the answers are in collapsed text blocks. These blocks are indicated by a triangle icon followed by a label (usually 'answer') and can be opened by clicking the icon or label: 

<details>
  <summary>Answer</summary>
     
  >This text should be visible only after clicking 'Answer'. Clicking the label again will hide this text again.
</details>

You will get more out of the tutorial if you take your time to think about the questions before checking the answers.

## Next steps

Next, we take a [look at the raw data and perform some initial processing](sumR).
