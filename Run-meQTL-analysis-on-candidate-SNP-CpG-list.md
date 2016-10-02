As a check, we perform some sensitivity analysis on the first set of probes using PLINK. 


Before running the methQTL associations we need to prepare the data for PLINK.

```
16a-preparegctafiles.sh
```

We run PLINK only on the first probeset. 

    ./16-gcta.sh

The script will perform the meQTL analysis on the first set of probes, using `nthreads` threads in parallel (specified in the `config` file). 

### Check and upload the results

The results are in `results/16`.

To check that everything ran successfully, please run:

```
./check_upload.sh 16 check
```

This should tell you that `Section 16 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 16 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 16.

This procedure will be repeated at the end of each section.