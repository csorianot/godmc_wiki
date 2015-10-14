This page will show you how to use `R/meffil` for normalisation and post-normalisation QC after you have used it for sample QC.

[Meffil](https://github.com/perishky/meffil) performs normalization and post normalization QC for data measured with Illumina 450k arrays. It is using parallelization and reads idat files directly. It is therefore much faster than minfi. Please email [josine.min@bristol.ac.uk] or [g.hemani@bristol.ac.uk] if you have any questions.

- Load meffil and set how many cores to use for parallelization

```
library(meffil)
options(mc.cores=16)
```

- Load qc.objects and qc.summary you have generated during the sample QC process 

```
load("qc.objects.clean.Robj")
length(qc.objects)
load("qcsummary.clean.Robj")
```
or generate qc.objects with the code below:
```
samplesheet <- meffil.create.samplesheet(path_to_idat_files)
qc.objects <- meffil.qc(samplesheet, cell.type.reference="blood gse35069 complete", verbose=TRUE)
save(qc.objects,file="qc.objects.clean.Robj")
```

- Plot residuals remaining after fitting control matrix to decide on the number PCs to include in the normalization below. The residuals should consistently decrease with increasing numbers of components.  For ARIES there was a dramatic drop around 10 PCs.

```
y<-meffil.plot.pc.fit(qc.objects)
ggsave(y$plot,filename="pc.fit.pdf",height=6,width=6)
```

-And now perform quantile normalization. Bad CpGs due to poor detection scores which we identified in the qc.summary are removed.

```
norm.objects <- meffil.normalize.quantiles(qc.objects, number.pcs=pc)
save(norm.objects,file=paste(norm.obj.pc",pc,".Robj",sep=""))

norm.beta <- meffil.normalize.samples(norm.objects, cpglist.remove=qc.summary$bad.cpgs$name)
save(norm.objects,file=paste(norm.obj.pc",pc,".Robj",sep=""))
```

- In the normalization report, principal components analysis will be performed on the control matrix and on the most variable probes. In addition associations will be calculated between the first 10 PCs and the batch variables using anova and a linear model (t-test). The t-test can be used to identify possible outliers. For example, if Slide is one of your batch variables, it gives a pvalue for each Slide rather than an overall p-value. Poor slides can be identified and removed post-normalization. In the norm.parameters variable, you can set your batch variables eg. Slide, plate, and tissue etc., the number of extracted PCs in the control matrix, the number of extracted PCs from the normalized betas, number of variable probes and the p value threshold used for the association testing. 

```
batch_var<-c(mybatchvariables)
norm.parameters<-meffil.normalization.parameters(norm.objects,variables=batch_var,control.pcs=1:10,probe.pcs=1:10,probe.range=20000,batch.threshold=0.01)
```
-Run normalization summary and make normalisation report. 

```
norm.summary <- meffil.normalization.summary(norm.beta, norm.objects)
meffil.normalization.report(norm.summary, output.file="normalization-report.html")
```
- This creates the file "normalization-report.html" in the current work directory. The file should open up in your web browser.
- Control probe scree plots: how many PCs do you need to explain at least 99% of the variance? From which PC is less than 1% of the variance explained?
- PCAplots of control probes: do you see any clusters indicating possible batch effects? Only batches with less than 10 levels are shown.
- Control probe associations of PCs with measured batch variables: The plots show anova test statistics which can be used to identify possible associations between a PC and a batch. For example in ALSPAC, we found significant associations for plate and slide.
- The linear model can be used to identify a possible association between a PC extracted from the control probes and a batch level. Are there any levels of a batch significant? The plots show coefficients with their 95% confidence interval of the t-test statistics. These can be used for example to identify poor slides.
- PCAplots of normalized betas: do you see any clusters indicating possible batch effects? Only batches with less than 10 levels are shown.
- Normalized probe associations with measured batch variables: The anova test is used to identify possible associations between a PC and a batch. 
- The linear model can be used to identify a possible association between a PC extracted from normalised betas and a batch level. Are there any levels of a batch significant? The plots show coefficients with their 95% confidence interval of the t-test statistics. These can be used for example to identify poor slides.
For ALSPAC, we found one slide significantly associated with several PCs. We removed this slide from the normalization matrix. The associations could also inform you which batch variables you should include in your downstream methQTL analysis.