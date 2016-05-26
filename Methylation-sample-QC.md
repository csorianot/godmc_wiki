We have developed an R package called [meffil](https://github.com/perishky/meffil) to perform QC for data measured with Illumina 450k arrays. **Please make sure you have installed version 1.0.0 or higher.** You can check the version like this:

```
packageVersion("meffil")
``` 

It is using parallelization and reads `.idat` files directly. It is therefore much faster than `minfi` and uses much less memory. Please email [josine.min@bristol.ac.uk](josine.min@bristol.ac.uk) or [g.hemani@bristol.ac.uk](g.hemani@bristol.ac.uk) if you have any questions.

### Installation

Install the `meffil` R package for the first time. On the command line:

```r
source("http://bioconductor.org/biocLite.R")
biocLite("illuminaio")
biocLite("limma")
biocLite("IlluminaHumanMethylation450kmanifest")
biocLite("IlluminaHumanMethylation450kanno.ilmn12.hg19")
biocLite("CopyNumber450kData")
biocLite("DNAcopy")
install.packages("markdown")
install.packages("knitr")
install.packages("devtools")
library(devtools)
install_github("perishky/meffil")
```

### Generating QC objects

Load meffil and set how many cores to use for parallelization

```r
library(meffil)
options(mc.cores=16)
```

Generate a samplesheet with your samples. The samplesheet can be generated automatically from the `idat` basenames by giving the directory with `idat` files or it can be done manually. It should contain at least the following necessary columns: `Sample_Name`, `Sex` (possible values "M","F" or "NA") and `Basename`. 
`Basename` is the path to the idat file without the _Red.idat and _Grn.idat extension.  It tries to parse the basenames to guess if the Sentrix plate and positions are present. 

```r
samplesheet <- meffil.create.samplesheet(path_to_idat_files)
```

Please note: 
- If you have your idat files in more than one directory you need to make a samplesheet for each directory followed by `rbind` to combine the samplesheets into one sample sheet. Please contact us if you have idat files in multiple directories. 
- If you have more idat files in your directory than needed you can simply delete the rows from the sample sheet.
- At this point it is worthwhile to manually modify the samplesheet data frame to replace the actual sample IDs in the `Sample_Name` column if necessary, and to add the sex values to the `Sex` column. Don't change these column names though.

Alternatively, if you have already a samplesheet with batch variables you can read it in using:
```r
samplesheet <- meffil.read.samplesheet(base=".",pattern="mysamplesheet.csv")
```

Next perform the background correction, dye bias correction, sex prediction and cell count estimates. The `meffil.qc` function processes your `idat` files and returns a qc.object for each sample. You need to specify which cell type reference you need. You can find cell type references with the `meffil.list.cell.type.references()` function. Currently there are whole blood and cord blood references implemented. Cell count predictions are done using the [Houseman et al.](http://www.ncbi.nlm.nih.gov/pubmed/22568884). meffil and minfi are all using the houseman algorithm for estimating cell counts. In meffil we do something slightly different by normalizing each sample individually to the cell type reference dataset rather than normalizing all samples in a dataset to the cell type reference in order to avoid having cell count estimates depend on the other samples being included in the normalization.

```r
meffil.list.cell.type.references()
qc.objects <- meffil.qc(samplesheet, cell.type.reference="blood gse35069 complete", verbose=TRUE)
save(qc.objects,file="qc.objects.Robj")
```

You can check if your samples are there by checking the number of samples and extracting the sample names.

```r
length(qc.objects)
```

Sample names can be checked like this

```r
names(qc.objects)
```

You can check data for the first sample

```r
names(qc.objects[[1]])
qc.objects[[1]]$sample.name
```

### Obtaining genotype data to check for ID mismatches

It is important that the SNPs in your genotype data and the methylation levels in this data have corresponding IDs. This can be done as part of the QC process by extracting a set of control SNPs from your genotype data.

In R:

```r
writeLines(meffil.snp.names(), con="snp-names.txt")
```

In the UNIX command shell (you will need `plink1.90`, available [here](https://www.cog-genomics.org/plink2))

```r
plink --bfile dataset --extract snp-names.txt --recode A --out genotypes
```

In R:

```r
genotypes <- meffil.extract.genotypes("genotypes.raw")
genotypes <- genotypes[,match(samplesheet$Sample_Name, colnames(genotypes))]
```

### Generating a QC report

We can now set some parameters for the QC of the raw data.

```r
        qc.parameters <- meffil.qc.parameters(
	beadnum.samples.threshold             = 0.1,
	detectionp.samples.threshold          = 0.1,
	detectionp.cpgs.threshold             = 0.1, 
	beadnum.cpgs.threshold                = 0.1,
	sex.outlier.sd                        = 5,
	snp.concordance.threshold             = 0.95,
	sample.genotype.concordance.threshold = 0.8
)
```

    beadnum.samples.threshold` = fraction of probes that failed the threshold of 3 beads.
    detectionp.samples.threshold` = fraction of probes that failed a detection.pvalue threshold of 0.01.
    beadnum.cpgs.threshold` = fraction of samples that failed the threshold of 3 beads.
    detectionp.cpgs.threshold` = fraction of samples that failed the detection.pvalue threshold of 0.01.
    sex.outlier.sd` = number of standard deviations to determine whether sample is sex outlier 
    snp.concordance.threshold` = concordance threshold to include snps to calculate sample concordance 
    sample.genotype.concordance.threshold` = concordance threshold to determine whether sample is outlier

We can now summarise the QC analysis of the raw data. 

```r
qc.summary <- meffil.qc.summary(
	qc.objects,
	parameters = qc.parameters,
	genotypes=genotypes
)

save(qc.summary, file="qcsummary.Robj")
```

We can also make a `qc.report`.

```r
meffil.qc.report(qc.summary, output.file="qc-report.html")
```

This creates the file `qc-report.html` in the current work directory. The file should open up in your web browser.


### Removing bad samples

After checking the qc report you can select bad samples to remove. I would remove all outliers in `qc.summary$bad.samples` except the outliers based on control probes as there are often a few probes for each control type only.  For the control probe categories, I would remove slides with dye bias `Control probe (dye.bias)` which are captured by the normalisation probes `(normC + normG)/(normA + normT)` or bisulfite conversion control probes (`Control probe (bisulfite1)` and `Control probe (bisulfite2)`) .

```r
outlier <- qc.summary$bad.samples
table(outlier$issue)
index <- outlier$issue %in% c("Control probe (dye.bias)", 
                              "Methylated vs Unmethylated",
                              "X-Y ratio outlier",
                              "Low bead numbers",
                              "Detection p-value",
                              "Sex mismatch",
                              "Genotype mismatch",
                              "Control probe (bisulfite1)",
                              "Control probe (bisulfite2)")

outlier <- outlier[index,]
```

You can remove bad samples prior to performing quantile normalization. Bad probes can be removed later on.

```r
length(qc.objects)
qc.objects <- meffil.remove.samples(qc.objects, outlier$sample.name)
length(qc.objects)
save(qc.objects,file="qc.objects.clean.Robj")
```

Rerun QC summary on clean dataset

```r
qc.summary <- meffil.qc.summary(qc.objects,parameters=qc.parameters,genotypes=genotypes)
save(qc.summary, file="qcsummary.clean.Robj")
```

Rerun QC report on clean dataset

```r
meffil.qc.report(qc.summary, output.file="qc-report.clean.html")
```