## PCAR: a computational <u>P</u>ipeline for calling <u>C</u>HM and scoring the <u>A</u>llele-specific <u>R</u>egulatory roles

![label1](https://img.shields.io/badge/version-v1.1.1-yellow)	![label2](https://img.shields.io/badge/license-MIT-green)

**| [Overview](#overview) | [Installation](#installation) | [Usage](#usage)  | [Module: callchm](#module-callchm) | [Module: scoreasr](#module-scoreasr) | [Test data and examples of operation](#demo) |**
![Figure1](Schema.png?⁩raw=True "Title")
## Overview

PCAR was developed for calling CpG-rich genomic loci with high H3K9me3 signal and DNA methylation level (CHM) and scoring the allele-specific regulatory foles based on the features of known imprinting control regions (ICRs). Please contact Hui Yang(1810550@tongji.edu.cn) if you have any questions or suggestions.

## Installation
- System requirements

  This pipeline has been tested on *Linux* operating systems:
  - Linux: Ubuntu 16.04
  - RAM: 380 GB
  - CPU: 40 cores, 2.40 GHz/core
  
- Software used

	- [ChromHMM (v1.22)](http://compbio.mit.edu/ChromHMM) A software for learning and characterization chromatin states.
    - python (3.8.5) 
    - twobitreader A package for reading .2bit files. It can be installed by:
    ```shell
    $ conda install twobitreader -c bioconda
    ```
    - bedtools (v2.27.1)

- Download and unzip package from Github, then add "+x" to scripts to make it executable.
 ```shell
 $ unzip PCAR-main.zip
 $ cd PCAR-main
 $ chmod +x pcar callchm scoreasr pcar_getCpGnumberMultipleTs.py pcar_averageMethylInRegionMultipleThreads.sh methyl_processing.py
 ```
 
- Export path of PCAR suite to environment
```shell
# Get directory of PCAR scripts
$ pcarPATH=$(pwd)

# Add PCAR to ./bashrc file of your environment
$ echo "export ${pcarPATH}:\${PATH}" >> ~/.bashrc

# Or manually export to environment variable before running PCAR
$ export PATH=${pcarPATH}:${PATH}

# Test whether PCAR could be found
$ which pcar
"/YOUR_PATH_TO_PCAR/PCAR-main/pcar"
```
The installation takes approximately 30 seconds based on system above.

## Usage

```shell
$ pcar -m <callchm|scoreasr> [options]
        -h, --help -- help information
```

## Module *callchm* : CHM calling based on DNA methylation and H3K9me3


```shell
$ pcar -m callchm -H H3K9me3.rmDup.bam -M methyl.sam.G.bed -Z genomesize -Q genomesequence [options]

Options:
-H, --h3k9me3 HFILE     H3K9me3 ChIP-seq sequence alighment after removing duplicates. REQUIRED.
-C, --control CFILE	Input ChIP-seq sequence alignment after removing duplicates. Defalt: "".
-M, --methyl MFILE      DNA methylation level of CpG sites estimated using mcall. REQUIRED.
-Z, --gsiz ZFILE        Two-column file: <chromosome name><tab><size in bases> downloaded from UCSC. REQUIRED.
-Q, --gseq QFILE        Genome sequence file in twoBit format downloaded from UCSC. REQUIRED.
-G, --gver GVER         Genome build version. Default: mm10.
-B, --binsize <int>     The number of base pair in a bin determining the resolution of the model learning and segmentation. Default: 200 base pairs.
-P, --poissonthresh <float>	This option specifies the tail probability of the poisson distribution that the binarization threshold should correspond to. Default: 0.0001.
-N, --name NAME         Name will be used to generate file names. Default: NA.
-T, --threads <int>     Number of threads to use. Default: 1.
-O, --outdir OUTDIR     If specified all output files will be written to that directory. Default: the currenting working directory.
-D, --definition <CHM|CH-nonM|CM-nonH>		Choosing the definition of region to be called. Default: CHM.
```

## Module *scoreasr* : scoring allele-specific regulatory potential

```shell
$ pcar -m scoreasr [options]

Options:
-I, --asepi ASEpiFile   File containing DNA methylation level and H3K9me3 in parents. Default: Epi.txt in current working directory.
-R, --asexpr ASExprDir  Directory containing expression data. Default: current working directory.
-U, --regu ReguDir      Directory containing regulation data. Default: current working directory.
-N, --name Name         Name will be used to generate file names. Default: NA.
-O, --outdir OutDir     If specified all output files will be written to that directory. 
```

## Demo
We provided two small [test datasets](https://github.com/hyang-bio/PCAR/tree/main/Test) (mouse in mm10) for users to test PCAR: [Test_callchm](https://github.com/hyang-bio/PCAR/tree/main/Test/Test_callchm) containing data in mouse chr19 for calling CHM, while [Test_scoreasr](https://github.com/hyang-bio/PCAR/tree/main/Test/Test_callchm/Test_scoreasr) containing data needed for scoring the Allele-specific Regulatory roles. 

### Running callchm module
```shell
$ pcar -m callchm -H Test/Test_callchm/Test.H3K9me3.rmDup.bam -M Test/Test_callchm/Test.sam.G.bed -Z Test/Test_callchm/Test.chrom.sizes -Q Test/Test_callchm/Test.2bit -T 6 -N Test
```
The CHMs identified are saved in the Test.CHM.bed. *callchm* based on *[Test_callchm](https://github.com/hyang-bio/PCAR/tree/main/Test/Test_callchm)* takes approximately 2 minutes based on system above.

### Running scoreasr module
##### File preparation
* Epi.txt containing DNA methylation level and H3K9me3 in parents. Format is tab-separated as follows: 

| #Chrom | Start |End | M_mat_stage<sub>1</sub> | M_pat_stage<sub>1</sub> | H_mat_stage<sub>1</sub> | H_pat_stage<sub>1</sub> | M_mat_stage<sub>2</sub> | M_pat_stage<sub>2</sub> | H_mat_stage<sub>2</sub> | H_pat_stage<sub>2</sub> | ... |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| chr1 | 3878000 | 3878600 | 0.64 | 0.70 | 0.21 | 0.20 | 0.63 | 0.33 | 0.32 | 0.37 | ... |
| chr1 | 4921800 | 4922400 | 0.36 | 0.98 | 0.34 | 0.28 | 0.71 | 0.92 | 0.30 | 0.35 | ... |
| chr1 | 5040400 | 5041000 | 0.18 | 0.73 | 0.13 | 0.14 | 0.46 | 0.52 | 0.07 | 0.77 | ... |
| chr1 | 5047000 | 5047600 | 0.49 | 0.81 | 0.08 | 0.27 | 0.13 | 0.42 | 0.03 | 0.28 | ... |
| chr1 | 5071800 | 5073000 | 0.19 | 0.70 | 0.12 | 0.01 | 0.02 | 0.74 | 0.16 | 0.88 | ... |

* Expr_genes.promoter.txt, Expr_transposableElements.promoter.txt containing expression information in log<sub>2</sub> (FPKM+1) in at least 2 stages. .Format is tab-separated as follows:

| #Chrom_promoter | Start_promoter | End_promoter | GeneName | E_mat_stage<sub>1</sub> | E_pat_stage<sub>1</sub> | E_mat_stage<sub>2</sub> | E_pat_stage<sub>2</sub> |  ... |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| chr1 | 3669498 | 3673498 | NM_001011874 | 0.00 | 0.00 | 0.00 | 0.01 | ... |
| chr1 | 4358303 | 4362303 | NM_001370921 | 0.13 | 0.47 | 0.00 | 0.00 | ... |
| chr1 | 4407241 | 4411241 | NM_001195662 | 0.00 | 0.00 | 0.00 | 0.00 | ... |
| chr1 | 4358314 | 4362314 | NM_011283 | 0.19 | 0.00 | 0.00 | 0.01 | ... |
| chr1 | 4495354 | 4499354 | NM_001289464 | 0.00 | 0.00 | 0.00 | 0.00 | ... |
| chr1 | 4495354 | 4499354 | NM_001289465 | 0.00 | 0.00 | 0.00 | 0.00 | ... |

Note: Transposable element transcripts annotation are assembled in pre-implantation embryos as [Shao et al](https://github.com/wanqingshao/TE_expression_in_scRNAseq) and stored in [mm10_te_tx.gff3](https://github.com/hyang-bio/PCAR/tree/main/Test/Test_scoreasr/mm10_te_tx.gff3). 

* Zfp57motif.methyl.txt containing DNA methylation information of Zfp57 motifs. Format is tab-separated as follows:

| #Chrom_motif | Start_motif | End_motif | M_mat_stage<sub>1</sub> | M_pat_stage<sub>1</sub> | M_mat_stage<sub>2</sub> | M_pat_stage<sub>2</sub> | ... |
| --- | --- | --- | --- | --- | --- | --- | --- |
| chr1 | 10805313 | 10805319 | NA | 0.01 | 1.00 | 0.00 |
| chr1 | 35898368 | 35898374 | 0.50 | 0.89 | 0.29 | 0.78 |
| chr1 | 37875379 | 37875385 | 1.00 | 0.03 | 1.00 | 0.00 |
| chr1 | 60901305 | 60901311 | NA | 0.06 | 0.04 | 1.00 |
| chr1 | 63200202 | 63200208 | 1.00 | 0.00 | NA | 0.61 |
| chr1 | 63200221 | 63200227 | 1.00 | 0.00 | NA | 0.58 |
 
* KnownIG.promoter.bed, LncRNA.bed, Ctcf.bed: Annotation of known imprinted genes' promoters, lncRNA, Ctcf binding sites in bed format.

```shell
$ pcar -m scoreasr -I Test/Test_scoreasr/Epi.txt -R Test/Test_scoreasr -U Test/Test_scoreasr -N Test
```

The final score and details for allele-specific regulatory roles are saved in Test.score.txt and associated allele-specific expressed genes and transposable elements could be searched in Test.score_asexpr_genes.txt and Test.score_asexpr_transposableElements.txt respectively. *scoreasr* based on *[Test_scoreasr](https://github.com/hyang-bio/PCAR/tree/main/Test/Test_callchm/Test_scoreasr)* takes approximately 8 minutes based on system above.
