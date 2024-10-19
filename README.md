# Whole Exome Sequencing Analysis

## Project Overview
This project outlines the steps used to process whole exome sequencing (WES) data for variant identification in a trio consisting of father, mother, and proband. The analysis involves raw data preparation, quality control, alignment, variant calling, and annotation.

## Data Source
The data used in this project is the same as that used in the [GALAXY training](https://training.galaxyproject.org/archive/2019-02-07/topics/variant-analysis/tutorials/exome-seq/tutorial.html) platform. However, while the GALAXY platform is a web-based tool with a graphical user interface (GUI), this analysis was carried out on a Linux-based system using the high-performance computing (HPC) cluster provided by the Alvarado-Serrano Lab.

## Workflow Adaptation
Although the workflow structure follows a similar approach to the GALAXY WES pipeline, the tools used in this project differ. This pipeline relies on open-source, command-line bioinformatics tools such as BWA, Samtools, Bcftools, and SnpEff for processing and analysis, leveraging the computing power of an HPC cluster.

## Folder Structure
  data/raw_fastq: Contains the raw sequencing reads for father, mother, and proband. These files were downloaded from the [zenodo repository](https://zenodo.org/records/61377) used in the GALAXY platform. Due to file size limitations, files larger than 100 MB are not available in this GitHub repository.

  software: Includes setup files for software installation, such as Anaconda, and dependencies required for running the bioinformatics tools.

  results: Stores the output files generated during the analysis. These include:

  - Quality control results from FastQC.
  - Mapped files in BAM format after read alignment.
  -  Variant calling files, including VCF files generated during variant identification and filtering.
  -  Annotated VCF files after using SnpEff for variant annotation.
    
Please note that due to GitHub’s storage limitations, many large files (e.g., FASTQ files, BAM files, and VCF files) are not included in this repository. These files can be regenerated by following the analysis pipeline with access to the original data.

## Analysis Workflow

### Environment Setup:
  The analysis environment was set up using Conda to manage bioinformatics tools, ensuring reproducibility and version control.

### Data Acquisition:
  Raw sequencing reads for the trio were downloaded, and a reference genome (hg19, chromosome 8) was obtained to use in the alignment step.

### Quality Control:
  FastQC was used to evaluate the quality of the raw reads.

### Read Alignment:
  Reads were aligned to the reference genome using BWA, and the resulting files were processed to remove duplicates and sort the aligned reads.

### Variant Calling:
  Variants were identified using Bcftools from the aligned BAM files. Filtering was applied to retain high-quality variants.

### Annotation:
  The filtered variants were annotated using SnpEff to provide functional information about the identified variants.

## Notes
All commands and software used in this project were executed in a Linux environment on the HPC cluster provided by the Alvarado-Serrano Lab.
This pipeline offers a reproducible, command-line based alternative to the web-based GALAXY platform, focusing on flexibility and the ability to handle larger datasets in an HPC environment.
Due to storage limitations, not all files are provided here. The core analysis pipeline is documented, and files can be regenerated by following the instructions.

## Well, if you scrolled up to here !!
### I assume, you are excited about the results as well !!
Please [click here](results/final_result.md) to go to the results page.
**OR**
If you are excited **to see the result** of the analysis, please download [this file](results/variant_calling_files/snpEff_summary.html) and open in your favorite browser.

## Contact
For any questions or further details about the analysis, please contact [**Mohit Poudel**](https://twitter.com/MohitPoudel11)

## Special Thanks:
[**Alavarado-Serrano LAB, Ohio Univesity**](https://alvarado-s.weebly.com)
![Logo](https://github.com/poudelmohit/portfolio/blob/main/assets/lablogo-small.png)
