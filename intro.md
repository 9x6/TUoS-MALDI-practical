# Metabolomics practical - data analysis

## Introduction

This practical originally would build on the hands-on metabolomics work scheduled as part of APS6625. Due to the COVID-19 outbreak this has become rather a lot less hands on. Nevertheless, we can still have a look at some of the data that was collected earlier to get a taste of how this data is structured and how a basic analysis can be performed using freely accessible tools.

## Software

Normally we would aim to have all software installed on computers in the lab, though we would still encourage setting up at least a working R environment on your own device as this is useful for multiple courses and assignments. As we cannot use the lab computers in the lab for this practical (remote access cannot be set up, sorry), please check if you are able to install the software listed below. All software is freely available for academic users. Unfortunately there are some platform restrictions on Proteowizard, as discussed below.
*If you are on a slow connection, or have a limited data allowance, skip Proteowizard and do not download the RAW data in the data section*

### Proteowizard

Proteowizard comes with a programme called msconvert that can be used to convert the data that comes from the machine into an open data format. To do this, it uses closed-source vendor-specific libraries which only work on Windows systems. If you do not have access to a windows system, or if installing fails, download the pre-processed data files from the data section below instead of the RAW data. For the determined and advanced non-windows user a docker image is available, however this also assumes knowledge of docker as well as the command line version of msconvert, which we will not cover here.
Proteowizard can be downloaded [here](http://proteowizard.sourceforge.net/download.html). The 64 bit installer is usually the best option. The e-mail field is optional.

### R

As the University of Sheffield uses R as preferred statistics platform, you'll probably already have it installed. If you don't, it can be downloaded by selecting a local mirror via [r-project.org](https://r-project.org). In the UK, Bristol isn't a bad choice, the download page for the Windows version is [here](https://www.stats.bris.ac.uk/R/bin/windows/base/). R will soon introduce a major new version (4.0). If you are offered this, please look for the last 3.6.x version under previous releases instead. Older versions of R may also work, but have not been tested.

### RStudio

RStudio provides a more comfortable working environment for R. As with R itself, there's a good chance you already have this installed. If not, it can be downloaded [here](https://rstudio.com/products/rstudio/download/#download).

## Data

*If you have a limited data allowance, do not download the RAW data.*

The [raw data files](https://drive.google.com/open?id=1oY4WlWQrMBI9cYQs5AaBZ5ysH0oXWlNg) (~560 Mb) are available if you are logged in to a University of Sheffield account.

If you have a limited connection, [pre-processed input files](https://drive.google.com/open?id=1U6qHw7jATcopLD4IetoPnd-DWJDuEne9) (~15 Mb) can be downloaded here (University of Sheffield account required).

## Next Steps

Pre-processing the data
