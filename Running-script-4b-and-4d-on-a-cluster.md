### Optional: Running on the cluster
#### Script 4b
We can run the `04b-methylation_adjustment1.sh` and `04d-methylation_adjustment2.sh` in parallel on a cluster. In the `config` file you can specify how many chunks to break the entire analysis into, e.g. 

    meth_chunks="100"

will split the ~480k probes into 100 chunks of ~4.8k each. If you run

    ./04b-methylation_adjustment1.sh

Then the script runs the entire 480k probes using `nthreads` threads. However if you specify a number between 1 and 100 as an argument to this script, e.g.:

    ./04b-methylation_adjustment1.sh 1

then the script will run only the 1st chunk of the entire dataset, i.e. approximately, probes 1-4800.

To utilise this on a cluster requires two steps:

1. Creating a submission script that works for your cluster
2. Aggregating the individual results from each chunk into a single file at the end.


An example of how a submission script, e.g. `submit_04b.sh` for our cluster would be written is as follows:


```bash

#!/bin/bash

#PBS -N meth_04b
#PBS -o meth_04b-output
#PBS -e meth_04b-error
#PBS -l walltime=12:00:00
#PBS -t 1-100
#PBS -l nodes=1:ppn=16
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/
./04b-methylation_adjustment1.sh ${PBS_ARRAYID}

```

then to run:

    qsub submit_04b.sh

On our cluster this creates a batch of 100 jobs, each with the `${PBS_ARRAYID}` variable set to a value between 1 and 100. 

You can check whether your jobs are completed by grepping `Successfully` from the log files (`results/04/logs_b`):
```
grep Successfully log* |wc
```
After these jobs have completed, assuming there have been no errors, there will be 100 separate `.RData` files that need to be combined together. To do this simply run:

    resources/methylation/aggregate_adjustment1.sh

This will combine your 100 chunks into a single file for use moving forward in the analysis.

#### Script 4d
Similarly, for the `04d-methylation_adjustment2.sh` script you would create a submission script (e.g. `submit_04d.sh`):

```bash

#!/bin/bash

#PBS -N meth_04d
#PBS -o meth_04d-output
#PBS -e meth_04d-error
#PBS -l walltime=12:00:00
#PBS -t 1-100
#PBS -l nodes=1:ppn=16
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/
./04d-methylation_adjustment2.sh ${PBS_ARRAYID}

```

Then:

    qsub submit_04d.sh

You can check if your jobs ran successfully by using this line in `results/04/logs_d`:
```
grep Successfully log.txt_[0-9]* |wc

```

and once the jobs are completed:

    resources/methylation/aggregate_adjustment2.sh