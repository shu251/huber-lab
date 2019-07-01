### Internal Huber lab guide
Data management with HPC & best practices


### Set up
```
# Download git repository
git clone https://github.com/shu251/huber-lab.git


# Conda
module load anaconda # load anaconda for use
conda info --envs # view your environments
# create conda environment from Sarah
conda env create -f snake-18S-env.yaml --name snake-18S 

# Enter environment
source activate snake-18S

```

### Test case _Mid-Cayman_
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

### Download files from SRA using R script and bash **TWO WAYS**

First enter interactive or scavenger so we can do stuff. Go to SRA explorer website: https://ewels.github.io/sra-explorer/#
Search for BioProject: PRJNA258374, select *bash script for downloading FastQ files*

```
cd raw_data
nano download-MCR.sh # make new bash script
head -n 10 download-MCR.sh >> download-MCR_10.sh #subset top 10
bash download-MCR_10.sh
# CANCEL and let's write a SLURM script
```
1.  copy in base SLURM
2.  modify shell command
3. run test
4.  submit SLURM
5. monitor with squeue


### Fastqc example
If current conda environment doesn't have fastqc and multiqc
_Google it!_

```
conda install -c bioconda fastqc
conda install -c bioconda multiqc 
```


### Best practices:

**Workflow**
1. Set up my working environment: files in correct directories, conda environment specs, ensure all pr$
2. Enter interactive or scavenger (set a timer...) to start testing and compiling base code
2a. begin SLURM script construction
3. Test memory of script to run (i.e. SLURM or snakemake dry run)
3a. prep dataset by making a subset
3b. conduct dry run
4. submit script! either with sbatch or gen-profile with snakemake
