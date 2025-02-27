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

## Install QIIME2
1. Load anaconda
```
module load anaconda3
```
2. Just in case you have an environment, remove it first
```
conda env remove -n qiime2-amplicon-2024.2
```
3. Just in case you have an activated conda environment:
```
conda deactivate
```
4. Download installer
```
wget wget https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.2-py38-linux-conda.yml
```
5. Create an environment
```
conda env create -n qiime2-amplicon-2024.2 --file qiime2-amplicon-2024.2-py38-linux-conda.yml
```
6. Once we’ve made it, we can activate this new environment:

```
conda activate qiime2-amplicon-2024.2
```
7. And now we can remove the file that we used to install it:
```
rm qiime2-amplicon-2024.2-py38-linux-conda.yml
```


# Organize repository into folders
1. Create a folder `raw-data` by using the `mkdir` command
```
mkdir mock-data
```
2. cd into the newly created folder and download the data
```
cd mock-data
wget https://www.ibdmdb.org/downloads/raw/HMP2/16S/2018-01-08/206534.fastq.gz
wget https://www.ibdmdb.org/downloads/raw/HMP2/16S/2018-01-08/206536.fastq.gz
wget https://www.ibdmdb.org/downloads/raw/HMP2/16S/2018-01-08/206538.fastq.gz
```
3. Unzip the data. Note that sometimes when we run bioinformatics programs they can uncompress the files within the program, but other times we need to uncompress (unzip) them first.

```
gunzip *.fastq.gz
```

4. Use `less` to scroll through the file, and remember to press `q` when you want to stop looking at the file.
```
less 206534.fastq
```

## We will first perform quality control (QC) on our data
1. First we’ll be running fastqc, and to do that, we’ll first make a directory for the output to go:
```
cd .. #Goes one directory back (up one level)
mkdir fastqc_mock_out
```
2. Load required module (Loads the program/software you will use)
```
module load FastQC
```
3. Now we’ll run fastqc
```
fastqc -t 4 mock-data/*.fastq -o fastqc_mock_out
```
The arguments that we’re giving fastqc are:

