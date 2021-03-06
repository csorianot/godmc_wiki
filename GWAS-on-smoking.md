To perform the predicted smoking GWAS on all samples:

    ./10-gwas_smoking.sh 1

This takes approximately 2 minutes for sample size of 100, but time will increase quadratically with increasing sample size.

**By default**, where age ranges are both above and below 25 years, to perform the predicted smoking GWAS on subjects older than 25 years:

    ./10-gwas_smoking.sh 2

To perform the predicted smoking GWAS on subjects younger than 25 years:

    ./10-gwas_smoking.sh 3

If there are fewer than 100 samples with age over 25 years or age younger than 25 years then those age-specific GWASs will not be run. To see the GWASs that are eligible at this stage check the `results/10/gwas_list.txt` file. e.g. If you see:

```
smoking_prediction
smoking_prediction_ge25
```

then just run batches 1 and 2:

```
./10-gwas_smoking.sh 1
./10-gwas_smoking.sh 2
```

### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 10 check
```

This should tell you that `Section 10 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 10 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 10`.

This procedure will be repeated at the end of each section.