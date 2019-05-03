# Dedup
Identifying deduplication using apache pig.
## Introduction
Today genetics data analysis represents a huge challenge on the analytics capability to process the data due to its individual file size. The capability of deduping as a formal approach to verify the quality of a file faces that challenge. Normally the approach would be to rely on high performance computing to do it. Here we describe an approach of doing dedup applied to genetic data and used in the paper by Aideen et al (2019) by using containers possible to be executed on "normal" computers and with timelines fairly aceptable.
## Tools
https://hortonworks.com/downloads/# \
https://filezilla-project.org \
https://pig.apache.org  
## Method = Tools + Wrap
The dedup approach was to find a complet dedup read in each pair of the dedup files.
Tasks done on the OS system
### 1. Raw data
Fastq files with a size between 5 and 1.8 Gb.
### 2. Creating the files
From the fastq files where created files with only the reads and with numbered lines.
#### 2.1 Extracting only the reads
    awk 'NR % 4==2' filename.fastq > filename.csv
#### 2.2 Creating the lines numbers
    nl -s "," filename.csv > filename_numbered.csv
### 3. Spliting the files higher than 2Gb
#### 3.1 Reading the number of lines
    wc - l filename_numbered.csv
#### 3.2 Spliting the file
    split -a -l (numberoflines/2) filename_numbered.csv
    





