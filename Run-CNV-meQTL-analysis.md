We have now performed the standard meQTL analysis and the variance-meQTL analysis. We will now perform a CNV-meQTL analysis, whereby we are testing for associations between genetic copy number variants and methylation levels. 

**WARNING** Several cohorts have analysed this script but ended with huge results files. We need to look at this in more depth and recommend to wait with running this script.

Again, the procedure is exactly the same except it should be somewhat faster as there are only ~480000 CNV positions to evaluate. The CNV data have been split into `genetic_chunks` chunks which can each be run independently on a different node on the cluster. An example job submission script (e.g. `submit_cnvmqtl.sh`) would be:

```bash

#!/bin/bash

#PBS -N cnvmqtl
#PBS -o cnvmqtl-output
#PBS -e cnvmqtl-error
#PBS -l walltime=12:00:00
#PBS -t 1-500
#PBS -l nodes=1:ppn=16
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/
./07-mcnv.sh ${PBS_ARRAYID}

```

then to submit to the cluster:

    qsub submit_cnvmqtl.sh

And this will distribute the entire surface across 500 nodes, each node parallelising across 16 cores.


### Upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 07 check
```

This should tell you that `Section 07 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 07 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 07.

This procedure will be repeated at the end of each section.