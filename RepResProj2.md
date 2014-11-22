# Reproducible Research: Peer Assessment 2
## Data Processing
We begin by loading libraries and setting a few global parameters:

```r
  library(knitr)
  #opts_chunk$set(echo=TRUE)       ## set global parameter for echo
  setwd("~/datasciencecoursera/RepResProj2/")
```

We first download and unzip the data (if necessary):

```r
  #Download file if it does not exist

  if (!file.exists("repdata-data-StormData.csv.bz2")) {
      message("Downloading data...")
      fileURL <- "http://bit.ly/1uNSAQY"
      zipfile = "repdata-data-StormData.csv.bz2"
      download.file(fileURL, destfile=zipfile, method="curl")
  }
```

We then read the data into R

```r
  # Load the csv file and assign it to a variable
  file   = "repdata-data-StormData.csv.bz2"
  raw    =  read.csv(file,stringsAsFactors = FALSE)
```

## Synopsis
The purpose of the analysis is to determine which types of events are most harmful with respect to population health in the United States.

## Results

To begin the analysis, we explore the overall layout of the data:

```r
dim(raw)
```

```
## [1] 902297     37
```

```r
names(raw)
```

```
##  [1] "STATE__"    "BGN_DATE"   "BGN_TIME"   "TIME_ZONE"  "COUNTY"    
##  [6] "COUNTYNAME" "STATE"      "EVTYPE"     "BGN_RANGE"  "BGN_AZI"   
## [11] "BGN_LOCATI" "END_DATE"   "END_TIME"   "COUNTY_END" "COUNTYENDN"
## [16] "END_RANGE"  "END_AZI"    "END_LOCATI" "LENGTH"     "WIDTH"     
## [21] "F"          "MAG"        "FATALITIES" "INJURIES"   "PROPDMG"   
## [26] "PROPDMGEXP" "CROPDMG"    "CROPDMGEXP" "WFO"        "STATEOFFIC"
## [31] "ZONENAMES"  "LATITUDE"   "LONGITUDE"  "LATITUDE_E" "LONGITUDE_"
## [36] "REMARKS"    "REFNUM"
```

End line. + 2
