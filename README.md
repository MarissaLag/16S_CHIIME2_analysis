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

****removed last line: 


--o-denoising-stats not_passing/


***because, resulted in error: 


######error April 21 - FileNotFoundError: [Errno 2] No such file or directory: 'not_passing/.qza'



***since data is good quality, all samples will pass***





qiime feature-table summarize --i-table table-dada2.qza --o-visualization table-dada2.qzv --m-sample-metadata-file Metadata_Marissa.txt


qiime feature-table tabulate-seqs --i-data rep-seqs-dada2.qza --o-visualization rep-seqs-dada2.qzv



***error April26: There was an issue with loading the file Metadata.txt as metadata:


  Metadata file path doesn't exist, or the path points to something other than a file. Please check that the path exists, has read permissions, and points to a regular file (not a directory): Metadata.txt


  There may be more errors present in the metadata file. To get a full report, sample/feature metadata files can be validated with Keemei: https://keemei.qiime2.org


  Find details on QIIME 2 metadata requirements here: https://docs.qiime2.org/2019.4/tutorials/metadata/



***above error because Metadata file needs to be made, error fixed after Metadata_Marissa.txt made 




***********Assign Taxonomy using silva-132-99-nb-classifier.qza ************




qiime feature-classifier classify-sklearn --i-classifier $PWD/silva-132-99-nb-classifier.qza --i-reads /$PWD/rep-seqs-dada2.qza --o-classification taxonomy-data2.qza


qiime metadata tabulate --m-input-file taxonomy-data2.qza --o-visualization taxonomy-data2.qzv




*****Filter and remove mitochondria and chloroplast*************



#### remove 16S DNA from oyster tissue/algae etc. 



qiime taxa filter-table --i-table table-dada2.qza --i-taxonomy taxonomy-data2.qza --p-exclude D_4__Mitochondria,D_3__Chloroplast --o-filtered-table table-no-mitochondria-no-chloroplast_data2.qza


qiime feature-table summarize --i-table table-no-mitochondria-no-chloroplast_data2.qza --o-visualization table-no-mitochondria-no-chloroplast_data2.qzv --m-sample-metadata-file Metadata_Marissa.txt




##visualize??? Still getting errors


qiime demux summarize --i-data table-no-mitochondria-no-chloroplast_data2.qza --o-visualization table-no-mitochondria-no-chloroplast_data2.qzv




##################Filter the table to contain sequences contributing >0.05% of the total abundance-rarify#######



qiime feature-table filter-features --i-table table-no-mitochondria-no-chloroplast_data2.qza --p-min-frequency 61 --o-filtered-table 0.05percent-filtered-table.qza


qiime feature-table summarize --i-table 0.05percent-filtered-table.qza --o-visualization 0.05percent-filtered-table.qzv --m-sample-metadata-file Metadata_Marissa.txt



###################create bar plots (not with rarified table)##############


qiime taxa barplot --i-table 0.05percent-filtered-table.qza --i-taxonomy taxonomy-data2.qza --m-metadata-file Metadata_Marissa.txt --o-visualization 0.05_taxa-bar-plots-unfiltered.qzv


#Generate a tree for phylogenetic diversity analyses


#multiple sequence alignment


qiime alignment mafft --i-sequences rep-seqs-dada2.qza --o-alignment aligned-rep-seqs.qza


#mask (or filter) the alignment to remove positions that are highly variable


qiime alignment mask --i-alignment aligned-rep-seqs.qza --o-masked-alignment masked-aligned-rep-seqs.qza




#FastTree to generate a phylogenetic tree from the masked alignment


qiime phylogeny fasttree --i-alignment masked-aligned-rep-seqs.qza --o-tree unrooted-tree.qza






#The FastTree program creates an unrooted tree, so in the final step in this section we apply midpoint rooting to place the root of the tree at the midpoint of the longest tip-to-tip distance in the unrooted tree


qiime phylogeny midpoint-root --i-tree unrooted-tree.qza --o-rooted-tree rooted-tree.qza




############rarefied data to depth of 5000, based on sequences.qzv  ############


qiime diversity core-metrics-phylogenetic --i-phylogeny rooted-tree.qza --i-table 0.05percent-filtered-table.qza --p-sampling-depth 5000 --m-metadata-file Metadata_Marissa.txt --output-dir core-metrics-results


####### create bar plots (with rarified table) ############


qiime taxa barplot --i-table core-metrics-results/rarefied_table.qza --i-taxonomy taxonomy-data2.qza --m-metadata-file Metadata_Marissa.txt --o-visualization taxa-bar-plots-rarefied.qzv


#t######to look at the rarefied table #######


Mkdir exported-rarefied-5000-table00000.0.






qiime feature-table summarize --i-table core-metrics-results/rarefied_table.qza --o-visualization rarefied_table.qzv --m-sample-metadata-file Metadata_Marissa.txt



### If you want to get otu table in biom and txt format do the script below


Qiime tools export rarified-5000-table.qza —output-dir exported-biom

Biom convert -t rarified-5000-table.biom -o rarified-5000-table.tsv —to-tsv biom head -I feature-table.tsv

###can’t get biom conversion to work…



# export data


qiime tools export core-metrics-results/rarefied_table.qza --output-dir exported-rarefied-5000-table


qiime tools export taxonomy-data2.qza --output-dir exported-taxonomy








####qiime emperor plot


—I-pcoa core-metrics-results/unweighted_unifrac_results.qza\

—m-metadata-file sample-Metadata_Marissa.txt \

—p-custom-axes Treatment \

—o-visualization core-metrics-results/unweighted-unifrac-emperor.qzv



