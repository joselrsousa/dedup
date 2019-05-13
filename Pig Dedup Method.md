# Dedup Method
Identifying deduplication using apache pig.
## Introduction
Today genetics data analysis represents a huge challenge on the analytics capability to process the data due to its individual file size. The capability of deduping as a formal approach to verify the quality of a file faces that challenge. Normally the approach would be to rely on high performance computing to do it. Here we describe an approach of doing dedup applied to genetic data and used in the paper by Aideen et al (2019) by using containers possible to be executed on "normal" computers and with timelines fairly acceptable.
## Tools
https://hortonworks.com/downloads/# \
https://filezilla-project.org \
https://pig.apache.org  
## Method = Tools + Wrap
The dedup approach was to find a complet dedup read in each pair of the dedup files.
### Tasks done on the OS system
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
    
### Filezilla upload of the files
The transformedfiles are upload to the folder of the user maria_dev.

### Hortonworks environment
The upload files are processed to produce the dedup files.
### 1. OS enviroment
#### 1.1 Move the files to HDFS
    hdfs dfs -put filename_numbered.csv /user/maria_dev
#### 1.2 Write the script
        sd1 = LOAD 'Z00065a_R1_numbered.csv' USING PigStorage(',') AS (line:int,read:chararray);
        sd2 = LOAD 'Z00065a_R2_numbered.csv' USING PigStorage(',') AS (line:int,read:chararray);
        --sd1_1 = FOREACH sd1 GENERATE c1,UniqueID(c1) AS id1;
        --sd2_1 = FOREACH sd2 GENERATE c2,UniqueID(c2) AS id2;
        sd3 = JOIN sd1 BY line,sd2 BY line;
        sd4 = FOREACH sd3 GENERATE $0 as line:int,CONCAT($1,$3) AS read:chararray;
        --sd5 = FOREACH sd4 GENERATE line:int,SUBSTRING(read,24,124) AS read:chararray;
        sd6 = GROUP sd4 by read;
        sd7 = FOREACH sd6 GENERATE sd4.line,sd4.read,COUNT(sd4.read) as count;
        sd8 = FILTER sd7 BY count>1;
        STORE sd8 INTO 'Z00065all_a';
        -- the commented lines demonstrate how to extract a substring to compare
#### 1.3 Run the script
        pig -x tez scriptfilename
The script also show how it can be used to only find for dedup in certain read positions.
