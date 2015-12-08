### Genetic data

The genetic data must be
- Imputed to 1000 genomes reference panel, ideally phase 3. Phased haplotypes available [here](https://mathgen.stats.ox.ac.uk/impute/1000GP_Phase3.html). For guidelines on how to perform imputation see [here](http://genome.sph.umich.edu/wiki/IMPUTE2:_1000_Genomes_Imputation_Cookbook) and [here](https://github.com/explodecomputer/godmc/wiki/Genetic-imputation). A pipeline is also available [here](https://github.com/explodecomputer/imputePipePBS). Please contact us if you have any queries about this.
- Filtered to have MAF > 0.01 and imputation quality score > 0.8
- Converted to best guess binary plink format without a probability threshold 
- All remaining SNPs combined into a single fileset (*i.e.* not a separate fileset for each chromosome)

#### Fam file and Sample IDs

**IMPORTANT:** Please ensure that the second column of the `.fam` file (individual IDs) contains unique sample IDs. These sample IDs should be the IDs that are used in the phenotype, covariate, methylation and structural variants data. The first column of the `.fam` file (family IDs) is not important for these analysis. e.g. They can just be the same as the second column. Please also ensure that the individual IDs don't contain any underscores.

1A 1A  0  0  2 -9
2A 2A  0  0  2 -9
3A 3A  0  0  2 -9

#### Bim file

Please ensure that your alleles coded in column 5 and 6 are coded as A,C,T,G. To perform a meta-analysis across cohorts, alleles should be matching across cohorts. Please use full allele coding for INDELS and no I,R, D recoding. The pipeline changes the IDs of you (second column) so you don't need to worry about this.

1 chr1:1068669:INDEL  0 1068669                GT    G
1 chr1:1068832:INDEL  0 1068832 CGCCGCCTGCCTGCCCG    C
1 chr1:1069474:INDEL  0 1069474         AAAAAAAAG    A
1 chr1:1069475:INDEL  0 1069475          AAAAAAAG    A
1 chr1:1081403:INDEL  0 1081403                GC    G
1 chr1:1084475:INDEL  0 1084475                 A   AG

#### Imputation quality
We also require imputation quality scores for each SNP. Some instructions on how to get imputed data into the desired format are below.

    SNP MAF Info
    rs1 0.02 0.88

#### `Impute2` imputed data 

Assuming you have used [impute2](https://mathgen.stats.ox.ac.uk/impute/impute_v2.html) to perform the imputation and generated `bgen` dosage files, you can use [qctool](http://www.well.ox.ac.uk/~gav/qctool/#overview) to filter on MAF and info and prepare a snp quality file for the pipeline following the bash script below.

It is recommended that you use `plink1.90` to do this, available for download [here](https://www.cog-genomics.org/plink2)

```bash
# First copy your sample file to a new file
cp data.sample filtered.sample
# If you need to filter out samples you need to generate a new sample file.
 
In R:
 d<-read.table("data.sample",header=T,stringsAsFactors=F)
 e<-read.table("exclusion.samples.txt",stringsAsFactors=F)
 d2<-d[(which(d[,1]%in%e[,1]==F)),]
 write.table(d2,"filtered.sample",sep=" ",col.names=T,row.names=F,quote=F)

#Then run the bash script below.

#!/bin/bash
for i in {1..22}
do

    # First filter out snps (info<0.8) and maf <0.01 and samples using qctool.
    
    qctool -g data_chr${i}.bgen -s data.sample -og filteredchr${i}.bgen -maf 0.01 1 -info 0.8 1 -excl-samples   exclusion.samples.txt
    # Now calculate summary stats on the filtered data. Please note you need to exclude the samples from the .sample file manually. 
       
    qctool -g filteredchr${i}.bgen -s filtered.sample -snp-stats data_chr${i}.snp-stats
     
    # Now extract the best guess data from the bgen files, variants with a "." will be recoded to chr:pos_allele1_allele2.

    plink1.9 --bgen filteredchr${i}.bgen snpid-chr --sample filtered.sample --set-missing-var-ids @:#\$1,\$2 --make-bed --out data_chr${i}_filtered --hard-call-threshold 0.499999 --fill-missing-a2
    
    # Rename the SNP IDs if necessary to avoid possible duplicates
    
    cp data_chr${i}_filtered.bim data_chr${i}_filtered.bim.orig
    awk '{
        if (length($5) == "1" && length($6) == "1") 
            print $1, "chr"$1":"$4":SNP", $3, $4, $5, $6;
        else 
            print $1, "chr"$1":"$4":INDEL", $3, $4, $5, $6;
    }' data_chr${i}_filtered.bim.orig > data_chr${i}_filtered.bim

    # For simplicity remove any duplicates

    cp data_chr${i}_filtered.bim data_chr${i}_filtered.bim.orig2
    awk '{
        if (++dup[$2] > 1) { 
            print $1, $2".duplicate."dup[$2], $3, $4, $5, $6 
        } else { 
            print $0 }
    }' data_chr${i}_filtered.bim.orig2 > data_chr${i}_filtered.bim
    grep "duplicate" data_chr${i}_filtered.bim | awk '{ print $2 }' > duplicates.chr${i}.txt
    
    plink1.90 --bfile data_chr${i}_filtered --exclude duplicates.chr${i}.txt --make-bed --out data_chr${i}_filtered


# Relabel the SNP IDs and extract relevant columns in the snp-stats file
    # Assumes column 4 is the position
    # Assumes columns 7 and 8 are the allele names
    # Assumes column 15 is the MAF
    # Assumes columns 19 is the info score

awk -v chr=$i '{
        if (length($7) == "1" && length($8) == "1") 
            print "chr"chr":"$4":SNP", $15, $19;
        else 
            print "chr"chr":"$4":INDEL", $15, $19;
    }' data_chr${i}.snp-stats > data_chr${i}.info

    # Remove duplicates from snp-stats
    fgrep -v -w -f duplicates.chr${i}.txt <data_chr${i}.info chr${i}_filtered.info
done

# Merge them into one dataset

for i in {2..22}
do 
    echo "data_chr${i}_filtered"
done > mergefile.txt

plink1.90 --bfile data_chr1_filtered --merge-list mergefile.txt --make-bed --out data_filtered

# Combine info files into a single file

head -n1 chr01_filtered.info > data_filtered.info

for i in {1..22}
do
    awk ' NR>1 {print $0}' < chr${i}_filtered.info |cat >> data_filtered.info
done

```


The result should be three plink files: `data_filtered.bed`, `data_filtered.bim`, `data_filtered.fam`, and the info file: `data_filtered.info`. Copy them to `godmc/input_data` and set the following variable in your `config` file:

    bfile_raw="${home_directory}/input_data/data_filtered"

and

    quality_scores="${home_directory}/input_data/data_filtered.info"
    quality_type="impute2"