# Metagenomic-analysis---Pipeline

**Pipeline: Stepwise Metagenomic Analysis of Microbiome (Using WSL/Linux + Conda)**

**Step 1: Data Retrieval**
Use prefetch or fasterq-dump/fastq-dump to download SRA data.

# Create data directory
mkdir -p sra_data

# Download SRA file (already done)
prefetch ERR14218664

# Extract FASTQ (paired-end)
fastq-dump --outdir sra_data --gzip --skip-technical --readids --read-filter pass --dumpbase --split-files --clip sra_data/ERR14218664/ERR14218664.sra


**Step 2: Quality Control – FASTQC**
Run fastqc on raw reads

# Create output dir
mkdir -p fastqc_results

# Run FASTQC
fastqc sra_data/*.fastq.gz -o fastqc_results

**Step 3: Summarize Quality Reports – MULTIQC**
Aggregate FASTQC reports

# Create output dir
mkdir -p multiqc_report

# Run MultiQC
multiqc fastqc_results -o multiqc_report

# Open HTML Report (on Windows)
wslview multiqc_report/multiqc_report.html

**Step 4: Host Decontamination (Optional but important)**
Remove host (e.g., human) sequences using bowtie2

# Download and index reference genome
bowtie2-build GRCh38.fa human_index

# Align and remove human reads
bowtie2 -x human_index -1 sample_1.fastq.gz -2 sample_2.fastq.gz --very-sensitive -S aligned.sam --un-conc-gz clean_reads.fastq.gz

**Step 5: Taxonomic Classification**
Use Kraken2 or Kaiju for taxonomic classification:
# Kraken2 (example)
kraken2 --db /path/to/kraken2-db --threads 4 --paired clean_reads_1.fastq.gz clean_reads_2.fastq.gz --report kraken_report.txt --output kraken_output.txt

**Step 6: Functional Profiling**
Use HUMAnN3 to identify microbial pathways and gene families:

# HUMAnN3
humann --input merged_reads.fastq.gz --output humann_output --threads 4
You may need to concatenate reads for single input:
zcat clean_reads_1.fastq.gz clean_reads_2.fastq.gz > merged_reads.fastq

**Step 7: Assembly** (Optional)
**Assemble metagenomes using MEGAHIT:**
megahit -1 clean_reads_1.fastq.gz -2 clean_reads_2.fastq.gz -o assembly_output

**Step 8: Binning** (Optional)
Group contigs into bins using MetaBAT2:
metabat2 -i assembly_output/final.contigs.fa -o bins_dir/bin

**Step 9: Visualization**
Use Krona, Pavian, or R packages like phyloseq, MicrobiomeAnalyst.

ktImportTaxonomy kraken_output.txt -o taxonomy_report.html
wslview taxonomy_report.html

**Step 10: Statistical and Diversity Analysis**
Alpha and beta diversity, ordination (e.g., PCoA), etc., using:

QIIME 2 (if 16S)

R with phyloseq, vegan, DESeq2 for shotgun
