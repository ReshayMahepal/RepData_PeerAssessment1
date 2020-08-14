---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site:

Dataset: Activity monitoring data [52K]
The variables included in this dataset are:
- steps: Number of steps taking in a 5-minute interval (missing values are coded as \color{red}{\verb|NA|}NA)
- date: The date on which the measurement was taken in YYYY-MM-DD format
- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Requirements:

1. Code for reading in the dataset and/or processing the data
2. Histogram of the total number of steps taken each day
3. Mean and median number of steps taken each day
4. Time series plot of the average number of steps taken
5. The 5-minute interval that, on average, contains the maximum number of steps
6. Code to describe and show a strategy for imputing missing data
7. Histogram of the total number of steps taken each day after missing values are imputed
8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
9. All of the R code needed to reproduce the results (numbers, plots, etc.) in the report


## Loading and preprocessing the data

The csv file containing the activity data is loaded into a data frame called actData using the code below.The structure of the data, dimensions as well as the first and last 10 rows are viewed


```r
        actData <- data.frame(read.csv("activity.csv", header = TRUE))
        str(actData)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
        dim(actData)
```

```
## [1] 17568     3
```

```r
        head(actData, 5)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
```

```r
        tail(actData, 5)
```

```
##       steps       date interval
## 17564    NA 2012-11-30     2335
## 17565    NA 2012-11-30     2340
## 17566    NA 2012-11-30     2345
## 17567    NA 2012-11-30     2350
## 17568    NA 2012-11-30     2355
```

The data loaded is further processed to remove the missing values. The data is stored in a data frame named actDataComplete.


```r
        actDataComplete <- data.frame(actData[complete.cases(actData),])
        str(actDataComplete)
```

```
## 'data.frame':	15264 obs. of  3 variables:
##  $ steps   : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 2 2 2 2 2 2 2 2 2 2 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
        dim(actDataComplete)
```

```
## [1] 15264     3
```

```r
        head(actDataComplete, 5)
```

```
##     steps       date interval
## 289     0 2012-10-02        0
## 290     0 2012-10-02        5
## 291     0 2012-10-02       10
## 292     0 2012-10-02       15
## 293     0 2012-10-02       20
```

```r
        tail(actDataComplete, 5)
```

```
##       steps       date interval
## 17276     0 2012-11-29     2335
## 17277     0 2012-11-29     2340
## 17278     0 2012-11-29     2345
## 17279     0 2012-11-29     2350
## 17280     0 2012-11-29     2355
```

## What is mean total number of steps taken per day?

The code below illustrates the method used to compute the total steps daily. The resulting data set is stored in the "totalSteps" object.


```r
        totalSteps <- group_by(actDataComplete, date)
        totalSteps <- summarise(totalSteps, dailySteps = sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
        head(totalSteps, 5)
```

```
## # A tibble: 5 x 2
##   date       dailySteps
##   <fct>           <int>
## 1 2012-10-02        126
## 2 2012-10-03      11352
## 3 2012-10-04      12116
## 4 2012-10-05      13294
## 5 2012-10-06      15420
```

```r
        mean(totalSteps$dailySteps)
```

```
## [1] 10766.19
```

The histogram illustrating the "Total number of steps taken each day" is generated in the *base* plotting system.


```r
        hist(totalSteps$dailySteps, main = "Total number of steps taken each day", xlab = "Total Steps per day", col = "red")
```

<img src="RepData_PA1_files/figure-html/histogram total steps per day-1.png" style="display: block; margin: auto;" />

The mean and median are found in the summary below:


```r
      sumTbl <- data.frame(Mean=mean(totalSteps$dailySteps), Median = median(totalSteps$dailySteps), stringsAsFactors=FALSE) 
      sumTbl <- xtable(sumTbl, digits = c(0,0,0))
      print(sumTbl, type ="html")
```

<!-- html table generated in R 3.6.3 by xtable 1.8-4 package -->
<!-- Sat Aug 15 01:10:53 2020 -->
<table border=1>
<tr> <th>  </th> <th> Mean </th> <th> Median </th>  </tr>
  <tr> <td align="right"> 1 </td> <td align="right"> 10766 </td> <td align="right"> 10765 </td> </tr>
   </table>

## What is the average daily activity pattern?

The object "dailyAct" contains the average steps ("aveSteps") per 5 minute interval across all days. 


```r
        dailyAct <- group_by(actDataComplete, interval)
        dailyAct <- summarise(dailyAct, aveSteps = mean(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
        head(dailyAct, 5)
```

```
## # A tibble: 5 x 2
##   interval aveSteps
##      <int>    <dbl>
## 1        0   1.72  
## 2        5   0.340 
## 3       10   0.132 
## 4       15   0.151 
## 5       20   0.0755
```

