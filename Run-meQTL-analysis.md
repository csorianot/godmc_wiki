At this point we have normalised and adjusted the methylation data and converted it to the correct format. We have also QC'd the genetic data and saved it to the correct format. We can now perform the meQTL analysis.

This process is computationally expensive, it is performing an association of every SNP against every probe. So the data has been setup to parallelise. The SNP data has been split into `meth_chunks` chunks. This variable is specified in the `config` file:

    meth_chunks="100"

is the default setting. So for example if there are 8000000 SNPs in total then when we run

    ./05-mqtl.sh 1

The script will perform the meQTL analysis using SNPs 1-80000 (the first chunk of 100), using `nthreads` threads in parallel (also specified in the `config` file). 

Going through 1-100 sequentially is likely to take some time so it is recommended that you parallelise across a cluster. To do this, you need to create a job submission script that will work on your cluster. Each cluster is different, but to show an example of how it works on our cluster (which uses a PBS scheduler), we would create a script (e.g. `submit_mqtl.sh`) like this:


```bash

#!/bin/bash

#PBS -N mqtl
#PBS -o job_reports/mqtl-output
#PBS -e job_reports/mqtl-error
#PBS -l walltime=12:00:00
#PBS -t 1-100
#PBS -l nodes=1:ppn=16
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/
./05-mqtl.sh ${PBS_ARRAYID}

```

Then, when this is submitted:

    qsub submit_mqtl.sh

it will create a batch of 100 jobs, each running with the variable `$PBS_ARRAYID` set to a value between 1-100, and each individual job further parallelised across 16 threads. 