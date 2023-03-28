# Problem_Set_11

In this week's problem set we will be conducting a PCA on human SNP data.

There are several pre-processing steps that I conducted that I will outline below. 

### Pre-processing 1 - Get a specific region of Chromosome 19

Due to time and processor constraints we will be anlayzing only a small portion of Human Chromosome 19 

I started with a VCF file of all positions in Chr 19 in the 1000 Genomes data VCF. This involves using **VCFtools** which is already loaded onto the UNC-Charlotte HPC and can be accessed using ```ml vcftools```

The command to conduct this filtering is

```vcftools --gzvcf ALL.chr19.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz --recode --chr 19 --from-bp 33349004 --to-bp 33935179```

This will filter the VCF file to only include the positions on chromosome 19 from 33,349,004 to 33,935,179


### Pre-processing 2 - Filter linked sites using PLINK

As we discussed in class our data contains variants that are linked (non-independently inherited). We want to include only 1 representative from each of these inherited blocks. 

For this we use PLINK - another software that is already loaded onto the HPC cluster 

The command is:

```--vcf chr19_human_data_edited.recode.vcf --indep-pairwise 50 10 0.1 --out chr19_human_pruned``` 

The command ```--indep-pairwise 50 10 0.1``` 

50 denotes we have set a window of 50 Kb. The second argument, 10 is our window step size - meaning we move 10 bp each time we calculate linkage. Finally, we set an r2 threshold - i.e. the threshold of linkage we are willing to tolerate. Here we prune any variables that show an r2 of greater than 0.1

This is the output from PLINK

```PLINK v2.00a3.3LM AVX2 Intel (3 Jun 2022)      www.cog-genomics.org/plink/2.0/
(C) 2005-2022 Shaun Purcell, Christopher Chang   GNU General Public License v3
Logging to chr19_human_pruned.log.
Options in effect:
  --indep-pairwise 50 10 0.1
  --out chr19_human_pruned
  --vcf chr19_human_data_edited.recode.vcf

Start time: Tue Mar 28 12:46:58 2023
63990 MiB RAM detected; reserving 31995 MiB for main workspace.
Using up to 8 compute threads.
--vcf: 18182 variants scanned.
--vcf: chr19_human_pruned-temporary.pgen +
chr19_human_pruned-temporary.pvar.zst + chr19_human_pruned-temporary.psam
written.
2504 samples (0 females, 0 males, 2504 ambiguous; 2504 founders) loaded from
chr19_human_pruned-temporary.psam.
18182 variants loaded from chr19_human_pruned-temporary.pvar.zst.
Note: No phenotype data present.
Calculating allele frequencies... done.
--indep-pairwise (1 compute thread): 3938/18182 variants removed.
Variant lists written to chr19_human_pruned.prune.in and
chr19_human_pruned.prune.out .
End time: Tue Mar 28 12:46:58 2023```


### Pre-processing 3 - Filter the VCF file based on the pruning done in PLINK

We run VCFtools once more to filter our VCF file with the pruned SNPS

```vcftools --vcf chr19_human_data_edited.recode.vcf --recode --snps chr19_human_pruned.prune.out --out chr19_human_pruned```