- -t 4: the number of threads to use. Sometimes “threads” will be shown as --threads, --cpus, --processors, --nproc, or similar. Basically, developers of packages can call things whatever they like, but you can use the help documentation to see what options are available. We’re using 4 here because that’s the maximum that we have available. See below (htop) for how we find out about how many we have available.
- mock-data/*.fastq: the fastq files that we want to check the quality of.
- -o fastqc_mock_out: the folder to save the output to.

4. Next we’ll run multiqc. The name suggests it might be performing QC on multiple files, but it’s actually for combining the output together of multiple files, so we can run it like this:

```
multiqc fastqc_mock_out --filename multiqc_mock.html
```
So we’ve given as arguments:

- fastqc_mock_out: the folder that contains the fastqc output.
- --filename multiqc_mock.html: the file name to save the output as.

## Visualize QC file and answer the questions.
1. Add, Commit and Push multiqc_mock.html to your GitHub repository.
2.Download file and doeble click to open in web browser.


# NOW LET'S CARRY OUT QC ON REAL DATA!
1. Create a symlink to the data like this, yup, you not always have to copy or move files around!
```
ln -s /ocean/projects/agr250001p/shared/week-7-data/raw_data .
```
2. Create a directory for the fastqc output:
```
mkdir fastqc_raw_data_out
```
3. Run fastqc
```
fastqc -t 4 raw_data/*.fastq.gz -o fastqc_raw_data_out
```
4. Run multiqc
```
multiqc fastqc_raw_data_out --filename multiqc_raw_data.html
```

## Let's run Quimme to study the microbiome
1. Create a Symlink for the metadata file from the share folder
```
ln -s /ocean/projects/agr250001p/shared/week-7-data/Blueberry_metadata_reduced.tsv .
```
2. Inspect the metadata file
```         
head Blueberry_metadata_reduced.tsv
```
- Answer questions 11 and 12

## A few formatting steps required by quimme
1. Lwt's create the following directory
```         
mkdir reads_qza
```
2. You had installed the latest version of QIIME2, so you can either activate that environment with the command below.

```         
conda activate qiime2-amplicon-2024.2
```
- When you are finished this tutorial you can deactivate the environment using:

```         
conda deactivate
```
3. Feed qiime the raw reads
```         
qiime tools import \
  --type SampleData[PairedEndSequencesWithQuality] \
  --input-path raw_data/ \
  --output-path reads_qza/reads.qza \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt
```
- cassava format, files follow a pattern `SampleID_L001_R1_001.fastq.gz`

4. remove primer sequences from reads
```
qiime cutadapt trim-paired \
  --i-demultiplexed-sequences reads_qza/reads.qza \
  --p-cores 4 \
  --p-front-f ACGCGHNRAACCTTACC \
  --p-front-r ACGGGCRGTGWGTRCAA \
  --p-discard-untrimmed \
  --p-no-indels \
  --o-trimmed-sequences reads_qza/reads_trimmed.qza
```
5 Visualize your data now
```         
qiime demux summarize \
  --i-data reads_qza/reads_trimmed.qza \
  --o-visualization reads_qza/reads_trimmed_summary.qzv
```
- Git add, commit and push `reads_trimmed_summary.qzv` Download and open in https://view.qiime2.org/

### Denoising reads
1. Join pair-end reds
```         
qiime vsearch merge-pairs \
  --i-demultiplexed-seqs reads_qza/reads_trimmed.qza \
  --output-dir reads_qza/reads_joined
```
2. Filter out low-quality reads
```         
qiime quality-filter q-score \
  --i-demux reads_qza/reads_joined/merged_sequences.qza \
  --o-filter-stats filt_stats.qza \
  --o-filtered-sequences reads_qza/reads_trimmed_joined_filt.qza
```
- Git add, commit and push `reads_trimmed_joined_filt.qzv` Download and open in https://view.qiime2.org/
- Answer question 13
3. Actual denoising with deblur

```         
qiime deblur denoise-16S \
  --i-demultiplexed-seqs reads_qza/reads_trimmed_joined_filt.qza \
  --p-trim-length 390 \
  --p-sample-stats \
  --p-jobs-to-start 4 \
  --p-min-reads 1 \
  --output-dir deblur_output
```
- Note: this command may take up to 10 minutes or so to run.

4. Summarize dublur output
```         
qiime feature-table summarize \
  --i-table deblur_output/table.qza \
  --o-visualization deblur_output/deblur_table_summary.qzv
```
- Git add, commit and push `deblur_table_summary.qzv` Download and open in https://view.qiime2.org/

###  Run taxonomic classification
1. Create symlink to taxonomic data
```
ln -s /ocean/projects/agr250001p/shared/week-7-data/taxa .
```
2. Run taxonomic classification (Homework)
```         
qiime feature-classifier classify-sklearn \
  --i-reads deblur_output/representative_sequences.qza \
  --i-classifier /ocean/projects/agr250001p/shared/week-7-data/silva-138-99-nb-classifier.qza \
  --p-n-jobs 4 \
  --output-dir taxa
```
### Filtering resultant table
1. Filter out rare ASVs
```         
qiime feature-table filter-features \
  --i-table deblur_output/table.qza \
  --p-min-frequency 10 \
  --p-min-samples 1 \
  --o-filtered-table deblur_output/deblur_table_filt.qza
```
- replace X with desired frequency
2. Filter out contaminant and unclassified ASVs
```         
qiime taxa filter-table \
  --i-table deblur_output/deblur_table_filt.qza \
  --i-taxonomy taxa/classification.qza \
  --p-include p__ \
  --p-exclude mitochondria,chloroplast \
  --o-filtered-table deblur_output/deblur_table_filt_contam.qza
```
3. Subset and summarize filtered table
```         
qiime feature-table summarize \
  --i-table deblur_output/deblur_table_filt_contam.qza \
  --o-visualization deblur_output/deblur_table_filt_contam_summary.qzv
```
- Git add, commit and push `deblur_table_filt_contam_summary.qzv` Download and open in https://view.qiime2.org/
- Answer question 16 What is the minimum and maximum sequencing depth across all samples?

4. Copy final table to current directory
```         
cp deblur_output/deblur_table_filt_contam.qza deblur_output/deblur_table_final.qza
```
5. Produce final table summary
```         
qiime feature-table filter-seqs \
  --i-data deblur_output/representative_sequences.qza \
  --i-table deblur_output/deblur_table_final.qza  \
  --o-filtered-data deblur_output/rep_seqs_final.qza
```
```         
qiime feature-table summarize \
  --i-table deblur_output/deblur_table_final.qza \
  --o-visualization deblur_output/deblur_table_final_summary.qzv
```

- Git add, commit and push `deblur_table_final_summary.qzv` Download and open in https://view.qiime2.org/
