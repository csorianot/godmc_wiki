To perform the analysis on cell counts entropy:

    ./24-inv_gwas_cellcount_entropy.sh

**IMPORTANT**: We don't run this analysis on cord blood samples.


### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 24 check
```

This should tell you that `Section 24 has been successfully completed!`. Now please upload the results like this:

```
./check_upload.sh 24 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 24`.

This procedure will be repeated at the end of each section.
