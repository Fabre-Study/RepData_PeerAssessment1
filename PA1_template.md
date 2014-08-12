---
title: "Reproducible Research - Peer Assessment 1"
author: ' by Jorge Fabre'
output: html_document
---

**GITHUB link - **
<https://github.com/Fabre-Study/RepData_PeerAssessment1>

##---------------------------------------------------------------------------

## Loading and preprocessing the data

Unzip and Read the data from "activity.csv"

```{r,echo=TRUE}
unzip("repdata_data_activity.zip")
my.result <- read.csv("activity.csv")
```

Convert the class of the date.

```{r,echo=TRUE}
my.result$date <- as.Date(my.result$date)
```

Show the head of the dateset loaded.

```{r,echo=TRUE}
head(my.result)
```

## What is mean total number of steps taken per day?

Histogram with the total number of steps taken each day (no missing values).

```{r,echo=TRUE}
my.histogram <- aggregate(x=my.result$steps, by=list(my.result$date), FUN=sum, na.rm=TRUE)
names(my.histogram) <- c("date","steps")
library(ggplot2)
qplot(x=my.histogram$date, y=my.histogram$steps, xlab="Date", ylab="Steps", stat = "identity", geom="bar")
```

Reporting **mean** and **median** of toyal number of steps taken per day.

**Mean**
```{r,echo=TRUE}
mean(my.histogram$steps)
```

**Median**
```{r,echo=TRUE}
median(my.histogram$steps)
````

## What is the average daily activity pattern?

Histogram with the total number of steps taken each day (no missing values).
Create a new dataset with the aggregated data and plot the result.

```{r,echo=TRUE}
my.average.time <- aggregate(x=my.result$steps, by=list(my.result$interval), FUN=mean, na.rm=TRUE)
names(my.average.time) <- c("interval","steps")
qplot(x=my.average.time$interval, y=my.average.time$steps, xlab="Interval (5min)", ylab="Steps", stat = "identity", geom="line")
```

Which 5-minute interval, contains the maximum number of steps?

**5 minutes interval**
```{r,echo=TRUE}
which.max(my.average.time$steps)
```

## Imputing missing values

Total number of missing values in the dataset (dataset and steps)

**Missing values**
```{r,echo=TRUE}
sum(is.na(my.result))
sum(is.na(my.result$steps))
```

Strategy for filling in all of the missing values in the dataset. As suggested in the PA-1, it was used the mean for that 5-minute interval.

Merge the original dataset with "mean that 5-minute interval", and fill the missing values.
```{r,echo=TRUE}
my.result.imput <- merge(my.result, my.average.time, by.x="interval", by.y="interval")

my.result.imput$steps.x <-ifelse(is.na(my.result.imput$steps.x), my.result.imput$steps.y, my.result.imput$steps.x)

my.result.imput <- subset(my.result.imput, select=c("steps.x","date","interval"))
names(my.result.imput) <- c("steps","date","interval")
```

Create a new dataset that is equal to the original dataset.
```{r,echo=TRUE}
my.histogram.imput <- aggregate(x=my.result.imput$steps, by=list(my.result.imput$date), FUN=sum, na.rm=TRUE)
names(my.histogram.imput) <- c("date","steps")
```

Make a histogram of the total number of steps taken each day. 
```{r,echo=TRUE}
library(ggplot2)
qplot(x=my.histogram.imput$date, y=my.histogram.imput$steps, xlab="Date", ylab="Steps", stat = "identity", geom="bar")
```

Reporting **mean** and **median** of toyal number of steps taken per day.

**Mean**
```{r,echo=TRUE}
mean(my.histogram.imput$steps)
```

**Median**
```{r,echo=TRUE}
median(my.histogram.imput$steps)
````

The final result (mean and median) is different of the estimates used in the first part of this assessment. The main impact of imputing missing data is to make the mean and median closer.

## Are there differences in activity patterns between weekdays and weekends?

Set the regional setting for create the "weekday" and "weekend" indicator.
I am not from a country that use English regional setting.
```{r}
Sys.setlocale("LC_TIME","C")
```

Create a variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day. Aggredate the final result.

```{r}
my.result$dayweek <- factor(ifelse(weekdays(as.Date(my.result$date)) %in% c("Saturday", "Sunday"),"weekend","weekday"))

my.result.dayweek <- aggregate(x=my.result$steps, by=list(my.result$dayweek, my.result$interval), FUN=mean, na.rm=TRUE)
names(my.result.dayweek) <- c("dayweek","interval","steps")
```

Create a plot containing a time series plot (geom_line) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).
```{r}
ggplot(my.result.dayweek, aes(interval, steps)) + geom_line() + xlab("Interval") + ylab("Mean number of steps") + facet_grid(. ~ dayweek)
```

##---------------------------------------------------------------------------