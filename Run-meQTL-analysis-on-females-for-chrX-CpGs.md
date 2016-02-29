We have already performed the standard meQTL analysis. We are now going to perform the same kind of analysis again except this time we will look at ChrX methylation probes in females only. The procedure is exactly the same as in 05, the SNPs have been split into `genetic_chunks` chunks which can each be run independently on a different node on the cluster. 

    ./14-mqtl_females.sh 1


The script will perform the meQTL analysis using SNPs 1-16000 (the first chunk of 500), using `nthreads` threads in parallel (also specified in the `config` file). 

Going through 1-500 sequentially is likely to take some time so it is recommended that you parallelise across a cluster. For ~1800 samples, one chunk took 25 minutes to run using 16 cores.

An example job submission script (e.g. `submit_mqtlX_females.sh`) would be:

```bash

#!/bin/bash

#PBS -N mqtlXf
#PBS -o mqtlXf-output
#PBS -e mqtlXf-error
#PBS -l walltime=12:00:00
#PBS -t 1-500
#PBS -l nodes=1:ppn=16
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/    #EDIT THIS LINE
./14-mqtl_females.sh ${PBS_ARRAYID}

```

Then, when this is submitted:

    qsub submit_mqtlX_females.sh

it will create a batch of 500 jobs, each running with the variable `$PBS_ARRAYID` set to a value between 1-500, and each individual job further parallelised across 16 threads. 


### Check and upload the results

The results are in `results/14`.

Please open one of the *RData files and check if the output looks fine.

```
load("res.1.RData")
head(me$all$eqtls)
```

To check that everything ran successfully, please run:

```
./check_upload.sh 14 check
```

This should tell you that `Section 14 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 14 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 14.