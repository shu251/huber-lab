## Internal Huber lab guide   
:ocean: :volcano: :earth_americas: :computer: :microscope:
***
Data management with HPC & best practices

### Set up
```
# Download git repository
git clone https://github.com/shu251/huber-lab.git

# If you need to update:
git pull

# Conda
conda info --envs # view your environments

# Example - create conda environment from Sarah
conda env create -f snake-18S-env.yaml --name snake-18S 

## Enter environment
source activate snake-18S

```


### Test case _Mid-Cayman_

Below describes downloading raw data from Mid-Cayman Rise
***
BioProject: PRJNA258374
https://www.ncbi.nlm.nih.gov/sra?LinkName=bioproject_sra_all&from_uid=258374
* From 'Send to:' drop down menu, click 'File'
* Select 'RunInfo' Format and create file

Upload to huber-lab directory
```
scp $PATH/SraRunInfo.csv sarahhu@poseidon.whoi.edu:/vortexfs1/omics/huber/shu/huber-lab/
mkdir raw_data
mv *.csv raw_data/
```

### Download files from SRA by makig a bash script

First enter interactive or scavenger so we can do stuff. Go to SRA explorer website: https://ewels.github.io/sra-explorer/#
Search for BioProject: PRJNA258374, select *bash script for downloading FastQ files*

```
cd raw_data
nano download-MCR.sh # make new bash script
head -n 10 download-MCR.sh >> download-MCR_10.sh #subset top 10
bash download-MCR_10.sh
# CANCEL and let's write a SLURM script
```
Above the ```bash download-MCR_10.sh``` command will download 10 of the raw fastq files that we listed in download-MCR.sh. **Below** is an example slurm script to submit this job.  

This file is also provided in the repo as ```SLURM-MCR.txt```


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
