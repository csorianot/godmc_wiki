To perform the predicted smoking GWAS:

    ./10-gwas_smoking.sh

This takes approximately 2 minutes for sample size of 100, but time will increase quadratically with increasing sample size.

### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload 10 check
```

This should tell you that `Section 10 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload 10 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 10`.

This procedure will be repeated at the end of each section.