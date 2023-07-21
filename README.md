# 16S_CHIIME2_analysis

## 16S code based on Clara’s code from 2021

###step 1 was to make import manifest 

## Sample-ID’s have had direction identifiers removed. Therefore each sample ID will have 2 entries: one connected to the R1 (forward) fq.gz file and 

## a second connected to the R2 (reverse) fq.gz file. 

## with this import manifest we should be able to load the files into Qiime using the following code. 

###set wd


cd /Users/greent/Desktop/MarissaMB/Marissa_fastq.gz_data

qiime tools import \

  --type SampleData[PairedEndSequencesWithQuality] \

  --input-format PairedEndFastqManifestPhred33 \

  --input-path Manifest_M.csv \

  --output-path MWL_sequences.qza


###activate qiime2 environment


conda activate qiime2-2019.4



### SUCCESS! QZA built. 

### utilize a summary tool. 


qiime demux summarize --i-data MWL_sequences.qza --o-visualization MWL_sequences.qzv


### Success, Visualization built. 

### Qiime2 view online to see the visualizations. Each group will need to be trimmed. 


### threshold selected based on median value of 30 quality score (based off of QIIME2 qzv file visualization)

### Forward reads: left 16, length 294

### Reverse reads: left 7, length 228

### apply this to below code:


qiime dada2 denoise-paired \

--i-demultiplexed-seqs MWL_sequences.qza \

--p-trim-left-f 16 \

--p-trunc-len-f 294 \

--p-trim-left-r 7 \

--p-trunc-len-r 228 \

--p-n-threads 0 \

--o-representative-sequences rep-seqs-dada2.qza \

--o-table table-dada2.qza \

--verbose \

--o-denoising-stats stats-dada2.qza/

