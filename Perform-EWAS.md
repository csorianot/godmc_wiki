An EWAS will be performed on each of the phenotypes provided in the `phenotypes` file. For example, if you have provided a file with `Height` and `BMI` columns, then each of these will be analysed with the scripts below. If you have only provided one of the phenotypes please run the script for `Height` or `BMI` only. It uses pre-normalised methylation levels and phenotypes and uses all covariates eg. age, sex, cell counts, study specific covariates eg. batch, slide etc., genetic pcs and non-genetic methylation pcs. The scripts are using four different models to run EWAS implemented in `meffil`:

1. none=no covariates
2. all=user covariates
3. isva0=independent surrogate variables (unsupervised)
4. isva1=independent surrogate variables derived using "all" and "isva0" covariates (supervised)

See [here](http://www.ncbi.nlm.nih.gov/pubmed/21471010/) for a link to the isva paper.

To perform the EWAS run:

    ./08a-ewas.height.sh
    ./08b-ewas.bmi.sh

Please check that things look as expected by visually inspecting the Q-Q plots for each of the four models created in the `results/08/` directory.


### Upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 08 check
```

This should tell you that `Section 08 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 08 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 08.

This procedure will be repeated at the end of each section.