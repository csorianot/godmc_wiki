#### Convert `minimac3` imputed data to bestguess data (vcf files)

Please use this script [https://gist.github.com/epzjlm/2d7c1aded2ee24443d69] to extract imputation quality scores (r2) and MAF from the vcf files. You need to have gzipped vcf files as input. You run the script like this:
```
for i in {1..23}
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
   for i in {1..23}
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
    cp data_chr${i}.info data_chr${i}.info.orig
awk '{
        if (++dup[$1] > 1) {
            print $1".duplicate."dup[$1], $2, $3
        } else {
            print $0 }
}' data_chr${i}.info.orig > data_chr${i}.info

    fgrep -v -w -f duplicates.chr${i}.txt <data_chr${i}.info >chr${i}_filtered.info
done

# Merge them into one dataset

for i in {2..23}
do 
    echo "data_chr${i}_filtered"
done > mergefile.txt

plink1.90 --bfile data_chr1_filtered --merge-list mergefile.txt --make-bed --out data_filtered

# Combine info files into a single file

head -n1 chr1_filtered.info > data_filtered.info

for i in {1..23}
do
    awk ' NR>1 {print $0}' < chr${i}_filtered.info |cat >> data_filtered.info
done


The result should be three plink files: `data_filtered.bed`, `data_filtered.bim`, `data_filtered.fam`, and the info file: `data_filtered.info`. Copy them to `godmc/input_data` and set the following variable in your `config` file:

bfile_raw="${home_directory}/input_data/data_filtered"
and

quality_scores="${home_directory}/input_data/data_filtered.info"
quality_type="minimac3"
```