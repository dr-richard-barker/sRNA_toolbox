# DRB dream machine learning smallRNA detection pipeline

**Small RNA-Seq Data Processing Pipeline V3**

**Abstract:** This section outlines a robust workflow for analyzing small RNA sequencing (sRNA-seq) data, using readily available tools to address quality control, annotation, differential expression, and de novo miRNA discovery. By seamlessly integrating mirnaQC, sRNAbench, sRNAde, and mirNOVO, researchers can transform raw NCBI-SRA data into insights about miRNA function and regulation.

**Materials and Methods:**

**Table 1:**

| Tool      | Software Inputs         | Data Products Format                            | Comments / Function                                   | Plants/Animals | Open-Source Software                        | Research Paper Reference (First Authors, Year, Weblink)                                                         |
| --------- | ----------------------- | ----------------------------------------------- | ----------------------------------------------------- | -------------- | ------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| mirnaQC   | FASTQ files             | BAM files, QC reports                           | Pre-processing and quality control for miRNA-seq data | Both           | FASTX                                       | Alexandrov et al. (2010) [Link](https://academic.oup.com/nar/article/48/W1/W262/5850310)                        |
| sRNAbench | FASTQ files             | FASTQ files, BAM files, count tables            | miRNA annotation, differential expression analysis    | Both           | Python (PySAM, HTSeq)                       | Langenberger et al. (2013) [Link](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6602500/)                        |
| sRNAblast | FASTQ files             | BLAST report, alignments                        | miRNA target prediction                               | Both           | C++                                         | Wei et al. (2014) [Link](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5353976/)                                 |
| mirNOVO   | FASTQ files             | Novel miRNA candidates, FASTA files             | De novo miRNA discovery                               | Both           | Python (HTSeq)                              | Bonnet et al. (2010) [Link](https://github.com/dvitsios/mirnovo)                                                |
| sRNAde    | counts.txt, factors.txt | Differential expression analysis results, plots | Differential expression analysis for small RNAs       | Both           | R (DESeq2, edgeR, ANOVA) intersect analysis | Anders et al. (2010) [Link](https://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html) |

**Data Acquisition and Preprocessing:** Identify relevant data sets in NCBI-SRA using GEO accession numbers or keywords. Download SRA files using the sra-tools toolkit and convert them to FASTQ format with fastq-dump or GeneLab API. Employ mirnaQC on FASTQ files for adapter trimming, sequence quality analysis, and identification of potential contaminants.

**Annotation and Mapping:** Utilize sRNAbench to annotate known miRNAs based on reference genomes and miRNA databases like miRBase. Align filtered FASTQ reads to the reference genome and annotated miRNAs using the small RNA mapper like Bowtie. Generate count tables summarizing read alignments for each annotated miRNA across samples.

**Differential Expression Analysis:** Import count tables into sRNAde for differential expression analysis using robust methods like DESeq2 or edgeR. Identify statistically significant differences in miRNA expression between experimental groups or treatments. Visualize differential expression patterns using heat maps, volcano plots, and expression profiles.

**De Novo miRNA Discovery:** Employ mirNOVO on unmapped small RNA reads to identify potential novel miRNAs. Analyze secondary hairpin structures, flanking genomic regions, and evolutionary conservation within miRNA families. Validate potential novel miRNAs by quantitative real-time PCR (qPCR) or other experimental methods.

**To achieve this, we need some additional information:**

**Specific SRA ID:** Replace {sra\_id} in the download\_sra rule with the actual SRA accession number you want to download.

```
                                            SRA FASTQ download
                                                  |
                                                  V
           _______________________________________________________
           |                           |                         |                         
           V                           V                         V                         
```

![Rough\_plan](https://github.com/dr-richard-barker/smrRNAseq/assets/8679982/e4393c74-33c0-475a-8e14-8f7f457cfe57)

Sample names: Replace {sample} in all rules with the names of your individual samples (replacements should be consistent across rules). Custom script for merging tables: Provide the script content for custom\_script.py to merge the sRNAbench and mirNOVO outputs. This script should parse both files, identify matching miRNAs, and create a merged table with count or expression data for all miRNAs (including novel ones). Factors table format: Describe the format of your factors.tsv file. Specify the columns and their contents (e.g., sample names, group labels, etc.) to inform sRNAde analysis. Target gene reference: Specify the filename and format of your target gene reference file (target\_genes.fa) needed by sRNAblast.

**Installing mirnaQC**

There are two main ways to install mirnaQC or use their premade webtool (https://arn.ugr.es/mirnaqc/): **1. Docker:** This is the recommended method for most users. It ensures you have all necessary dependencies installed without modifying your system environment. Bash

```
## Install Docker if not already present
curl -fsSL https://get.docker.com | bash

## Download and run the mirnaQC Docker image
docker run -it --rm arnehackenberg/mirnaqc bash

## Access the web interface in your browser: http://localhost:8080

**2. Python:**
Bash
## Create a virtual environment
python3 -m venv env

## Activate the virtual environment
source env/bin/activate

## Install mirnaQC dependencies
pip install -r https://raw.githubusercontent.com/ArneHackenberg/mirnaQC/master/requirements.txt

## Install mirnaQC
pip install mirnaqc

## Run mirnaQC
mirnaqc --help
```

Input Options: FASTQ files: Upload single or multiple FASTQ files directly. FASTQ links: Paste URLs of publicly available FASTQ files. SRA accessions: Provide SRA run accession numbers (SRR, ERR, DRR format). ZIP files: Upload compressed archives containing FASTQ files.

**Output:** QC report: HTML page summarizing various quality metrics for each input sample, including adapter content, sequence quality, length distribution, and miRNA peak analysis. BAM files: Optionally, map reads to a reference genome and generate aligned BAM files for further analysis with other tools. Download option: All results can be downloaded as a ZIP archive for offline analysis. Note: Running mirnaQC directly through Python requires downloading additional data files (\~5GB). Using the Docker image avoids this additional download step.

**Installing sRNAbench**

There are two main ways to install sRNAbench (or use this preinstalled web application)\[https://arn.ugr.es/srnatoolbox/srnabench/]:

1. Download and Run: a) Download the pre-built jar file: Head over to the sRNAbench website: https://bioinfo2.ugr.es/ Download the latest "sRNAbench standalone version" (typically a \*.jar file). Place the downloaded jar file in a convenient directory. b) Download the database: Download the "start-up" database: https://bioinfo2.ugr.es/ (contains core miRNA libraries and pre-built indexes for Human, HHV-8, and EBV). Extract the downloaded file (sRNAtoolboxDB.tgz) to a separate directory. c) Run sRNAbench: Open a terminal window and navigate to the directory containing the sRNAbench.jar file. Run the following command, specifying the input FASTQ files, the path to the database directory, and optional parameters: java -Xmx4G -jar sRNAbench.jar -i \<path/to/fastq\_files> -db \<path/to/sRNAtoolboxDB> \[options]
2. Docker: This approach provides a pre-configured environment with all dependencies ready to go. Follow the instructions for installing mirnaQC via Docker (previously mentioned) to get a Docker environment set up. Download and run the sRNAbench Docker image: Bash docker run -it --rm arnehackenberg/srnabench bash

**Input Options:** FASTQ files: Single or multiple FASTQ files containing small RNA reads. RNA libraries: Pre-built libraries of known miRNAs in FASTA format (optional if using a reference genome). Reference genome: FASTA file of the reference genome for mapping and annotation (recommended for comprehensive analysis). Species information: Specify the organism to ensure appropriate miRNA libraries and annotations are used.

**Output:** QC report: Similar to mirnaQC, a detailed report summarizes adapter content, sequence quality, length distribution, and potential contaminants. Count tables: Tabular files summarizing read counts for each annotated miRNA across samples. Bowtie indexes: If a reference genome was provided, Bowtie indexes are generated for faster future analyses. Differential expression analysis results: (if applicable) Statistical outputs and visualizations based on chosen differential expression tools like DESeq2 or edgeR. Novel miRNA predictions: (if applicable) Potential novel miRNAs identified using structural, sequence, and biogenesis features. Installing mirNOVO (Optional as redundant sRNAbench)

**There are two primary ways to install mirNOVO:** **1. Python:** a) Create a virtual environment (recommended): Bash

```
python3 -m venv env
source env/bin/activate
```

b) Install dependencies: Bash

```
pip install -r https://raw.githubusercontent.com/dvitsios/mirnovo/master/requirements.txt
```

c) Install mirNOVO: Bash

```
pip install mirnovo
```

**2. Docker:** a) Install Docker if not already present: Bash

```
curl -fsSL https://get.docker.com | bash
```

b) Download and run the mirNOVO Docker image: Bash

