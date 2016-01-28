We will be performing GWAS on the following phenotypes

- Age accelerated residuals (covariates: sex and smoking)
- Predicted smoking (covariates: age and sex)
- Cellcount entropy (the relative diversity of cell counts in the sample) (covariates: smoking, age and sex)
- Abundance of each cell type (covariates: smoking, age and sex)

These GWASs will be performed using a linear mixed model (LMM) to account for population structure and/or relatedness. Specifically, we will be using the GCTA `--mlma-loco` routine ([Yang et al 2014](http://www.nature.com/ng/journal/v46/n2/abs/ng.2876.html)). 

In addition we have also provided scripts to perform a multivariate LMM (MVLMM) to perform multi-trait GWAS on all cell count estimates jointly. This is performed using GEMMA software ([Zhou and Stephens 2014](http://www.nature.com/nmeth/journal/v11/n4/full/nmeth.2848.html)).

To perform the age accelerated residuals GWAS:

    ./09-gwas_aar.sh

This takes approximately 2 minutes for sample size of 100, but time will increase quadratically with increasing sample size.

### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 09 check
```

This should tell you that `Section 09 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 09 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 09`.

This procedure will be repeated at the end of each section.