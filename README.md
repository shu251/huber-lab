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
mv extract-sra.R raw_data/

```

### Download files from SRA using R script and bash **TWO WAYS**

First enter interactive or scavenger so we can do stuff.


```
cd raw_data
Rscript extract-sra.R
# R code will generate 2 bash scripts to download from SRA
### Note: R is already loaded because we are using this cool conda environment

# Download forward and reverse reads:
bash download_read1.sh
bash download_read2.sh
```


### Write with SLURM script
