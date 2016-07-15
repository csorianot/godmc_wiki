At this point we have normalised and adjusted the methylation data and converted it to the correct format. We have also QC'd the genetic data and saved it to the correct format. We can now perform the meQTL analysis.

This process is computationally expensive, it is performing an association of every SNP against every probe. So the data has been setup to parallelise. The SNP data has been split into `genetic_chunks` chunks using `script 2b`. The genetic chunks can be found here: `./processed_data/genetic_data/tabfile/data.tab.{1-500}` This variable is specified in the `config` file:

    genetic_chunks="500"

is the default setting. So for example if there are 8000000 SNPs in total then when we run

    ./05-mqtl.sh 1

The script will perform the meQTL analysis using SNPs 1-16000 (the first chunk of 500), using `nthreads` threads in parallel (also specified in the `config` file). 

Going through 1-500 sequentially is likely to take some time so it is recommended that you parallelise across a cluster or your server. 

If you don't have access to a cluster you can parallelise your jobs using the package GNU parallel.
```
seq 100 | parallel -j 10 --workdir $PWD ./05-mqtl.sh {}
```
If you have 
If you have access to a cluster, you need to create a job submission script that will work on your cluster. For ~1800 samples, one chunk took 25 minutes to run using 16 cores. Each cluster is different, but to show an example of how it works on our cluster (which uses a PBS scheduler), we would create a script (e.g. `submit_mqtl.sh`) like this:


```bash

#!/bin/bash

#PBS -N mqtl
#PBS -o mqtl-output
#PBS -e mqtl-error
#PBS -l walltime=12:00:00
#PBS -t 1-500
#PBS -l nodes=1:ppn=16
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/    #EDIT THIS LINE
./05-mqtl.sh ${PBS_ARRAYID}

```

Then, when this is submitted:

    qsub submit_mqtl.sh

it will create a batch of 500 jobs, each running with the variable `$PBS_ARRAYID` set to a value between 1-500, and each individual job further parallelised across 16 threads. 


### Check and upload the results

The results are in `results/05`.

Please open one of the *RData files and check if the output looks fine.

```
load("res.1.RData")
head(me$all$eqtls)
```

To check that everything ran successfully, please run:

```
./check_upload.sh 05 check
```

This should tell you that `Section 05 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 05 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 05.

This procedure will be repeated at the end of each section.