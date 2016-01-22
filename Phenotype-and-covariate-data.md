
## Covariates
Your covariates file should be a text file (white space separated) with `IID` in the first column followed by headed columns for covariates. You need to specify whether your covariate is a categorical (`factor`) or continuous (`numeric`) variable. For example: `{Name}_factor` or `{Name}_numeric`.  The following covariates are **required**:

- `Sex_factor`: A column of `M`'s for males and `F`'s for females
- `Age_numeric`: In years

The following covariates are strongly recommended (but not necessary):

- `Slide_factor`: The slide ID for the methylation array
- Any other important batches

The pipeline will create genetic and methylation principal components for you so you don’t need to add PCs to this file. An example of what your file should look like is below:

    IID Sex_factor Age_numeric Slide_factor
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

We use cell counts as covariates in the methQTL analysis and as phenotypes in the GWA analysis.

### Cell counts to use as covariates
To use cell counts as covariates you can either provide predicted cell counts or directly measured cell counts. If you have directly measured cell counts then ideally you should provide cell counts for 7 cell types (Bcells, CD4T, CD8T, Neutrophils, Eosinophils, Monocytes, Natural Killer cells). If you use predicted cell counts, you can extract cell counts from idat files using `meffil` which has several blood reference datasets implemented including two reference datasets for cord blood. `Meffil` is using the Houseman method to predict cell counts.


If no cell counts file has been provided, we will estimate cell counts on the normalized betas using `meffil` during the course of running the pipeline. Note that in estimating cell counts we separate granulocytes into neutrophils and eosinophils, so please do the same if you wish to provide your own cell count estimates.

### Cell counts as GWA phenotypes
The GWA on cell counts will be done on Houseman predicted cell counts for the following 6 cell types:
Bcells, CD4T, CD8T, Neutrophils, Monocytes, Natural Killer cells. Please note that we splitted Granulocytes into Neutrophils and Eosinophils. 

### Predict cell counts from idat files (Houseman or cord blood)

###Extraction of cell counts
   ```r
   
   cc<-t(sapply(qc.objects, function(obj) obj$cell.counts$counts))
   cc<-data.frame(IID=row.names(cc),cc)
   write.table(cc,"cellcounts.txt",sep="\t",row.names=F,col.names=T,quote=F)
   ```

### Cell counts file(s) format
If you are providing your own cell count estimates, please make sure you use the same format as the phenotypes and covariate files and save it as a separate file. Your header should be the same as in the example below. 

```
IID Bcell CD4T CD8T Eos Mono Neu NK
0.01737535 0.0000000 3.750239e-18 -3.255819e-19 0.06323692 0.8449611 0.09284286
0.07131901 0.0532302 1.147492e-02 0.000000e+00 0.05687753 0.7669148 0.07237679
0.16438292 0.1637806 -8.673617e-19 -1.042249e-19 0.11164848 0.3932823 0.25849168
```