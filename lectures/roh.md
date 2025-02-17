---
layout: page
title: Runs of Homozygosity
summary: "Identification of Runs of Homozygosity from targeted sequencing data"
sidebar_link: true
---

- *Runs of Homozygosity* (ROHs) are sizeable stretches of consecutive homozygous markers encompassing genomic regions where the two haplotypes are identical
- Haplotypes in *ROHs* can underlie identity either by **state** (IBS) or by **descent** (IBD). IBD occurs when two haplotypes originate from a common ancestor (Figure 1), a condition whcih we refer to as **autozygosity**
- *Autozygosity* in an individual is the hallmark of high levels of genomic inbreeding (g*F*), often occurring in the offspring from consanguineous unions where parents are related as second-degree cousins or closer
- The most well-known medical impact of parental consanguinity is the increased risk of rare autosomal recessive diseases in the progeny, with excess risk inversely proportional to the disease-allele frequency ([Bittles, 2001](https://onlinelibrary.wiley.com/doi/full/10.1034/j.1399-0004.2001.600201.x?sid=nlm%3Apubmed))

---

![homozygosity_mapping]({{site.url}}{{site.baseurl}}/images/homozygosity_mapping.png)
**Figure 1**. Schematic of haplotype flow along a consanguineous pedigrees to form autozygous *ROHs* in the progeny (from [McQuillan et al., 2008](https://www.sciencedirect.com/science/article/pii/S000292970800445X?via%3Dihub))

---

## Homozygosity mapping

- We refer to *homozygosity mapping* ([Lander and Botstein, 1987](https://science.sciencemag.org/content/236/4808/1567.long)) as an approach that infers *autozygosity* from the detection of long ROHs to map genes associated with recessive diseases
- A classical strategy to identify *ROHs* is based on SNP-array data and analysis with [PLINK](http://zzz.bwh.harvard.edu/plink/)
- The combination of *homozygosity mapping* with exome sequencing has boosted genomic research and diagnosis of recessive diseases in recent years ([Alazami et al., 2015](https://www.sciencedirect.com/science/article/pii/S2211124714010444?via%3Dihub); [Monies et al., 2017](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5502059/))
- *ROHs* may be of clinical interest also since they can unmask **uniparental isodisomy**
- Computational approaches have been developed to identify *ROHs* directly in exome data, allowing **simultaneous detection of variants and surrounding ROHs** from the same datasets.


## Computational approaches

As for CNVs, the sparse nature and small size of exonic targets are a challenge for the identification of continuous strethces of homozygous markers throughout the genome. This may affect the performance of tools originally tailored to SNPs when applied to exome data. 

In Figure 2 a synthetic overview is given of algorithms, input data and output file types in bioinformatic tools for exome-based *ROH* detection. *ROHs* are divided in three size classes reflecting different mechanisms that have shaped them (according to [Pemberton et al., 2012](https://www.sciencedirect.com/science/article/pii/S0002929712003230?via%3Dihub)). 

Usually, long ROH (approximately >1.5 Mb) arise as a result of close parental consanguinity, but also short-medium *ROHs* can be of medical interest in populations as involved in disease susceptibility, natural selection and founder effects ([Ceballos et al., 2018](https://www.nature.com/articles/nrg.2017.109)).

---

![roh_exome_methods]({{site.url}}{{site.baseurl}}/images/roh_exome_methods.png)
**Figure 2**. Summary of the tools for *ROH* detection from exome data (from [Pippucci et al., 2014](https://www.karger.com/Article/Pdf/362412))

---

### H<sup>3</sup>M<sup>2</sup>

*H<sup>3</sup>M<sup>2</sup>* ([Magi et al., 2014](https://academic.oup.com/bioinformatics/article/30/20/2852/2422169)) is based on an heterogeneous [*Hidden Markov Model*](https://en.wikipedia.org/wiki/Hidden_Markov_model) (HMM) that incorporates inter-marker distances to detect *ROHs* from exome data.

*H<sup>3</sup>M<sup>2</sup>* calculates B-allele frequencies of a set of polymorphic sites throughout the exome as the ratio between allele *B* counts and the total read count at site *i*:

*BAF*<sub>i</sub> = *N<sub>b</sub>/N*

*H<sup>3</sup>M<sup>2</sup>* retrieves *BAF*<sub>i</sub> **directly from BAM files** to predict the heterozygous/homozygous genotype state at each polymorphic position *i*:

- when *BAF*<sub>i</sub> \~ 0 the predicted genotype is **homozygous reference**
- when *BAF*<sub>i</sub> \~ 0.5 the predicted genotype is **heterozygous**
- when *BAF*<sub>i</sub> \~ 1 the predicted genotype is **homozygous alternative**

*H<sup>3</sup>M<sup>2</sup>* models *BAF* data by means of the *HMM* algorithm to discriminate between regions of homozygosity and non-homozygosity according to the *BAF* distribution along the genome (Figure 3).

---
  
![h3m2_baf]({{site.url}}{{site.baseurl}}/images/h3m2_baf.png)
**Figure 3**. ***BAF* data distribution** Panels a, b and c show the distributions of BAF values against the genotype calls generated by the HapMap consortium on SNP-array data ( a ), the genotype calls made by SAMtools ( b ) and the genotype calls made by GATK ( c ). For each genotype caller, the distribution of BAF values is reported for homozygous reference calls (HMr), heterozygous calls (HT) and homozygous alternative calls (HMa). R is the Pearson correlation coefficient. Panels d–g show the distribution of BAF values in all the regions of the genome ( d ), in heterozygous regions ( e ), in homozygous regions ( f ) and in the X chromosome of male individuals ( g ). For each panel, the main plot reports the zoomed histogram, the left subplot shows the BAF values against genomic positions, whereas the right subplot shows the entire histogram of BAF values. (from [Magi et al., 2014](https://academic.oup.com/bioinformatics/article/30/20/2852/2422169))
