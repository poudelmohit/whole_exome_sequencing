# 1. Creating Conda Env and setting up:

```bash
    
conda create -n wes -y
conda activate wes

conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge

conda activate wes

conda install bioconda::fastqc -y
conda install bioconda::trimmomatic -y
conda install bioconda::bwa -y
conda install bioconda::samtools -y
conda install bioconda::bcftools -y
conda install bioconda::snpeff -y

# due to version conflict, I am creating new env for 
# GEMINI (tool required in Step 5: Annotation)

conda create -n gemini_env python=3.8
conda activate gemini_env
conda install -c bioconda gemini

## 
```bash
    mkdir data/raw_fastq software/ results/

## 1.1 Obtaining raw reads:
```
    cd data/raw_fastq

    wget https://zenodo.org/record/3243160/files/father_R1.fq.gz
    wget https://zenodo.org/record/3243160/files/father_R2.fq.gz
    wget https://zenodo.org/record/3243160/files/mother_R1.fq.gz
    wget https://zenodo.org/record/3243160/files/mother_R2.fq.gz
    wget https://zenodo.org/record/3243160/files/proband_R1.fq.gz
    wget https://zenodo.org/record/3243160/files/proband_R2.fq.gz
    
    # unzipping files:
    gunzip *.gz

## 1.2 Downloading reference sequence:
```bash
    cd ..
    wget https://zenodo.org/record/3243160/files/hg19_chr8.fa.gz
    gunzip hg19_chr8.fa.gz

# 2. Quality Control:
```bash
    mkdir fastqc_results
    cd raw_fastq && ls

    fastqc *.fq

    # moving fastqc results to different directory:

    mv *.html *.zip ../fastqc_results
    ls ../fastqc_results
    # Quality of the reads looks fine to me. Skipping trimming for now.

# 3. Mapping paired-end raw reads with reference fasta:
```bash
    mkdir ../mapped_files && cd $_
    
    mv ../hg19_chr8.fa ./

## 3.1 Indexing the reference fasta file:
```bash
    bwa index hg19_chr8.fa 

## 3.2 Mapping the paired-end reads (sam and bam):
```bash
    bwa mem hg19_chr8.fa ../raw_fastq/father_R?.fq > father.sam
    bwa mem hg19_chr8.fa ../raw_fastq/mother_R?.fq > mother.sam
    bwa mem hg19_chr8.fa ../raw_fastq/proband_R?.fq > proband.sam

    # Converting into bam format:

  for file in $(ls *.sam); do

    base=$(basename $file .sam)    # Convert SAM to BAM and keep only properly paired reads
  
    samtools view -S -b $file | samtools view -b -f 2 > ${base}.mapped.bam    # Sort the BAM file
  
    samtools sort ${base}.mapped.bam -o ${base}.sorted.bam    # Remove duplicates
  
    samtools rmdup ${base}.sorted.bam ${base}.sorted.rmdup.bam    # Index the final BAM file
  
    samtools index ${base}.sorted.rmdup.bam

done

# 4. Variant Calling:
```bash
    mkdir ../variant_calling

    bcftools mpileup -f hg19_chr8.fa father.sorted.rmdup.bam mother.sorted.rmdup.bam proband.sorted.rmdup.bam -Oz -o ../variant_calling/trio.pileup.vcf.gz

    cd ../variant_calling
    
    # calling variants:
    bcftools call -mv -Oz -o trio.variants.vcf.gz trio.pileup.vcf.gz

    # indexing the VCF file:
    bcftools index trio.variants.vcf.gz
    

    # fitering the raw VCF file:

    bcftools filter -i 'QUAL>30' -Oz -o trio.filtered.variants.vcf.gz trio.variants.vcf.gz

# 5. Annotation:




    




    