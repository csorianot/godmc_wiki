To perform the GWAS of each cell type, assuming we have 7 cell types, we need to run:

```
./12-gwas_cellcounts.sh 1
./12-gwas_cellcounts.sh 2
./12-gwas_cellcounts.sh 3
./12-gwas_cellcounts.sh 4
./12-gwas_cellcounts.sh 5
./12-gwas_cellcounts.sh 6
./12-gwas_cellcounts.sh 7
```

In order to check how many cell types there are, you can check the following file:

```
less results/12/cellcounts_columns.txt
```

This takes approximately 2 minutes for sample size of 100 for each cell type, but time will increase quadratically with increasing sample size.

### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 12 check
```

This should tell you that `Section 12 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 12 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 12`.

This procedure will be repeated at the end of each section.