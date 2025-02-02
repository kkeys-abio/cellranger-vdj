# klkeys/cellranger-vdj: Usage

## Introduction

## Running the pipeline

The typical command for running the pipeline is as follows:

```bash
nextflow run klkeys/cellranger-vdj --input 'samplesheet.tsv' --genome 'GRCh38' -profile docker
```

This will launch the pipeline with the `docker` configuration profile. See below for more information about profiles.

Note that the pipeline will create the following files in your working directory:

```bash
work            # Directory containing the nextflow working files
results         # Finished results (configurable, see below)
.nextflow_log   # Log file from Nextflow
# Other nextflow hidden files, eg. history of pipeline runs and old logs.
```

### Updating the pipeline

When you run the above command, Nextflow automatically pulls the pipeline code from GitHub and caches it.
For subsequent pipeline runs, Nextflow will always use the cached version if available - even if the pipeline has been updated since.
To ensure that you're running the latest version of the pipeline, regularly update the cached version of the pipeline:

```bash
nextflow pull klkeys/cellranger-vdj
```

### Reproducibility

It is best practice to specify the pipeline version when running the pipeline on your data.
This ensures that a specific version of the pipeline code and software are used when you run your pipeline.
If you keep using the same tag, you'll be running the same version of the pipeline, even if there have been changes to the code since.

