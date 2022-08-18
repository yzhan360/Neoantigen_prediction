INPUT
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Paired-end reads: R1 & R2
Reference indexes: hg38 & rdna
Compartment bed files: intron, exon & inter-genic
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


SYSTEM PREREQUISITES
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
OS: UNIX/LINUX 
Resources: 1-8 cores & 8-64GB of memory
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


PIPELINE PREREQUISITES
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
SOFTWARE PACKAGES: fastqc_v0.11.3, cutadapt_v1.15.0, trimgalore_v0.6.3, rsem_v1.3.0, star/2.5.2b, samtools_v0.1.19, bowtie2_v2.2.5, picard_v2.18.2, bedtools_v2.22.1, java_v1.8.0_45, md5sum_v8.4, R_v3.3.1 & perl_v5.10.1 (any version of md5sum, R & perl would do)
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


PIPELINE USAGE
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Create a directory for scripts & copy all the scripts in to it
mkdir scripts
cd scripts

# Create a text file containing the samplenames
for i in ../$PROJECTID_*; do samplename=`echo $i | sed -e s/...$PROJECTID_//`; echo $samplename; done > samplenames.txt

# Create the analysis directories & copy the scripts to analysis directory
qsub -v projectid=$PROJECTID 01_runCreateDirectories.sh

# Run the md5sum check
qsub 02_runMd5Sum.sh

# Get the read count and match R1/R2 counts
qsub 03_runGetCount.sh

# Run fastqc
for i in `cat samplenames.txt`; do cd /data/ngsc4data/$PROJECTID/$PROJECTID_$i; qsub -v projectid=$PROJECTID 04_runFastqc.sh; done

# Run Trimgalore, RSEM alignments, RDNA alignments, picard stats & compartment counts
cd ../scripts/
for i in `cat samplenames.txt`; do cd /data/ngsc4data/$PROJECTID/$PROJECTID_$i; qsub -v in=$i,projectid=$PROJECTID 05_runRNASeq_workflow.sh; done

# Generate stats summary
cd ../scripts
qsub 06_runGenerateStatsSummary.sh

# Generate expected counts summary 
cd ../$PROJECTID_000_analysis/RSEM/tables/genes/
qsub 07_runSummarizeExpectedCounts.sh

# Replace empty gene-name entries with NAs
cd ../isoforms/
qsub 08_runFixGeneNames.sh
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
