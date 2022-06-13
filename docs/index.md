# Instructions for Bioinformaticians using the Lund Stem Cell Center LS2 compute platform.

Congratulations, you have been deemed worthy enough to be granted access to LS2.

In order to keep the system orderly we have set out some guidelines which everyone should follow to maintain a smoothly running platform. Please read this document *carefully* before you use the system.

## Getting access to LS2

LS2 is a secure system where we handle human sequence data under GDPR rules, therefore there are a few steps involved to get access.

The first thing you need to do is make an account at [SUPR](https://supr.snic.se/). Once done, contact [Shamit Soneji](shamit.soneji@med.lu.se) stating which SCC lab you belong to. You will then be added to the list of users, after which, you will log back in to SUPR and apply for an account at LUNARC.

They will send you a user agreement that you will need to print, sign, scan, and then send a PDF of it to Shamit for counter-signing. Someone from LUNARC will then contact you with your password.

In the meantime you need to contact [LDC](servicedesk@lu.se) and get a fixed IP address for your network port and VPN. When you have this, email it to Shamit for communicaton to LUNARC.

LUNARC also uses two-factor authentication using Pocket Pass. Follow the instructions [here](https://lunarc-documentation.readthedocs.io/en/latest/authenticator_howto/) to set this up. 

## Connecting to Aurora

The server address is aurora-ls2.lunarc.lu.se. You can connect using a terminal:

```
ssh <username>@aurora-ls2.lunarc.lu.se
```
It will ask for your password, and then your two-factor code from your Pocket Pass.

 You can also connect using Thinlinc which gives you a full Linux desktop. Follow the instuctions [here](https://lunarc-documentation.readthedocs.io/en/latest/using_hpc_desktop/) to use that. 

## Data - where to put my data/scripts/results?

LS2 has four primary drives fs1, fs3, fs5 and fs7 found at:


/projects/fs1/\<username\>

/projects/fs3/\<username\>

/projects/fs5/\<username\>

/projects/fs7/\<username\>


Each of these drives are mirrored nightly to a unit of the same size. No snapshots are stored, this is merely to protect against mechanical failure.

These drives should be used in a certain way, and this is how:

1. **Raw data** - this (and ONLY this) should go to /projects/**fs5**/\<username\>
    Raw data are e.g. fastq files (gzipped!!) or raw read folders.

2. **Processed data** (BAMS, counts etc) should be directed to either **fs1** or **fs3** depending on which one you were assigned to when given access. Intermediate files such as SAM files etc should be removed to save space.

3. Raw data that has been published should be *moved* to **fs7** for achive.

4. Scripts should be stored on fs1/fs3.


<That is not much -right? So where will all the intermediate data files end up?>

<Files that are reproducible (as your scripts do no random sampling or something like that) or even temporary should
be put on a not backed up disk. Ideally you should save them on the calculation nodes in a folder on the node.>
The slurm system is set up to give you a temporary folder on the odes for each slurm job you start. The name of your temporary folder is stored in the $SNIC_TMP [environment variable](https://linuxize.com/post/how-to-set-and-list-environment-variables-in-linux/). This variable does not exist on the frontend. The folder on the node is private to you and will be deleted once the job is finished. So in the last step of your script you need to save your results.

<Even these results should not be backed up (if they can be reproduced - try it if you are unsure). Hence they should also be stored on a not backed up nas.>

I strongly recommend [to create soft links](https://www.cyberciti.biz/faq/creating-soft-link-or-symbolic-link/) to the main data folder that you use. I for example have two links - one for the main data storage accessible under "\~/NAS/" and one for the primary data storage under "\~/DATA":
```
ln -s /projects/fs1/stefanl ~/NAS
ln -s /projects/fs5/stefanl/ ~/DATA
```
This way you keep quite flexible in changing the NAS mount points in the future.
Do not overdo this! You need to know where all your data is as you are responsible - OK?

At the moment my main data analysis 'place' is "\~/NAS" and therefore "/projects/fs1". This is backed up, hence the separation between scripts and results has not been applied even to my workspace. But please check - the "/projects/fs5" is actually only meant for RAW data - raw sequencing output or fastq.gz files or whichever other raw data you are analyzing.


## Make scripts reproducible

The easiest way to achieve that is letting someone take care of your analysis software.
R and python packages are updating rather frequently and updated packages on the one hand might change the results
and on the other hand are really hard to install without an internet connection.

[Singularity images](https://sylabs.io/guides/3.6/user-guide/quick_start.html) are the way to go.
Singularity version 3.6 is installed on aurora-ls2. If you have an immediate need for a package you might think about starting your own singularity image and upload it to aurora-ls2. But before you do this check out the folder "/projects/fs1/common/singularityImages/" there might already exist an image that contains the software you are looking for.

I recommend reading [this pdf](pdfs/HowToUseSingularityOnLsens2.pdf) about how to use singularity images and especially how to use my SingSingCell image.

## Scripts

This is a VERY broad field. In general all Bash scripts should be [SLURM scripts](https://lunarc-documentation.readthedocs.io/en/latest/batch_system/) on aurora-ls2 to make them run on the calculation nodes. I assume most slurm scripts would access some software installed on the aurora system. Software on aurora and likely also the comming cosmos, is handled using the module system - [please read up on it!](https://lunarc-documentation.readthedocs.io/en/latest/aurora_modules/).

This is an extremely simple script that I use to start my singularity images on the node:
```
#! /bin/bash
#SBATCH -n 5
#SBATCH -N 1
#SBATCH -t 24:00:00
#SBATCH -A lsens2018-3-3
#SBATCH -J SingSing
#SBATCH -o SingSing.%j.out
#SBATCH -e SingSing.%j.err
module purge
module load Singularity/default SingSingCell/1.3 
exit 0
```
Let's look into that. All entries starting with '#' are bash comments and evaluated by the SLURM engine.
I'll scan through the options I normally use: 

1.[#SBATCH -n 5] is the amount of processors you want for this job.
    I use this also to reserve a certain amount of memory for my software. Our nodes have 40 cores each and 376Gb of RAM.
    By requesting 5 nodes I also hope to obtain a total of 376Gb * 5/40 = 47 Gb of RAM.

2.[#SBATCH -N 1] denotes the amount of nodes you request - keep this at 1 all the time.

3.[#SBATCH -t 24:00:00] kill this process after 24 h

4.[#SBATCH -A lsens2018-3-3] state the project you are part of

4.[#SBATCH -J name] the name of your job

5.[#SBATCH -o name.%j.out] the out file of your job (%j adds the job ID to this file)

6.[#SBATCH -e name.%j.err] the error file of you job


After this come the lines that load the SLURM modules you want to load.
In this specific situation the singularity image is loaded and the Jupyter notebook is started.

After the loading of the modules you would then add your Bash script and run your commands.
In this specific script the command is run inside the SingSingCell/1.3 start up phase:
```
singularity run -B/projects,/local /projects/fs1/common/software/SingSingCell/SingleCells_v1.3.sif
```

Please finish up with a trailing "exit 0" as this likely helps the SLURM error detection system.

Please be considerate to your co-workers. Our resources are limited and so is ls2, too. Try to use only as many resources as you actually need at a time. And think about what your software is able to do. Most of the available softwares do not benefit from more than 10 cores. And I know of not a single Bioinformatic program that can use two nodes at the same time.

### Nextflow and NF-core

Nextflow and the [NF-core modules](https://nf-co.re/) are an epic time saver for us bioinformaticians. They cover main bioinformatic issues like single cell mapping or ChIP analysis. And this to an extend and depth that I am normally not applying to my projects.
This is a real step forward and I think we should all get to know the basics of that. Nextflow pipelines have a potential to rid us of all the bash scripting we have been used to. It even frees us from thinking about SLURM or the amount of processors we need for a given task. 

Hence please try to use the nextflow pipelines installed in "/projects/fs1/common/nextflow/". A minimal help on how to start these pipelines on aurora-ls2 can be obtained by looking at the test script for these pipelines "/home/stefanl/common/nextflow/test_all_blade.sh". This way you can see how the input files should be structured and how the pipelines are called. This was quite a lot of work to install them - so please try to use them. I hope [this guide](pdfs/NextFlow_Pipelines_on_aurora_ls2.pdf) can help you.


### R and Python scripts

R and Python are the languages most of the bioinformatic packages are implemented in. Therefore these scripts are different from the Bash/SLURM scripts, they contain a lot of analysis specififc settings and are therefore extremely valuable.
In other words - BACK THEM UP ;-)

I recommend to use the [Python notebooks](https://jupyter.org/) to interact with both R and Python. With aurora-ls2 being a secure no-internet system installing packages for both languages is a pain. Therfore I have focused on Singularity to package up the most used packages in both R and Python and provide them as [a Jupyter enabled singularity image called SingSingCell](pdfs/HowToUseSingularityOnLsens2.pdf).

