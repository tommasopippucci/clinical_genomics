---
layout: page
title: Bioinformatic analysis in NGS
summary: "Basics of file formats and bioinformatic workflows for NGS data analysis"
sidebar_link: true
---

> This is an introductory page to the basics of bioinformatic analysis of NGS data, but it is not to be intended as a tutorial on NGS pipelines and variant calling. This on-line training activity is indeed focused on data analysis for clinical interpretation. If you are looking for training on variant calling, visit this **Galaxy** tutorial on [Exome sequencing data analysis for diagnosing a genetic disease](https://galaxyproject.github.io/training-material/topics/variant-analysis/tutorials/exome-seq/tutorial.html).

# Next Generation Sequencing

- *Next (or Second) Generation Sequencing* (NGS/SGS) is an umbrella-term covering a number of approaches to DNA sequencing that have been developed after the first, widespread and for long time most commonly used Sanger sequencing.
- *NGS* is also known as *Massive Parallel Sequencing* (MPS), a term that makes explicit the paradigm shared by all these technologies, that is to sequence in parallel a massive library of spatially separated and clonally amplified DNA templates. 
    - For a comprehensive review of the different *NGS* technologies see [Goodwin et al., 2016](https://www.nature.com/articles/nrg.2016.49), which also includes an introduction to the third generation methods allowing sequencing of long single-molecule reads.


## NGS in the clinic

In the span of less than a decade, NGS approaches have pervaded clinical laboratories revolutionizing genomic diagnostics and increasing yield and timeliness of genetic tests.

In the context of disorders with a recognized strong genetic contribution such as neurogenetic diseases, *NGS* has been firmly established as the strategy of choice to rapidly and efficiently diagnose diseases with a Mendelian basis. A general diagnostic workflow for these disorders currently embraces different *NGS*-based diagnostic options as illustrated in Figure 1.

---

![ngs_neuro_diagnostics]({{site.url}}{{site.baseurl}}/images/ngs_neuro_diagnostics.png)
**Figure 1**. General workflow for genetic diagnosis of neurological diseases. (\*If considering high-yield single-gene testing of more than 1–3 genes by another sequencing method, note that next-generation sequencing is often most cost-effective. †Genetic counselling is required before and after all genetic testing; other considerations include the potential for secondary findings in genomic testing, testing parents if inheritance is sporadic or recessive, and specialty referral. From [Rexach et al., 2019](https://www.thelancet.com/journals/laneur/article/PIIS1474-4422(19)30033-X/fulltext))

---

Currently, most common *NGS* strategies in clinical laboratories are the so-called **targeted sequencing** methods that, as opposed to **genome sequencing** covering the whole genomic sequence, focus on a pre-defined set of regions of interest (the **targets**). The targets can be selected by **hybrid capture** or **amplicon sequencing**, and the target-enriched libraries is then sequenced. The most popular target designs are:

- **gene panels** where the coding exons of only a clinically-relevant group of genes are targeted
- **exome sequencing** where virtually all the protein-coding exons in a genome are simultaneously sequenced


## Basics of NGS bioinformatic analysis

Apart from the different width of the target space in exome and gene panels, these two approaches usually share the same experimental procedure for NGS library preparation. After clonal amplification, the fragmented and adapter-ligated DNA templates are sequenced from both ends of the *insert* to produce short reads in opposite (**forward** and **reverse**) orientation (**paired-end** sequencing). 

Bioinformatic analysis of NGS data usually follows a general three-step workflow to variant detection. Each of these three steps is marked by its "milestone" file type containing sequence data in different formats and metadata describing sequence-related information collected during the analysis step that leads to generation of that file.

---

| NGS workflow step | File content | File format | File Size (individual exome) |
| --- | --- | --- | --- |
| Sample to reads | Unaligned reads and qualities | **fastQ** | gigabytes |
| Reads to alignments | Aligned reads and metadata | **BAM** | gigabytes |
| Alignments to variants | Genotyped variants and metadata | **VCF** | megabytes |

---


Here are the different formats explained: 

- **[fastQ](https://en.wikipedia.org/wiki/FASTQ_format)** (sequence with quality): the *de facto* standard for storing the output of high-throughput sequencing machines
  - Usually not inspected during data analysis
- **[BAM](https://samtools.github.io/hts-specs/SAMv1.pdf)** (binary sequence alignment/map): the most widely used TAB-delimited file format to store alignments onto a reference sequence
  - Aligned reads
- **[VCF](http://samtools.github.io/hts-specs/VCFv4.3.pdf)** (variant call format): the standard TAB-delimited format for genotype information associated with each reported genomic position where a variant call has been recorded

Another useful file format is [BED](https://www.ensembl.org/info/website/upload/bed.html), to list genomic regions of interest such as the exome or panel targets.

The steps of the ***reads-to-variants*** workflow can be connected through a bioinformatic **pipeline** ([Leipzig et al., 2017](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5429012/)), consisting in read alignments, post-alignment BAM processing and variant calling.


### Alignment

As generated by the sequencing machines, *paired-end* reads are written to two *fastQ* files in which *forward* and *reverse* reads are stored separately together with their qualities. *FastQ* files are taken as input files by tools (the **aligners**) that align the reads onto a reference genome. One of the most used aligners is [BWA](http://bio-bwa.sourceforge.net/) among the many that have been developed (Figure 2).

---

![aligners_timeline]({{site.url}}{{site.baseurl}}/images/aligners_timeline.png)
**Figure 2**. Aligners timeline 2001-2012 (from [Fonseca et al., 2012](https://academic.oup.com/bioinformatics/article/28/24/3169/245777))

---

    
During the bioinformatic process, *paired-end* reads from the two separate *fastQ* files are re-connected in the alignment, where it is expected that they will:
- map to their correct location in the genome
- be as distant as the insert size of the fragment they come from
- be in opposite orientations
a combination which we refer to as **proper pairing**. All these data about *paired-end* reads mapping are stored in the BAM file and can be used to various purposes, from alignment quality assessment to structural variant detection.

In Figure 3, the [Integrative Genomic Viewer (IGV)](http://software.broadinstitute.org/software/igv/) screenshot of an exome alignment data over two adjacent *ASXL1* exons is shown. Pink and violet bars are *forward* and *reverse* reads, respectively. The thin grey link between them indicates that they are *paired-end* reads. The stack of reads is concentrated where exons are as expected in an exome, and the number of read bases covering a given genomic location *e* (depicted as a hill-shaped profile at the top of the figure) defines the **depth of coverage (DoC)** over that location:

*DoC*<sub>e</sub>=*number of read bases over e/genomic length of e*

---

![coverage]({{site.url}}{{site.baseurl}}/images/coverage.png)
**Figure 3**. Exome data visualization by [*IGV*](http://software.broadinstitute.org/software/igv/)

---


### Post-alignment BAM processing

Regarding post-alignment *pipelines*, the most famous for germline SNP and InDel calling is probably that developed as part of the GATK toolkit (Figure 4).

---

![gatk_germline_pipeline]({{site.url}}{{site.baseurl}}/images/gatk_germline_snps_indels.png)
**Figure 4**. After-alignment pipeline for germline SNP and InDel variant calling according to [GATK Best Practices](https://software.broadinstitute.org/gatk/best-practices/workflow?id=11145)

---

    
According to GATK best practices, in order to be ready for variant calling the BAM file should undergo the following processing:
- [marking duplicate reads](https://software.broadinstitute.org/gatk/documentation/tooldocs/4.0.4.0/picard_sam_markduplicates_MarkDuplicates.php) to flag (or discard) reads that are mere optical or PCR-mediated duplications of the actual reads
- [recalibrating base quality scores](https://software.broadinstitute.org/gatk/documentation/article?id=11081) to correct known biases of the native base-related qualities
While GATK BAM processing is beyond doubt important to improve data quality, it has to be noticed that it is not needed to obtain variant calls and that non GATK-based pipelines may not use it or may use different quality reparametrization schemes. Duplicate flagging or removal is not recommended in *amplicon* sequencing experiments.


### Variant calling

The process of variant detection and genotyping is performed by *variant callers*. These tools use probabilistic approaches to collect evidence that non-reference read bases accumulating over a given locus support the presence of a variant. To be confident that a variant is a true event, its supporting evidence should be significantly stronger than chance; e.g. the C>T on the left of the screenshot in Figure 5 is supported by all its position-overlapping reads, claiming for a variant. In contrast, the C>A change on the right of the screenshot is seen only once over many reads, challenging its interpretation as a real variant. 

---

![igv_screenshot_variant]({{site.url}}{{site.baseurl}}/images/igv_variant.png)
**Figure 5**. Variant visualization by *IGV*

---

The GATK variant calling pipeline first produces a **genomic VCF ([gVCF](https://gatkforums.broadinstitute.org/gatk/discussion/4017/what-is-a-gvcf-and-how-is-it-different-from-a-regular-vcf))**, whose main difference with *VCF* is that it records all sites in a target whether there is a variant or not, while *VCF* contains only information for variant sites, preparing multiple samples for **joint genotyping** and creation of a **multi-sample** *VCF* whose variants can undergo quality **filtering** in order to obtain the final set of quality-curated variants ready to be annotated.

In downstream analyses, annotations can be added to *VCF* files themselves or information in *VCF* files can be either annotated in TAB- or comma- deimited files to be visually inspected for *clinical* variant searching or used as input to **prioritization** programs.
