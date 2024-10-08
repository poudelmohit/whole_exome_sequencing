# Setting up directories:
```bash 

    mkdir -p data/raw_fastq software/ results/
```
# Installing Conda:
```bash

    cd software && wget https://repo.anaconda.com/archive/Anaconda3-2024.06-1-Linux-x86_64.sh
    chmod +x Anaconda3-*.sh
    bash Anaconda3-*.sh # Press Enter and 'Yes'
    # Please restart the terminal
    conda config --set auto_activate_base false
```
# 1. Creating Conda Env and setting up:

```bash

    conda create -n wes -y
    conda activate wes

    conda config --add channels defaults
    conda config --add channels bioconda
    conda config --add channels conda-forge

    conda install bioconda::fastqc -y
    conda install bioconda::trimmomatic -y
    conda install bioconda::bwa -y
    conda install bioconda::samtools -y
    conda install bioconda::bcftools -y
    conda install bioconda::snpeff -y
    conda deactivate
    # due to version conflict, I am creating new env for 
    # GEMINI (tool required in Step 5: Annotation)

    conda create -n gemini_env python=2.7 -y
    conda activate gemini_env
    conda install -c bioconda gemini -y
    conda deactivate
```


## 1.1 Obtaining raw reads:

```bash
    cd data/raw_fastq

    wget https://zenodo.org/record/3243160/files/father_R1.fq.gz
    wget https://zenodo.org/record/3243160/files/father_R2.fq.gz
    wget https://zenodo.org/record/3243160/files/mother_R1.fq.gz
    wget https://zenodo.org/record/3243160/files/mother_R2.fq.gz
    wget https://zenodo.org/record/3243160/files/proband_R1.fq.gz
    wget https://zenodo.org/record/3243160/files/proband_R2.fq.gz

    # unzipping files:
    gunzip *.gz
```

## 1.2 Downloading reference sequence:

```bash
    cd ..
    wget https://zenodo.org/record/3243160/files/hg19_chr8.fa.gz
    gunzip hg19_chr8.fa.gz
```

# 2. Quality Control:

```bash
    mkdir ../results/fastqc_results
    cd raw_fastq && ls
    fastqc *.fq

    # moving fastqc results to different directory:

    mv *.html *.zip ../../results/fastqc_results
    ls ../../results/fastqc_results
    # Quality of the reads looks fine to me. Skipping trimming for now.
```

# 3. Mapping paired-end raw reads with reference fasta:

```bash
    mkdir ../mapped_files && cd $_
    mv ../hg19_chr8.fa ./
```

## 3.1 Indexing the reference fasta file:

```bash
    ls
    bwa index hg19_chr8.fa
```

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
```

# 4.1 Variant Calling:

```bash
    mkdir ../variant_calling_files

    bcftools mpileup -f hg19_chr8.fa father.sorted.rmdup.bam mother.sorted.rmdup.bam proband.sorted.rmdup.bam -Oz -o ../variant_calling_files/trio.pileup.vcf.gz

    cd ../variant_calling_files
    
    # calling variants:
    bcftools call -mv -Oz -o trio.variants.vcf.gz trio.pileup.vcf.gz

    # indexing the VCF file:
    bcftools index trio.variants.vcf.gz
    

    # fitering the raw VCF file:

    bcftools filter -i 'QUAL>30' -Oz -o trio.filtered.variants.vcf.gz trio.variants.vcf.gz
```


# 5.1 Annotation:

```bash
    conda activate wes

    # downloading the snpeff database:
    snpEff download GRCh37.75

    snpEff -Xmx8G GRCh37.75 trio.filtered.variants.vcf.gz > reannotated.vcf.gz
    ls -lh 
```
# 5.2 Renaming the sample names in annotated file:

```bash
    bcftools view -h annotated.vcf.gz | grep "#CHROM"

    echo "father.sorted.rmdup.bam father" > sample_map.txt
    echo "mother.sorted.rmdup.bam mother" >> sample_map.txt
    echo "proband.sorted.rmdup.bam proband" >> sample_map.txt

    cat sample_map.txt

    # using bcftools to modify the sample name in annotated vcf file:
    bcftools reheader -s sample_map.txt -o reannotated.vcf.gz annotated.vcf.gz

    # Verifying the renamed samples:
    bcftools view -h reannotated.vcf.gz | grep "#CHROM"
    
    # copying all variant calling files to results section:
    cp -r ../variant_calling_files results
```

# 5.3 Creating a pedigree file:

```bash

    touch family.ped
    echo "#family_id    name     paternal_id    maternal_id    sex    phenotype" > family.ped
    echo "FAM           father   0              0              1      1" >> family.ped
    echo "FAM           mother   0              0              2      1" >> family.ped
    echo "FAM           proband  father         mother         1      2" >> family.ped
```

# 5.4 Installing GEMINI:
```

    cd ../../software
    wget https://raw.github.com/arq5x/gemini/master/gemini/scripts/gemini_install.py

    python gemini_install.py ~/gemini ~/gemini

    ls
```

# 5.5 Creating a GEMINI Database:
```bash
    conda activate gemini_env
    conda install openssl=1.0
    gemini load -h

    gemini load --cores 4 -v reannotated.vcf.gz -t snpEff -p family.ped db.gemini
    pip install ipyparallel==4.0
    pip install ipython-cluster-helper==0.6.4
    pip show gemini ipython-cluster-helper ipyparallel
    pip show gemini
    gemini install-data
```