qiime diversity core-metrics-phylogenetic --i-phylogeny rooted-tree.qza --i-table 0.05percent-filtered-table.qza --p-sampling-depth 5000 --m-metadata-file Metadata_Marissa.txt --output-dir core-metrics-results/--o-unweighted_unifrac_pcoa_results.qza




####PERMANOVA


qiime diversity beta-group-significance \

  --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza \

  --m-metadata-file Metadata_Marissa.txt \

  --m-metadata-column Tank_treatment \

  --o-visualization core-metrics-results/unweighted-unifrac-donor-significance.qzv


****no differences (unweighted) between tank treatments at p sampling length of 5000****


qiime diversity beta-group-significance \

  --i-distance-matrix core-metrics-results/weighted_unifrac_distance_matrix.qza \

  --m-metadata-file Metadata_Marissa.txt \

  --m-metadata-column Tank_treatment \

  --o-visualization core-metrics-results/weighted-unifrac-donor-significance.qzv


****no differences (unweighted) between tank treatments at p sampling length of 5000****



qiime diversity beta-group-significance \

  --i-distance-matrix core-metrics-results/weighted_unifrac_distance_matrix.qza \

  --m-metadata-file Metadata_Marissa.txt \

  --m-metadata-column Age \

  --o-visualization core-metrics-results/weighted-unifrac-donor-significance.qzv


*****significant (p = 0.0001) age effect (weighted)****



***PcoA based analysis


qiime longitudinal volatility \

  --m-metadata-file ./Metadata_Marissa_other.txt \

  --m-metadata-file ./core-metrics-results/unweighted_unifrac_pcoa_results.qza \

  --p-state-column Age \

  --p-individual-id-column Library_Name \

  --p-default-group-column ‘Tank_treatment’ \

  --p-default-metric 'Axis 2' \

  --o-visualization ./pc_vol.qzv




#####look at larvae only###


###1st make new metadata file with only larvae


###2nd subset table.qza, ie. extract larvae data


Qiime feature-table filter-samples --i-table 0.05percent-filtered-table.qza --m-metadata-file Marissa_Metadata_larvae.txt --o-filtered-table larvae_filtered_table.qza 


###


qiime diversity core-metrics-phylogenetic --i-phylogeny rooted-tree.qza --i-table larvae_filtered_table.qza --p-sampling-depth 2900 --m-metadata-file Marissa_Metadata_larvae.txt --output-dir core-metrics-results-larvae


###generate taxa-plots


qiime taxa barplot --i-table larvae_filtered_table.qza --i-taxonomy taxonomy-data2.qza --m-metadata-file Marissa_Metadata_larvae.txt --o-visualization taxa-plts-larvae.qzv



###PERMANOVA with larvae data



qiime diversity core-metrics-phylogenetic --i-phylogeny rooted-tree.qza --i-table 0.05percent-filtered-table.qza --p-sampling-depth 2900 --m-metadata-file Metadata_Marissa_larvae.txt --output-dir core-metrics-results_larvae/—o-unweighted_unifrac_pcoa_results_larvae.qza



qiime diversity core-metrics-phylogenetic --i-phylogeny rooted-tree.qza --i-table larvae_filtered_table.qza --p-sampling-depth 2900 --m-metadata-file Marissa_Metadata_larvae.txt --output-dir core-metrics-results_larvae/--o-unweighted_unifrac_pcoa_results_larvae.qza



###permanova###


qiime diversity beta-group-significance \

  --i-distance-matrix core-metrics-results-larvae/unweighted_unifrac_distance_matrix.qza \

  --m-metadata-file Marissa_Metadata_larvae.txt \

  --m-metadata-column Tank_treatment \

  --o-visualization core-metrics-results-larvae/unweighted-unifrac-donor-significance.qzv


****no signify differences for unweighted***


qiime diversity beta-group-significance \

  --i-distance-matrix core-metrics-results-larvae/weighted_unifrac_distance_matrix.qza \

  --m-metadata-file Marissa_Metadata_larvae.txt \

  --m-metadata-column Tank_treatment \

  --o-visualization core-metrics-results-larvae/weighted-unifrac-donor-significance.qzv


***no difference for weighted***



###try with spat data###


###1st make new metadata file with only spat


###2nd subset table.qza, ie. extract spat data


Qiime feature-table filter-samples --i-table 0.05percent-filtered-table.qza --m-metadata-file Marissa_Metadata_spat.txt --o-filtered-table spat_filtered_table.qza 


###


qiime diversity core-metrics-phylogenetic --i-phylogeny rooted-tree.qza --i-table spat_filtered_table.qza --p-sampling-depth 5000 --m-metadata-file Marissa_Metadata_spat.txt --output-dir core-metrics-results-spat


###generate taxa-plots


qiime taxa barplot --i-table spat_filtered_table.qza --i-taxonomy taxonomy-data2.qza --m-metadata-file Marissa_Metadata_spat.txt --o-visualization taxa-plts-spat.qzv



###PERMANOVA with spat data



qiime diversity beta-group-significance \

  --i-distance-matrix core-metrics-results-spat/unweighted_unifrac_distance_matrix.qza \

  --m-metadata-file Marissa_Metadata_spat.txt \

  --m-metadata-column Tank_treatment \

  --o-visualization core-metrics-results-spat/unweighted-unifrac-donor-significance.qzv


****no significant differences for unweighted***


qiime diversity beta-group-significance \

  --i-distance-matrix core-metrics-results-spat/weighted_unifrac_distance_matrix.qza \

  --m-metadata-file Marissa_Metadata_spat.txt \

  --m-metadata-column Tank_treatment \

  --o-visualization core-metrics-results-spat/weighted-unifrac-donor-significance.qzv


****no significant differences for weighted***

