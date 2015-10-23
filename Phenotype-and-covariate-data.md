
## Covariates
Your covariates file should be a text file (white space separated) with `IID` in the first column followed by headed columns for covariates. The following covariates are **required**:

- `Sex`: A column of `M`'s for males and `F`'s for females
- `Age`: In years

The following covariates are strongly recommended (but not necessary):

- `Slide`: The slide ID for the methylation array
- Any other important batches

The pipeline will create genetic and methylation principal components for you so you don’t need to add PCs to this file. An example of what your file should look like is below:

    IID Sex Age Slide
    id1 F 30 12345678
    id2 M 42 12345678
    id3 M 76 87654321

Please ensure there are no missing values in the `Sex` and `Age` columns, even if your data is all the same age or all the same sex. Please also ensure that the capitalisation of column headers in the example above is followed.

## Phenotypes

We are expecting that you will provide `Height` and `BMI` phenotypes. Please provide them in the same format as the covariates, e.g.:

    IID Height BMI
    id1 1.62 23.1
    id2 1.89 20.6
    id4 1.70 22.7

Note the capitalisation!!

We are expecting height in metres and BMI in kg/m2 units.

## Cell counts

Ideally you should provide cell counts for 7 cell types (Bcells, CD4T, CD8T, Neutrophils, Eosinophils, Monocytes, Natural Killer cells). Please note that we separated Granulocytes in neutrophils and eosinophils. These cell counts can be either estimated using meffil if you have access to idat files or directly measured. If you don't have access to idat files and don't have directly measured cell counts, the pipeline will estimate cell counts from betas by setting "provided cellcounts" to "NULL" in the config file. Please find below some code to estimate cell counts from idat files.
```
library(meffil)
options(mc.cores=8)
samplesheet <- meffil.create.samplesheet(path_to_idat_files) ##please edit
qc.objects <- meffil.qc(samplesheet, cell.type.reference="blood gse35069 complete", verbose=TRUE)
save(qc.objects,file="qc.objects.Robj")
cc<-mclapply(qc.objects,function (qc.object) meffil.estimate.cell.counts(qc.object,cell.type.reference="blood gse35069 complete")$counts)
cc2<-data.frame(IID=names(cc),t(do.call(cbind,cc)))
write.table(cc2,"cellcounts.txt",sep="\t",quote=F,row.names=F,col.names=T)
```

Please make sure you provide cell counts in the same format as the phenotypes/covariates and save it as a separate file. Your header should be the same as in the example below. 

    IID Bcell CD4T CD8T Eos Mono Neu NK
    0.01737535 0.0000000 3.750239e-18 -3.255819e-19 0.06323692 0.8449611 0.09284286
    0.07131901 0.0532302 1.147492e-02 0.000000e+00 0.05687753 0.7669148 0.07237679
    0.16438292 0.1637806 -8.673617e-19 -1.042249e-19 0.11164848 0.3932823 0.25849168