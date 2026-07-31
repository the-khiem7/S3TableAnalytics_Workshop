---
title: "About Datasets"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b>6.1. </b>"
---

## 1. What is VCF?

A file format for storing genetic variations found in a genome. Primarily used to record SNPs, indels, etc.

This file format was developed by the 1000 Genomes Project, and currently VCF version 4.5 is commonly used.

VCF files are text-based and used to store genomic data for various species, not just human genomes.

**Applications**

- Genomic analysis
  - Exploring variations in specific genes
  - Analyzing allele frequency
- Disease research
  - Analyzing whether specific variations are associated with diseases
  - GWAS (Genome-Wide Association Study) research
- Visualization
  - VCF data can be viewed in IGV (Integrative Genomics Viewer)

## 2. VCF File Structure

A VCF file consists of two main parts:

- **Header**
  - Contains the file's metadata, starting with the `#` character.
  - Includes file format, reference genome version, field descriptions, etc.
- **Body**
  - Contains the actual variant information in a tabular format.
  - Composed of fields such as CHROM, POS, ID, REF, ALT, etc.

### 2.1. VCF Header

The VCF file header includes metadata starting with `##` and column headers starting with `#CHROM`.

```text
##fileformat=VCFv4.2
##source=GATK
##reference=hg19
##INFO=<ID=DP,Number=1,Type=Integer,Description="Total Depth">
##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
#CHROM  POS     ID      REF     ALT     QUAL    FILTER  INFO    FORMAT  SAMPLE1 SAMPLE2
```

- `##fileformat`: VCF file version
- `##source`: Information about the software that generated the file
- `##reference`: Reference genome version
- `##INFO`: Definition of data to be included in the INFO field
- `##FORMAT`: Description of FORMAT fields for individual samples
- `#CHROM`: Data column names

### 2.2. VCF Body

Variant information fields. Stored in a tab-separated format, containing variant information.

```text
#CHROM  POS     ID      REF  ALT  QUAL  FILTER  INFO                 FORMAT    SAMPLE1  SAMPLE2
chr1    12345   rs123   G    A    99.0  PASS    DP=100;AF=0.5        GT:DP     0/1:50   1/1:40
chr2    67890   .       T    C    60.0  PASS    DP=80;AF=0.3         GT:DP     0/0:20   0/1:60
```

- **CHROM**: Chromosome where the variant occurred (e.g., chr1, chr2, etc.)
- **POS**: Position of the variant on the chromosome
- **ID**: Unique ID of the variant (variant database ID)
- **REF**: Reference genome's base sequence
- **ALT**: Alternative base sequence where the variant occurred
- **QUAL**: Quality score of the variant (higher score means higher reliability)
- **FILTER**: Flag indicating which of the provided filters were passed
- **INFO**: Additional variant information (e.g., DP=100, AF=0.5)
  - `DP=100`: Read Depth (how many times the variant was read)
  - `AF=0.5`: Allele Frequency (frequency of the alternative base)
  - `MQ=60`: Mapping Quality
  - ...
- **FORMAT**: Expandable list of fields for sample description

| Tag Name | Description |
|----------|-------------|
| GT | Genotype of the sample for this position. e.g., 0/0 – means homozygous reference; e.g., 0/1 – means heterozygous, having REF/ALT allele; e.g., 1/1 – means homozygous alternate |
| AD | Unfiltered Allele Depth (comma-separated) |
| DP | Filtered Depth |
| PL | Normalized Phred-scaled likelihoods for predicted genotypes |
| GQ | Genotype Quality, Phred-scaled confidence indicating the probability that GT is correct |
| MQ | RMSMappingQuality |

## 3. What is Glow?

Glow is an open-source toolkit for working with genomic data at biobank-scale and beyond.

The toolkit is natively built on Apache Spark, the leading unified engine for big data processing and machine learning, enabling genomics workflows to scale to population levels.

### 3.1. Introduction

- Genomic data is growing to big data scale, doubling every 7 months globally.
- However, most genomics data tools run on a single node and are not scalable.
- Glow emerged to address this issue.
- Built on Apache Spark and Delta Lake, enabling distributed computation and storage for genetic data.

### 3.2. Features

- **Data ingestion**: Can read VCF, BGEN, Plink formats into Apache Spark DataFrames.
- **Built-in functionality**: Common tasks like calculating quality control statistics, running regression tests, and performing simple transformations are provided as Spark functions that can be called from Python, SQL, Scala, or R.
- **Data transformation**: Provides dataset transformation features such as variant normalization and lift over.
- **Extensibility**: Can add User-defined Functions, a feature of Apache Spark.
- **Connects bioinformatics and big data ecosystems**: Best practices used by data engineers and data scientists across industries.
- Built on Apache Spark and Delta Lake, enabling distributed computation and storage for genetic data.
- **Integration**: Can be combined with datasets such as electronic medical records, real-world evidence, and medical images to generate additional insights.
