To perform the GWAS on cell count diversity:

    ./11-gwas_cellcount_entropy.sh

This takes approximately 2 minutes for sample size of 100, but time will increase quadratically with increasing sample size.
*IMPORTANT:* We don't run this analysis on cord blood samples.

### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 11 check
```

This should tell you that `Section 11 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 11 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 11`.

This procedure will be repeated at the end of each section.