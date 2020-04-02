## Extracting features

These steps are best performed in a new markdown file. We'll be using a number of packages:
```r
library(MALDIquantForeign)
library(MALDIquant)
```

### Metadata

Before we can process our samples, we need to create a list of which samples we want to process.
Previously, we've got a list of files using Sys.glob. Here, we'll use that list to start a dataframe that holds information on our samples.

We've been told the following about our samples
- Samples differ on three levels
  - Soil type (3 types)
  - Biological replicate (3 per soil type)
  - Technical replicate (3 per biological replicate)
- The samples were run in order of soil type, biological rep, technical rep. E.g. the first sample is Soil A, biorep 1, techrep 1, the next is Soil A, biorep 1, techrep 1, and so on, with the last being Soil O, biorep 3, techrep 3.
- The order of soil is A (Agricultural), F (Forest), O (Organic)

To generate the annotation quickly based on this information, we can build vectors of relevant values and bind them 
We can repeat an element of a vector using the rep() function. E.g: `rep('A', 3)` gives us A A A.\
A vector itself can also be repeated e.g.: `rep(c('A','B'), 3)` gives us A B A B A B.\
This can also be done by element: `rep(c('A','B'), each=3)` gets us A A A B B B.\
You can even nest a rep statement in another: `rep(rep(c('A','B'), each=3), 2)` = A A A B B B A A A B B B.

Given the structure below, how can we fill in the missing annotations (i.e. replace the 0s)?
```r
samples<-as.data.frame(Sys.glob("SUM/*.sum.mzML"), stringsAsFactors = F)
colnames(samples)[1]<-"filename"
samples$soil<-0
samples$biorep<-0
samples$techrep<-0
samples$bio_id<-paste0(samples$soil,samples$biorep)
samples$tech_id<-paste0(samples$bio_id,".",samples$techrep)
samples
```
<details>
  <summary>Expected output</summary>

|   | filename | soil | biorep | techrep | bio_id | tech_id|
|---|---------:|:----:|:------:|:-------:|:------:|:------:|
|1|SUM/HW-290120-007.sum.mzML|A|1|1|A1|A1.1|
|2|SUM/HW-290120-008.sum.mzML|A|1|2|A1|A1.2|
|3|SUM/HW-290120-009.sum.mzML|A|1|3|A1|A1.3|
|4|SUM/HW-290120-010.sum.mzML|A|2|1|A2|A2.1|
|5|SUM/HW-290120-011.sum.mzML|A|2|2|A2|A2.2|
|6|SUM/HW-290120-012.sum.mzML|A|2|3|A2|A2.3|
|7|SUM/HW-290120-013.sum.mzML|A|3|1|A3|A3.1|
|8|SUM/HW-290120-014.sum.mzML|A|3|2|A3|A3.2|
|9|SUM/HW-290120-015.sum.mzML|A|3|3|A3|A3.3|
|10|SUM/HW-290120-016.sum.mzML|F|1|1|F1|F1.1|
|11|SUM/HW-290120-017.sum.mzML|F|1|2|F1|F1.2|
|12|SUM/HW-290120-018.sum.mzML|F|1|3|F1|F1.3|
|13|SUM/HW-290120-019.sum.mzML|F|2|1|F2|F2.1|
|14|SUM/HW-290120-020.sum.mzML|F|2|2|F2|F2.2|
|15|SUM/HW-290120-021.sum.mzML|F|2|3|F2|F2.3|
|16|SUM/HW-290120-022.sum.mzML|F|3|1|F3|F3.1|
|17|SUM/HW-290120-023.sum.mzML|F|3|2|F3|F3.2|
|18|SUM/HW-290120-024.sum.mzML|F|3|3|F3|F3.3|
|19|SUM/HW-290120-025.sum.mzML|O|1|1|O1|O1.1|
|20|SUM/HW-290120-026.sum.mzML|O|1|2|O1|O1.2|
|21|SUM/HW-290120-027.sum.mzML|O|1|3|O1|O1.3|
|22|SUM/HW-290120-028.sum.mzML|O|2|1|O2|O2.1|
|23|SUM/HW-290120-029.sum.mzML|O|2|2|O2|O2.2|
|24|SUM/HW-290120-030.sum.mzML|O|2|3|O2|O2.3|
|25|SUM/HW-290120-031.sum.mzML|O|3|1|O3|O3.1|
|26|SUM/HW-290120-032.sum.mzML|O|3|2|O3|O3.2|
|27|SUM/HW-290120-033.sum.mzML|O|3|3|O3|O3.3|

