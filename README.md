# Herpesviruses mimic zygotic genome activation

This repository includes scripts used for the paper: 

```diff 
- Herpesviruses mimic zygotic genome activation to promote viral replication
```


<img width="595" alt="image" src="https://github.com/jiangtan01/Herpesviruses-mimic-zygotic-genome-activation-to-promote-viral-replication/assets/67500766/e82a2419-6969-4387-b935-7a7c92958f88">




Accompanying sequencing data is available in the GEO of this paper.



## overview

* Bulk RNA-Seq analysis
* CHIP-seq analysis
* CUT&Run analysis
* single cell RNA-seq analysis

## the installation of snakePipes2.7.2

for the mapping of Bulk RNA-Seq, CHIP-seq and single cell RNA-seq analysis, all of them did by the pipeline snakePipes. The version used 2.7.2 or 2.2.3

for the installation of snakePipes, you need to install either miniconda or anaconda first. More detail please check the introduction of snakePipes from MPI: https://snakepipes.readthedocs.io/en/stable/content/setting_up.html#installing-snakepipes

the snakePipes2.7.2 install code is: 
```diff
conda create -n snakePipes -c mpi-ie -c conda-forge -c bioconda snakePipes2.7.2
```

you can run it on a linux system of workstation, typicall install time may take several hours. The expected run time depends on the size of different input fastq file and different jobs. Typically need from several hours to several days. 
  
## example for the usage of snakePipes

Bulk RNA-Seq analysis of PAA treatment upon HSV-1 infection at 8phi, data from GEO: GSM5608614. The expected output are the file of bam, bigwig and DE gene list.

```diff
cd /path/to/data/ && conda activate snakepipes2.7.2

mRNA-seq -i /path/to/data/GSM5608614 -o mRNA.all.snakepipes.file.out --sampleSheet sampleInfo.8hpi.tsv -j 20 hg38_HSV1_GFP_1107
```
CHIP-seq analysis of histone marker H3K27ac upon HSV-1 infection,	data from GEO: GSE124803. The expected output are the file of bam, bigwig and bed.
```diff
cd /path/to/data/ && conda activate snakepipes2.2.3

DNA-mapping -i /path/to/data/GSE124803 -o /path/to/snakepipes.out -j 20 --DAG --fastqc --dedup --properPairs --mapq 3 --bwBinSize 25 hg38

ChIP-seq -d /path/to/snakepipes.out -j 20 hg38 /path/to/chip_config.yaml
```  
single cell RNA-seq analysis for  Merkel cell carcinoma tumor patient, data from GEO: GSE226438. The expected output are the file of bam and matrix.
```diff
cd /path/to/data/ && conda activate snakePipes/2.7.2

scRNAseq -j 20 -i /path/to/data/GSE226438 -o snakepipes.out --myKit 10Xv3 --STARsoloCoords 1,16,17,12 hg38_MCV  
```







