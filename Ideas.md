1. Can we predict ethnicity using methylation? How do we regress out genetic variance?
2. Is the correlation structure between CpGs conserved across datasets? Can we use WGCNA to look at preservation of the correlation structure across different datasets.
3. Why are CpGs correlated? Can we look at multiple CPGs in one regions?
4. DataSHIELD opportunities to share individual level data in a secure way. 
For example: 
   A. GCTA SNP heritability analysis where all individuals are required to estimate the pairwise relationships across all samples
   B. The 'whole methylome’ estimation, the extension of GCTA to estimate the contribution of all methylation levels to a complex trait
5. We can test to see if SNPs causing methylation are enriched for genomic annotations. And we can test if the genetically influenced methylation positions themselves are enriched for genomic annotations. But what if we combined this to look in two dimensions? E.g. are methylation levels with annotation A more likely to be influenced by a SNP in annotation B? Gib has some ideas about how to generate an appropriate null to test for significance of 2D enrichments.
6. Are the methQTL CpGs enriched for genomic annotations? 
7. Can we find master regulators eg. SNPs that associate with multiple CpGs?
8. Are the SNP-methylation pairs that make an mQTL in regions of chromosome interactions? E.g. we take the list of interacting chromosome regions from hi-c or promoter capture hi-c and see if our SNP-methylation pairs can be explained by virtue of being in loci that physically interact in the cell. Chromosome interactions are cell type  specific duo would hope to see  differential enrichment for different tissue types. Might be particularly interesting for trans-mQTLs.
9. How can we integrate methQTLs with complex traits (GoSHIFTER, GARFIELD etc.). 
Bayesian fine mapping approaches such as Maller et al..
10. Integration with gene expression: i) Lookup of CpG genes in GTEx. ii) Two sample MR approach a. get all cismQTLs in GoDMC b. Find all of those SNP effects in trans eQTLanalyses. Use GTEx and blood eQTL browser.
11. Can we run a EWAS on polygenic risk scores?
12. Can we find chromosome X and Y trans effects?
13. Are methQTL SNPs annotated to enhancers or superenhancers?