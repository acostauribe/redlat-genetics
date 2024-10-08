# Quality control pipeline for genomic data
#### Developed by Juliana Acosta-Uribe for the ReD-Lat Consortium 2024

## Description
This is a pipeline designed to perform Quality Control of genomica data, either from Genome, Exome or SNP Arrays.\
It also allows to start either with a vcf.gz file or a plink fileset. 

## Requirements
-[R](https://www.r-project.org/)\
-[RStudio](https://posit.co/download/rstudio-desktop/)\
-[plink](https://www.cog-genomics.org/plink2/)\
-[king](https://www.kingrelatedness.com/)

If you are working with Exome or Genome data, you will also need:\
-[vcftools](https://vcftools.github.io/man_latest.html)\
-[bcftools](https://samtools.github.io/bcftools/bcftools.html)

## Installation
Ensure that all required software is installed on your system. Specify the path to each of the required software in the rmd 

## Usage
The [quality_control.rmd](quality_control.rmd) pipeline is designed to be run as an [R markdown](https://rmarkdown.rstudio.com/lesson-1.html) file in [RStudio](https://posit.co/download/rstudio-desktop/). This way you can run it in a step-by-step mode and generate a rendered quality control report. 

Download `quality_control.rmd` to your local RStudio and modify steps *1. Set up your environment* and *2. Customize the quality control process* to fit your quality control goals. The rest of the code does not require modifications. (More information below)

You could also edit `quality_control.rmd` and run it directly from the r command line if you already have the `sample_data.txt` and the `problematic_relatedness.txt` files in your workspace. 

```r
library(rmarkdown) 
render("path/to/quality_control.Rmd")
```

## Arguments
- <genome_vcf>.vcf.gz - bgzipped genome or Exome VCF file, or 
- <genome_plink>.bed/<genome_plink>.fam/<genome_plink>.bim - Plink formatted dataset

If you are starting with a *<genome_vcf>.vcf*, or your *<genome_plink>.fam* does not have the sex/family of the samples already specified please provide an additional file `sample_data.txt` that includes all the individuals in your data and has the following header:

**IID** ID of the sample as it is in the plink IID or in the VCF\
**SEX** should be specified (1=male, 2=female, 0=no data)\
**FID** Family ID of the sample\
**PID** Paternal ID of the sample. If this individual is also in the dataset, the PID should be identical to the father's IID\
**MID** Maternal ID of the sample. If this individual is also in the dataset, the MID should be identical to the mother's IID\
**PHENO** Optional, if you are interested in any downstream analyses involving the phenotype (1=control, 2=case, -9=missing)

⚠️ The **FID** given to the parents needs to match the same **FID** given to that individual for their genome.

A trio would look like this (order of the columns does not matter)

| FID  | IID | PID | MID | SEX | PHENO |
|------|-----|-----|-----|-----|-------|
| FID1 | SON | DAD | MOM | 1   | 2     |
| FID1 | MOM | 0   | 0   | 2   | 1     |
| FID1 | DAD | 0   | 0   | 1   | 1     |

R is expecting a tab delimited file. If your file is delimited by spaces you can fix it with the following bash command 

```bash
sed -i 's/ /\t/g'  sample_data.txt`
```


## Output
The script generates several intermediate files and graphs of the QC process. The final output is a quality controled dataset which is determined based on specific criteria defined in the R script.

## Script Components
- quality_control.rmd: The main script that runs the workflow.
- data_preprocessing.sh: A script to rename samples, recode variants according to the reference genome, assign IDs to all variable in bim file and normalize INDELs. (requires plink2 in addition to previous software)


## Workflow

> **Always Take a look at your files before beginning:** \
> If you start with a plink dataset: 
> - Do the Individual IDs (column 2 in *file.fam*) match what you were expecting? 
> - Have the families been given a Family ID (column 1 in *file.fam*)?  
> - Do the Individuals have their sex assigned (column 5 in *file.fam*)?  
> - Do the variants in the *.bim* have an identifier (column 2 in *file.bim*)? Some analyses will require this information, and we may have to incorporate it to the *file.fam*/*file.bim* if its not already there. \
> ⚠️ Make sure your files are properly aligned and the alleles are being called from the correct strand. INDELs should be [left aligned and normalized](https://samtools.github.io/bcftools/bcftools.html#norm).

1. **Set up your environment**
- Install required packages
- Set your working directory
- Set up path to software
- Give the name of your starting file
- Define the type of data you will be working with. ('GENOME', 'EXOME', 'ARRAY')
- Specify your reference genome ('hg19', 'hg38')
- Import sample data (if provided)

2. **Customize the quality control process**
- Do you want to do filter variant calls for genotype depth (DP) and Genotype Quality (GQ)?
- Do you want to keep only the variants with PASS in the VQSR filtering?
- Do you want to check for known and cryptic relatedness among samples?
- Do you want to create directories and to organize your data as you go?
  
3. **Start Quality Control process**
- Generate and plot basic statistics:
- *I. General statistics* [.vchk](https://samtools.github.io/bcftools/bcftools.html#stats)
- *II. Sample based metrics* [metrics](https://vcftools.sourceforge.net/man_latest.html#OUTPUT%20OPTIONS)
-- missingness per sample (.imiss)
-- depth per sample (.idepth)
- *III. Site based metrics*
-- Missingness per site (.lmiss)
-- Mean depth per site (.ldepth.mean)
--- VQSR quality [vqsr](https://gatk.broadinstitute.org/hc/en-us/articles/360035531612-Variant-Quality-Score-Recalibration-VQSR-)
  
4. **Genotype Quality control (Only for Exome and Genome data)**
- Filter your genotypes according to genotype depth (DP) and genotype quality (GQ)
- Filter your sites according to their Variant Quality Score Recalibration ranks
- *Genotypes with DP and GQ below desired thresholds are replaced with missing values*
  
5. **Filter for missingness** 
- Identify samples and variants (sites) missing more than 10% of data and remove them. The 10% or 0.1 threshold can be modified as desired.
- *Samples and variants with missingness over the desired threshold will be removed*

6. **Calculate heterozygosity** 
- Use plink to calculate sample heterozygosity and then exclude individuals that are beyoond 3 standard deviations from the cohort mean. 
- *Heterozygosity outliers will be removed*

7. **Check sex**
- Use plink to determine chromosomal sex according to X and Y chromosomes 
- *Individuals with discrepancy between genetic and disclosed sex will be removed*
   
8. **Identify duplicates** 
- Use King to detect duplicate samples
- *Duplicate samples are removed*
    
9. **Calculate relatedness(optional)**
- Use King to identify samples that are up tho 4th degree of relatedness
- ⚠️ You have to manually check if King's inference match your previous kinship knowledge.
- *Samples with cryptic relatedness are removed*

10. **Remove variants & Individuals and with missingness ≥5%**
- Do a final removal of samples and variants with high missigness.
- *Samples and variants with missingness over the desired threshold will be removed*

All of these steps are extensively described in [quality_control.rmd](quality_control.rmd) 

![alt text](QC.png)

## References
This pipeline integrated multiple workflows for quality control. 
Some of our resources were:

- Broad Institute gnomAD
  
   [gnomAD v3.1](https://gnomad.broadinstitute.org/news/2020-10-gnomad-v3-1-new-content-methods-annotations-and-data-availability/#sample-and-variant-quality-control/)
  
   [gnomAD v3.0](https://gnomad.broadinstitute.org/news/2019-10-gnomad-v3-0/)

- [Anderson, C. A.](https://doi.org/10.1038/nprot.2010.116), Pettersson, F. H., Clarke, G. M., Cardon, L. R., Morris, A. P., & Zondervan, K. T. (2010). Data quality control in genetic case-control association studies. Nature protocols, 5(9), 1564-1573.

- [Panoutsopoulou, K.](https://doi.org/10.1007/978-1-4939-7868-7_3), & Walter, K. (2018). Quality control of common and rare variants. Chapter in: Genetic Epidemiology: Methods and Protocols, Methods in Molecular Biology, vol. 1793, Springer Science Business Media, LLC, part of Springer Nature 2018


## Contact
For support or contributions, please contact me at:
 
`acostauribe@ucsb.edu`
