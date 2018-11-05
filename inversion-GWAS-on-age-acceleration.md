We will evaluate the association between genetic inversions and phenotypes derived from methylation data (age accelerated residuals, cell count entropy and abundance of each cell type) and BMI and height. For the phenotypes derived from methylation data, we will run the same models used in main GoDMC pipeline:

- Age accelerated residuals vs Inversions + sex and smoking
- Predicted smoking vs Inversions + age and sex
- Cell count entropy vs Inversions + smoking, age and sex
- Abundance of each cell type vs Inversions + smoking, age and sex
We will use the age and sex normalized BMI and height and we will not include other covariates:
- zBMI vs Inversions
- zHeight vs Inversions

We will use the same approach than in GoDMC pipeline. Therefore, we will run linear mixed models (LMMs) accounting for population structure and/or relatedness with GCTA. 

Each analysis was coded in a different script. Nonetheless, due to the low number of inversions tested, all the analyses are run in few minutes. 

To perform GWAS on age accelerated residuals:
    
    ./22-inv_gwas_age.sh





### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 22 check
```

This should tell you that `Section 22 has been successfully completed!`. Now please upload the results like this:

```
./check_upload.sh 22 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 22`.

This procedure will be repeated at the end of each section.