First, go to the [klkeys/cellranger-vdj releases page](https://github.com/klkeys/cellranger-vdj/releases) and find the latest version number - numeric only (eg. `1.3.1`). Then specify this when running the pipeline with `-r` (one hyphen) - eg. `-r 1.3.1`.

This version number will be logged in reports when you run the pipeline, so that you'll know what you used when you look back in the future.

### Input sample sheet

A sample sheet containing sample metadata and paths to the input fastq files is needed to run the pipeline and should be provided with the `--input` parameter.

Example sample sheet:

```tsv
sample	fastq_1	fastq_2
subsampled_sc5p_v2_hs_B_1k_b	s3://amplifybio-bioinformatics-genomic-references/test-data/cellranger-vdj/subsampled_sc5p_v2_hs_B_1k_b_fastqs/subsampled_sc5p_v2_hs_B_1k_b_S1_L001_R1_001.fastq.gz	s3://amplifybio-bioinformatics-genomic-references/test-data/cellranger-vdj/subsampled_sc5p_v2_hs_B_1k_b_fastqs/subsampled_sc5p_v2_hs_B_1k_b_S1_L001_R2_001.fastq.gz
subsampled_sc5p_v2_hs_B_1k_b	s3://amplifybio-bioinformatics-genomic-references/test-data/cellranger-vdj/subsampled_sc5p_v2_hs_B_1k_b_fastqs/subsampled_sc5p_v2_hs_B_1k_b_S1_L002_R1_001.fastq.gz	s3://amplifybio-bioinformatics-genomic-references/test-data/cellranger-vdj/subsampled_sc5p_v2_hs_B_1k_b_fastqs/subsampled_sc5p_v2_hs_B_1k_b_S1_L002_R2_001.fastq.gz
```

* `sample`: Sample ID. If samples were pooled in the same GEM, but sequenced in different lanes, then they will be processed together with `cellranger vdj`. For more info, see [here](https://support.10xgenomics.com/single-cell-vdj/software/pipelines/latest/what-is-cell-ranger).
* `fastq\_1`: Read 1 FASTQ for the current sample
* `fastq\_2`: Read 2 FASTQ for the current sample

### Genome reference data

Genome reference data can be provided in several ways.

#### Using 10x genomics pre-built references

The pipeline requires a V(D)J reference. 10X Genomics provides a prebuilt one [here](https://support.10xgenomics.com/single-cell-vdj/software/downloads/latest).
Download it and stash it in a place where the pipeline can find it, either local storage or cloud storage.
Point the pipeline to the saved reference with

```bash
    --vdj_reference path/to/reference
```

## Core Nextflow arguments

> **NB:** These options are part of Nextflow and use a _single_ hyphen `-`. In contrast, pipeline parameters use a double-hyphen `--`.

### `-profile`

Use this parameter to choose a configuration profile. Profiles can give configuration presets for different compute environments.

Several generic profiles are bundled with the pipeline which instruct the pipeline to use software packaged using different methods (Docker, Singularity, Podman, Shifter, Charliecloud, Conda) - see below.

> We highly recommend the use of Docker or Singularity containers for full pipeline reproducibility.
When this is not possible, Conda is also supported.
Note that `cellranger` is not supported on Conda; this pipeline _must_ use Docker or Singularity.

The pipeline also dynamically loads configurations from [https://github.com/nf-core/configs](https://github.com/nf-core/configs) when it runs, making multiple config profiles for various institutional clusters available at run time.
For more information and to see if your system is available in these configs please see the [nf-core/configs documentation](https://github.com/nf-core/configs#documentation).

Note that multiple profiles can be loaded, for example: `-profile test,docker` - the order of arguments is important!
They are loaded in sequence, so later profiles can overwrite earlier profiles.

If `-profile` is not specified, the pipeline will run locally and expect all software to be installed and available on the `PATH`.
Running Nextflow without a specified `-profile` is _not_ recommended.

* `docker`
  * A generic configuration profile to be used with [Docker](https://docker.com/)
  * Pulls software from Docker Hub: [`nfcore/cellranger`](https://hub.docker.com/r/nfcore/cellranger)
* `singularity`
  * A generic configuration profile to be used with [Singularity](https://sylabs.io/docs/)
  * Pulls software from Docker Hub: [`qbicpipelines/cellranger`](https://hub.docker.com/r/qbicpipelines/cellranger/)
* `podman`
  * A generic configuration profile to be used with [Podman](https://podman.io/)
  * Pulls software from Docker Hub:[`qbicpipelines/cellranger`](https://hub.docker.com/r/qbicpipelines/cellranger/)
* `shifter`
  * A generic configuration profile to be used with [Shifter](https://nersc.gitlab.io/development/shifter/how-to-use/)
  * Pulls software from Docker Hub: [`qbicpipelines/cellranger`](https://hub.docker.com/r/qbicpipelines/cellranger/)
* `charliecloud`
  * A generic configuration profile to be used with [Charliecloud](https://hpc.github.io/charliecloud/)
  * Pulls software from Docker Hub: [`qbicpipelines/cellranger`](https://hub.docker.com/r/qbicpipelines/cellranger/)
* `conda`
  * Please only use Conda as a last resort i.e. when it's not possible to run the pipeline with Docker, Singularity, Podman, Shifter or Charliecloud.
  * A generic configuration profile to be used with [Conda](https://conda.io/docs/)
  * Pulls most software from [Bioconda](https://bioconda.github.io/)
* `test`
  * A profile with a complete configuration for automated testing
  * Includes links to test data so needs no other parameters

### `-resume`

Specify this when restarting a pipeline. Nextflow will used cached results from any pipeline steps where the inputs are the same, continuing from where it got to previously.

You can also supply a run name to resume a specific run: `-resume [run-name]`. Use the `nextflow log` command to show previous run names.

### `-c`

Specify the path to a specific config file (this is a core Nextflow command). See the [nf-core website documentation](https://nf-co.re/usage/configuration) for more information.

#### Custom resource requests

Each step in the pipeline has a default set of requirements for number of CPUs, memory and time.
For most of the steps in the pipeline, if the job exits with an error code of `143` (exceeded requested resources) it will automatically resubmit with higher requests (2x original, then 3x original).
If it still fails after three times then the pipeline is stopped.

While these default requirements will hopefully work for most people with most data, you may find that you want to customise the compute resources that the pipeline requests.
You can do this by creating a custom config file.
For example, to give the workflow process `star` 32GB of memory, you could use the following config:

```nextflow
process {
  withName: star {
    memory = 32.GB
  }
}
```

To find the exact name of a process you wish to modify the compute resources, check the live-status of a nextflow run displayed on your terminal or check the nextflow error for a line like so: `Error executing process > 'bwa'`.
In this case the name to specify in the custom config file is `bwa`.

See the main [Nextflow documentation](https://www.nextflow.io/docs/latest/config.html) for more information.

If you will run `nf-core` pipelines regularly, then it may be wise to request that your custom config file is uploaded to the `nf-core/configs` git repository.
Before you do this please can you test that the config file works with your pipeline of choice using the `-c` parameter (see definition above).
You can then create a pull request to the `nf-core/configs` repository with the addition of your config file, associated documentation file (see examples in [`nf-core/configs/docs`](https://github.com/nf-core/configs/tree/master/docs)), and amending [`nfcore_custom.config`](https://github.com/nf-core/configs/blob/master/nfcore_custom.config) to include your custom profile.

If you have any questions or issues please send us a message on [Slack](https://nf-co.re/join/slack) on the [`#configs` channel](https://nfcore.slack.com/channels/configs).

### Running in the background

Nextflow handles job submissions and supervises the running jobs. The Nextflow process must run until the pipeline is finished.

The Nextflow `-bg` flag launches Nextflow in the background, detached from your terminal so that the workflow does not stop if you log out of your session.
The logs are saved to a file.

Alternatively, you can use `screen` / `tmux` or a similar tool to create a detached session which you can log back into at a later time.
Some HPC setups also allow you to run nextflow within a cluster job submitted your job scheduler (from where it submits more jobs).

#### Nextflow memory requirements

In some cases, the Nextflow Java virtual machines can start to request a large amount of memory.
We recommend adding the following line to your environment to limit this (typically in `~/.bashrc` or `~./bash_profile`):

```bash
NXF_OPTS='-Xms1g -Xmx4g'
```
