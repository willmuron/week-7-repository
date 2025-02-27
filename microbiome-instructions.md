## Fork and Clone directory
- Make sure you clone it to your ocean folder.
You should know how to do this now! `Woohoo`

## Install MultiQC

This software has been already installed in our shared ocean directory and it will work fine while you are in that specific direectory. However, for you to use this file from any directory in the HPC, we must first add it to the environment path and source it:

```
echo 'export PATH=/ocean/projects/agr250001p/shared/software/MultiQC:$PATH' >> ~/.bashrc
source ~/.bashrc
```
- Use `multiqc --help` to test the software

# Organize repository into folders
1. Create a folder `raw-data` by using the `mkdir` command
```
mkdir raw-data
```
2. cd into the newly created folder and download the data
```
cd raw-data
wget https://www.ibdmdb.org/downloads/raw/HMP2/16S/2018-01-08/206534.fastq.gz
wget https://www.ibdmdb.org/downloads/raw/HMP2/16S/2018-01-08/206536.fastq.gz
wget https://www.ibdmdb.org/downloads/raw/HMP2/16S/2018-01-08/206538.fastq.gz
```
