---
title: "Reproducible Research - Course Project 1"
author: "Vaishnavi Doraiswamy"
date: "January 04, 2017"
output: html_document
---

To parse dates we're going to use lubridate, dplyr to parse data, and to plot some graphs ggplot2


```r
library(knitr)
opts_chunk$set(echo = TRUE, results = 'asis')
library(data.table)
library(ggplot2) 
```

Reading the data and preprocessing it


```r
activity_data <- read.csv('activity.csv', header = TRUE, sep = ",",colClasses=c("numeric", "character", "numeric"))

activity_data$date <- as.Date(activity_data$date, format = "%Y-%m-%d")
activity_data$interval <- as.factor(activity_data$interval)
```



```r
steps_per_day <- aggregate(steps ~ date,activity_data, sum)
colnames(steps_per_day) <- c("date","steps")
```


Histogram of the total number of steps taken each day:


```r
ggplot(steps_per_day, aes(x = steps)) +  geom_histogram(fill = "green", binwidth = 1000) + labs(title="Histogram of Steps Taken per Day", 
             x = "Number of Steps per Day", y = "Number of times in a day(Count)") + theme_bw() 
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)

Mean and median number of steps taken each day. 


```r
mean   <- mean(steps_per_day$steps, na.rm=TRUE)
median <- median(steps_per_day$steps, na.rm=TRUE)
```

Time series plot of the average number of steps taken.


```r
steps_per_interval <- aggregate(activity_data$steps, by = list(interval = activity_data$interval), FUN=mean, na.rm=TRUE)
steps_per_interval$interval <- 
        as.integer(levels(steps_per_interval$interval)[steps_per_interval$interval])
colnames(steps_per_interval) <- c("interval", "steps")
ggplot(steps_per_interval, aes(x=interval, y=steps)) +   
        geom_line(color="orange", size=1) +  
        labs(title="Average Daily Activity Pattern", x="Interval", y="Number of steps") +  
        theme_bw()
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

The 5-minute interval that, on average, contains the maximum number of steps

```r
max_interval <- steps_per_interval[which.max(  
        steps_per_interval$steps),]
```

Code to describe and show a strategy for imputing missing data

Total number of missing values in the dataset

```r
missing_vals <- sum(is.na(activity_data$steps))
na_fill <- function(data, pervalue) {
        na_index <- which(is.na(data$steps))
        na_replace <- unlist(lapply(na_index, FUN=function(idx){
                interval = data[idx,]$interval
                pervalue[pervalue$interval == interval,]$steps
        }))
        fill_steps <- activity_data$steps
        fill_steps[na_index] <- na_replace
        fill_steps
}

rdata_fill <- data.frame(  
        steps = na_fill(activity_data, steps_per_interval),  
        date = activity_data$date,  
        interval = activity_data$interval)
```
Check if there are anymore missing values in the dataset

```r
sum(is.na(rdata_fill$steps))
```

[1] 0
Histogram of the total number of steps taken each day after missing values are imputed

```r
fill_steps_per_day <- aggregate(steps ~ date, rdata_fill, sum)
colnames(fill_steps_per_day) <- c("date","steps")

ggplot(fill_steps_per_day, aes(x = steps)) + 
       geom_histogram(fill = "blue", binwidth = 1000) + 
        labs(title="Histogram of Steps Taken per Day", 
             x = "Number of Steps per Day", y = "Number of times in a day(Count)") + theme_bw() 
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png)
Calculating the mean and median

```r
steps_mean_fill   <- mean(fill_steps_per_day$steps, na.rm=TRUE)
steps_median_fill <- median(fill_steps_per_day$steps, na.rm=TRUE)
```
##Do these values differ from the estimates from the first part of the assignment?

Yes, there is a slight difference

##What is the impact of imputing missing data on the estimates of the total daily number of steps?

As we infer from the histograms, it seems that the impact of imputing missing values has increase our peak, but it's not affect negatively our predictions.


Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends


```r
weekdays_steps <- function(data) {
    weekdays_steps <- aggregate(data$steps, by=list(interval = data$interval),
                          FUN=mean, na.rm=T)
    # convert to integers for plotting
    weekdays_steps$interval <- 
            as.integer(levels(weekdays_steps$interval)[weekdays_steps$interval])
    colnames(weekdays_steps) <- c("interval", "steps")
    weekdays_steps
}

data_by_weekdays <- function(data) {
    data$weekday <- 
            as.factor(weekdays(data$date)) # weekdays
    weekend_data <- subset(data, weekday %in% c("Saturday","Sunday"))
    weekday_data <- subset(data, !weekday %in% c("Saturday","Sunday"))

    weekend_steps <- weekdays_steps(weekend_data)
    weekday_steps <- weekdays_steps(weekday_data)

    weekend_steps$dayofweek <- rep("weekend", nrow(weekend_steps))
    weekday_steps$dayofweek <- rep("weekday", nrow(weekday_steps))

    data_by_weekdays <- rbind(weekend_steps, weekday_steps)
    data_by_weekdays$dayofweek <- as.factor(data_by_weekdays$dayofweek)
    data_by_weekdays
}

data_weekdays <- data_by_weekdays(rdata_fill)


ggplot(data_weekdays, aes(x=interval, y=steps)) + 
        geom_line(color="violet") + 
        facet_wrap(~ dayofweek, nrow=2, ncol=1) +
        labs(x="Interval", y="Number of steps") +
        theme_bw()
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png)
