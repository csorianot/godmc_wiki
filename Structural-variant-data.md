To generate copy number variants from Ilumina 450k methylation arrays you will need the original `.idat` files for your samples because copy number variation is estimated from array intensities. The functions to generate these data are provided in the `R/meffil` package, and it should be a relatively quick process (can run in parallel) with a relatively small memory footprint.


To install in R:
```
source("http://bioconductor.org/biocLite.R")
biocLite("illuminaio")
biocLite("limma")
biocLite("IlluminaHumanMethylation450kmanifest")
biocLite("IlluminaHumanMethylation450kanno.ilmn12.hg19")
biocLite("CopyNumber450kData")
install.packages("markdown")
install.packages("knitr")
install.packages("devtools")
library(devtools)
install_github("perishky/meffil")
```
Load meffil and set how many cores to use for parallelization
```
    library(meffil)
    options(mc.cores=16)
```
Generate a samplesheet with your samples. The samplesheet can be generated automatically from the idat basenames by giving the directory with idat files or it can be done manually. It should contain at least the following necessary columns: `Sample Name`, `Sex` (possible values `M`, `F` or `NA`) and `Basename`. It tries to parse the basenames to guess if the Sentrix plate and positions are present. 

    samplesheet <- meffil.create.samplesheet(path_to_idat_files)

At this point please ensure that the `Sample_Name` column contains the actual sample IDs that are being used for the other data types. Please also add the sex values to the `Sex` column. Don't change these column names though.

Now estimate the CNVs:
```
    cnv <- meffil.calculate.cnv(samplesheet)    
```
A matrix of genetic copy number variation at each probe can now be generated:
```
    cnv <- meffil.cnv.matrix(cnv)
```
Please save this object to the `godmc/input_data` folder:
```
    save(cnv, file="/path/to/godmc/input_data/cnv.RData")
```
and make sure that the object name that you are saving is `cnv`, as this is the name that the pipeline will be expecting.