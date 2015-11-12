### Normalise phenotypes

For the EWAS we need to normalise the BMI and height phenotypes. The following will be done:

- Normalise height using inverse-normal transformation separately for males and females
- Adjust height for age and age squared (if they are nominally associated)
- Perform the same for BMI

To do this, run:

    ./03a-phenotype_data.sh

This script also generates plots of the phenotypes. Please check that the plots in `log_files/phenotype_data/ewas_phenotypes.pdf` look sane.

### Test that the genotype and phenotype quality

Next, as a check for the genotype and phenotype data we will generate a genetic prediction for height using the SNPs from [Wood et al 2014](http://www.ncbi.nlm.nih.gov/pubmed/25282103) and test its correlation with the height phenotype (if it is provided). To do this, run:

    ./03b-height_prediction.sh

Please check the plots in `log_files/height_prediction/heightcor_plot.pdf` to make sure that you are seeing a reasonable correlation. Anything below r-square of 0.1 is cause for concern!!


### Upload the results

To check that everything ran successfully, please run:

```
./check_upload 03 check
```

This should tell you that `Section 03 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload 03 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 03.

This procedure will be repeated at the end of each section.