</details>

<details>
  <summary>Solution</summary>

```r
samples$soil<-c(rep('A',9),rep('F',9),rep('O',9))
samples$biorep<-c(rep(rep(1:3, each=3),3))
samples$techrep<-c(rep(1:3,9))
samples$bio_id<-paste0(samples$soil,samples$biorep)
samples$tech_id<-paste0(samples$bio_id,".",samples$techrep)
samples
```

</details>

### Importing data

After summing, the size of the data is significantly smaller. In this project, we'll therefore be able to load all the samples into memory at the same time, which makes it easier to process all samples in one go. In our `samples` dataframe we have a list of files in the column `filename`. The MALDIquantForeign package provides the function `importMzMl` to load mzML files:
```{r load files}
suppressWarnings(rawdata<-importMzMl(samples$filename, centroided = F, verbose=F))
length(rawdata)
```
The code chunk suppresses warnings for the import function, as it will produce a bunch of warnings about mismatching checksums. Whilst there is a standard for how these should be calculated, it appears that there are different implementations that can cause mismatches. At this stage the relevant question is:

>Did we load the expected number of files?

### Transformation and smoothing

We can take a look at a full spectrum and a small section with some more detail like so:

```r
mzroi<-c(200,210) #We'll reuse these coordinates later.
inroi<-c(0,10000)
plot(rawdata[[1]])
plot(rawdata[[1]], xlim=mzroi, ylim=inroi)
```
This creates two plots. In Rstudio, you can switch between plots by clicking the small previews above the current plot.
One of the first steps to take is to transform the raw intensities by taking the square root. This causes the variance between measurements (e.g. the same peak in technical replicates) to be less dependent on the mean intensity (of e.g. the peak).

```r
spectra<-transformIntensity(rawdata, method="sqrt")
plot(spectra[[1]], xlim=mzroi, ylim=sqrt(inroi))
```
This does highlight the noise in the data - the signal does not look very smooth. A common step in signal processing is to smooth out the signal. Using information from neighbouring data points, we try to apprimate what the underlying signal is likely to be. A widely used smoothing approach is applying a Savitzky-Golay filter. The important question is how many data points should we use to define the neighbourhood. In other words, how much should the data be smoothed? As a rule of thumb, as many data points in the smooth as there are in the top half of the narrowest peak that should be included. From our random zoomed-in part of the spectrum, we take the 2nd largest peak: 
```r
plot(spectra[[1]], xlim=c(205.5,206.5), ylim=c(0,40), type='b')
```
>Assuming the signal between 206.0 and 206.2 is a single peak, how many data points should we smooth over?\
>Optional tip: to add a horizontal line to a plot, use `abline`, see `?abline`.
<details>
<summary>Answer</summary>
>There are 11 data points between when it first goes above 20 to when the twin peaks drop below 20 counts.
</details>
</br>
>Many filters need a half window size to be specified. For a point for which we are calculating the smoothed value, the half window size is the number of preceding as well as the number following data points to include. What half window size could we use here?
<details>
<summary>Answer</summary>
>For the Savitzky golay filter the total window ('neighbourhood') is 2 * halfwindow + 1 (the data point itself). If we have a full window of 11, the halfwindow size is 5 ((11-1)/2).
</details>
</br>

Next, test out the effect of smoothing
```r
testspectrumhw5<-smoothIntensity(spectra[[1]], method="SavitzkyGolay", halfWindowSize=5)
plot(testspectrumhw5, xlim=c(205.5,206.5), ylim=c(0,40), type='b')
```
This still preserves part of the 'twin' peak shape.
>What happens if you increase the halfWindowSize?
<details>
<summary>Answer</summary>

>For example:
```r
testspectrumhw11<-smoothIntensity(spectra[[1]], method="SavitzkyGolay", halfWindowSize=11)
plot(spectra[[1]], xlim=c(205.5,206.5), ylim=c(0,40), type='b', main="No smoothing")
plot(testspectrumhw5, xlim=c(205.5,206.5), ylim=c(0,40), type='b', main="SG halfWindowSize 5")
plot(testspectrumhw11, xlim=c(205.5,206.5), ylim=c(0,40), type='b', main="SG halfWindowSize 11")
```
>Cycling through the plots, we can see that details that were still being retained with less smoothing (such as the twin peak shape) are lost. Too much smoothing may smooth out actual signal, rather than just noise.
</details>
</br>

We stick with a halfWindowSize of 5, and apply it to all spectra.
```r
spectra<-smoothIntensity(spectra, method="SavitzkyGolay", halfWindowSize=5)
```

