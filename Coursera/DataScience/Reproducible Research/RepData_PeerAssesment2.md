# Public health and economic problems for communities and municipalities caused by storms and other severe weather events
Onur Akpolat  
21. October 2014  

##Synopsis

Storms and other severe weather events can cause both public health and economic problems for communities and municipalities. This report is based on data from the U.S. National Oceanic and Atmospheric Administration's (NOAA). It shows the most harmful events that can be affect the united states, in order to take some decisions related to the future investments in planning and management of damages.

The two questions this report answers are:

1. Across the United States, which types of events (as indicated in the `EVTYPE` variable) are most harmful with respect to population health?

2. Across the United States, which types of events have the greatest economic consequences?

##Data Processing

In this section the initial configuration as well as loading and processing the data will be described.

### Initial configuration

The initial configuration consists of loading the required packages and initializing some variables.


```r
#Variables
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
#Directories
if (!file.exists("data")){
  dir.create("data")
}
#R-Packages:
IsRutilsInstalled <- require("R.utils")
```

```
## Loading required package: R.utils
## Loading required package: R.oo
## Loading required package: R.methodsS3
## R.methodsS3 v1.6.1 (2014-01-04) successfully loaded. See ?R.methodsS3 for help.
## R.oo v1.18.0 (2014-02-22) successfully loaded. See ?R.oo for help.
## 
## Attaching package: 'R.oo'
## 
## The following objects are masked from 'package:methods':
## 
##     getClasses, getMethods
## 
## The following objects are masked from 'package:base':
## 
##     attach, detach, gc, load, save
## 
## R.utils v1.34.0 (2014-10-07) successfully loaded. See ?R.utils for help.
## 
## Attaching package: 'R.utils'
## 
## The following object is masked from 'package:utils':
## 
##     timestamp
## 
## The following objects are masked from 'package:base':
## 
##     cat, commandArgs, getOption, inherits, isOpen, parse, warnings
```

```r
if(!IsRutilsInstalled){
    install.packages("R.utils")
    library(R.utils)
    }

IsGgplot2Installed <- require("ggplot2")
```

```
## Loading required package: ggplot2
```

```r
if(!IsGgplot2Installed){
    install.packages("ggplot2")
    library("ggplot2")
    }

IsReshape2Installed <- require("reshape2")
```

```
## Loading required package: reshape2
```

```r
if(!IsReshape2Installed){
    install.packages("reshape2")
    library("reshape2")
    }
```

### Loading the data

This sections describes the downloading the data from the web and loading it int a data frame.


```r
if (!file.exists("data/repdata-data-StormData.csv.bz2")){
        download.file(url, destfile="data/repdata-data-StormData.csv.bz2", method="curl")
}
weatherData <- read.csv("data/repdata-data-StormData.csv.bz2", header=TRUE, fileEncoding="ISO-8859-1")
```

### Cleanup and process data

First the event types need to be cleaned up.


```r
#Food-Data
weatherData$EVTYPE <- gsub("^.*((?!FLASH).).*FLOOD.*$","FLOOD",weatherData$EVTYPE, perl=TRUE)
weatherData$EVTYPE <- gsub("^URBAN.*$","FLOOD",weatherData$EVTYPE)
weatherData$EVTYPE <- gsub("^RIVER.*$","FLOOD",weatherData$EVTYPE)
#Tornado-Data
weatherData$EVTYPE <- gsub(".*TORNADO.*","TORNADO",weatherData$EVTYPE)
#Thunderstorm-Wind-Data
weatherData$EVTYPE <- gsub("^.*THUNDER.*WIND.*$","THUNDERSTORM WIND",weatherData$EVTYPE)
weatherData$EVTYPE <- gsub(".*TSTM.*","THUNDERSTORM WIND",weatherData$EVTYPE)
```

Second, PROPDMGEXP and CROPDMGEXP need to be normalized so the damage values can be determined.


