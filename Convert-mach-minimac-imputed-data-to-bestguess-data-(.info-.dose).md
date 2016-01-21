[GCTA](http://cnsgenomics.com/software/gcta/data_management.html) can convert both .mldose and .mlinfo as .dose and .info files to plink best guess format if the files are gzipped. However, chr and pos are set to 0. In the example below, chromosomal position are extracted from the SNP ids. Please contact us if you have ids other than {CHR}:{POS}:{A1}:{A2} or {CHR}:{POS}.

```
#!/bin/bash
for i in {1..22}
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
    zcat chr$i.info.gz | awk '$5>0.01 && $7>0.8 || NR>1 {print $1,$5,$7}' |perl -pe 's/Rsq/Info/g' >         chr{i}_filtered.info
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

 head -n1 chr1_filtered.info > data_filtered.info

 for i in {1..22}
 do   
   awk 'NR>1 {print $0}' < chr${i}_filtered.info |cat >> data_filtered.info
 done

# The result should be three plink files: data_filtered.bed, data_filtered.bim, data_filtered.fam, and the info file: data_filtered.info. Copy them to godmc/input_data and set the following variable in your config file:

bfile_raw="${home_directory}/input_data/data_filtered"
and

quality_scores="${home_directory}/input_data/data_filtered.info"
quality_type="minimac2"
```