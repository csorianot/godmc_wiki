### Genetic data

The genetic data must be
- Imputed to 1000 genomes reference panel, ideally phase 3. Phased haplotypes available [here](https://mathgen.stats.ox.ac.uk/impute/1000GP_Phase3.html). For guidelines on how to perform imputation see [here](http://genome.sph.umich.edu/wiki/IMPUTE2:_1000_Genomes_Imputation_Cookbook) and [here](https://github.com/explodecomputer/godmc/wiki/Genetic-imputation). A pipeline is also available [here](https://github.com/explodecomputer/imputePipePBS). Please contact us if you have any queries about this.
- Filtered to have MAF > 0.01 and imputation quality score > 0.8
- Converted to best guess binary plink format without a probability threshold 
- All remaining SNPs combined into a single fileset (*i.e.* not a separate fileset for each chromosome)

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
We also require imputation quality scores for each SNP. Some instructions on how to get imputed data into the desired format are below.

    SNP MAF Info
    rs1 0.02 0.88

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

#### Convert `MACH/minimac` imputed data to bestguess data (.mldose and .mlinfo files or .dose and info files)

GCTA can convert both mldose and mlinfo as .dose and .info files to plink best guess format if the files are gzipped. However, chr and pos are set to 0. In the example below, chromosomal position are extracted from the SNP ids. Please contact us if you have ids other than {CHR}:{POS}:{A1}:{A2} or {CHR}:{POS}.

```
#!/bin/bash
for i in {10..10}
do 
#gcta --dosage-mach-gz chr$i.mldose.gz chr$i.mlinfo.gz --maf 0.01 --imput-rsq 0.8 --make-bed --out data_chr${i}_filtered
gcta --dosage-mach-gz chr$i.dose.gz chr$i.info.gz --maf 0.01 --imput-rsq 0.8 --make-bed --out data_chr${i}_filtered

awk -F':' '{print $2}' <data_chr${i}_filtered.bim |awk '{print $1}'>pos$i.txt
paste pos$i.txt data_chr${i}_filtered.bim |awk '{print '$i',$3,$4,$1,$6,$7}' >data_chr${i}_filtered.bim2
mv data_chr${i}_filtered.bim2 data_chr${i}_filtered.bim

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
    
    #filter info/maf file
    awk '{print $2}' < data_chr{i}_filtered.bim >data_chr{i}_filtered.id 
   zcat chr$i.info.gz | awk '$5>0.01 && $7>0.8 || NR>1 {print $1,$5,$7}' |perl -pe 's/Rsq/Info/g' >      chr{i}_filtered.info
   fgrep -v -w -f duplicates.chr${i}.txt <chr{i}_filtered.info >chr{i}_filtered.info2
   mv chr{i}_filtered.info2 chr{i}_filtered.info
done

# Merge them into one dataset

for i in {2..22}
do 
    echo "data_chr${i}_filtered"
done > mergefile.txt

plink1.90 --bfile data_chr1_filtered --merge-list mergefile.txt --make-bed --out data_filtered

# Combine info files into a single file

```
 head -n1 chr1_filtered.info > data_filtered.info

 for i in {1..22}
 do   
   awk 'NR>1 {print $0}' < chr${i}_filtered.info |cat >> data_filtered.info
 done
```

The result should be three plink files: data_filtered.bed, data_filtered.bim, data_filtered.fam, and the info file: data_filtered.info. Copy them to godmc/input_data and set the following variable in your config file:

bfile_raw="${home_directory}/input_data/data_filtered"
and

quality_scores="${home_directory}/input_data/data_filtered.info"
quality_type="minimac2"
```

#### Convert `minimac3` imputed data to bestguess data (vcf files)

Please use this script [https://gist.github.com/epzjlm/2d7c1aded2ee24443d69] to extract imputation quality scores (r2) and MAF from the vcf files. You need to have gzipped vcf files as input. You run the script like this:
```
for i in {1..22}
do
perl get_vcf_chr_pos_info.pl chr$i.vcf.gz MAF,R2 >mafinfo.minimac3.chr$i.txt

    # Assumes column 2 is the position
    # Assumes columns 4 and 5 are the allele names
    # Assumes column 8 is the MAF
    # Assumes columns 9 is the info score

awk -v chr=$i '{
        if (($4 == "A" || $4 == "T" || $4 == "C" || $4=="G") &&  ($5 == "A" || $5 == "T" || $5 == "C" || $5 == "G")) 
            print "chr"chr":"$2":SNP", $8, $9;
        else 
            print "chr"chr":"$2":INDEL", $8, $9;
    }' mafinfo.minimac3.chr$i.txt |perl -pe 's/R2/Info/g'|perl -pe 's/chr[0-9][0-9]\:POS\:INDEL/SNP/g'|perl -pe 's/chr[0-9]\:POS\:INDEL/SNP/g' |awk '$2>0.01 && $3>0.8 {print $0}' >data_chr${i}.info

awk 'NR>1 {print $1}' <data_chr${i}.info >data_chr${i}.keep
done


    # Then run the bash script below to convert your data to best guess and to filter out SNPs with MAF<0.01 and info<0.08

   #!/bin/bash
   for i in {1..22}
   do
   vcftools --gzvcf chr$i.vcf.gz --plink --chr $i --out chr$i
   plink1.90 --ped chr$i.ped --map chr$i.map --make-bed --out data_chr${i}_filtered 
   
 # Rename the SNP IDs if necessary to avoid possible duplicates
    
    cp data_chr${i}_filtered.bim data_chr${i}_filtered.bim.orig
    awk '{
        if (($5 == "A" || $5 == "T" || $5 == "C" || $5=="G") &&  ($6 == "A" || $6 == "T" || $6 == "C" || $6=="G")) 
            print $1, "chr"$1":"$4":SNP", $3, $4, $5, $6;
    else 
        print $1, "chr"$1":"$4":INDEL", $3, $4, $5, $6;
   }' data_chr${i}_filtered.bim.orig > data_chr${i}_filtered.bim
  

   # Keep SNPs with MAF>0.01 or info>0.8
   plink1.90 --bfile data_chr${i}_filtered --make-bed --out data_chr${i}_filtered --extract data_chr${i}.keep
   
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


    # Remove duplicates from maf/info file
    fgrep -v -w -f duplicates.chr${i}.txt <data_chr${i}.info >chr${i}_filtered.info
done

# Merge them into one dataset

for i in {2..22}
do 
    echo "data_chr${i}_filtered"
done > mergefile.txt

plink1.90 --bfile data_chr1_filtered --merge-list mergefile.txt --make-bed --out data_filtered

# Combine info files into a single file

head -n1 chr1_filtered.info > data_filtered.info

for i in {1..22}
do
    awk ' NR>1 {print $0}' < chr${i}_filtered.info |cat >> data_filtered.info
done

```


The result should be three plink files: `data_filtered.bed`, `data_filtered.bim`, `data_filtered.fam`, and the info file: `data_filtered.info`. Copy them to `godmc/input_data` and set the following variable in your `config` file:

bfile_raw="${home_directory}/input_data/data_filtered"
and

quality_scores="${home_directory}/input_data/data_filtered.info"
quality_type="minimac3"
```