```r
weatherData$PROPDMGEXP <- gsub("^[^0-9A-Za-z]$","1",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^0*$","1",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^[Kk]$","1000",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^[Mm]$","1000000",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^[Bb]$","1000000000",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^[Hh]$","10000",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^2$","100",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^3$","1000",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^4$","10000",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^5$","100000",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^6$","1000000",weatherData$PROPDMGEXP)
weatherData$PROPDMGEXP <- gsub("^7$","10000000",weatherData$PROPDMGEXP)
weatherData$PROPCOST <- as.numeric(weatherData$PROPDMGEXP) * weatherData$PROPDMG
weatherData$CROPDMGEXP <- gsub("^[^0-9A-Za-z]$","1",weatherData$CROPDMGEXP)
weatherData$CROPDMGEXP <- gsub("^0*$","1",weatherData$CROPDMGEXP)
weatherData$CROPDMGEXP <- gsub("^[Kk]$","1000",weatherData$CROPDMGEXP)
weatherData$CROPDMGEXP <- gsub("^[Mm]$","1000000",weatherData$CROPDMGEXP)
weatherData$CROPDMGEXP <- gsub("^[Bb]$","1000000000",weatherData$CROPDMGEXP)
weatherData$CROPCOST <- as.numeric(weatherData$CROPDMGEXP) * weatherData$CROPDMG
```

Last, aggregations are performed and the plotting will be prepared.


```r
#Aggregation for question1
weatherData$FATALITIES <- weatherData$FATALITIES*5
firstQuestionData <- aggregate(weatherData[,c("FATALITIES", "INJURIES")],by=list(weatherData$EVTYPE),FUN="sum")
firstQuestionData <- cbind(firstQuestionData, TOTAL = apply(firstQuestionData[c("FATALITIES","INJURIES")],1,function(x) sum(x)))
firstQuestionData <- firstQuestionData[with(firstQuestionData,order(-TOTAL,-FATALITIES,-INJURIES)),]
names(firstQuestionData)[1] <- "EVTYPE"
firstQuestionData$EVTYPE <- as.factor(firstQuestionData$EVTYPE)
firstQuestionTop10 <- firstQuestionData[1:10,]
firstQuestionPlotdata <- melt(firstQuestionTop10,id="EVTYPE",measure.vars=c("INJURIES","FATALITIES"))

#Aggregation for question2
secondQuestionData <- aggregate(weatherData[,c("PROPCOST", "CROPCOST")],by=list(weatherData$EVTYPE),FUN="sum")
secondQuestionData <- cbind(secondQuestionData, TOTAL = apply(secondQuestionData[c("PROPCOST", "CROPCOST")],1,function(x) sum(x)))
secondQuestionData <- secondQuestionData[with(secondQuestionData,order(-TOTAL,-PROPCOST,-CROPCOST)),]
names(secondQuestionData)[1] <- "EVTYPE"
secondQuestionData$EVTYPE <- as.factor(secondQuestionData$EVTYPE)
secondQuestionTop10 <- secondQuestionData[1:10,]
secondQuestionPlotdata <- melt(secondQuestionTop10,id="EVTYPE",measure.vars=c("PROPCOST","CROPCOST"))
```
##Results 

This section provides the answers to the intial questions, by plotting and answering the results.

1. Across the United States, which types of events (as indicated in the `EVTYPE` variable) are most harmful with respect to population health?

```r
ggplot(firstQuestionPlotdata,aes(x=reorder(EVTYPE,value),y=value/1000,fill=EVTYPE)) +
  geom_bar(stat="identity") +
  coord_flip() +
  labs(x="",y="Count of injury and weigthed fatality in thousands") +
  ggtitle("Storm Impact on Human Health\n") +
  theme(plot.title = element_text(lineheight=.8, face="bold",size=20)) +
  guides(fill=F)
```

![](RepData_PeerAssesment2_files/figure-html/firstQuestionPlot-1.png) 
**Answer:** The most harmful type of event for the population health is the **Tornado**, because it caused the most amount of *injuries and fatalities* in the United states.

2. Across the United States, which types of events have the greatest economic consequences?

```r
ggplot(secondQuestionPlotdata,aes(x=reorder(EVTYPE,value),y=value/1000000000,fill=EVTYPE)) +
  geom_bar(stat="identity") +
  coord_flip() +
  labs(x="",y="Cost in billions") +
  ggtitle("Storm Impact on Economics\n") +
  theme(plot.title = element_text(lineheight=.8, face="bold",size=20)) +
  guides(fill=F)
```

![](RepData_PeerAssesment2_files/figure-html/secondQuestionPlot-1.png) 
**Answer:** the greatest economic consequences caused by type of event is the **Floods**, because it caused the most amount of  damages in properties in the United states.
