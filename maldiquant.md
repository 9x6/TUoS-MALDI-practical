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

### Visualising data

We can take a look at a full spectrum like so:
```{r}
plot(rawdata[[1]])
```
