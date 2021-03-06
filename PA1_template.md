Reproducible Research Peer Assessment 1
========================================================

This markdown file was generated on Sun Jun 15 4:07:16 PM 2014 UTC using R version 3.0.3 (2014-03-06).

## Introduction

This assignment was completed to fulfill part of the requirements of the Johns Hopkins University Reproducible Research course.  The purpose is to demonstrate the use of the [knitr](http://cran.r-project.org/web/packages/knitr/) package as a means to achieve literate statistcal programming.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.


## Set parameters, load in and preprocess the data


```r
setwd("~/GitHub/RepData_PeerAssessment1-1")
Sys.setlocale("LC_TIME", "English")
suppressPackageStartupMessages(require(data.table))
suppressPackageStartupMessages(require(ggplot2))
suppressPackageStartupMessages(require(scales))
options(scipen = 1, digits = 7)
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
if(!file.exists("activity.csv")){
        download.file(fileUrl, destfile = "activity_monitoring_data.zip", 
                      method="auto")       
        unzip("activity_monitoring_data.zip")
}
activityData <- data.table(read.csv("activity.csv", header = TRUE, sep = ",",
        na.strings = "NA", colClasses = c("numeric", "Date", "numeric")))
activityData$interval <- sprintf("%04d", activityData$interval)
activityData$interval <- as.POSIXct(activityData$interval, format = "%H%M")
```

First the working directory was established.  *Note: if the reader wishes to execute the code herein, he/she may need to change the working directory.*  Next two packages are loaded: [data.table](http://cran.r-project.org/web/packages/data.table/index.html) for efficient processing of data tables, [ggplot2](http://ggplot2.org/) for producing the plots, and [scales](http://cran.r-project.org/web/packages/scales/index.html) for aid in formatting axes on plots. Finally the data were downloaded and loaded in as a data.table.


## What is mean total number of steps taken per day?


```r
transformedDataByDate <- activityData[, list(total = sum(steps)), by = date]
stepsMean <- round(mean(transformedDataByDate$total, na.rm = TRUE))
stepsMedian <- round(median(transformedDataByDate$total, na.rm = TRUE))
ggplot(transformedDataByDate, aes(x = total)) + geom_histogram(color = "blue", 
        fill = "white", binwidth = 1000) + geom_vline(aes(xintercept = 
        stepsMedian), color = "red", linetype = "dashed", size=1) + ylim(0, 18) + 
        ggtitle("Total Number of Steps Taken Per Day: Missing Data Ignored") + 
        xlab("Number of steps per day") + ylab("Count") + theme_bw()
```

![plot of chunk stepsbedayhistNAomit](figure/stepsbedayhistNAomit.png) 


Next the data was transformed, plotted, and the mean and median number of steps per day were calculated.  In this particular case, any missing values within the data were simply ignored.

The mean number of steps, rounded to the nearest step, taken per day was **10766**.
The median number of steps, rounded to the nearest step,  taken per day was **10765**.


## What is the average daily activity pattern?


```r
transformedDataByInterval <- activityData[, list(average = mean(steps, na.rm = 
        TRUE)), by=interval]
ggplot(transformedDataByInterval, aes(x = interval, y = average)) + 
        geom_line(color = "blue") + ggtitle("Average Daily Activity Pattern") +  
        ylab("Number of Steps") + xlab("Time of Day") + 
        scale_x_datetime(labels = date_format("%H:%M"), breaks = 
        date_breaks("2 hours")) + theme_bw()
```

![plot of chunk avgdailypattern](figure/avgdailypattern.png) 

```r
maxInterval<-strftime(transformedDataByInterval[which.max(
        transformedDataByInterval$average),interval], format = "%H:%M")
```

Next the average daily activity pattern was plotted.  

The 5- minute time interval, on average across all the days in the dataset, containing 
the maximum number of steps ends at **08:35**.


## Determing the effect of imputing missing values


```r
numberNA <- length(is.na(activityData$steps)[is.na(activityData$steps) == TRUE])
imputedActivityData <- activityData
imputedActivityData$steps <- ifelse(is.na(activityData$steps), 
        transformedDataByInterval[match(activityData$interval, 
        transformedDataByInterval$interval), transformedDataByInterval$average], 
        activityData$steps)
```

In the original data set, there exists **2304** missing values in the dataset.


As a result, missing values were imputed by substituting the mean for that 5-minute interval across all days for the missing value.  Total number of steps taken per day were then replotted for the data including imputed values.  


```r
transformedImputedDataByDate <- imputedActivityData[, list(total = sum(steps)),
        by = date]
stepsImputedMean <- round(mean(transformedImputedDataByDate$total))
stepsImputedMedian <- round(median(transformedImputedDataByDate$total))
ggplot(transformedImputedDataByDate, aes(x = total)) + geom_histogram(color =
        "blue", fill = "white", binwidth = 1000) + geom_vline(aes(
        xintercept = stepsMedian), color = "red", linetype = "dashed", 
        size = 1) + ylim(0, 18) + ggtitle(
        "Total Number of Steps Taken Per Day: Missing Data Imputed") + 
        xlab("Number of steps per day") + ylab("Count") + theme_bw()
```

![plot of chunk stepsbedayhistimputeNA](figure/stepsbedayhistimputeNA.png) 

Then the histograms corresponding to missing values that were ignored and missing values that were imputed were overlaid to visualize the results of imputation. 


```r
transformedDataByDateRemove <- transformedDataByDate
transformedDataByDateRemove$datasource <- "Remove Missing"
transformedDataByDateImpute <- transformedImputedDataByDate
transformedDataByDateImpute$datasource <- "Impute Missing"
transformedDataByDateCombine <- rbind(transformedDataByDateRemove,
        transformedDataByDateImpute)
ggplot(transformedDataByDateCombine, aes(x = total, fill = datasource)) + 
        geom_histogram(color = "black", binwidth = 1000, alpha = .5, 
        position = "identity")+ xlab("Number of steps per day") + ylab("Count") + 
        theme_bw() + theme(legend.position = c(0.875, 0.9),
        legend.background = element_rect(color = "black", size = 1, 
        linetype = "solid"))
```

![plot of chunk stepsbedayhistomitandimputeNA](figure/stepsbedayhistomitandimputeNA.png) 

Changes in mean and median as result of imputation were negligible.  For example,
the preimutation mean and median were **10766** and **10765**, 
steps respectively, while the postimputation mean and median were **10766** and **10766** steps respectively.

These results are perhaps expected as often entire days worth of data went missing from the original data set, and the imputed values were themselves derived from average figures. As a result, the imputed data were simply appended to the existing mean.  Perhaps a more sophisticated imputation approach is warranted.


## Are there differences in activity patterns between weekdays and weekends?


```r
imputedActivityDataWeek <- imputedActivityData
imputedActivityDataWeek$weekdayorweekend <- ifelse(weekdays(
        imputedActivityDataWeek$date) %in% c("Saturday", "Sunday"), "weekend", 
        "weekday")
transformedimputedActivityDataWeek <- imputedActivityDataWeek[, list(
        average = mean(steps)), by = list(interval, weekdayorweekend)]
ggplot(transformedimputedActivityDataWeek, aes(x = interval, y = average)) + 
        geom_line(color = "blue")+ facet_wrap(~weekdayorweekend, ncol = 1) +
        ggtitle("Average Daily Activity Pattern: Weekday vs. Weekend") + 
        ylab("Number of steps") + xlab("Time of Day") + 
        scale_x_datetime(labels = date_format("%H:%M"), breaks = 
        date_breaks("2 hours")) + theme_bw() 
```

![plot of chunk avgdailypatternbydayofweek](figure/avgdailypatternbydayofweek.png) 

Next, we sought to determine whether differences in the activity pattern existed
between weekdays and weekends.  Average daily activity data were stratified by weekday or weekend and then replotted.  While the periods of inactivity were similar regardless of the days of the week, the periodocity of activity during active periods were clearly different.