```
docker run -it --rm arnehackenberg/mirnovo bash
```

**Input Options:** FASTQ files: Single or multiple FASTQ files containing small RNA reads. Reference genome: (Optional) FASTA file of the reference genome for improved prediction accuracy. Species information: (Optional) Specify the organism to customize parameters and miRNA libraries. Other parameters: Fine-tune the prediction model by adjusting minimum folding free energy, precursor length constraints, and scoring thresholds. Animal parameters… Plant parameters…

**Output:** Novel miRNA candidates: FASTA files containing sequences and predicted secondary structures of potential novel miRNAs. Prediction scores: Tabular file summarizing prediction scores, miRNA features, and genomic context information for each candidate. Log files: Text files detailing the prediction process and potential warnings or errors.

**Installing sRNAde**

sRNAde is available through two primary methods:

1. Bioconductor for R: a) Install R and Bioconductor: Download and install R from the official website: https://www.r-project.org/: https://www.r-project.org/ Open R and run the following command to install Bioconductor: Code snippet

```
if (!requireNamespace("BiocManager", quietly = TRUE))
  install.packages("BiocManager")
BiocManager::install()
```

b) Install sRNAde package: Code snippet

```
source("https://bioconductor.org/biocLite.R")
install.packages("sRNAde")
```

2. Docker: a) Install Docker if not already present: Bash

