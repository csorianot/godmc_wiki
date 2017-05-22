### Option 1 (preferred)

Ideally, we would prefer it if you normalise and QC your data using the [meffil](https://github.com/perishky/meffil) package. **Please make sure you have installed version 1.0.0 or higher.** You can check the version like this:

```
packageVersion("meffil")
``` 

Meffil has been optimised for speed and memory, and instructions on how to do this can be found here:

1. [Perform sample QC](https://github.com/MRCIEU/godmc/wiki/Methylation-sample-QC)
2. [Normalise QC'd samples](https://github.com/MRCIEU/godmc/wiki/Methylation-sample-QC)


### Option 2

If you prefer to use your own normalisation then please ensure that the data you provide is as follows:
- Beta values should be used to measure methylation levels.
- We prefer that you would use functional normalisation.
- The methylation data is an R matrix object where CpGs should be in rows and sample IDs should be columns. 
- The rownames must be the `cg` identifiers and the column names the IDs that correspond to the samples, and that correspond to sample IDs in the other datasets. 
- Please note that you should avoid underscores in your sample identifiers. 
- The methylation matrix should be saved as a `.RData` file and the methylation matrix object should be called `norm.beta`. e.g. in R:

        save(norm.beta, file="/path/to/godmc/input_data/methylation.RData")

- You should avoid spaces in your file name.