We have developed a R package called [meffil](https://github.com/perishky/meffil) to perform QC for data measured with Illumina 450k arrays. It is using parallelization and reads `.idat` files directly. It is therefore much faster than minfi. Please email [josine.min@bristol.ac.uk] or [g.hemani@bristol.ac.uk] if you have any questions.

- Install the meffil R package for the first time 

On the command line:

```
mkdir meffil
cd meffil
wget https://github.com/perishky/meffil/archive/master.zip
unzip master.zip
mv meffil-master meffil
R CMD INSTALL meffil
```

- Please make sure you update meffil to the latest version.

In R:

```
library(devtools)
install_github("perishky/meffil")
```

- Load meffil and set how many cores to use for parallelization

```
library(meffil)
options(mc.cores=16)
```

- Generate a samplesheet with your samples. The samplesheet can be generated automatically from the idat basenames by giving the directory with idat files or it can be done manually. It should contain at least the following necessary columns: "Sample Name", "Sex" (possible values "M","F" or "NA") and "Basename". It tries to parse the basenames to guess if the Sentrix plate and positions are present. At this point it is worthwhile to manually modify the samplesheet data.frame to replace the actual sample IDs in the Sample_Name column if necessary, and to add the sex values to the Sex column. Don't change these column names though.

```
samplesheet <- meffil.create.samplesheet(path_to_idat_files)
```

- Perform the background correction, dye bias correction, sex prediction and cell count estimates. This function loads all your idat files and returns a qc.object for each sample. 

```
qc.objects <- meffil.qc(samplesheet, cell.type.reference="blood gse35069", verbose=TRUE)
save(qc.objects,file="qc.objects.Robj")
```

You can check if your samples are there by checking the number of samples and extracting the sample names.

```
length(qc.objects)
```


Sample names can be checked like this

```
names(qc.objects)
```

You can check data for the first sample

```
names(qc.objects[[1]])
qc.objects[[1]]$sample.name
```

Extract snp.betas from your qc.objects

```
snp.betas<-meffil.snp.betas(qc.objects)
save(snp.betas,file="snp.betas.Robj")
```

- To compare GWA array genotypes to methylation array genotypes, a matrix of of GWA genotypes needs to be obtained (rows=SNPs, columns=samples). You can generate a GWA genotype matrix using PLINK. Please note that sample names should be matching for the two matrices.

In R:

```
writeLines(meffil.get.snp.probes(), con="snp-names.txt")
```

command shell > 

```
plink --bfile dataset --extract snp-names.txt --recode A --out genotypes.raw
```

In R:

```
plinkfiles <- list.files(path_to_plink_files, pattern="*.raw")
plinkfiles<-paste(path_to_plink_files, plinkfiles,sep="")
genotypes <- meffil.extract.genotypes(plinkfiles)
genotypes <- genotypes[,match(samplesheet$Sample_Name, colnames(genotypes))]
```

- We can now set some parameters for the QC of the raw data.

```
qc.parameters <-meffil.qc.parameters(beadnum.samples.threshold=0.1,detectionp.samples.threshold=0.1, detectionp.cpgs.threshold = 0.1, beadnum.cpgs.threshold = 0.1,sex.outlier.sd=5, snp.concordance.threshold=0.95, sample.genotype.concordance.threshold=0.8)
```

beadnum.samples.threshold = fraction of probes that failed the threshold of 3 beads.
detectionp.samples.threshold = fraction of probes that failed a detection.pvalue threshold of 0.01.
beadnum.cpgs.threshold = fraction of samples that failed the threshold of 3 beads.
detectionp.cpgs.threshold = fraction of samples that failed the detection.pvalue threshold of 0.01.
sex.outlier.sd= number of standard deviations to determine whether sample is sex outlier 
snp.concordance.threshold= concordance threshold to include snps to calculate sample concordance 
sample.genotype.concordance.threshold=concordance threshold to determine whether sample is outlier

- We can now summarise the QC analysis of the raw data. 

```
qc.summary <- meffil.qc.summary(qc.objects,parameters=qc.parameters,genotypes=genotypes)
save(qc.summary, file="qcsummary.Robj")
```

- We can also make a qc.report.

```
meffil.qc.report(qc.summary, output.file="qc-report.html")
```

This creates the file "qc-report.html" in the current work directory. The file should open up in your web browser.

- After checking the qc report you can select bad samples to remove. I wouldn't remove outliers based on control probes as there only a few probes for each category. However, I would remove slides with dyebias which is captured by the normalisation probes ((normC + normG)/(normA + normT)).

```
outlier<-qc.summary$bad.samples
table(outlier$issue)
w<-which(outlier$issue%in%c("Control probe (dye.bias)","Methylated vs Unmethylated","X-Y ratio outlier","Low bead numbers","Detection p-value","Sex mismatch","Genotype mismatch"))
outlier<-outlier[w,]
```

- You can remove bad samples prior to performing quantile normalization. Bad probes can be removed later on.

```
length(qc.objects)
qc.objects <- meffil.remove.samples(qc.objects, outlier)
length(qc.objects)
save(qc.objects,file="qc.objects.clean.Robj")
```

- Rerun qc summary on clean dataset
```
qc.summary <- meffil.qc.summary(qc.objects,parameters=qc.parameters,genotypes=genotypes)
save(qc.summary, file=”qcsummary.clean.Robj")
```

- Rerun qc report on clean dataset

meffil.qc.report(qc.summary, output.file="qc-report.clean.html")

-Extract cell counts on clean dataset. It gives you 7 cell counts (Bcells, CD4T, CD8T, Neutrophils, Eosinophils, Monocytes, Natural Killer cells).

```
counts <- t(sapply(qc.objects, meffil.estimate.cell.counts, cell.type.reference="blood gse35069 complete", verbose = T))
cell.counts<-data.frame(IID=row.names(cell.counts),cell.counts)
write.table(cell.counts,"cellcounts.txt",sep="\t",quote=F,row.names=F,col.names=T)
```