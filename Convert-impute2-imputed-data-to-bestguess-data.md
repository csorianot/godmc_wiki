#### Convert `Impute2` imputed data to bestguess data

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

    plink1.90 --bgen filteredchr${i}.bgen snpid-chr --sample filtered.sample --set-missing-var-ids @:#\$1,\$2 --make-bed --out data_chr${i}_filtered --hard-call-threshold 0.499999
    
    # Rename the SNP IDs if necessary to avoid possible duplicates
    
    cp data_chr${i}_filtered.bim data_chr${i}_filtered.bim.orig
    awk '{
        if (($5 == "A" || $5 == "T" || $5 == "C" || $5=="G") &&  ($6 == "A" || $6 == "T" || $6 == "C" || $6=="G")) 
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
    # Assumes columns 5 and 6 are the allele names
    # Assumes column 15 is the MAF
    # Assumes columns 19 is the info score

awk -v chr=$i '{
        if (($5 == "A" || $5 == "T" || $5 == "C" || $5=="G") &&  ($6 == "A" || $6 == "T" || $6 == "C" || $6=="G")) 
            print "chr"chr":"$4":SNP", $15, $19;
        else 
            print "chr"chr":"$4":INDEL", $15, $19;
    }' data_chr${i}.snp-stats > data_chr${i}.info

    # Remove duplicates from snp-stats
    fgrep -v -w -f duplicates.chr${i}.txt <data_chr${i}.info >chr${i}_filtered.info
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