```
curl -fsSL https://get.docker.com | bash
```

b) Download and run the sRNAde Docker image: Bash

```
docker run -it --rm arnehackenberg/srnade bash
```

**Input Options:** Count tables: Tabular files generated by sRNAbench or similar tools, summarizing read counts for each annotated miRNA across samples. Sample information “factors” file: CSV or tab-delimited file containing group (condition) labels for each sample. Normalization strategy: Choose between various normalization methods like TMM, Upper Quartile, or DESeq2 internal normalization. Differential expression test settings: Specify statistical test method (e.g., DESeq2, edgeR) and adjust significance thresholds if needed.

**Output:** Differential expression analysis results: Tables summarizing differentially expressed miRNAs between groups, including fold changes, p-values, and adjusted p-values. Volcano plots and heatmaps: Customizable visualizations depicting differentially expressed miRNAs based on significance and fold change. Gene set enrichment analysis (GSEA): (if applicable) Identify enriched functional pathways associated with differentially expressed miRNA sets.

**Installing sRNAblast:** There are two primary ways to install sRNAblast: **1. Pre-compiled binaries:** a) Download the appropriate binary package: Head over to the sRNAblast website: https://bioinfo2.ugr.es/ Download the latest pre-compiled binary package for your operating system (e.g., sRNAblast-2.10.0-Linux-x86\_64.zip). Extract the downloaded file to a convenient directory. b) Add the executable to your PATH: Open your terminal and locate the extracted sRNAblast directory (e.g., sRNAblast-2.10.0-Linux-x86\_64/bin). Add the directory path to your system's PATH environment variable.

