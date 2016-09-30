At this point we have a candidate list of SNP-CpG pairs from phase 1. To define a cis association we used a 1 MB distance between CpG and SNP. We have selected SNP-CpG pairs in _cis_ that are found at least once in one of the phase 1 cohorts with a value < 1e-5. For _trans_, we restricted the candidate list to SNP-CpG pairs that were found in at least 2 cohorts at pvalues <1e-5. In phase 2, we are going to perform meQTL associations using linear mixed models using GCTA.

We have splitted the CpGs in 74 probesets. 

    ./16-gcta.sh 1

The script will perform the meQTL analysis on the first set of probes, using `nthreads` threads in parallel (also specified in the `config` file). 

Going to 74 probesets will take some time so it is recommended that you parallelise across a cluster or your server.

If you don't have access to a cluster you can parallelise your jobs using the package GNU parallel.
```
seq 74 | parallel -j 10 --workdir $PWD ./16-gcta.sh {}
```

If you have access to a cluster, you need to create a job submission script that will work on your cluster. See below an example of how it works on our cluster (which uses a PBS scheduler), we would create a script (e.g. `submit_gcta.sh`) like this:
```
#!/bin/bash

#PBS -N gcta
#PBS -o gcta.o
#PBS -e gcta.e
#PBS -l walltime=100:00:00
#PBS -t 1-74
#PBS -l nodes=1:ppn=2
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc #EDIT this line
./16-gcta.sh ${PBS_ARRAYID}

```

Then, when this is submitted:

```
qsub submit_gcta.sh
```

It will create a batch of 74 jobs, each running with the variable `$PBS_ARRAYID` set to a value between 1-74, and each individual job further parallelised across 2 threads.
