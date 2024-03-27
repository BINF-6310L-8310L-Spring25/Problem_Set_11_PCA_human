# Problem_Set_12

In this week's problem set we will be conducting a PCA on human SNP data.

There are several pre-processing steps that I conducted that I will outline below. 

### Pre-processing 1 - Get a specific region of Chromosome 19

Due to time and processor constraints we will be anlayzing only a small portion of Human Chromosome 19 

Here is a snapshot of the region we are looking at. I used the 1000 Genomes data to look at the common SNPs (there are so many it's just a black line at the bottom) 

![Screenshot 2023-03-29 075252](https://user-images.githubusercontent.com/47755288/228527739-9ed8e3ca-28c5-4ac9-b18d-4cdc0ec311fc.png)



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
End time: Tue Mar 28 12:46:58 2023
```


### Pre-processing 3 - Filter the VCF file based on the pruning done in PLINK

We run VCFtools once more to filter our VCF file with the pruned SNPS

```vcftools --vcf chr19_human_data_edited.recode.vcf --recode --snps chr19_human_pruned.prune.out --out chr19_human_pruned```


## PROBLEM SET 

Below, you will find the questions for this week's problem-set 


### Problem A - Import VCF file 

Using the vcfR package read in the VCF file chr19_human_pruned.recode.vcf

# Quesstion A1
- How many variants are there in the data file?

# Question A2
- How many individuals are in the data file?

&nbsp;
&nbsp;

### Problem B - Data quality checks 

Use the ```extract.gt()``` function to obtain the genotype file from the VCF. 

Transpose the genotype file 

# Question B1 
- How many missing data points are in our data? 

Use the ```remove_constant()``` function from the ```janitor``` package to remove columns that are invariant in our dataset

# Question B2
- How many rows (SNPs) were removed from our dataset?

&nbsp;
&nbsp;

### Problem 3 - PCA C

Use ```prcomp()``` to run a PCA on our SNP data 

Use the ```fviz_eig()`` function from the ```factoextra``` package create the scree plot

# Question C1
- Approximately how much variation is explained on the first principal component (dimension 1)?

&nbsp;
&nbsp;

### Problem D - Add labels to the PCA data

We will use ggplot to visualize our PCA because it is more flexible. Follow the below steps to format the data and import data labels.

1. Save the PCA coordiantes from the PCA output into a new matrix. 
2. Convert this matrix to a data frame.
3. We want to add labels to our PCA points. The label data can be found in the file ```igsr_samples.tsv```. Read in the label data. 

To merge these two we will use the join functions of titdyr. These functions are _very_ useful because they will automatically merge the data based on a shared column name. 

4. Create a new column in your PCA output coordinate data frame that contains the row names of the PCA. Call this variable ```Sample.name``` The command should look something like ```pca_points$Sample.name<-rownames(pca_points)```
5. Use the ```left_join()``` function to merge your PCA points with the sample data. You want the PCA points to be on the _left side_ of this command. This will keep all observations in the first dataset and keep only the matching data points in the second dataset 
6. You can use the function ```table(dat$col)``` to summarize a column or row in a dataframe. 

# Question D1
- In the data, how many individuals are found in the AFR (African) Superpopulation?

# Question D2
- In the data, how many individuals are female (sex)

&nbsp;
&nbsp;

### Problem E - Visualize the data

Use ggplot to produce a plot on PC1 and PC2. Color the points with the SuperPopulation groups

# Question E1
- What Superpopulation appears to be _the most distinct_ from the other populations?


Use ggplot to produce a plot on PC1 and PC2. Color the points with the Sex groups

# Question E2
- Does our data cluster by sex?


### Problem F - Get Clusters 

Use thhe ```fviz_nbclust()``` function to plot the number of clusters **in the PCA coordinates for PC1 and PC2**against the within sum-of-squares. Remember, we are looking for where the "elbow" of that plot is (aka where the improvement starts to decrease in magnitude)

# Question F1
- Based on this how many clusters would you propose for k-means clustering? 


Cluster the PC coordinates based on this number of clusters and visualize the results and color the points by these clusters
We want to look at how well our PCA reflects the Super Populations. Extract the columns that contain the Super Population and the cluster assignments. Use the ```table()``` function to summarize the counts for each population in the groups. 

# Question F2
- Do any of your clusters contain only (or mostly) a single superpopulation? 


### Knit and upload your document to the Canvas site

