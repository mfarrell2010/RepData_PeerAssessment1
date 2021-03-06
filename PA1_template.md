---
title: "Reproducible Research"
author: "Michael Farrell"
date: "Tuesday, September 15, 2015"
output: html_document
---

## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

The data used for this research was collected from an activity monitory. This device collects data at 5 minute intervals throughout the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day

## Loading and preprocessing the data
Here the activity file is loaded and date data transformed into a data data type.





```r
# Load the data (i.e. read.csv())
actmon <- read.table(unz("activity.zip","activity.csv"), header=T, sep=",")

#Process/transform the data (if necessary) into a format suitable for your analysis
actmon$date <- as.Date(actmon$date)
```

## What is mean total number of steps taken per day?

First, we will look at the the total number of steps taken each day.


```r
# Calculate the total number of steps taken per day
avg_steps <- aggregate(x = actmon$steps , by = list(actmon$date), FUN = sum ,na.rm=TRUE)
names(avg_steps) <- c("date","steps")
```

Now here is a histogram showing a breakdown of the total number of steps taken each day.


```r
p <- ggplot(avg_steps, aes(x = steps)) +
     ggtitle("Histogram of Daily Steps") +
     xlab("Steps") +
     geom_histogram(binwidth=1000, fill="steelblue")
p
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 


Here are the mean and median of those total daily step values.

```r
mean(avg_steps$steps)
```

```
## [1] 9354.23
```

```r
median(avg_steps$steps)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

Here is a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across the day.


```r
steps_by_int <- group_by(actmon, interval)
avg_by_int <- summarise(steps_by_int,
                       avg_steps = mean(steps, na.rm = TRUE))
plot(avg_by_int, type='l',
     main='Daily Activity Pattern',
     xlab='Time of Day',
     ylab='Steps')
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

Here we can calculate the 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max_steps <- max(actmon$steps, na.rm=TRUE)
max_time <- actmon[actmon$steps==806 & !is.na(actmon$steps),]
max_time
```

```
##       steps       date interval
## 16492   806 2012-11-27      615
```
## Imputing missing values

We can see that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data

First, let's look at the total number of intervals that are missing values.


```r
nrow(actmon[is.na(actmon$steps),])
```

```
## [1] 2304
```

Let's consider a strategy for filling in all of the missing values in the dataset. Since, we have the mean value of steps for each interval, let's use that value for those intervals that are missing values.


```r
# Here we create a new dataset that is equal to the original dataset but with the missing data filled in
actmon.imputed <- merge(x = actmon, y = avg_by_int, by = "interval", all.x = TRUE)
actmon.imputed$imputed_steps = actmon.imputed$steps
```




```r
# Get average steps again, now using the imputed dataset
imputed_steps_by_date <- aggregate(x = actmon.imputed$imputed_steps , by = list(actmon.imputed$date), FUN = sum ,na.rm=TRUE)

names(imputed_steps_by_date) <- c("date","steps")
histplot <- ggplot(imputed_steps_by_date,aes(x = steps)) +
  ggtitle("Histogram of Daily steps - Imputed") +
  xlab("Steps") +
  geom_histogram(binwidth=1000, fill="steelblue")
histplot
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

Here are the mean and median of those total daily step values.


```r
mean(imputed_steps_by_date$steps)
```

```
## [1] 9354.23
```

```r
median(imputed_steps_by_date$steps)
```

```
## [1] 10395
```

## Are there differences in activity patterns between weekdays and weekends?

A variable is created and the dataset split into two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
# Here is a variable indicating if the date is a weekday or weekend date.
actmon.imputed$weekday <- as.factor(ifelse(weekdays(actmon.imputed$date) %in% c("Saturday","Sunday"), "Weekend", "Weekday")) 

# Now agregating by the interval and weekday/weekend factor.
avg_by_in_wkday  <- aggregate(x = actmon.imputed$steps, 
                              by = list(actmon.imputed$interval,actmon.imputed$weekday), FUN = mean ,na.rm=TRUE)
names(avg_by_in_wkday) <- c("interval","weekday","steps")
```

Now here is panel containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days.


```r
p_wkday <- ggplot(avg_by_in_wkday,aes(interval,steps)) +
  ggtitle("Steps by Time - Weekday vs. Weekend") +
  facet_grid(. ~ weekday) +
  geom_line(size = 1)
p_wkday
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13-1.png) 
