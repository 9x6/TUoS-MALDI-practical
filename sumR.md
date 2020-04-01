## Processing scans

Time for a new Rmarkdown file. Suggested naming: '01-ProcessScans.Rmd'.\
Most of the example code is not required, the part after the first code chunk can be removed. This part of the tutorial uses the MSnbase package, which we can load in the same setup block:

```r
knitr::opts_chunk$set(echo = TRUE)
library(MSnbase)
```

We have 27 input files, each corresponding to a single sample. After conversion, all our files are in mzML format.
What does an example sample look like?

```r
profiledata<-readMSData("RAW/HW-290120-007.mzML", centroided. = F, msLevel. = 1, mode = 'onDisk')
profiledata
```
>How many scans are there in this file?
>How do these scans relate to each other?
<details>
<summary>Answer</summary>

>There are 119 scans (according to the spectra data)\
>How do the scans relate? The spectra data block suggests there is a retention time. Technically this may be the case, but thinking back to the lab practical, no chromatography was involved. The samples were spotted on a plate, and are supposedly a uniform sample - meaning there is no relevant separation that should show across scans. We will get a more accurate picture is we sum the scans together.
</details>
</br>

An issue with summing the scans is that small differences in mz values between scans may exist. We can match individual data points between scans with a small tolerance that accounts for the variation between samples. But which tolerance should we allow? How much variation is there between subsequent scans?

```r
MzScattering<-unlist(estimateMzScattering(profiledata, timeDomain = T))
max(MzScattering)/sqrt(50)
```
N.B. this is slightly misusing a function designed for use with LCMS. It does, however, give us a ballpark figure to work with. As we have TOF data, we specify timeDomain to be true. This will mean functions actually operate on the square root of mz values. To figure out what the worst-case scattering is relative to the mzs being compared, we divide the scattering by square root (as this is in timeDomain) of the lowest mz (50).

>How many ppm (parts per million) is the worst case scattering, rounded down to the nearest integer?
<details>
<summary>Answer</summary>

>1.5e-5 = 15e-6 = 15 ppm
</details>
</br>
We can then sum up the scans. combineSpectra does what we want, but most of the options are specified in meanMzInts(). From the documentation (run ?meanMzInts() for a list)

```r
profilespectra<-Spectra(spectra(profiledata)) #Extract the spectra
pdsum15<-MSnbase::combineSpectra(profilespectra, weighted=F, timeDomain=T, ppm=15, intensityFun=sum, unionPeaks=T)
profile15<-as(pdsum15, "MSnExp") #Format conversion
```

To illustrate what's happening when we sum (left is linear y-axis, right is log y-axis), we can plot our summed data together with (in this case) 19 individual scans. Note missing values cause gaps in the plot

```r
par(mfrow=c(1,2))
plot(x=MSnbase::mz(filterMz(profile15[[1]], mz=c(380,380.3))),
     y=MSnbase::intensity(filterMz(profile15[[1]], mz=c(380,380.3))),
     col="red", type='l', xlab='mz', ylab='counts')
for(n in seq(from=10, to=100, by=5)){
  lines(x=MSnbase::mz(filterMz(profiledata[[n]], mz=c(380,380.3))),
        y=MSnbase::intensity(filterMz(profiledata[[n]], mz=c(380,380.3))),
        col="grey")
}
plot(x=MSnbase::mz(filterMz(profile15[[1]], mz=c(380,380.3))),
     y=MSnbase::intensity(filterMz(profile15[[1]], mz=c(380,380.3))),
     col="red", type='l', xlab='mz', ylab='counts', log="y")
for(n in seq(from=10, to=100, by=5)){
  lines(x=MSnbase::mz(filterMz(profiledata[[n]], mz=c(380,380.3))),
        y=MSnbase::intensity(filterMz(profiledata[[n]], mz=c(380,380.3))),
        col="grey")
}
par(mfrow=c(1,1))
```

>Why does the summed signal look cleaner?
<details>
<summary>Answer</summary>

>The random noise in the individual scans is less of an issue when added together, as it cancels out.
</details>
</br>

>Supposing we would want the average rather than the sum, how can that be achieved by changing the code block 'sum test' above? 
<details>
<summary>Answer</summary>

>The function `combineSpectra` takes an argument called `intensityFun` which is set to `sum` in the block above. Documentation (`?meanMzInts`) shows the default is actually base::mean, which would average the scans. We can therefore replace `sum` by `base::mean` or just remove the entire `intensityFun=sum` argument.
</details>
</br>

>Why can using an average sometimes be a good idea?
<details>
<summary>Answer</summary>

>In some cases the number of scans won't be equal between files. Summing them up would give us different ranges for the intensities between samples that are a technical artefact. In most cases, however, we would rely on another normalisation method to make the signals of different samples more comparable, as there are also other sources of variation that we would want to correct for. An minor advantage to averaging is that the values of an averaged spectrum are much closer to the values in the individual spectra that were used to calculate the average. This can be simulated by dividing y values in the `plot` code for the summed spectra (but not the `lines` code for the individual spectra) by 119 (i.e. `y=(MSnbase::intensity(filterMz(profile15[[1]], mz=c(380,380.3))))/119`)
</details>
</br>
Running this manually on every file would be a lot of coding. We're going to assume that the tolerance that we've chosen will apply to all samples in this data set. We can write a bit of code (a function) that given an input, produces an output by following the steps we tell it to do.



```r
createSumFilenames<-function(filename, out.folder="SUM"){
  file.path(out.folder,sub("\\.mzML",".sum.mzML",basename(filename)))
}
mergeScans<-function(filename, out.folder="SUM", tol.ppm=15){
  filename.out<-createSumFilenames(filename, out.folder)
  if(file.exists(filename.out)){
    warning(paste("Not processing",filename,"as output file",filename.out,"already exists"))
  } else {
    profiledata<-readMSData(filename, centroided. = F, msLevel. = 1, mode = 'onDisk')
    profiledata.merge<-MSnbase::combineSpectra(Spectra(spectra(profiledata)),
                                               weighted=F, 
                                               timeDomain=T, 
                                               ppm=tol.ppm, 
                                               intensityFun=sum, 
                                               unionPeaks=T)
    writeMSData(as(profiledata.merge, "MSnExp"), file=filename.out, outformat="mzml")
    message(paste0("Processed ",filename))
  }
}
```

The custom function mergeScans takes three arguments: a filename, the name for an output directory (that needs to exist) and tol.ppm, which defaults to 15 if not specified.
An output filename is created by substituting part of the input file name. The Input directory is stripped at the same time, so we can just prepend the output folder to get the full path for an output file path.
When we know where we want to write, we check whether there is already data there, as it stops us from doing long(ish) calculations that we already did if we rerun our code. Remember to remove the files if we decide that we need to change the processing in the function in such a way that it would generate new output.
The rest of the function does the same summing as we tested previously.

So we then need a vector or a list of filenames, which we can get using some wildcard matching:
```r
infiles<-Sys.glob("RAW/*.mzML")
infiles
```

Which we can then feed to `sapply`, that will run a function of our choice using each value of infiles as an input for individual runs of the function we tell it to use:

```r
sapply(infiles, mergeScans)
```

**Note that this code takes a while to finish** \
After the code has run, we'll have a summed spectrum for each sample in the SUM directory of our project directory.

## Next steps

Now the data has been summed, we can start to [extract features from the spectra](maldiquant).
