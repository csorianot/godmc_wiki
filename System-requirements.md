####You need to have at least 
```
R.3.2.0
```

In the repo dir you need to run
```
wget http://homepages.uni-regensburg.de/~wit59712/easyqc/EasyQC_9.2.tar.gz
```

####These are the Rpackages you need:
```
install.packages("lattice")
install.packages("ggplot2") 
install.packages("data.table")
install.packages("MatrixEQTL")
install.packages("parallel")
install.packages("matrixStats")
install.packages("plyr")
install.packages("Cairo")
install.packages("plotrix")
install.packages("EasyQC_9.2.tar.gz")

#You only need to install this package if you have relatedness in your samples
install.packages("GenABEL") 


source("http://bioconductor.org/biocLite.R")
biocLite("impute")

#You only need to install this package if you have relatedness in your samples
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