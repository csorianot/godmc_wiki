##You need to have at least 
```
R.3.2.0
```
###These are the Rpackages you need:
```
install.packages("lattice")
install.packages("ggplot2") 
install.packages("data.table")
install.packages("MatrixEQTL")
install.packages("parallel")
install.packages("GenABEL")
install.packages("matrixStats")
install.packages("plyr")
install.packages("Cairo")
install.packages("plotrix")
install.packages("EasyQC_9.0.tar.gz")

source("http://bioconductor.org/biocLite.R")
biocLite("SNPRelate")
biocLite("GENESIS")
```

###If you haven't installed meffil yet, here are some instructions:
```
source("http://bioconductor.org/biocLite.R")
biocLite("illuminaio")
biocLite("limma")
biocLite("IlluminaHumanMethylation450kmanifest")
biocLite("IlluminaHumanMethylation450kanno.ilmn12.hg19")
biocLite("CopyNumber450kData")
biocLite("DNAcopy")
install.packages("markdown")
install.packages("knitr")
install.packages("devtools")
library(devtools)
install_github("perishky/meffil")
```