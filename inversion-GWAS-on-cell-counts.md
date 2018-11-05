To perform the analysis on cell counts, assuming we have 7 cells types, we need to run:

    ./25-inv_gwas_cellcounts.sh 1
    ./25-inv_gwas_cellcounts.sh 2
    ./25-inv_gwas_cellcounts.sh 3
    ./25-inv_gwas_cellcounts.sh 4
    ./25-inv_gwas_cellcounts.sh 5
    ./25-inv_gwas_cellcounts.sh 6
    ./25-inv_gwas_cellcounts.sh 7

**IMPORTANT**: We don't run this analysis on cord blood samples.

You can check the following file to check how many cell type you have:

    less results/12/cellcounts_columns.txt


### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 25 check
```

This should tell you that `Section 25 has been successfully completed!`. Now please upload the results like this:

```
./check_upload.sh 25 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 25`.

This procedure will be repeated at the end of each section.
