# Aurora documentation from Bioinformaticians for Bioinformaticians

Please read this before you use the aurora-ls2 system!

## Data - where to put data / scripts / results ?

Backup is very expensive for us as we need to buy more hardware to back things up.
Therefore we try to minimize the amount of data that needs to get backed up.

What we think is necessary to be backed up is:

1. Your raw data - this (and ONLY this) should go to /projects/fs5/\<username\>
    Raw data are e.g. fastq files (gzipped!!) or raw read folders.

2. Your scripts - this is where all your work time got invested!
    They should be stored in your home directory as this is also updated.

That is not much -right? So where will all the intermediate data files end up?

Files that are reproducible (as your scripts do no random sampling or something like that) or even temporary should
be put on a not backed up disk. Ideally you should save them on the calculation nodes in a folder on the node.
The slurm system is set up to give you a tempoorary folder on the odes for each slurm job you start. The name of your temporary folder is stored in the $SNIC_TMP [environment variable](https://linuxize.com/post/how-to-set-and-list-environment-variables-in-linux/). This variable does not exist on the frontend. The folder on the node is private to you and will be deleted once the job is finished. So in the last step of your script you need to save your results.

Even these results should not be backed up (if they can be reproduced - try it if you are unsure). Hence they should also be stored on a not backed up nas.

I strongly recommend [to create soft links](https://www.cyberciti.biz/faq/creating-soft-link-or-symbolic-link/) to the main data folder that you use. I for example have the main data storage accessible under "\~/NAS/". 
```
ln -s /projects/fs1/stefanl ~/NAS
ln -s /projects/fs5/stefanl/ ~/DATA
```
This way you keep quite flexible in changing the NAS mount points in the future.
Do not overdo this! You need to know where all your data is as you are responsible - OK?


## Make scripts reproducible

The easiest way to achieve that is letting someone take care of your analysis software.
R and python packages are updating rather frequently and updated packages on the one hand might change the results
and on the other hand do create a problem if every user installes them in a private directory.

[Singularity images](https://sylabs.io/guides/3.6/user-guide/quick_start.html) are the way to go.
Singularity version 3.6 is installed on aurora-ls2. If you have an immediate need for a package you might think about starting your own singularity image and upload it to aurora-ls2. But before you do this check out the folder "/projects/fs1/common/singularityImages/" there might already exist an image that contains the software you are looking for.

I recommend reading [this pdf](pdfs/HowToUseSingularityOnLsens2.pdf) about how to use singularity images and especially how to use my SingSingCell image.  

## Scripts

This is a VERY broad field. In general all Bash scripts should be [SLURM scripts](https://lunarc-documentation.readthedocs.io/en/latest/batch_system/) on aurora-ls2 to make them run on the calculation nodes. I assume most slurm scripts would access some software installed on the aurora system. Software on this system, is handled using the module system - [please read up on it!](https://lunarc-documentation.readthedocs.io/en/latest/aurora_modules/).

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

4.[#SBATCH -J] the name of your job

5.[#SBATCH -o] the out file of your job (%j adds the job ID to this file)

6.[#SBATCH -e] the error file of you job


After this come the lines that load the SLURM modules you want to load.
In this specific situation the singularity image is loaded and the Jupyter notebook is started.
After the loading of the modules you would then add your Bash script and run your commands.

Please finish up with a trailing "exit 0" as this likely helps the SLURM error detection system.

Please be considerate to your co-workers. Our resources are limited and so is ls2, too. Try to use only as many resources as you actually need at a time. And think about what your software is able to do. Most of the available softwares do not benefit from more than 10 cores. And I know of not a single Bioinformatic program that can use two nodes at the same time.

### Nextflow and NF-core

Nextflow and the [NF-core modules](https://nf-co.re/) are an epic time saver for us bioinformaticians. They cover main bioinformatic issues like single cell mapping or ChIP analysis. And this to an extend and depth that I am normally not applying to my projects.
This is a real step forward and I think we should all get to know the basics of that. Nextflow pipelines have a potential to rid us of all the bash scripting we have been used to. It even frees us from thinking about SLURM or the amount of processors we need for a given task. 

Hence please try to use the nextflow pipelines installed in "/projects/fs1/common/nextflow/". A minimal help on how to start these pipelines on aurora-ls2 can be obtained by looking at the test script for these pipelines "/home/stefanl/common/nextflow/test_all_blade.sh". This way you can see how the input files should be structured and how the pipelines are called. This was quite a lot of work to install them - so please try to use them using [this guide](pdfs/NextFlow_Pipelines_on_aurora_ls2.pdf) ;-).


### R and Python scripts

R and Python are the languages most of the bioinformatic packages are implemented in. Therefore these scripts are different from the Bash/SLURM scripts, they contain a lot of analysis specififc settings and are therefore extremely valuable.
In other words - BACK THEM UP ;-)

I recommend to use the [Python notebooks](https://jupyter.org/) to interact with both R and Python. With aurora-ls2 being a secure no-internet system installing packages for both languages is a pain. Therfore I have focused on Singularity to package up the most used packages in both R and Python and provide them as [a Jupyter enabled singularity image called SingSingCell](pdfs/HowToUseSingularityOnLsens2.pdf).


