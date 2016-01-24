This page will show you how to use `R/meffil` for normalisation and post-normalisation QC after you following the [sample QC](Methylation-sample-QC)

[Meffil](https://github.com/perishky/meffil) performs normalization and post normalization QC for data measured with Illumina 450k arrays. It uses multi-threading and reads `idat` files directly to maximise speed and minimise memory use. It is therefore much faster than minfi.


### Load in the QC objects

Load meffil and set how many cores to use for parallelization

```r
library(meffil)
options(mc.cores=16)
```

Load the `qc.objects` and `qc.summary` you generated during the sample QC process

```r
load("qc.objects.clean.Robj")
length(qc.objects)
load("qcsummary.clean.Robj")
```

or generate qc.objects with the code below:

```r
samplesheet <- meffil.create.samplesheet(path_to_idat_files)
qc.objects <- meffil.qc(samplesheet, cell.type.reference="blood gse35069 complete", verbose=TRUE)
save(qc.objects,file="qc.objects.clean.Robj")
```

### Estimate the number of principal components to use

Next we need to estimate how many principal components (PCs) to use to adjust the methylation levels for technical effects. We can use 10-fold cross validation to estimate the residual variance after fitting `n` number of PCs. The residuals should consistently decrease with increasing numbers of PCs. The plot generated is similar to an elbow curve. The idea is to choose the number of PCs at which the residuals decreases abruptly. Sometimes there is more than one elbow and I would choose the one with the highest number of PCs. For ARIES there was a dramatic drop around 10 PCs.

```r
y <- meffil.plot.pc.fit(qc.objects)
ggsave(y$plot,filename="pc.fit.pdf",height=6,width=6)
```

Set the number of PCs to use going forward, e.g. to use 10 PCs:

```r
pc <- 10
```

### Perform quantile normalisation

And now perform quantile normalization. Bad CpGs due to poor detection scores which we identified in the `qc.summary` are also removed.

```r
norm.objects <- meffil.normalize.quantiles(qc.objects, number.pcs=pc)
save(norm.objects,file=paste("norm.obj.pc",pc,".Robj",sep=""))

norm.beta <- meffil.normalize.samples(norm.objects, cpglist.remove=qc.summary$bad.cpgs$name)
save(norm.beta,file=paste("norm.beta.pc",pc,".Robj",sep=""))
```

### Generate normalisation report

In the normalization report, principal components analysis will be performed on the control matrix and on the most variable probes. 

In addition associations will be calculated between the first 10 PCs and the batch variables that were specified in the `samplesheet`. The association tests can be used to identify possible outliers. For example, if `Slide` is one of your batch variables, it gives a p-value for each `Slide` rather than an overall p-value. Poor slides can be identified and removed post-normalization. 

In the `norm.parameters` variable, you can set your batch variables eg. `Slide`, `plate`, and `tissue` etc., the number of extracted PCs in the control matrix, the number of extracted PCs from the normalized betas, number of variable probes and the p-value threshold used for the association testing.

Set the parameters to use in normalisation:


#Set your batch variables
#It is important to code your batch variables as a factor in order to look at the associations between PCs and your batch variables eg. Slide, sentrix_row, sentrix_col, Sex and other batch should be coded as a factor. You can check this with: 
```r

str(norm.objects[[1]]$samplesheet)

#You change it by running a loop

for (i in 1:length(norm.objects)){
norm.objects[[i]]$samplesheet$Slide<-as.factor(norm.objects[[i]]$samplesheet$Slide)
norm.objects[[i]]$samplesheet$Sex<-as.factor(norm.objects[[i]]$samplesheet$Sex)
norm.objects[[i]]$samplesheet$sentrix_row<-as.factor(norm.objects[[i]]$samplesheet$sentrix_row)
norm.objects[[i]]$samplesheet$sentrix_col<-as.factor(norm.objects[[i]]$samplesheet$sentrix_col)
}

batch_var<-c("Slide", "plate","Sex")
norm.parameters <- meffil.normalization.parameters(
	norm.objects,
	variables=batch_var,
	control.pcs=1:10,
	probe.pcs=1:10,
	probe.range=20000,
	batch.threshold=0.01
)
```

Run pcs, normalization summary and make normalisation report. 

```r
pcs <- meffil.methylation.pcs(norm.beta,probe.range=20000)
save(pcs,file="pcs.norm.beta.Robj")
norm.summary <- meffil.normalization.summary(norm.objects, pcs=pcs,parameters=norm.parameters)
meffil.normalization.report(norm.summary, output.file="normalization-report.html")
```

This creates the file `normalization-report.html` in the current work directory. The file should open up in your web browser.

### QC report details -please check!

- Control probe scree plots: how many PCs do you need to explain at least 99% of the variance? From which PC is less than 1% of the variance explained?
- PCAplots of control probes: do you see any clusters indicating possible batch effects? Only batches with less than 10 levels are shown. Row effects are often captured with the control probes.
- Control probe associations of PCs with measured batch variables: The plots show anova test statistics which can be used to identify possible associations between a PC and a batch. For example in ARIES, we found significant associations for plate and slide.
- The linear model can be used to identify a possible association between a PC extracted from the control probes and a batch level. Are there any levels of a batch significant? The plots show coefficients with their 95% confidence interval of the t-test statistics. These can be used for example to identify poor slides.
- PCAplots of normalized betas: do you see any clusters indicating possible batch effects? Only batches with less than 10 levels are shown.
- Normalized probe associations with measured batch variables: The anova test is used to identify possible associations between a PC and a batch. 
- The linear model can be used to identify a possible association between a PC extracted from normalised betas and a batch level. Are there any levels of a batch significant? The plots show coefficients with their 95% confidence interval of the t-test statistics. These can be used for example to identify poor slides.

For ARIES, we found one slide significantly associated with several PCs. We removed this slide from the normalization matrix. The associations could also inform you which batch variables you should include in your downstream methQTL analysis.

###Extraction of cell counts
   ```r
   cc<-t(sapply(qc.objects, function(obj) obj$cell.counts$counts))
   cc<-data.frame(IID=row.names(cc),cc)
   write.table(cc,"cellcounts.txt",sep="\t",row.names=F,col.names=T,quote=F)
   ```