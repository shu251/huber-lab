## Internal Huber lab guide   
:ocean: :volcano: :earth_americas: :computer: :microscope:

Data management with HPC & best practices   

_last update August 2020_


### Contents of repo:

* Download this repo with list of useful commands (```cmds-hpc.txt```)
* Organize with conda environments on HPC & best practices
* see ```slurm-examples``` for base text for slurm commands
* Download sequences from provided BioProject (Mid-Cayman rise example)
* How to run mothur with slurm on poseidon
 
```
# Download git repository
git clone https://github.com/shu251/huber-lab.git

# If you need to update:
git pull
```

## Conda
[Read up on managing environments](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html).   
I also wrote a [blog post](https://alexanderlabwhoi.github.io/post/anaconda-r-sarah/) that details how I manage several R versions using conda. It gives some insight into why conda is a really good tool to use!   

Also see an explanation of the 'Concept of Conda' [here](https://geohackweek.github.io/datasharing/01-conda-tutorial/) with a helpful infographic. Below I've also included a protocol for creating a new conda environment to install R.


See some examples below:
```
conda info --envs # view your environments

# Example - create conda environment
conda create --name sarahissocool

## Enter environment
conda activate sarahissocool
```
Now each line in your terminal will be preceeded by ```(sarahissocool)```. When you're in this new environment, google things like "conda install R" or "conda install megahit". In the case of the latter, you can install megahit like this:

```
## Enter environment
conda activate sarahissocool 

conda install -c bioconda megahit
```
Whenever you need to use megahit, open up this environment and it will be enabled.


## Create a conda environment for R

1. Log on to access Poseidon
2. You can be located anywhere on poseidon to create a conda environment
3. Create a conda environment that is called **R**
```
conda create --name R
```
> This may take a bit of time, it will show a _Solving environment:_ note while it is creating it. When it asks you to Proceed, type *y* (yes).

4. To view all your conda environments, type the command: ```conda info --envs```. You should now see *R* listed as one of your conda environments.
5. Enter (or activate) your newly created conda environment
```
conda activate R
```
> Note that instead of ```(base)``` preceeding your username/log in account on each line, it will now show ```(R)```.

6. Install R in your *R* conda environment!

```
conda install -c r r 
```
> this may also take some time! It will say things like "Collecting package metadata" and again "Solving Environment" (with the command line version of the _spinning wheel_. There will eventually be a long long list of items that you will be asked to install. Proceed by selecting *y*. The environment will then work on preparing and verifying this transaction. 

7. Enable/start up R to see what version you are running by typing ```R```.
8. Lets install our favorite package and make sure it loads:
```
install.packages("tidyverse")
```
> You will be prompted to select a CRAN mirror, any should work. Type in the relevant number and hit enter. If you run into an error, document it (copy and paste it somewhere) so we can look at it if needed.

While the above may take some time it should end with somethign like this:
```
The downloaded source packages are in
	‘/tmp/RtmpkAnSzH/downloaded_packages’
Updating HTML index of packages in '.Library'
Making 'packages.html' ... done
```

9. Now test to ensure you can load tidyverse
```
library(tidyverse)
```
10. To exit R on the command line, type ```quit()```. Do not save your workspace. 


## Best practices workflow for using HPC

1. Set up my working environment: files in correct directories, conda environment specs, ensure all progra$
2. Enter interactive or scavenger (set a timer...) to start testing and compiling base code
2a. begin SLURM script construction
3. Test memory of script to run (i.e. SLURM or snakemake dry run)
3a. prep dataset by making a subset
3b. conduct dry run
4. submit script! either with sbatch or gen-profile with snakemake


## Download sequences from public repository
BioProject: PRJNA258374 - https://www.ncbi.nlm.nih.gov/sra?LinkName=bioproject_sra_all&from_uid=258374

1. First enter interactive or scavenger so we can do stuff. 
2. Go to SRA explorer website: https://ewels.github.io/sra-explorer/#
3. Search for BioProject: PRJNA258374, select *bash script for downloading FastQ files*
4. copy and paste that into a text file called "download-MCR.sh" - A bash script to download the first ten sequences are here as ```download-MCR_10.sh```

```
mkdir raw-seqs #Make a new directory to store your new sequences

# use 'nano' text editor to create download-MCR.sh for all sequences or use the provided on
nano download-MCR.sh # make new bash script

# Execute bash script like so:
bash download-MCR_10.sh

```

An alternative is to send this to slurm (which is recommended if you have lots of sequences to download).
See the slurm script: ```slurm-examples/slurm-mcr.txt``` for this.


### Slurm example:
```
#!/bin/bash
#SBATCH --partition=compute           # Queue selection
#SBATCH --job-name=download-MCR        # Job name
#SBATCH --mail-type=ALL               # Mail events (BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=u    # Where to send mail
#SBATCH --ntasks=1                    # Run a single task
#SBATCH --cpus-per-task=8             # Number of CPU cores per task (if 8 == 8 threads)
#SBATCH --mem=10000                     # Job memory request (if 10,000 == 10000MB == 10GB)
#SBATCH --time=03:00:00               # Time limit hrs:min:sec
#SBATCH --output=MCR-download-SRA.log   # Standard output/error
#export OMP_NUM_THREADS=8       # 8 threads

bash raw_data/download-subset.sh
```

Submit the above script like so: ```sbatch SLURM-mcr.txt```   
You will get a message tell you what your submit job ID is. Record this, as you'll want to track how long this job will take.

_Best practices for writing and submitting a SLURM job_:
1.  copy in base SLURM (you can use the ```prep-SLURM.txt``` file in this repo as a starting point)
2.  modify shell command
3. run test (make a small batch of your samples that you want to run through first, this helps with de-bugging, etc)
4.  submit SLURM (```sbatch prep-SLURM.txt```)
5. monitor with squeue (```squeue -u sarahhu```)

***

## Setting up a conda environment! 
See this website on how to create and manage conda environments:
https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#building-identical-conda-environments   

There's another example [here, for qc-ing and trimming sequences reads](https://github.com/shu251/qc-trim).


### Example with curating mapping files!

Do you have conda installed?
```
conda --version
```
Poseidon is running conda v4.7.10 as of 9-17-2019   


I created this conda environment with these steps:
```
# Created environment
conda env create --name map_curation

# To combat the issues with that one script, literally googled "conda install biopython" and "conda install BCbio" to find the below install commands:
conda install -c conda-forge biopython
conda install -c bioconda bcbio-gff

# Exported map_curation environment to share
conda env export > map_curation.yaml
```

**To make your own** see ```/vortexfs1/omics/huber/envs``` to find the environment file (ends with .yaml)
```
# From anywhere on the Poseidon server:
# Create your conda environment
conda env create --name map_curation --file /vortexfs1/omics/huber/envs/map_curation.yaml 

# To enter the conda environment:
conda activate map_curation
```
At this point your command line will change to be proceeded by "(map_curation)"
Within this environment you can run all that biopython stuff. If you want additional programs added that are related to curation mapping files, add them via a conda install while you are activated in the environment! Then we can keep updating them. 


## Setting up to run mothur

1. Create your own conda environment called 'mothur'
```
conda create --name mothur
```

2. Activate this environment
```
conda activate mothur
```
Your command line should now look like this each line preceeding with '(mothur)', instead of base:
```
(mothur) [sarahhu@poseidon-l1 huber-lab]$
```

3. Install the latest version of mothur
```
conda install -c bioconda mothur
```

4. Test to make sure install was successful
```
# Start mothur
mothur

# Command line should now look like this:

mothur >

# Quit
quit()

```

To exit this mothur environment, ```conda deactivate mothur```

*Now*, whenever you want to run mothur, type ```conda activate mothur```, and this mothur enabled environment will launch.

5. To run mothur interactively, start up an interactive or scavenger session. See above instructions for this! Then re-activate your mothur conda environment and you should be able to run mothur.

6. Run mothur using slurm!  You need to write all your mothur commands in their own standalone text file. See example from Amy called ```mothur-commands.txt```. Now you need to write a *slurm* script that activates mothur and executes your .txt file!
  
Example slurm script to activate and run:
```
module load anaconda/5.1  # activate anaconda, so we can use conda environments
source activate mothur    # activate the mothur conda env
mothur mothur-commands.txt  # execute the mothur commands
```

See example files: ```mothur-commands.txt``` and ```mothur-submit-slurm.txt```. _Note_ that mothur is very memory intensive, so you may need to play around with the memory requirements listed at the top of your slurm script. 