The time series plot illustrating the average steps per 5 minute interval across all days from October 2012 to November 2012 is generated in the *ggplot2* plotting system.


```r
        ggplot(dailyAct, aes(x=interval, y=aveSteps)) + geom_line(color="red", size=1) + xlab("Interval") + ylab("Average no. of steps") + ggtitle("Average steps per 5 minute intervals, Oct-Nov 2012") + theme(plot.title = element_text(hjust = 0.5))
```

<img src="RepData_PA1_files/figure-html/time series plot-1.png" style="display: block; margin: auto;" />

The five minute interval with the maximum average steps is extracted using the R script as follows:


```r
        maxVal <- dailyAct[which.max(dailyAct$aveSteps),]
        print(maxVal)
```

```
## # A tibble: 1 x 2
##   interval aveSteps
##      <int>    <dbl>
## 1      835     206.
```

## Imputing missing values

The number of rows in the orginal data set that are incomplete and missing data is determined using the *Mice Package*. The *md.pattern* function is used to identify variables in the data set that are missing data. An explanantion of the package can be found at [datascience+](https://datascienceplus.com/imputing-missing-data-with-r-mice-package/).


```r
        missingData <- md.pattern(actData)
```

<img src="RepData_PA1_files/figure-html/missing data-1.png" style="display: block; margin: auto;" />

```r
        print(missingData)
```

```
##       date interval steps     
## 15264    1        1     1    0
## 2304     1        1     0    1
##          0        0  2304 2304
```

As seen above, across the 3 colums, there are **2304 rows** from the "steps" variable is missing data and incomplete.

The *mice* function then provides a simple method for imputing missing data. In the function below, we see that the *mice* function makes use of the *mean* method, i.e. *an unconditional mean imputation* to iteratively determine a suitable mean to impute as the missing values. The complete data set with imputed values is stored in the object "completeData"


```r
        tempData <- mice(actData,m=5,maxit=10,meth='mean',seed=500)
```

```
## 
##  iter imp variable
##   1   1  steps
##   1   2  steps
##   1   3  steps
##   1   4  steps
##   1   5  steps
##   2   1  steps
##   2   2  steps
##   2   3  steps
##   2   4  steps
##   2   5  steps
##   3   1  steps
##   3   2  steps
##   3   3  steps
##   3   4  steps
##   3   5  steps
##   4   1  steps
##   4   2  steps
##   4   3  steps
##   4   4  steps
##   4   5  steps
##   5   1  steps
##   5   2  steps
##   5   3  steps
##   5   4  steps
##   5   5  steps
##   6   1  steps
##   6   2  steps
##   6   3  steps
##   6   4  steps
##   6   5  steps
##   7   1  steps
##   7   2  steps
##   7   3  steps
##   7   4  steps
##   7   5  steps
##   8   1  steps
##   8   2  steps
##   8   3  steps
##   8   4  steps
##   8   5  steps
##   9   1  steps
##   9   2  steps
##   9   3  steps
##   9   4  steps
##   9   5  steps
##   10   1  steps
##   10   2  steps
##   10   3  steps
##   10   4  steps
##   10   5  steps
```

```
## Warning: Number of logged events: 50
```

```r
        completeData <- complete(tempData,1)
        head(completeData)
```

```
##     steps       date interval
## 1 37.3826 2012-10-01        0
## 2 37.3826 2012-10-01        5
## 3 37.3826 2012-10-01       10
## 4 37.3826 2012-10-01       15
## 5 37.3826 2012-10-01       20
## 6 37.3826 2012-10-01       25
```

```r
        tail(completeData)
```

```
##         steps       date interval
## 17563 37.3826 2012-11-30     2330
## 17564 37.3826 2012-11-30     2335
## 17565 37.3826 2012-11-30     2340
## 17566 37.3826 2012-11-30     2345
## 17567 37.3826 2012-11-30     2350
## 17568 37.3826 2012-11-30     2355
```

```r
        summary(completeData)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 37.38   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##                   (Other)   :15840
```

The method used to generate the "totalSteps" data set is employed to create the "comDatSteps" data set, within which the steps are summed per day. A simliar histogram is plotted using the new data set. 


```r
        comDatSteps <- group_by(completeData, date)
        comDatSteps <- summarise(comDatSteps, dailySteps = sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
        head(comDatSteps, 5)
```

```
## # A tibble: 5 x 2
##   date       dailySteps
##   <fct>           <dbl>
## 1 2012-10-01     10766.
## 2 2012-10-02       126 
## 3 2012-10-03     11352 
## 4 2012-10-04     12116 
## 5 2012-10-05     13294
```

```r
        tail(comDatSteps, 5)
```

```
## # A tibble: 5 x 2
##   date       dailySteps
##   <fct>           <dbl>
## 1 2012-11-26     11162 
## 2 2012-11-27     13646 
## 3 2012-11-28     10183 
## 4 2012-11-29      7047 
## 5 2012-11-30     10766.
```

```r
        mean(comDatSteps$dailySteps)
```

```
## [1] 10766.19
```

```r
        summary(comDatSteps)
```

```
##          date      dailySteps   
##  2012-10-01: 1   Min.   :   41  
##  2012-10-02: 1   1st Qu.: 9819  
##  2012-10-03: 1   Median :10766  
##  2012-10-04: 1   Mean   :10766  
##  2012-10-05: 1   3rd Qu.:12811  
##  2012-10-06: 1   Max.   :21194  
##  (Other)   :55
```

```r
        hist(comDatSteps$dailySteps, main = "Total number of steps taken each day", xlab = "Total Steps per day", col = "green")
```

<img src="RepData_PA1_files/figure-html/histogram using imputed data set-1.png" style="display: block; margin: auto;" />

The mean and median of the imputed data set("comDatSteps") is calculated as follows:



```r
      meanMedTbl <- data.frame(Mean=mean(comDatSteps$dailySteps), Median = median(comDatSteps$dailySteps), stringsAsFactors=FALSE) 
      meanMedTbl <- xtable(meanMedTbl, digits = c(0,0,0),)
      print(meanMedTbl, type ="html")
```

<!-- html table generated in R 3.6.3 by xtable 1.8-4 package -->
<!-- Sat Aug 15 01:11:03 2020 -->
<table border=1>
<tr> <th>  </th> <th> Mean </th> <th> Median </th>  </tr>
  <tr> <td align="right"> 1 </td> <td align="right"> 10766 </td> <td align="right"> 10766 </td> </tr>
   </table>

As per the table above, we see that the mean value **remains the same** as per that determined when considering only the complete cases of the original data set. The median on the other hand has **increased by 1 from 10765 to 10766**. Similarly comparing the 2 histograms plotted, it is evident that imputing the missing data using the method as described above had a negilible impact of the estimates of the daily total number of steps.

## Are there differences in activity patterns between weekdays and weekends?

The *weekday()* function was used to determine the weekday or "dayName", and the variable added to the "completeData" data set. using the ifelse function, the "dayName" could be classified as weekend or weekday. However, in order to use this method, the date variable had to be converted from the factor class to the date class  


```r
        completeData <- data.frame(completeData)
        completeData$date <- as.Date(completeData$date, format = "%Y-%m-%d")
        completeData$dayName <- weekdays(completeData$date)
        completeData$dayDes = ifelse(completeData$dayName == "Saturday" | completeData$dayName == "Sunday","weekend","weekday")
        str(completeData)
```

```
## 'data.frame':	17568 obs. of  5 variables:
##  $ steps   : num  37.4 37.4 37.4 37.4 37.4 ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
##  $ dayName : chr  "Monday" "Monday" "Monday" "Monday" ...
##  $ dayDes  : chr  "weekday" "weekday" "weekday" "weekday" ...
```

```r
        head(completeData, 5)
```

```
##     steps       date interval dayName  dayDes
## 1 37.3826 2012-10-01        0  Monday weekday
## 2 37.3826 2012-10-01        5  Monday weekday
## 3 37.3826 2012-10-01       10  Monday weekday
## 4 37.3826 2012-10-01       15  Monday weekday
## 5 37.3826 2012-10-01       20  Monday weekday
```

```r
        tail(completeData, 5)
```

```
##         steps       date interval dayName  dayDes
## 17564 37.3826 2012-11-30     2335  Friday weekday
## 17565 37.3826 2012-11-30     2340  Friday weekday
## 17566 37.3826 2012-11-30     2345  Friday weekday
## 17567 37.3826 2012-11-30     2350  Friday weekday
## 17568 37.3826 2012-11-30     2355  Friday weekday
```

The comparison of the average steps per 5 minute intervals over weekends and weekdays is illustrated using a time series plot in the *ggplot2* plotting system 


```r
        compDailyData <- aggregate(steps ~ interval + dayDes, completeData, mean)
        ggplot(data = compDailyData, aes(x = interval, y = steps)) + 
        geom_line(color="green", size=1) +
        facet_grid(.~dayDes) +
        ggtitle("Average steps per 5 minute intervals, Oct-Nov 2012") +
        xlab("Interval") +
        ylab("Average no. of steps") +
        theme(plot.title = element_text(hjust = 0.5))
```

<img src="RepData_PA1_files/figure-html/time series plot for weekend and weekday-1.png" style="display: block; margin: auto;" />








