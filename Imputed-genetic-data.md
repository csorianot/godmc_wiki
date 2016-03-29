### Genetic data

The genetic data must be
- Imputed to 1000 genomes reference panel, ideally phase 3. Phased haplotypes available [here](https://mathgen.stats.ox.ac.uk/impute/1000GP_Phase3.html). For guidelines on how to perform imputation see [here](http://genome.sph.umich.edu/wiki/IMPUTE2:_1000_Genomes_Imputation_Cookbook) and [here](https://github.com/explodecomputer/godmc/wiki/Genetic-imputation). A pipeline is also available [here](https://github.com/explodecomputer/imputePipePBS). The Haplotype Reference Consortium offers a free imputation service (including 1000G Phase 3) which you can use [here] (http://www.haplotype-reference-consortium.org/data-access). Please contact us if you have any queries about imputation.
- Filtered to have MAF > 0.01 and imputation quality score > 0.8
- **IMPORTANT:** Converted to best guess binary plink format without a probability threshold 
- All remaining SNPs combined into a single fileset (*i.e.* not a separate fileset for each chromosome)
- No spaces in the file names
- All autosomes and chrX

#### Fam file and Sample IDs

**IMPORTANT:** Please ensure that the second column of the `.fam` file (individual IDs) contains unique sample IDs. These sample IDs should be the IDs that are used in the phenotype, covariate, methylation and structural variants data. The first column of the `.fam` file (family IDs) is not important for these analysis. e.g. They can just be the same as the second column. Please also ensure that the individual IDs don't contain any underscores or a header.

    1A 1A  0  0  2 -9
    2A 2A  0  0  2 -9
    3A 3A  0  0  2 -9

#### Bim file and allele coding

**IMPORTANT:** ensure that your alleles coded using the original impute2 or minimac coding. To perform a meta-analysis across cohorts, alleles should be matching across cohorts. Please use full sequence allele coding for INDELs if you have used impute2.  If you have used minimac, INDELs need to be coded as I, R, D. The pipeline harmonizes the allele coding to an universal coding. The pipeline also changes the SNV ids for you (second column) so you don't need to worry about marker names. Please don't use a header.

###### impute2
    1 chr1:1068669:INDEL  0 1068669                GT    G
    1 chr1:1068832:INDEL  0 1068832 CGCCGCCTGCCTGCCCG    C
    1 chr1:1069474:INDEL  0 1069474         AAAAAAAAG    A
    1 chr1:1069475:INDEL  0 1069475          AAAAAAAG    A
    1 chr1:1081403:INDEL  0 1081403                GC    G
    1 chr1:1084475:INDEL  0 1084475                 A   AG

##### minimac
              
    20 chr20:69481 0 69481 R D
    20 chr20:70480 0 70480 R D
    20 chr20:70484 0 70484 R D
    20 chr20:72104 0 72104 R D
    20 chr20:74543 0 74543 R I
    20 chr20:86164 0 86164 R D

#### Bed file
**IMPORTANT:** We use best guess genotypes without a probability threshold in our pipeline. 
Please make sure you don't filter on a probability threshold as matrixQTL (software used to calculate methQTLs) can't handle missingness properly and will set missing genotypes to the genotypic mean. We use best guess genotypes as matrixQTL can't handle dosages. Please see below how you can prepare best guess files.

#### Imputation quality
We also require imputation quality scores for each SNP. Some instructions on how to get imputed data into the desired format including your imputation quality file are below.

    SNP MAF Info
    rs1 0.02 0.88

#### Convert imputed data to bestguess data
  - [Convert impute2 imputed data to bestguess data (.gen/bgen files)](https://github.com/MRCIEU/godmc/wiki/Convert-impute2-imputed-data-to-bestguess-data)
  - [Convert mach/minimac imputed data to bestguess data (.info/.dose or .mlinfo/.mldose files)](https://github.com/MRCIEU/godmc/wiki/Convert-mach-minimac-imputed-data-to-bestguess-data-(.info-.dose-or-.mlinfo-.mldose-files))
  - [Convert minimac3 imputed data to bestguess data (vcf files)](https://github.com/MRCIEU/godmc/wiki/Convert-minimac3-imputed-data-to-bestguess-data-(vcf-files))