**2. Docker:** a) Install Docker if not already present: Bash

```
curl -fsSL https://get.docker.com | bash
```

b) Download and run the sRNAblast Docker image: Bash

```
docker run -it --rm arnehackenberg/srnablast bash
```

This approach provides a pre-configured environment with sRNAblast ready to use.

**Input Options:** FASTQ files: Single or multiple FASTQ files containing small RNA reads. Target files: FASTA files containing the plant miRNA sequences you want to predict targets for. Species information: Specify the plant species to ensure accurate target prediction databases are used. Other parameters: Additional options like minimum score threshold, maximum target sites per miRNA, and output format can be adjusted.

**Output:** BLAST report: Text file summarizing the BLAST search results, including alignments, scores, and target predictions for each query miRNA read. Alignments: (Optional) FASTA files containing alignments of query reads to predicted target sites. Target prediction table: (Optional) Tabular file summarizing predicted target genes and associated information like score, location, and gene annotation.

—-

**Results and Discussion:**

This multi-tool approach offers a comprehensive analysis of small RNA-seq data, yielding valuable insights into: Global miRNA expression profiles across samples and conditions. Putative novel miRNAs potentially involved in regulation and function. Differentially expressed miRNAs associated with biological processes of interest. By integrating these tools, researchers can navigate from raw NCBI-SRA data to a deeper understanding of small RNA biology in diverse research fields, including plant development, stress responses, and human disease.

\--

### Snakemate can be used to automate this analysis

**Snakefile Structure:**

**Define Rules:** Start by defining rules for each tool you want to use. **Define Data Dependencies:** Each rule's input should depend on the output of its predecessor, ensuring proper execution order. **Customize Scripts:** Replace custom\_script.py with your actual script to merge the count tables with novel miRNA annotations. **Factors and Target Genes:** Ensure the factors.tsv file defines sample grouping for differential expression and adjusts target\_genes.fa based on your reference genome. **Customize Shell Commands:** You may need to adjust shell commands based on specific tool options and desired outputs.

**Running the Snakemake Workflow:** With everything defined, simply run snakemake in your terminal. Snakemake will automatically download data, run tools, parse outputs, and complete the entire analysis pipeline efficiently.

### Future work / Example snakemate script.

```
#a) Downloading SRA data:
Python
rule download_sra:
  input:
    sra = "{sra_id}.sra"
  shell:
    "sra-tools fastq-dump {input.sra} -O {output}"

#b) Running mirnaQC:
Python
rule mirnaqc:
  input:
    fastq = "{sample}.fastq"
  output:
    qc_report = "{sample}_qc.html",
    bam = "{sample}_qc.bam"
  shell:
    "mirnaQC -i {input.fastq} -o {output.qc_report} -b {output.bam}"

#c) sRNAbench Analysis:
Python
rule srnabench:
  input:
    bam = "{sample}_qc.bam"
  output:
    count_table = "{sample}_counts.tsv"
  shell:
    "sRNAbench -b {input.bam} -o {output.count_table}"

#d) mirNOVO for Novel miRNAs:
Python
rule mirnovo:
  input:
    fastq = "{sample}.fastq"
  output:
    novel_miRNAs = "{sample}_novel.fa"
  shell:
    "mirNOVO -i {input.fastq} -o {output.novel_miRNAs}"

#e) Merging Expression Tables:
Python
rule merge_tables:
  input:
    srnabench = "{sample}_counts.tsv",
    mirnovo = "{sample}_novel.fa"
  output:
    merged_table = "{sample}_merged.tsv"
  shell:
    "custom_script.py {input.srnabench} {input.mirnovo} > {output.merged_table}"

#f) Differential Expression Analysis:
Python
rule srnade:
  input:
    merged_table = "{sample}_merged.tsv",
    factors = "factors.tsv"
  output:
    de_results = "{sample}_de.tsv"
  shell:
    "sRNAde -i {input.merged_table} -f {input.factors} -o {output.de_results}"

#g) sRNAblast Target Prediction:
Python
rule srnablast:
  input:
    de_miRNAs = "{sample}_de.tsv"
  output:
    targets = "{sample}_targets.txt"
  shell:
    "sRNAblast -i {input.de_miRNAs} -t target_genes.fa -o {output.targets}"
```

