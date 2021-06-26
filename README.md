Cut\&Run Analysis Pipeline Example
================
Chris Sansam and Tyler Noble
6/25/21

  - [Scripts and Functions to Run](#scripts-and-functions-to-run)
      - [Locations on Cluster:](#locations-on-cluster)
      - [Locations on Github](#locations-on-github)
  - [Setup directory](#setup-directory)
  - [Run script](#run-script)
  - [Cleanup folder](#cleanup-folder)

# Scripts and Functions to Run

## Locations on Cluster:

<smb://data/Sansam/hpc-nobackup/scripts/CutAndRun_2020Aug1/analyzeCutAndRunPeaks2.sh>
<smb://data/Sansam/hpc-nobackup/scripts/CutAndRun_2020Aug1/functions/peakAnalysisFunctions02.sh>

## Locations on Github

<https://github.com/SansamLab/SansamLab-SansamLabClusterScripts/blob/main/CutAndRun_2020Aug1/analyzeCutAndRunPeaks2.sh>
<https://github.com/SansamLab/SansamLab-SansamLabClusterScripts/blob/main/CutAndRun_2020Aug1/functions/peakAnalysisFunctions02.sh>

# Setup directory

``` bash
# make and change into directory
mkdir 2021Jun25_CutRunAnalysisExample
cd 2021Jun25_CutRunAnalysisExample
# copy input table from github to directory
wget https://raw.githubusercontent.com/SansamLab/CutAndRunPipeLineExample/main/MZ1sheet_v2.txt
# substitute dos line endings
sed -i.bak 's/\r$//' MZ1sheet_v2.txt
```

# Run script

``` bash
cd 2021Jun25_CutRunAnalysisExample
sbatch --wrap="/Volumes/Sansam/hpc-nobackup/scripts/CutAndRun_2020Aug1/analyzeCutAndRunPeaks2.sh MZ1sheet_v2.txt"
```

# Cleanup folder

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
