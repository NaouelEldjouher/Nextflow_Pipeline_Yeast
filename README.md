# Nextflow Yeast Variant Calling Pipeline

> **Project Status: Archival / Educational**  
> *Note: This repository represents one of my early projects focusing on local workflow execution and Nextflow DSL1. My current production pipelines (e.g., AMR-Flow) utilize Nextflow DSL2, containerized environments (Docker/Singularity), and cloud infrastructure (AWS/Terraform).*

## 🧬 Project Overview

This project aims to reproduce a previously conducted study on the Yeast Genome by migrating the analysis into a reproducible, automated **Nextflow** pipeline. 

The workflow processes raw Illumina sequencing data to identify genomic variants, covering SRA retrieval, quality control, alignment, and variant calling using the GATK best practices framework.

Original study reference and methodology can be found [here](https://github.com/Naoueldjouher/Chip-seq_Yeast_Genome/blob/main/README.md).

## ⚙️ Pipeline Architecture

The pipeline is modularized into the following sequential processes:

1. **`downloadsrr.nf`**: Retrieves raw sequencing data using the SRA Toolkit.
2. **`trimming.nf`**: Performs quality control and adapter trimming via Trimmomatic.
3. **`alignment.nf`**: Aligns high-quality reads to the reference genome using Bowtie2.
4. **`sortandpicard.nf`**: Converts SAM to BAM, sorts reads, and adds Read Groups using Samtools and Picard.
5. **`Gatk.nf`**: Performs variant calling utilizing the Genome Analysis Toolkit (GATK).

## 🚀 Getting Started

### Prerequisites
To run this pipeline locally, ensure the following tools are installed and accessible in your `$PATH`, or update the `params` in the Nextflow script to point to their local binaries:
* [Nextflow](https://www.nextflow.io/)
* Java
* SRA Toolkit
* FastQC & Trimmomatic
* Bowtie2 & Samtools
* Picard & GATK4

### Installation
Clone the repository to your local machine:
```bash
git clone [https://github.com/Naoueldjouher/Nextflow_Pipeline_Yeast.git](https://github.com/Naoueldjouher/Nextflow_Pipeline_Yeast.git)
cd Nextflow_Pipeline_Yeast
```
## Usage
Execute the pipeline by specifying the target SRR accession number:
```bash
nextflow run main.nf --srr_numbers SRR1811834
```
(Note: Parameters such as tool paths, output directories, and thread counts can be customized directly in the nextflow.config file or passed via command line).

## Scientific & Technical Notes
During the development and testing of this pipeline, a few specific bioinformatics challenges were encountered and addressed. Documenting these ensures transparency and reproducibility.
### 1. The Illumina R2 Quality Discrepancy
Following the Trimmomatic step, a significant size discrepancy was observed between the forward and reverse paired-end reads for sample SRR1811834:

* ***SRR1811834_R1_paired_trimmed.fastq.gz:*** 605 MB

* ***SRR1811834_R2_paired_trimmed.fastq.gz:*** 44 MB

* ***Analysis:*** This is a well-documented phenomenon in Illumina sequencing, where the reverse read (R2) often suffers from a severe quality drop-off compared to the forward read (R1). Because Trimmomatic is configured to drop reads that fall below a specific quality threshold, the vast majority of the R2 reads were correctly discarded.

* ***Resolution:*** To maintain data integrity and prevent poor alignments, the downstream Bowtie2 alignment step was adjusted to proceed in single-end mode utilizing only the robust Pair_1 reads.

### 2. GATK ApplyBQSR in Non-Human Genomes
During the GATK Base Quality Score Recalibration (BQSR) step, the pipeline threw an error indicating no mismatches were found between the reads and the reference genome.

* ***Analysis:*** BQSR relies heavily on a highly accurate, pre-existing database of "known variants" (such as dbSNP in humans) to mask true biological variants and recalculate machine error rates. The Yeast genome often lacks a standardized, comprehensive VCF of known variants that aligns perfectly with specific experimental reference genomes.

* ***Resolution:*** After validating the integrity of the BAM files with Picard's ValidateSamFile, the ApplyBQSR step was intentionally bypassed. For organisms lacking robust known-variant databases, bypassing BQSR is a standard bioinformatics practice to prevent the model from incorrectly recalibrating base qualities.








