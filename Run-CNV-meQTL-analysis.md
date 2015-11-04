We have now performed the standard meQTL analysis and the variance-meQTL analysis. We will now perform a CNV-meQTL analysis, whereby we are testing for associations between genetic copy number variants and methylation levels. 

Again, the procedure is exactly the same except it should be somewhat faster as there are only ~480000 CNV positions to evaluate. The CNV data have been split into `genetic_chunks` chunks which can each be run independently on a different node on the cluster. An example job submission script (e.g. `submit_cnvmqtl.sh`) would be:

```bash

#!/bin/bash

#PBS -N cnvmqtl
#PBS -o cnvmqtl-output
#PBS -e cnvmqtl-error
#PBS -l walltime=12:00:00
#PBS -t 1-1000
#PBS -l nodes=1:ppn=16
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/
./05c-cnvmqtl.sh ${PBS_ARRAYID}

```

then to submit to the cluster:

    qsub submit_cnvmqtl.sh

And this will distribute the entire surface across 1000 nodes, each node parallelising across 16 cores.