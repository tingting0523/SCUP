SCUP-pipeline
--------------------------------
Version: 1.0.0
Updated date: 2026.11.30

Dependencies
--------------------------------
upload simulate data here
HiSAT2 version: 2.2.1
Samtools version: 1.19.2
GATK version: 4.3.0.0
LiRA version: 2022(3bc0ael)
SCcaller version: 2.0.0
Whatshap version: 1.7
Python version: 3
Bash version: 5.2.21
SCSsim version: 1.0

Author
--------------------------------
Author: Tingting Chen
Email: chent2@southernct.edu


Pipelines
--------------------------------
SCUP (Single-Cell variant calling using Union and Phasing): A end-to-end pipeline for DNA single-cell single-cell variant detection: from fastq files to variants results. SCUP achieves a better balance between false positives (FPs) and false negatives (FNs) in single-cell variant detection.

Current SCUP-pipeline includes 5 component. See the flowchart, and usage below.
<img width="631" height="301" alt="image" src="https://github.com/user-attachments/assets/1e624c82-e886-4cc3-af63-9b327d4af80f" />

1. Data input and preprocessing: The pipeline requires a FASTA format reference genome(hg38) and FASTQ format DNA sequencing data(include single-cell sequencing data and bulk sequencing data) as input. During preprocessing, sequencing reads from the FASTQ files are aligned to the hg38 reference genome using HISAT2. The resulting alignments are stored in BAM format, which contains the genomic positions and alignment information of the sequencing reads. These BAM files are then used as input for the subsequent variant calling component. Following alignment, gSNVs are identified using the Genome Analysis Toolkit(GATK). GATK analyzes the aligned reads stored in BAM files and performs variant calling to identify genomic positions that differ from the reference genome.  
2. Variant Calling: SCcaller and LiRA were used to generate candidate variants from aligned sequencing reads and gSNVs which generated during the preprocessing stage. Each tool generated a separate VCF file for each cell. A filtering step is applied to the SCcaller VCF files to retain records with a dot (.) in the FILTER column. Records flagged in the FILTER column, including those with multiple alternative alleles caused by technical artifacts or fewer than four variant-supporting reads, are removed to eliminate low-quality variant calls and improve the accuracy of downstream analysis. Additionally, records with a homozygous reference genotype (0/0) are excluded. The remaining SCcaller candidates are then combined with LiRA candidates using a union-based strategy, retaining variants identified by either tool to improve recall.
3. Haplotype-based phasing: WhatsHap was applied independently to each single-cell sample and the bulk sample using the reference genome hg38, corresponding BAM files and candidate variants. The output is a VCF file for each sample with phased genotype and phase set (PS) information. The output is a VCF file for each sample with phased genotype and phase set (PS) information. The phased results from all samples were subsequently integrated, and related phase information across samples was merged to create new phase blocks. Candidate variants were then evaluated within these new phase blocks according to the haplotype-based filtering criteria and classified as either kept or filtered based on their haplotype consistency or not.
4. Joint filtering: variant was further jointly evaluated across multiple cells and variant calling tools when the haplotype inconsistency.
5. Output: a standard VCF file, with the mark and reason fields incorporated into the INFO column, producing the final output of the proposed pipeline.


Usage
--------------------------------
1. Data input and preprocessing
```bash
# 1: Mapping: please input the sample_number(node3cell1) as the parameter here
./createBam.sh node3cell1

# 2: GATK Calling
./gatk_cell.sh  
./gatk_bulk.log   
./gatk_joint.sh
```

2. Variant Calling
```bash
# SCcaller: please change the sample_number(node3cell1) as the parameter here
./runSccaller_parallel.bash node3cell1
awk 'BEGIN{FS="\t"} !/^#/ && $7=="." && $10 !~ /^0\/0/ {print}' node3cell1.vcfs > node3cell1.filter.vcf    # initial filter

# LiRA
lira setup --config config-bulk-SRR2976561.txt
lira split --config config-bulk-SRR2976561.txt --overwrite --parallel
lira setup --config config-SRR2976561.txt
lira split --config config-SRR2976561.txt --overwrite

```

3. Haplotype-based phasing
```bash
# Step 1: Run variant calling
python3 variant_calling.py

# Step 2: Evaluate results
python3 evaluate.py
```

4. Joint filtering
```bash
# Step 1: Run variant calling
python3 variant_calling.py

# Step 2: Evaluate results
python3 evaluate.py
```

5. Output



Release Notes
--------------------------------
V1.0.0, 2026.11.30, 1st release version
