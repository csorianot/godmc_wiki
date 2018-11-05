For predicted smoking, we also stratified the analysis in all samples, below 25 years and above 25 years. To perform the GWAS on all samples run:

    ./23-inv_gwas_smoking.sh 1

This script will print, in the first lines, the analysis available for your cohort. You need to run all the available cohorts before uploading the data. You can run the other batches by changing the last number:

    ./23-inv_gwas_smoking.sh 2
    ./23-inv_gwas_smoking.sh 3

### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 23 check
```

This should tell you that `Section 23 has been successfully completed!`. Now please upload the results like this:

```
./check_upload.sh 23 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 23`.

This procedure will be repeated at the end of each section.
