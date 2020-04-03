## Learning from the data

To further investigate the data, we will use MetaboAnalyst. We'll use the online version, which provides a graphical user interface and interactive graphics. MetaboAnalyst also exists as an R package that can be used for larger datasets or datasets that cannot be uploaded due to confidentiality issues. You'll notice that the online version shows the code that can be used to the analysis in the R package.

  1. Access [MetaboAnalyst](https://www.metaboanalyst.ca). Under 'Please cite' there are useful resources if you ever need to dive deeper into the tools. If you use metaboanalyst in published work, please cite one ore more of these publications (depending on relevance). At the bottom of the page there's a summary of what analyses can be done in metaboanalyst. When you're ready to start, follow the link at the top of top of the page labeled '[>> Click here to start<<](https://www.metaboanalyst.ca/MetaboAnalyst/ModuleView.xhtml)'.
  2. From the circle of tools select 'Statistical Analysis'
  3. Under item 1, choose Data type: Peak intensity table, samples in rows (unpaired). Select the file 'Intensities.csv' from your project directory via the 'Data file: choose file' button. Finally, click the 'submit' button to the right.
  4. Next you'll be presented with a **Data Integrity Check**. If all is well there will be no missing values, though there may be a warning about an empty row (which we ignore). As there are no missing values, skip to the next section.
  5. **Data Filtering**. The text above the tool outlines a number of different reasons why a variable may be non informative. We've performed some filtering of peaks that weren't very reproducible, but we haven't filtered any that have nearly identical intensities in all samples. IQR filtering will do that for us.
  6. **Normalization** What types of normalisation and transformation have we already performed?
     <details>
       <summary>Answer</summary>
     
     >We square-root transformed our data early on in the processing. We then applied TIC normalisation, which normalises to the _sum_ of each sample (i.e. 'sum' normalisation in the menu). We do not need to perform these steps again. For now, do not apply data scaling.
     </details>
     </br>
  7. **Analysis Paths**. Choose Principle Component Analysis (PCA). Select the '2D Scores Plot' tab. In a 2D scores plot, samples that are similar will be grouped closely together. Samples further away from each other are less similar. A coloured circle is drawn that represents the 95% confidence interval for a group of samples, in this case one for each of our soil types. What do you conclude about the similarity of the soils? Is there anything to note about reproducibility?
     <details>
       <summary>Answer</summary>
       
       >- The forest soil samples cluster closely together at some distance of the agricultural and organic soil samples.
       >- The agricultural and organic soils show little difference, with the 95% CI of the agricultural soil overlapping the 95% CI of the organic soil completely.
       >- All replicates are fairly close together, suggesting few differences and a highly reproducible result. The exception is one of the agricultural soil samples.
     </details>
     </br>
  8. Above the plot enable 'display sample names' and click 'Update' (to the right). Take a note of the replicate that is behaving differently.
  9. From the tabs at the top of the page, select the 'Loadings plot'. Rather than showing us the scores of the samples, this plot shows us in which direction the variables (in our case peaks) 'pull' our samples. Clicking on a datapoint will pull up a graph (may take a couple of seconds) and the m/z of the peak. There are three peaks that contribute a lot to the errant sample. For a combined view with labels, you can also try the 'Biplot' tab. What can we conclude from the loadings?
     <details>
       <summary>Answer</summary>
     
     >A very small minority of our peaks contributes to the variation captured by the second principle component. These peaks are potentially related, as they fall into a narrow m/z range. Perhaps the samples were not homogenous, and we ended up with a grain of a specific chemical in this replicate. Perhaps something else happened. Either way, it is an outlier that we can remove.
     </details>
     </br>
10. In the left menu pane, expand 'Processing' by clicking the triangle in front of it and select 'Data editor'. Find select the offending replicate, and exclude it (remember to click submit). Work towards the PCA again - we still don't need normalisation, but you will need to click to proceed. How has excluding one sample affected the replicate scores and the loadings? What is now the peak with the strongest influence on PC2?

     <details>
       <summary>Answer</summary>
     
     >The Y-axis covers a smaller range, so it appears that there is more spread on PC2. However, this is just the way things are shown. The variation between replicates in the agricultural and the organic soil is more similar, but it is larger than that in the forest soil. It still isn't possible to distinguish between the agricultural soil and the organic soil.\
     >The peak with the strongest influence has an m/z of 379.1352 (and the second largest is 380.1377)
     </details>
     </br>
11. As we didn't scale the data, large differences in intensity of a peak between samples will mean that peak has a large contribution to the variation. Often, the absolute size of the (random) variation is dependent on the size of the true value, meaning that we have a risk of just looking at the noise for more intense peaks rather than actual large differences between samples. The square root transformation mitigates this to some extent. That does still leave a distinction between prioritising absolute differences or relative differences (i.e. making an observation of a rare compound being present at twice the concentration as important as the doubling of the concentration of a common compound). Investigate the effects on the PCA by changing the scaling option in the normalisation options first to 'autoscaling' and then later to 'pareto'. What do you observe?
     <details>
       <summary>Answer</summary>
     
     >The scores will change, but the overall conclusions of the scores plot remains the same. In the loadings plot, however, we see a very different pattern when we apply autoscaling. In autoscaling the variation in each variable (peak) is normalised, so the contributions of variables tends to be much more even (but in different directions). Pareto scaling keeps some of the larger fold-changes, but will bring them closer to smaller fold changes. In this data set, there isn't much difference with the non-normalised data.\
     >Also: we still cannot see differences between agricultural and organic soil.
     </details>
     </br>
12. Let's look at the organic and the agricultural soil in isolation. Go back to the 'Data editor' and exclude the Forest soil samples (hint: click the first one, hold <kbd>SHIFT</kbd> and click the last one to select them in one go). Do the samples separate in the PCA?

13. Now select OrthoPLSDA. Do the samples separate in the scores plot?
