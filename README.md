
## ChIP-Seq Pipeline: README

### Overview
This pipeline processes ChIP-seq data to identify and analyze protein-DNA binding sites in Homo sapiens. The analysis focuses on HIF-2α binding in 786-O cells treated with DMSO and involves downloading data, quality control, alignment, duplicate removal, and peak calling using **MACS2**. Key genes visualized in the data include **VEGFA** and **EPO**.

### Prerequisites
Ensure the following modules are loaded on the HPC or your environment:
```bash
module load fastqc/0.12.1
module load bedtools/2.31.0
module load MACS2
module load trimmomatic/0.39
module load sra-toolkit/3.0.5
module load bwa/0.7.17
module load bowtie/2.5.1
module load samtools/1.17
```

### Steps

#### 1. Data Download
Download the raw data (FASTQ files) from the **SRA database** using the following commands:
```bash
fastq-dump --split-files SRR26965367
fastq-dump --split-files SRR26965368
fastq-dump --split-files SRR26965369
fastq-dump --split-files SRR26965370
```

#### 2. Quality Control (QC)
Run FastQC to assess the quality of the downloaded FASTQ files:
```bash
fastqc SRR26965367_1.fastq SRR26965367_2.fastq \
       SRR26965368_1.fastq SRR26965368_2.fastq \
       SRR26965369_1.fastq SRR26965369_2.fastq \
       SRR26965370_1.fastq SRR26965370_2.fastq
```
This will generate quality reports for all paired-end data files.

#### 3. Read Trimming
Trim low-quality reads and adapter sequences using **Trimmomatic**:
```bash
trimmomatic PE SRR26965367_1.fastq SRR26965367_2.fastq \
    SRR26965367_1_paired.fastq SRR26965367_1_unpaired.fastq \
    SRR26965367_2_paired.fastq SRR26965367_2_unpaired.fastq \
    SLIDINGWINDOW:4:20 MINLEN:36
```
Repeat for the other datasets (**SRR26965368**, **SRR26965369**, **SRR26965370**).

#### 4. Download and Prepare the Human Genome Reference
Download the **hg38** reference genome from UCSC:
```bash
curl -O http://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.fa.gz
```
Unzip the downloaded file:
```bash
gunzip hg38.fa.gz
```

#### 5. Index the Reference Genome
Use **Bowtie2** to index the reference genome for alignment:
```bash
bowtie2-build hg38.fa hg38
```

#### 6. Alignment of FASTQ Files
Align the trimmed FASTQ files against the hg38 reference using **Bowtie2**:
```bash
bowtie2 -x hg38 -1 SRR26965367_1.fastq -2 SRR26965367_2.fastq -S SRR26965367_aln.sam
```
Repeat for other datasets (**SRR26965368**, **SRR26965369**, **SRR26965370**).

#### 7. Convert SAM to BAM
Convert the aligned SAM files to BAM format using **Samtools**:
```bash
samtools view -S -b SRR26965367_aln.sam > SRR26965367_aln.bam
```
Repeat for all datasets.

#### 8. Sort BAM Files
Sort the BAM files for further processing:
```bash
samtools sort SRR26965367_aln.bam -o SRR26965367_sorted.bam
```
Repeat for other BAM files.

#### 9. Mark Duplicate Reads
Remove duplicate reads using **Picard**:
```bash
java -jar picard.jar MarkDuplicates I=SRR26965367_sorted.bam O=SRR26965367_dedup.bam M=SRR26965367_metrics.txt
```
Repeat for all sorted BAM files.

#### 10. Index BAM Files
Create index files for the deduplicated BAM files:
```bash
samtools index SRR26965367_dedup.bam
```
Repeat for all BAM files.

#### 11. Peak Calling with MACS2
Call peaks using **MACS2** for the ChIP-seq data:
```bash
macs2 callpeak -t SRR26965367_dedup.bam -f BAM -g hs -n SRR26965367 -q 0.01
```
Repeat for other datasets.

#### 12. Visualization in UCSC Genome Browser
The peaks data can now be visualized in the **UCSC Genome Browser** by uploading the peak files and exploring key genes such as **VEGFA** and **EPO**. You can also cross-reference the peaks with the genes of interest to determine binding sites.

### Key Genes
After running peak calling and visualizing the results, the following genes were found to have notable peaks:
- **VEGFA**
- **EPO**

These genes are involved in the cellular response to hypoxia, which aligns with the HIF-2α transcription factor's regulatory role.

### Final Notes
This pipeline processes ChIP-seq data to detect protein-DNA binding events, specifically identifying the interaction between HIF-2α and its target genes. The analysis highlights the regulatory role of nuclear speckles and gene expression control mechanisms in cancer. The pipeline leverages high-throughput sequencing technologies and bioinformatics tools to provide comprehensive insights into gene regulation.

