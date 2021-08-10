Cut\&Run Analysis Pipeline Example
================
Chris Sansam and Tyler Noble
6/25/21

  - [Align fastq files](#align-fastq-files)
      - [Scripts and Functions to Run for
        Alignment](#scripts-and-functions-to-run-for-alignment)
          - [Locations on Cluster:](#locations-on-cluster)
          - [Locations on Github](#locations-on-github)
      - [Make directory on the cluster](#make-directory-on-the-cluster)
      - [Generate input table for
        alignment](#generate-input-table-for-alignment)
      - [Run alignment pipeline](#run-alignment-pipeline)
  - [Downstream Analysis Pipeline](#downstream-analysis-pipeline)
      - [Scripts and Functions to Run for Downstream Analysis
        Pipeline](#scripts-and-functions-to-run-for-downstream-analysis-pipeline)
          - [Locations on Cluster:](#locations-on-cluster-1)
          - [Locations on Github](#locations-on-github-1)
      - [Setup directory](#setup-directory)
      - [Run script](#run-script)
      - [Cleanup folder](#cleanup-folder)
  - [Make a custom heat plot - STILL UNDER
    DEVELOPMENT\!](#make-a-custom-heat-plot---still-under-development)

last updated: 7/2/21

# Align fastq files

## Scripts and Functions to Run for Alignment

### Locations on Cluster:

<smb://data/Sansam/hpc-nobackup/scripts/batchTableMaker_ver03.sh>
<smb://data/Sansam/hpc-nobackup/scripts/o3-batcher_ver02.sh>
<smb://data/Sansam/hpc-nobackup/scripts/CutAndRun_2020Aug1/align-fastq-CutAndRun_3SpeciesChimeria.sh>

### Locations on Github

<https://github.com/SansamLab/SansamLab-SansamLabClusterScripts/blob/main/batchTableMaker_ver03.sh>
<https://github.com/SansamLab/SansamLab-SansamLabClusterScripts/blob/main/o3-batcher_ver02.sh>
<https://github.com/SansamLab/SansamLab-SansamLabClusterScripts/blob/main/CutAndRun_2020Aug1/align-fastq-CutAndRun_3SpeciesChimeria.sh>

## Make directory on the cluster

``` bash
mkdir /Volumes/Sansam/hpc-nobackup/2021Jul01_HeLaCutAndRunAlignment
cd /Volumes/Sansam/hpc-nobackup/2021Jul01_HeLaCutAndRunAlignment
```

## Generate input table for alignment

Using the batchTableMaker\_ver03.sh script, specify the container with
“-c” and a grep term with “-g”

``` bash
cd /Volumes/Sansam/hpc-nobackup/2021Jul01_HeLaCutAndRunAlignment

# Make input table for 2021 May 17 samples
/Volumes/Sansam/hpc-nobackup/scripts/batchTableMaker_ver03.sh \
-g 2021May17_TDN_Cut-Run_Hela_Asynch \
-c 2021
mv 2021.inputTable.txt May17_input_table.txt

# Make input table for 2021 May 26 samples
/Volumes/Sansam/hpc-nobackup/scripts/batchTableMaker_ver03.sh \
-g 2021May26_TDN_Cut-Run_Hela_Asynch_TICRR \
-c 2021
mv 2021.inputTable.txt May26_input_table.txt
```

## Run alignment pipeline

``` bash
cd /Volumes/Sansam/hpc-nobackup/2021Jul01_HeLaCutAndRunAlignment

/Volumes/Sansam/hpc-nobackup/scripts/o3-batcher_ver02.sh \
-i May17_input_table.txt \
-s /Volumes/Sansam/hpc-nobackup/scripts/CutAndRun_2020Aug1/\
align-fastq-CutAndRun_3SpeciesChimeria.sh \
-m paired \
-c "--cpus-per-task 6 --mem 48G" \
-- \
-r /Volumes/shared-refs/hg19-s288c-ecolli-chimera/hg19-s288c-ecolli-chimera \
-t 6

/Volumes/Sansam/hpc-nobackup/scripts/o3-batcher_ver02.sh \
-i May26_input_table.txt \
-s /Volumes/Sansam/hpc-nobackup/scripts/CutAndRun_2020Aug1/\
align-fastq-CutAndRun_3SpeciesChimeria.sh \
-m paired \
-c "--cpus-per-task 6 --mem 48G" \
-- \
-r /Volumes/shared-refs/hg19-s288c-ecolli-chimera/hg19-s288c-ecolli-chimera \
-t 6
```

The processed and indexed bam files as well as the readStats were moved
to the object store in the CutAndRunBams folder

# Downstream Analysis Pipeline

## Scripts and Functions to Run for Downstream Analysis Pipeline

### Locations on Cluster:

<smb://data/Sansam/hpc-nobackup/scripts/CutAndRun_2020Aug1/analyzeCutAndRunPeaks2.sh>
<smb://data/Sansam/hpc-nobackup/scripts/CutAndRun_2020Aug1/functions/peakAnalysisFunctions02.sh>

### Locations on Github

<https://github.com/SansamLab/SansamLab-SansamLabClusterScripts/blob/main/CutAndRun_2020Aug1/analyzeCutAndRunPeaks2.sh>
<https://github.com/SansamLab/SansamLab-SansamLabClusterScripts/blob/main/CutAndRun_2020Aug1/functions/peakAnalysisFunctions02.sh>

### Input table description
Column 1:  Treatment sample Object store container
Column 2:  Treatment sample Object store file prefix
Column 3:  Treatment sample filename
Column 4:  Control sample Object store container
Column 5:  Control sample Object store file prefix
Column 6:  Control sample filename
Column 7:  Sample label
Column 8:  Replicate grouping label

## Setup directory

``` bash
# make and change into directory
mkdir 2021Jun25_CutRunAnalysisExample
cd 2021Jun25_CutRunAnalysisExample
# copy input table from github to directory
wget https://raw.githubusercontent.com/SansamLab/CutAndRunPipeLineExample/main/MZ1sheet_v2.txt
# substitute dos line endings
sed -i.bak 's/\r$//' MZ1sheet_v2.txt
```

## Run script

``` bash
cd 2021Jun25_CutRunAnalysisExample
sbatch --wrap="/Volumes/Sansam/hpc-nobackup/scripts/CutAndRun_2020Aug1/analyzeCutAndRunPeaks2.sh MZ1sheet_v2.txt"
```

## Cleanup folder

``` bash
cd 2021Jun25_CutRunAnalysisExample

# Make function for creating directory and moving files
function cleanup {
mkdir ${1}
mv ${2} ${1}
}

# Delete all .bam and .bam.bai files
rm *.bam*

# Run function to move files into directories
cleanup FrIPs "FRiPs.txt"
cleanup logFiles2 "*.out"
cleanup readStats "*readStats"
cleanup LibraryNormBigwigs "*_LibSize.bw"
cleanup EColiNormBigwigs "*.bw"
cleanup AllPeaksBedFiles "*all.bed"
cleanup OptimalPeaksBedFiles "*optimal.bed"
cleanup MatricesForPeakHeatPlots "*.gz"
cleanup TabFilesForPeakHeatPlots "*.tab"
cleanup PdfsForPeakHeatPlots "*.pdf"
cleanup ReadCountsInAllPeaks "*counts.txt"
```

# Make a custom heat plot - STILL UNDER DEVELOPMENT\!

``` bash
# make an array with the sample names you want

sampleNames=(MTBP_250_Cis MTBP_250_MZ1)
array=($(for smpleName in ${sampleNames[@]}; do
    grep $smpleName MZ1sheet_v2.txt
done | awk '{print $8}'))
echo ${array[@]}

function getArrayColumn {
filename="$1"
shift
column="$1"
shift
local arr=("$@")
array=($(for smpleName in "${arr[@]}"; do
    grep $smpleName $filename
done | awk '{print $8}'))
echo ${array[@]}
}

sampleNames=(MTBP_250_Cis MTBP_250_MZ1)

test=($(getArrayColumn "${sampleNames[@]}"))
```