### Baseline correction

We now have a somewhat smooth signal. In the previous plots we saw a noisy line that hovered around 10, with an occasional peak. The level of this line may differ between samples, and may also be different at different m/z ratios. To make samples more comparable, we can correct for it. As with smoothing, it is possible to remove too much, so we take a look first:
```r
baselinetest <- estimateBaseline(spectra[[1]], method="SNIP", iterations=100)
plot(spectra[[1]])
lines(baselinetest, col='red')
```
In general it looks like there's not too much variation with this sample. However, estimated baseline shown in red looks a little bumpy. In close up:
```r
plot(spectra[[1]], xlim=mzroi, ylim=c(0,40))
lines(baselinetest, col='red')
```
We can make the baseline less likely to fit to individual points by increasing the number of iterations performed in the calculation:
```r
baselinetest <- estimateBaseline(spectra[[1]], method="SNIP", iterations=250)
plot(spectra[[1]], xlim=mzroi, ylim=c(0,40))
lines(baselinetest, col='red')
plot(spectra[[1]])
lines(baselinetest, col='red')
```
As it now looks smoother, we apply these settings for baseline removal. As an output to check what's happened, we include a couple of plots.
```r
spectra <- removeBaseline(spectra, method="SNIP", iterations=250)
plot(spectra[[1]], xlim=mzroi, ylim=c(0,40))
plot(spectra[[1]])
```
### TIC normalisation

In addition to removing the baseline, there are more steps to take to make signals more comparable. Some samples may have an overall higher (or lower) signal than other samples. We can check for this by calculating the total signal, or the total ion count (TIC). The assumption here is that the majority of the signal is similar between samples - which for soils may be a valid assumption, but this may not bet true is other settings. We can get the sums of the intensities to check for big differences:
```r
intensities<-sapply(spectra, function(spectrum){sum(intensity(spectrum))})
barplot(intensities/1e6, names.arg = samples$tech_id, xlab = "sample", ylab = "total intensity (in 1e6 counts)", las=2)
```
>Are there large differences?\
>Are any of these differences structured?
<details>
<summary>Answer</summary>
>There's about a two-fold difference between the least and the most intensive sample. The forest soil samples seem to have a higher overall intensity. There is a slight trend for the variation between technical replicates being smaller than that between biological replicates.
</details>
</br>
For the actual calibration, and comparison afterwards:
```r
spectra <- calibrateIntensity(spectra, method="TIC")
intensities<-sapply(spectra, function(spectrum){sum(intensity(spectrum))})
barplot(intensities, names.arg = samples$tech_id, xlab = "sample", ylab = "total intensity", las=2)
```
The differences should be lot smaller now.

### Peak alignment

Next we call some tentative peaks and see how well our samples are aligned
```r
testpeaks<-detectPeaks(spectra, halfWindowSize=5, SNR=3)
warps<-determineWarpingFunctions(testpeaks, plot = T, tolerance = 150e-6)
```

Check output pdf. Seems to warp slightly at the edges, but overall no big effect. Warp:

```r
spectra_aligned <- alignSpectra(spectra,
                        halfWindowSize=5,
                        SNR=3,
                        tolerance=150e-6,
                        warpingMethod="lowess")
```

What is the effect?
```r
testpeaks_after<-detectPeaks(spectra_aligned, halfWindowSize=5, SNR=3)
plot(spectra_aligned[[18]], xlim=c(1061.9,1062.9), ylim=c(0,0.006))
lines(spectra[[18]], col='grey', lty=3)
lines(spectra_aligned[[7]], col='blue')
lines(spectra[[7]], col='cyan3', lty=3)
points(testpeaks[[18]], col='grey', pch=4)
points(testpeaks[[7]], col='cyan3', pch=4)
points(testpeaks_after[[18]], pch=4)
points(testpeaks_after[[7]], col='blue', pch=4)
```

Looks like it might overcompensate a little, but does bring them closer together. Check if true:
```r
peakmzs_18<-mz(testpeaks[[18]])
peakmzs_07<-mz(testpeaks[[7]])
peakmzs_after_18<-mz(testpeaks_after[[18]])
peakmzs_after_07<-mz(testpeaks_after[[7]])
peakmzs_18[peakmzs_18 > 1062 & peakmzs_18 < 1063] - 
  peakmzs_07[peakmzs_07 > 1062 & peakmzs_07 < 1063]
peakmzs_after_18[peakmzs_after_18 > 1062 & peakmzs_after_18 < 1063] - 
  peakmzs_after_07[peakmzs_after_07 > 1062 & peakmzs_after_07 < 1063]
```

