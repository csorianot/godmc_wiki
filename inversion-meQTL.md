We will run a meQTL analysis using the chromosomal inversions. We will use the same strategy than in GoDMC phase 2 but including all CpGs and inversions. We will use the normalized and adjusted methylation data and the inversion genotypes inferred in the previous step.

You can run this step with:
    
    ./21-inversionmeQTL.sh

For a sample size of 400 and using 16 cores, this step took 1 hour to complete. 


### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 21 check
```

This should tell you that `Section 21 has been successfully completed!`. Now please upload the results like this:

```
./check_upload.sh 21 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 21`.

This procedure will be repeated at the end of each section.