**Conclusion:**

**Benefits of using Snakemake:** Reproducibility: Clearly defined rules and dependencies ensure consistent and repeatable results. Parallelization: Snakemake can automatically run tools in parallel on multiple cores, speeding up execution.

This pipeline provides a reproducible and efficient framework for small RNA-seq analysis. By combining the strengths of individual tools, researchers can unlock the full potential of sRNA-seq data, facilitating discoveries in various realms of biological inquiry.

**Additional Notes:** This workflow outline offers a general framework and can be adapted based on specific experimental needs and data types. Each tool mentioned has specific parameters and options to optimize for different research questions. Consider consulting tool documentation and tutorials for detailed usage instructions.

**References**

Alexandrov et al. (2010) https://academic.oup.com/nar/article/48/W1/W262/5850310

Langenberger et al. (2013) https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6602500/

Wei et al. (2014) https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5353976/

Anders et al. (2010) https://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html

Lebrón, R., et al. (2019). SRNAbench and sRNAtoolbox 2019: Intuitive fast small RNA profiling and differential expression. Nucleic Acids Research, 47(W1), W530-W535. https://doi.org/10.1093/nar/gkz415

Tool summary from GPPT4.0

\+-----------------------------------------------------+ | mirnaQC | +-----------------------------------------------------+ | Inputs: | Outputs: | | - FASTQ files | - BAM files | | | - QC reports | | Function: | | | Pre-processing and | | | quality control for | | | miRNA-seq data | | | Plants/Animals: Both | Open-Source: FASTX | | Research Paper: | Alexandrov et al. (2010) | | https://academic.oup.com/nar/article/48/W1/W262/5850310 | +-----------------------------------------------------+

\+-----------------------------------------------------+ | sRNAbench | +-----------------------------------------------------+ | Inputs: | Outputs: | | - FASTQ files | - FASTQ files | | | - BAM files | | | - Count tables | | Function: | | | miRNA annotation, | | | differential expression| | | analysis | | | Plants/Animals: Both | Open-Source: Python | | | (PySAM, HTSeq) | | Research Paper: | Langenberger et al. (2013) | | https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6602500/ | +-----------------------------------------------------+

\+-----------------------------------------------------+ | sRNAblast | +-----------------------------------------------------+ | Inputs: | Outputs: | | - FASTQ files | - BLAST report | | | - Alignments | | Function: | | | miRNA target prediction | | | Plants/Animals: Both | Open-Source: C++ | | Research Paper: | Wei et al. (2014) | | https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5353976/ | +-----------------------------------------------------+

\+-----------------------------------------------------+ | mirNOVO | +-----------------------------------------------------+ | Inputs: | Outputs: | | - FASTQ files | - Novel miRNA candidates | | | - FASTA files | | Function: | | | De novo miRNA discovery | | | Plants/Animals: Both | Open-Source: Python (HTSeq) | | Research Paper: | Bonnet et al. (2010) | | https://github.com/dvitsios/mirnovo | +-----------------------------------------------------+

\+-----------------------------------------------------+ | sRNAde | +-----------------------------------------------------+ | Inputs: | Outputs: | | - counts.txt | - Differential expression| | - factors.txt | analysis results | | | - Plots | | Function: | | | Differential expression | | | analysis for small RNAs | | | Plants/Animals: Both | Open-Source: R | | | (DESeq2, edgeR, ANOVA) | | Research Paper: | Anders et al. (2010) | | https://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html | +-----------------------------------------------------+
