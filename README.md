Herpesviruses mimic zygotic genome activation to promote viral replication

This repository includes workflows and scripts used for the paper: Herpesviruses mimic zygotic genome activation to promote viral replication

Accompanying sequencing data is available in GEO: GSE183764.

The analysis can be reran by recreating the specific environments and genome references. To do this make sure that conda is installed. Additionaly, mamba is required as well (in your base environment).

overview
confs - conda environments / config files to rerun analysis
Escapees - featureplots for escape genes
lfs - genome reference specific files
samplesheets - samplesheets and yaml files used in the downstream analyses.
scripts - collection of snakemake & R scripts.
prepare references and download custom files
A couple of files need to be downloaded:

SNP information (SNPdb)
GRCm38 genome fasta (genome)
Place these in a directory and fill in the config file confs/prepgenomes_config.yml appropriately (with absolute paths). Make sure to unpack and place genes.gtf.gz (in this repository under lfs/genes.gtf.gz) in the directory and the yaml file as well.

Next, create the prepgenomes environment and activate it:

mamba env create -f confs/prepgenomes.yml
mamba activate prepgenomes

The references can now be created with snakemake:

snakemake -s scripts/prepgenomes.smk -d prepgenomes_outputdirectory --configfile confs/prepgenomes_config.yml --cores 25

This will create the N-masked indices for both hybrid strains, as well as STAR & Bowtie2 indices.

snakePipes
Nearly all data is processed with a custom version of snakePipes.

mamba env create -f confs/snakepipes.yml

And the snakepipes version itself can be installed afterwards:

mamba activate snakepipesAS
pip install git+https://github.com/Akhtar-Lab-MPI-IE/snakepipes_allele_specific@allele_specific

The source code for the snakePipes version is available here.

After creating the conda environment, the environments, specific reference and cluster config need to be set up:

conda activate snakepipesAS

Create the snakePipes environments (optionally in a specific conda directory):

snakePipes createEnvs --condaDir /path/to/condaenv/directory --only CONDA_SHARED_ENV CONDA_CREATE_INDEX_ENV CONDA_RNASEQ_ENV CONDA_DNA_MAPPING_ENV CONDA_CHIPSEQ_ENV CONDA_ATAC_ENV CONDA_RMD_ENV CONDA_PREPROCESSING_ENV CONDA_SAMBAMBA_ENV CONDA_pysam_ENV

Create the reference:

createIndices -o references/mm10_as --gtfURL https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M9/gencode.vM9.annotation.gtf.gz --genomeURL ftp://ftp.ensembl.org/pub/release-78/fasta/mus_musculus/dna/Mus_musculus.GRCm38.dna_sm.primary_assembly.fa.gz --rmskURL https://github.com/Akhtar-Lab-MPI-IE/MSL2_ensures_biallelic_gene_expression/raw/main/lfs/rmsk.txt.gz --blacklist https://github.com/Akhtar-Lab-MPI-IE/MSL2_ensures_biallelic_gene_expression/raw/main/lfs/rar.bed --ignoreForNormalization lfs/ignore.txt mm10_as

Modify the cluster.yaml file appropriately (snakemake_cluster_cmd especially). The path to this file can be found with this command:

snakePipes info

Running the dataset specific workflows is explained in the snakePipes markdown. Note that there are two post-processing scripts included (allele_specific.degenes.category.R & allele_specific.degenes.cleanup.R), these require their own environment:

mamba env create -f confs/degenes.yml
mamba activate degenes

BSseq
The BSseq samples were processed outside of snakePipes, and have their own environment:

mamba env create -f confs/bss.yml mamba activate bssAS

Next, fill in the confs/bss_config.yaml file appropriately. The Nmaskpath is generated using the prepgenomes function from before, and is either the:

BlCa/CAST_EiJ_N-masked (BlCa/CaBl)
9sCa/129S1_SvImJ_CAST_EiJ_dual_hybrid.based_on_GRCm38_N-masked (9sCa)
If everything is prepared, run the dataset as followed:

snakemake -s scripts/bss.smk --configfile confs/bss.yml --cores 25 -d bsseq_outputdirectory

The differential methylation calling is exemplified in the Rscript and uses the CpG reports generated by the snakemake workflow.

Multiome
For the multiome runs (scRNA + scATAC combined), cellranger arc was used, with reference mm10-2020-A-2.0.0. To run allele-specific analysis for this data, reads were phased prior to cellranger arc runs. This can be reproduced with the by activating the prepgenomes environment again (see above):

mamba activate prepgenomes

and fill in the confs/phasemul_config.yml file accordingly:

fqdir_rna: absolute path to directory containing scRNA files
fqdir_atac: absolute path to directory containing scATAC files
bt_ix: bowtie2 index (including 'genome' postfix), created by prepgenomes
star_ix: star index folder, created by prepgenomes
snpfile: SNPfile.txt, created by prepgenomes
splitfq: absolute path to scripts/splitFQ.py (included in this repository)
The fastqfiles can then be split per genome by calling:

snakemake -s scripts/phase_multiome.smk --configfile confs/phasemul_config.yml --cores 10 -d phasemul_outputdirectory

The outputted files are properly named to use with cellranger-arc directly.