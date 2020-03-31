```r
knitr::opts_chunk$set(echo = TRUE)
library(MSnbase)
library(MALDIquantForeign)
```

We have 27 input files, each corresponding to a single sample.
Data is in mzML. What does an example sample look like?

```r
profiledata<-readMSData("RAW/HW-290120-007.mzML", centroided. = F, msLevel. = 1, mode = 'onDisk')
profiledata
```
We see 119 scans.
How do the scans relate? Supposedly uniform sample -> sum data for a clearer picture. Small differences between scans may exist, what is the tolerance we should allow? How much variation is there between subsequent scans?

```r
MzScattering<-unlist(estimateMzScattering(profiledata, timeDomain = T))
max(MzScattering)/sqrt(50)
```
N.B. this is slightly misusing a function designed for use with LCMS. It does, however, give us a ballpark figure to work with. As we have TOF data, we specify timeDomain to be true. This will mean functions actually operate on the sqrt of mz values. To figure out what the worst-case scattering is in ppm, we divide the scattering by square root (as this is in timeDomain) of the lowest mz (50).

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

Supposing we would want the average rather than the sum, how can that be achieved by changing the code block 'sum test' above? Why can using an average sometimes be a good idea?

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

Before we can process our samples, we need to create a list of which samples we want to process.
We can get a list (or, more acurately _a vector_) of files using Sys.glob, which allows wild-card matching of files. As we want to bind some more data to these filenames, we convert the vector into a data frame.
We've been told the following about our samples
- Samples differ on three levels
  - Soil type (3 types)
  - Biological replicate (3 per soil type)
  - Technical replicate (3 per biological replicate)
- The samples were run in order of soil type, biological rep, technical rep. E.g. the first sample is Soil A, biorep 1, techrep 1, the next is Soil A, biorep 1, techrep 1, and so on, with the last being Soil O, biorep 3, techrep 3.
- The order of soil is A (Agricultural), F (Forest), O (Organic)

We can repeat an element of a vector using the rep() function. E.g:
```r
rep('A', 3)
```
gives us A A A
A vector itself can also be repeated e.g.:
```r
rep(c('A','B'), 3)
```
gives us A B A B A B. This can also be done by element:
```r
rep(c('A','B'), each=3)
```
which gets us A A A B B B

Given the structure below, how can we fill in the missing annotations (i.e. replace the 0s)?
```r
samples<-as.data.frame(Sys.glob("RAW/*[0-9].mzML"), stringsAsFactors = F)
colnames(samples)[1]<-"filename"
samples$sum_filename<-createSumFilenames(samples$filename)
samples$soil<-0
samples$biorep<-0
samples$techrep<-0
samples$bio_id<-paste0(samples$soil,samples$biorep)
samples$tech_id<-paste0(samples$bio_id,".",samples$techrep)
samples
```
<details>
  <summary>Expected output</summary>

|   |filename|               sum_filename| soil | biorep | techrep | bio_id | tech_id|
|---|---:|---:|:---:|:---:|:---:|:---:|:---:|
|1|RAW/HW-290120-007.mzML|SUM/HW-290120-007.sum.mzML|A|1|1|A1|A1.1|
|2|RAW/HW-290120-008.mzML|SUM/HW-290120-008.sum.mzML|A|1|2|A1|A1.2|
|3|RAW/HW-290120-009.mzML|SUM/HW-290120-009.sum.mzML|A|1|3|A1|A1.3|
|4|RAW/HW-290120-010.mzML|SUM/HW-290120-010.sum.mzML|A|2|1|A2|A2.1|
|5|RAW/HW-290120-011.mzML|SUM/HW-290120-011.sum.mzML|A|2|2|A2|A2.2|
|6|RAW/HW-290120-012.mzML|SUM/HW-290120-012.sum.mzML|A|2|3|A2|A2.3|
|7|RAW/HW-290120-013.mzML|SUM/HW-290120-013.sum.mzML|A|3|1|A3|A3.1|
|8|RAW/HW-290120-014.mzML|SUM/HW-290120-014.sum.mzML|A|3|2|A3|A3.2|
|9|RAW/HW-290120-015.mzML|SUM/HW-290120-015.sum.mzML|A|3|3|A3|A3.3|
|10|RAW/HW-290120-016.mzML|SUM/HW-290120-016.sum.mzML|F|1|1|F1|F1.1|
|11|RAW/HW-290120-017.mzML|SUM/HW-290120-017.sum.mzML|F|1|2|F1|F1.2|
|12|RAW/HW-290120-018.mzML|SUM/HW-290120-018.sum.mzML|F|1|3|F1|F1.3|
|13|RAW/HW-290120-019.mzML|SUM/HW-290120-019.sum.mzML|F|2|1|F2|F2.1|
|14|RAW/HW-290120-020.mzML|SUM/HW-290120-020.sum.mzML|F|2|2|F2|F2.2|
|15|RAW/HW-290120-021.mzML|SUM/HW-290120-021.sum.mzML|F|2|3|F2|F2.3|
|16|RAW/HW-290120-022.mzML|SUM/HW-290120-022.sum.mzML|F|3|1|F3|F3.1|
|17|RAW/HW-290120-023.mzML|SUM/HW-290120-023.sum.mzML|F|3|2|F3|F3.2|
|18|RAW/HW-290120-024.mzML|SUM/HW-290120-024.sum.mzML|F|3|3|F3|F3.3|
|19|RAW/HW-290120-025.mzML|SUM/HW-290120-025.sum.mzML|O|1|1|O1|O1.1|
|20|RAW/HW-290120-026.mzML|SUM/HW-290120-026.sum.mzML|O|1|2|O1|O1.2|
|21|RAW/HW-290120-027.mzML|SUM/HW-290120-027.sum.mzML|O|1|3|O1|O1.3|
|22|RAW/HW-290120-028.mzML|SUM/HW-290120-028.sum.mzML|O|2|1|O2|O2.1|
|23|RAW/HW-290120-029.mzML|SUM/HW-290120-029.sum.mzML|O|2|2|O2|O2.2|
|24|RAW/HW-290120-030.mzML|SUM/HW-290120-030.sum.mzML|O|2|3|O2|O2.3|
|25|RAW/HW-290120-031.mzML|SUM/HW-290120-031.sum.mzML|O|3|1|O3|O3.1|
|26|RAW/HW-290120-032.mzML|SUM/HW-290120-032.sum.mzML|O|3|2|O3|O3.2|
|27|RAW/HW-290120-033.mzML|SUM/HW-290120-033.sum.mzML|O|3|3|O3|O3.3|

</details>

<details>
  <summary>Solution</summary>

```{r}
samples$soil<-c(rep('A',9),rep('F',9),rep('O',9))
samples$biorep<-c(rep(rep(1:3, each=3),3))
samples$techrep<-c(rep(1:3,9))
samples$bio_id<-paste0(samples$soil,samples$biorep)
samples$tech_id<-paste0(samples$bio_id,".",samples$techrep)
samples
```

</details>


```r
rawdata<-importMzMl(samples$filename, centroided = F)
```

Continues...
