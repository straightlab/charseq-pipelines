# ChAR-seq pipelines
This repository contains pipeline templates to process ChAR-seq raw sequencing reads. It also contains some notebooks describing how to prepare the ncessary files for pipeline (In progress).

Currently Available pipelines in [the pipeline folder](pipelines/)
- Paired End reads ("standard" pipeline)

Available notebooks (in progress)
- [Pipeline essentials](charseq_pipeline_essentials.pdf) : explains the key outputs of the pipeline

## Dependencies
- Standard bioformatics tools:
  - `samtools` (https://www.htslib.org)
  - `bedtools` (https://bedtools.readthedocs.io/en/latest/)
  - `picardtools` (https://broadinstitute.github.io/picard/)
  - `Bowtie2` :  the aligner used for aligning the DNA side of the reads (http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml)
  - `STAR` : the aligner used to align the RNA side of the reads (https://github.com/alexdobin/STAR)
  - `Salmon` : transcript abundance quantifer used for probabilistic annotations of reads mapping to multiple isoforms (https://salmon.readthedocs.io/en/latest/salmon.html)

- Other bioinformatics tools:
  - `pairix` : used to index and query the final pairs file (https://github.com/4dn-dcic/pairix)
  - `clumpify.sh` : from the BBmap suite, used to remove duplicates (https://sourceforge.net/projects/bbmap/)
  - `trimmomatic` : adapters trimming (http://www.usadellab.org/cms/?page=trimmomatic)
  - `pear` : to merge read1 and read2 from paired-end sequencing (https://cme.h-its.org/exelixis/web/software/pear/doc.html)
  - `snakemake` : our workflow management system (https://snakemake.readthedocs.io/en/stable/)
 
- Programming languages:
  - `julia 1.6 or 1.7` : the code to split the reads is written in Julia. Refer to the installation of chartools to install Julia and the required packahes.
  - `python >=v3.6` with the following packages: pandas, numpy, pysam, pyarrow, snakemake

- ChAR-seq specific packages:
  - `tagtools` : Custom package to match genomic annotations ("tags") to reads in a multimappers-aware way. Follow installation instructions at https://github.com/straightlab/tagtools
  - `chartools` : Custom package to manipulate and analyze ChAR-seq data. This is a work in progress that will eventually be cleanly packaged up and documented. A few functions from this package are necessary for pairs file generation. Follow installation instructions at https://github.com/straightlab/chartools-dev

We recommand installing all python related packages, including `tagtools` and `chartools` in a conda environment. For example, create a `charseq` environment with 
```bash
conda create -n charseq python=3.6
```

Then activate this environment and intall the other packages:
```bash
conda activate charseq
conda install pandas numpy pysam pyarrow snakemake
# then the pip instructions to install tagtools and chartools
```

Note that a lot of the bioinformatics tools above can also be installed through anaconda. In particular, pairix, bbmap, pear can be installed from the bioconda channel
```bash
conda install -c bioconda pairix bbmap
```

Bioconda channels may need to set up following instructions at `https://bioconda.github.io/user/install.html` (section 2, set up channels)



## How to run a pipeline
The pipelines are written in Snakemake and each pipeline consists in 3 files as described below. Simply copy these files to the desired run directory, edit the yaml configuration file and sample definition file appropiately, then simply execute snakemake

```bash
snakemake -pr -s pipeline.smk --configfile pipeline_config.yaml
```

Typically, the relevant python packages listed above (including snakemake) are installed in a conda environment, so activate the enviroment before running snakemake. 

The pipeline will generate output files into `./data`

The 3 files used by the pipeline are: 
- a snakemake file, `pipeline.smk`, which implements the pipeline steps. This file is portable across systems and do not need to be modified. There are no hardcoded sytems specific, sample specific, or tools configuration related parameters in this file.
- a yaml configuration file `pipeline_config.yaml`, which contains parameters for the various steps of the pipeline, such as configuration of the trimmer, aligners, read length cutoff, etc... Most parameters can be left unchanged but the paths to a few necessary resource files (such bowtie2 index, annotation files, etc..) are hardcoded and need to set up once. 
- a sample definition file `samples_def.yaml`, which contains the list of samples to process with for each : the path to fastq files, the sequence of the bridge (which can be different for each sample), and the path to the adapters definition fasta file used for adapter trimming.  

Important notes:
- One should always execute the pipeline in dry-run mode first, using the `-n` snakemake option!
- If the pipeline is ran on a cluster, it is ok to do dry runs on a login node. Of course however, once ready to fire up the pipeline, do not do so on a login node. Instead, either create a sbatch script or directly launch snakemake on an allocated compute node. 

## Preparation of annotation files
In order to run, the pipeline requires first creating annotation files for the genome of interest. These are used by tagtools to annote the reads with transcripts and gene names. The starting point to create these annotation files is a gff3 file for the genome of interest. Refer to xenopus_laevis example in the [notebooks](notebooks/xenopus_laevis) folder 

## Pipeline structures
### Basic pipeline
The schematic below shows the steps of the pipeline for paired-end sequencing reads.
![dag](images/pipeline_flow_PE.png)

