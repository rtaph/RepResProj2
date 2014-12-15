# Reproducible Research: Peer Assessment 2
******************************
## Most Harmful Weather Events

The purpose of the analysis is to determine which types of events are most harmful with respect to population health in the United States by using the [NOAA Storm Database](http://www.ncdc.noaa.gov/stormevents/). The paper attempts to answer two basic questions about severe weather events:

1. Across the United States, which types of events are most harmful with respect to population health?
2. Across the United States, which types of events have the greatest economic consequences?

### Synopsis

My analysis reveals that tornadoes and excessive heat are the most harmful weather event to human health in the United States. These conclusions are based on the total fatalities as well as injuries recorded for each type of event. Floods are the most economically damaging weather event, costing the economy approximately USD 144 billion per year.


### Data Processing
I begin the analysis by loading libraries and setting a few global parameters:


```r
  ## load needed libraries, set global options, and working directory
  library(knitr); library(plyr)
  opts_chunk$set(echo=TRUE)       
  setwd("~/Documents/Courses/datasciencecoursera/RepResProj2/")
```

I then check that the data file exists, download it (if needed), and unzip it:

```r
  #Download file if it does not exist
  if (!file.exists("repdata-data-StormData.csv.bz2")) {
      fileURL <- "http://bit.ly/1uNSAQY"
      zipfile = "repdata-data-StormData.csv.bz2"
      download.file(fileURL, destfile=zipfile, method="curl")
  }
```

The data is then read into R. As it is a large file, data is first read as character strings to improve performance and speed: 


```r
  # Load the data and assign it to a variable
  file   = "repdata-data-StormData.csv.bz2"
  raw    =  read.csv(file, stringsAsFactors = FALSE) # FALSE to optimize read speed
```

Information about the data can be summarized using the `str` command:


```r
str(raw)
```

```
## 'data.frame':	902297 obs. of  37 variables:
##  $ STATE__   : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ BGN_DATE  : chr  "4/18/1950 0:00:00" "4/18/1950 0:00:00" "2/20/1951 0:00:00" "6/8/1951 0:00:00" ...
##  $ BGN_TIME  : chr  "0130" "0145" "1600" "0900" ...
##  $ TIME_ZONE : chr  "CST" "CST" "CST" "CST" ...
##  $ COUNTY    : num  97 3 57 89 43 77 9 123 125 57 ...
##  $ COUNTYNAME: chr  "MOBILE" "BALDWIN" "FAYETTE" "MADISON" ...
##  $ STATE     : chr  "AL" "AL" "AL" "AL" ...
##  $ EVTYPE    : chr  "TORNADO" "TORNADO" "TORNADO" "TORNADO" ...
##  $ BGN_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ BGN_AZI   : chr  "" "" "" "" ...
##  $ BGN_LOCATI: chr  "" "" "" "" ...
##  $ END_DATE  : chr  "" "" "" "" ...
##  $ END_TIME  : chr  "" "" "" "" ...
##  $ COUNTY_END: num  0 0 0 0 0 0 0 0 0 0 ...
##  $ COUNTYENDN: logi  NA NA NA NA NA NA ...
##  $ END_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ END_AZI   : chr  "" "" "" "" ...
##  $ END_LOCATI: chr  "" "" "" "" ...
##  $ LENGTH    : num  14 2 0.1 0 0 1.5 1.5 0 3.3 2.3 ...
##  $ WIDTH     : num  100 150 123 100 150 177 33 33 100 100 ...
##  $ F         : int  3 2 2 2 2 2 2 1 3 3 ...
##  $ MAG       : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ FATALITIES: num  0 0 0 0 0 0 0 0 1 0 ...
##  $ INJURIES  : num  15 0 2 2 2 6 1 0 14 0 ...
##  $ PROPDMG   : num  25 2.5 25 2.5 2.5 2.5 2.5 2.5 25 25 ...
##  $ PROPDMGEXP: chr  "K" "K" "K" "K" ...
##  $ CROPDMG   : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ CROPDMGEXP: chr  "" "" "" "" ...
##  $ WFO       : chr  "" "" "" "" ...
##  $ STATEOFFIC: chr  "" "" "" "" ...
##  $ ZONENAMES : chr  "" "" "" "" ...
##  $ LATITUDE  : num  3040 3042 3340 3458 3412 ...
##  $ LONGITUDE : num  8812 8755 8742 8626 8642 ...
##  $ LATITUDE_E: num  3051 0 0 0 0 ...
##  $ LONGITUDE_: num  8806 0 0 0 0 ...
##  $ REMARKS   : chr  "" "" "" "" ...
##  $ REFNUM    : num  1 2 3 4 5 6 7 8 9 10 ...
```

Looking at the summary above, variables can be identified that match to the question of interest. In this analysis, I will use `EVTYPE` (the event type), `FATALITIES`, `INJURIES`, `PROPDMG` (monetary estimate of property damage), and `PROPDMGEXP` (unit used for the damage estimate).

Many of these variables need to be converted or manipulated into more workable formats:


```r
  # reformat data type of key variables
  raw$EVTYPE = as.factor(raw$EVTYPE)
  raw$BGN_DATE = as.POSIXlt(strptime(raw$BGN_DATE,format="%m/%d/%Y %H:%M:%S"))
  raw$DMG = ""
  raw$DMG = mapvalues(raw$PROPDMGEXP, c("B","M","m","K","H","h","0"),
                                      c(1e9,1e6,1e6,1e3,1e2,1e2,1))
  raw$DMG = as.numeric(raw$DMG) * raw$PROPDMG
```

```
## Warning: NAs introduced by coercion
```

```r
  raw$DMG = as.numeric(raw$DMG)
```

A new variable `DMG` is created to capture the monetary estimate of damages from weather events in a universal unit of measure. Although there are certain uncaught response types that cause NAs to be coerced, these cases are ambigous to interpret (even after reading the data help files). As they are few enough, they can likely be ignored without making a big difference on the exploratory analysis. 

By plotting the number of *unique* types of weather events per year below, it is apparent see that the initial period of data (~1950 to 1995) has few categorizations. I find it more likely that this absence of data is a result of lack of collection systems/standards, rather than an absence of particular types of events. I deem it more probable than not that including this initial period would bias the analysis away from type of events that only started being tracked recently.


```r
  t = table(format(raw$BGN_DATE,"%Y"))
  u  = as.numeric(tapply(raw$EVTYPE,raw$BGN_DATE[[6]], function(x) length(unique(x))))

  plot(t, type = "l", main = "Number of Records", ylab = "", col = "blue")
  par(new=T); plot(u, type='l', col = "red", lwd = 2, axes=F, xlab=NA, ylab=NA)
  axis(4, col = "red", lwd = 2)
  legend("topleft", c("Total","Unique"), lwd=c(2.5,2.5), col=c("blue","red"))
```

![](RepResProj2_files/figure-html/chunkExpl6-1.png) 

The number of unique weather events jumps sharply in 1995 (up to 387 records). Accordingly, I will work only with the subset of data from this date forward, as it is more representative, and will not bias the data towards type of weather events that were tracked earlier on in history.


```r
  df = raw[raw$BGN_DATE >= "1995-01-01 00:00:00",]
```
This subset nonetheless captures 75.6% of the observations in the raw data.

### Results

To answer the research questions, I will analyse which weather events cause the greatest number of fatalities, injuries, and economic damage.

To facilitate this, I have written a re-usable function that generate a summary of the absolute and relative impact of a particular parameter, by event type:


```r
  # a function that makes a summary table of a given parameter and calculates
  # relative impact (per weather event)
  mktab = function(variable){
    v = df[[variable]]
    dfa = aggregate(v ~ EVTYPE, data = df, sum)
    dfb = aggregate(v ~ EVTYPE, data = df, length)
    names(dfb)[2] = "OCCURENCES"; names(dfa)[2] = variable
    out = merge(dfa, dfb)
    out$RELIMPACT = out[[variable]] / out$OCCURENCES
    out = out[order(out[[variable]], decreasing = T),]
    out
  }
```

I use this function to explore the most harmful weather events, both on the basis of total fatalities and total injuries:


```r
  df1 = mktab("FATALITIES"); head(df1)
```

```
##             EVTYPE FATALITIES OCCURENCES  RELIMPACT
## 112 EXCESSIVE HEAT       1903       1673 1.13747759
## 665        TORNADO       1545      24251 0.06370871
## 134    FLASH FLOOD        930      52449 0.01773151
## 231           HEAT        924        755 1.22384106
## 357      LIGHTNING        725      14258 0.05084865
## 144          FLOOD        423      24473 0.01728435
```


```r
  df2 = mktab("INJURIES"); head(df2)
```

```
##             EVTYPE INJURIES OCCURENCES  RELIMPACT
## 665        TORNADO    21757      24251 0.89715888
## 144          FLOOD     6769      24473 0.27659053
## 112 EXCESSIVE HEAT     6525       1673 3.90017932
## 357      LIGHTNING     4627      14258 0.32451957
## 682      TSTM WIND     3625     128560 0.02819695
## 231           HEAT     2030        755 2.68874172
```

With respect to the second research question, I will use the `DMG` parameter. These property damage estimates were computed in the data processing stage using the `PROPDMG` and `PROPDMGEXP` variables. Using the same analysis as before, we obtain a table of the most costly events:


```r
  df3 = mktab("DMG"); head(df3)
```

```
##                EVTYPE          DMG OCCURENCES   RELIMPACT
## 49              FLOOD 143986183550      16830   8555328.8
## 129 HURRICANE/TYPHOON  69305840000         70 990083428.6
## 202       STORM SURGE  43193536000        167 258643928.1
## 243           TORNADO  24907463396      16081   1548875.3
## 43        FLASH FLOOD  15349730441      31719    483928.6
## 83               HAIL  14989598191      89714    167082.0
```

```r
  barplot(head(df3$DMG), main = "Weather Events Causing the Greatest Economic Damage, 1995-2011", ylab = "Estimated Cost (USD)")
  axis(1, at = 1:6, labels = head(df3$EVTYPE))
```

![](RepResProj2_files/figure-html/chunkExpl7-1.png) 

The above chart shows that on an absolute basis, floods have been the most costly weather event to Americans (USD 144 billion). This is followed by hurricanes/typhoons and storm surges.

However, by looking at the relative impact, we see that heaviest costs per event comes from `HEAVY RAIN/SEVERE WEATHER` (USD 2.5 billion).


```r
  df3[which.max(df3$RELIMPACT),]
```

```
##                       EVTYPE     DMG OCCURENCES RELIMPACT
## 99 HEAVY RAIN/SEVERE WEATHER 2.5e+09          1   2.5e+09
```

With only one observation above, this result appears suspect/mistaken. However, a review of the `REMARKS` variable for this entry reveals that this weather event was indeed substantial and costly:


```r
  df[df$EVTYPE %in% "HEAVY RAIN/SEVERE WEATHER" & df$PROPDMG == "2.5", "REMARKS"]
```

```
## [1] "A potent weather system stalled over southeast Louisiana from the evening of May 8 through mid day of May 10 producing two bouts of heavy rain and severe thunderstorms.  The first event occurred the evening of May 8 and early morning of May 9 producing several tornadoes and widespread heavy rain of 8 to 15 inches across the greater New Orleans metro area into southeast St. Tammany Parish.  The second bout of severe weather struck during the evening of May 9 and continued into the morning of May 10, producing rainfall of 10 to 15 inches primarily from St. Tammany Parish into south Mississippi.  Drainage capacity was overwhelmed by the torrential rainfall on each of these nights and water flooded tens of thousands of homes in southeast Louisiana.  By early June, the Federal Emergency Management Agency (FEMA) reported approximately 60,000 addresses representing single family units,  multifamily units, and businesses had filed for assistance due to weather damage.  In late May, the Red Cross estimated 36,000 homes in southeast Louisiana were affected by flood water.  Newspaper accounts indicated weather related damage to reach the $2.5 to $3.0 billion range.  Parish by parish detail follows. "
```

### Session Info


```r
  sessionInfo()
```

```
## R version 3.1.2 (2014-10-31)
## Platform: x86_64-apple-darwin13.4.0 (64-bit)
## 
## locale:
## [1] en_CA.UTF-8/en_CA.UTF-8/en_CA.UTF-8/C/en_CA.UTF-8/en_CA.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## loaded via a namespace (and not attached):
## [1] digest_0.6.6     evaluate_0.5.5   formatR_1.0      htmltools_0.2.6 
## [5] knitr_1.8        rmarkdown_0.3.10 stringr_0.6.2    tools_3.1.2     
## [9] yaml_2.1.13
```

