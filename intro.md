# Metabolomics practical - data analysis

## Introduction

This practical originally would build on the hands-on metabolomics work scheduled as part of APS6625. Due to the COVID-19 outbreak this has become rather a lot less hands on. Nevertheless, we can still have a look at some of the data that was collected earlier to get a taste of how this data is structured and how a basic analysis can be performed using freely accessible tools.

## Software

Normally we would aim to have all software installed on machines in the lab. As we cannot use these machines for this practical, please check if you are able to install the software listed below. All software is freely available for academic users. Unfortunately there are some platform restrictions on proteowizard, as discussed below.

### Proteowizard

Proteowizard comes with a programme called msconvert that can be used to convert the data that comes from the machine into an open data format. To do this, it uses closed-source vendor-specific libraries which only work on Windows systems. If you do not have access to a windows system, or if installing fails, converted data files can be provided. For advanced non-windows users a docker image is available, however this also assumes knowledge of the command line version msconvert.
Proteowizard can be downloaded [here](http://proteowizard.sourceforge.net/download.html). The 64 bit installer is usually the best option. The e-mail field is optional.

### R

As the University of Sheffield uses R as preferred statistics platform, you'll probably already have it installed. If you don't, it can be downloaded by selecting a local mirror via r-project.org. In the UK, Bristol isn't a bad choice, the download page for the Windows version is [here](https://www.stats.bris.ac.uk/R/bin/windows/base/). R will soon introduce a major new version (4.0). If you are offered this, please look for the last 3.6.x version under previous releases instead.

### RStudio

RStudio provides a more comfortable working environment for R. As with R itself, there's a good chance you already have this installed. If not, it can be downloaded [here](https://rstudio.com/products/rstudio/download/#download).



