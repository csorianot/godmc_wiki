We have already performed the standard meQTL analysis. We are now going to perform the same kind of analysis again except this time we will look at ChrX methylation probes in females only.

The procedure is exactly the same, the SNPs have been split into `genetic_chunks` chunks which can each be run independently on a different node on the cluster. An example job submission script (e.g. `submit_mqtlX.sh`) would be:


```
./14-mqtl_females.sh
```