In this case we can also consider to not align, as the random variation between the peaks in each sample is larger than the very mild correction we apply by warping the spectra.

We also still have some noise in the data - this will always be the case. In most applications we want to avoid looking at the noise too much, so often a Signal to noise ratio (SNR) is given as a criteria for peak calling. This means that to be be regarded as a peak, the peak height needs to be above a number of times the noise level determined by the SNR. We can investigate our data for noise like so:

```r
noise<-MALDIquant::estimateNoise(spectra_aligned[[1]])
plot(spectra_aligned[[1]], xlim = mzroi, ylim=c(0,0.006))
lines(noise, col="red")
lines(noise[,1],noise[,2]*5, col="purple")
lines(noise[,1],noise[,2]*10, col="blue")
```

Now it looks like an SNR of 5 (purple line) would already cut out some peaks. However, this picture may be misleading. Let's look at a much higher m/z range:

```r
plot(spectra_aligned[[1]], xlim=c(1060,1070), ylim=c(0,0.01))
lines(noise, col="red") #Red line: noise level
lines(noise[,1],noise[,2]*5, col="purple") #Purple line: noise level * 5
lines(noise[,1],noise[,2]*10, col="blue") #Blue line: noise level * 10
```

Here it look like we get a lot of repeating signal just above the 5 SNR line. An SNR of 8, or possibly even 10 might prove a better balance between restricting noise and throwing away signal.

We can then call the peaks with some additional refinements:
```r
peaks <- detectPeaks(spectra_aligned, method="MAD", halfWindowSize=5, SNR=8, refineMz = "descendPeak", signalPercentage=25)
```

As we saw previously, even after alignment there are small difference in the peak m/z values. For an illustration:
```r
plot(spectra_aligned[[18]], xlim=c(1061.9,1062.9), ylim=c(0,0.006))
for(n in seq_along(peaks)){
  points(peaks[[n]], pch = 4, col=rainbow(length(peaks))[[n]])
}
```

To get rid of those, we bin the Peaks from different samples within a certain tolerance together.
```r
peaks<-binPeaks(peaks, method="strict", tolerance = 50e-6)
plot(spectra_aligned[[18]], xlim=c(1061.9,1062.9), ylim=c(0,0.006))
for(n in seq_along(peaks)){
  points(peaks[[n]], pch = 4, col=rainbow(length(peaks))[[n]])
}
```
After this, we can gather our peaks and intensities:
```r
featureMatrix <- intensityMatrix(peaks, spectra_aligned)
```
As you'll have noticed, `intensityMatrix` is also reading in our aligned spectra. Why would it do that?

The `featureMatrix` object is, as the name suggests, a Matrix. One row for each sample, one column for each peak. How many peaks are there in the matrix? 
```r
ncol(featureMatrix)
```

Some peaks still appear to be random noise. Going back a step, we can filter peaks based on in which samples they are called. Based on the documentation for `filterPeaks`, what does the following do?
```r
peaks_byrep<-filterPeaks(peaks, minFrequency = 1, labels = samples$bio_id, mergeWhitelists = TRUE)
peaks_byrep_bysample<-filterPeaks(peaks_byrep, minFrequency = 0.5, labels = samples$soil, mergeWhitelists = TRUE)
```

We can then get a Matrix with just the reliable features:
```r
featureMatrix_filter <- intensityMatrix(peaks_byrep_bysample, spectra_aligned)
```

And check how many peaks we retained:
```r
ncol(featureMatrix_filter)
```

We now know where there are peaks, and what the intesity is in each sample. We don't yet know whether there are any differences between samples. A lot of analysis can be done in R, but alternatively we can export the data as csv files to enable import in other applications which are more user friendly:
```r
intensities_out<-data.frame(Sample=samples$tech_id,
                            Label=samples$soil,
                            featureMatrix_filter, check.names = F, stringsAsFactors = F)
intensities_out<-rbind(c(NA, "rt", rep(1, ncol(intensities_out)-2)), intensities_out) #Create fake RT
intensities_out_ex<-intensities_out %>% dplyr::filter(Sample %in% c("A3.3")) #Create fake RT
intensities_OF<-intensities_out %>% dplyr::filter(Label %in% c("O","F","m/z"))
write.csv(intensities_OF, row.names = F, file = "Intensities_OF.csv")
write.csv(intensities_out, row.names = F, file = "Intensities.csv")
write.csv(intensities_out_ex, row.names = F, file = "Intensities2.csv")
```
