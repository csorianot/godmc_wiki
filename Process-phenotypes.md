### Normalise phenotypes

For the EWAS we need to normalise the BMI and height phenotypes. The following will be done:

- Remove 5 SD outliers
- Normalise height using inverse-normal transformation separately for males and females
- Adjust height for age and age squared (if they are nominally associated) in males and females separately
- Standardise males and females separately before combining
- Perform the same for BMI

To do this, run:

    ./03a-phenotype_data.sh

**IMPORTANT**: 
- This script also generates plots of the phenotypes. Please check that the plots in `results/03/ewas_phenotypes.pdf` look sane.
- If you have **cord blood samples**, you don't need to run this script.

### Test the genotype and phenotype quality

Next, as a check for the genotype and phenotype data we will generate a genetic prediction for height using the SNPs from [Wood et al 2014](http://www.ncbi.nlm.nih.gov/pubmed/25282103) and test its correlation with the height phenotype (if it is provided). To do this, run:

    ./03b-height_prediction.sh

Please check the plots in `results/03/heightcor_plot.pdf` to make sure that you are seeing a reasonable correlation. Anything below r-square of 0.1 in adults is cause for concern!! We found a correlation of 0.29 in ARIES. 


### Upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 03 check
```

This should tell you that `Section 03 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 03 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 03.

This procedure will be repeated at the end of each section.