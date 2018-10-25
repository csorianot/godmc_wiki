We will evaluate the association between genetic inversions and phenotypes derived from methylation data (age accelerated residuals, cell count entropy and abundance of each cell type) and BMI and height. For the phenotypes derived from methylation data, we will run the same models used in main GoDMC pipeline:

- Age accelerated residuals vs Inversions + sex and smoking
- Predicted smoking vs Inversions + age and sex
- Cell count entropy vs Inversions + smoking, age and sex
- Abundance of each cell type vs Inversions + smoking, age and sex
We will use the age and sex normalized BMI and height and we will not include other covariates:
- zBMI vs Inversions
- zHeight vs Inversions

We will use the same approach than in GoDMC pipeline. Therefore, we will run linear mixed models (LMMs) accounting for population structure and/or relatedness with GCTA. 

### Age acceleration
Each analysis was coded in a different script. Nonetheless, due to the low number of inversions tested, all the analyses are run in few minutes. To perform age accelerated residuals:
    
    ./22a-inv_gwas_age.sh

### Predicted smoking
For predicted smoking, we also stratified the analysis in all samples, below 25 years and above 25 years. To perform the GWAS on all samples run:

    ./22b-inv_gwas_smoking.sh 1

This script will print, in the first lines, the analysis available for your cohort. You need to run all the available cohorts before uploading the data. You can run the other batches by changing the last number:

    ./22b-inv_gwas_smoking.sh 2
    ./22b-inv_gwas_smoking.sh 3

### Cell counts entropy
To perform the analysis on cell counts entropy:

    ./22c-inv_gwas_cellcount_entropy.sh

**IMPORTANT**: We don't run this analysis on cord blood samples.

### Cell counts 
To perform the analysis on cell counts, assuming we have 7 cells types, we need to run:

    ./22d-inv_gwas_cellcounts.sh 1
    ./22d-inv_gwas_cellcounts.sh 2
    ./22d-inv_gwas_cellcounts.sh 3
    ./22d-inv_gwas_cellcounts.sh 4
    ./22d-inv_gwas_cellcounts.sh 5
    ./22d-inv_gwas_cellcounts.sh 6
    ./22d-inv_gwas_cellcounts.sh 7

**IMPORTANT**: We don't run this analysis on cord blood samples.

You can check the following file to check how many cell type you have:

    less results/12/cellcounts_columns.txt

### Height
To run the association between inversions and height:

    ./22e-inv_gwas_height.sh

### BMI
To run the association between inversions and BMI:

    ./22f-inv_gwas_BMI.sh