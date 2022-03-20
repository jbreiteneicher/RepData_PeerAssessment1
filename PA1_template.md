---
title: "Reproducible Research: Peer Assessment 1"
date:   March 2022
output: 
  html_document:
    keep_md: true
---

## Loading and preprocessing the data
#### 1.Code for reading in the dataset and/or processing the data

```r
library(plyr, warn.conflicts = FALSE)
library(dplyr, warn.conflicts = FALSE)
```

```
## Warning: Paket 'dplyr' wurde unter R Version 4.1.3 erstellt
```

```r
library(ggplot2, warn.conflicts = FALSE)
mydata <- read.csv("activity.csv")
daily_steps <-aggregate(steps~date,data=mydata,sum,na.rm=TRUE)
mean_steps <- mean(daily_steps$steps, na.rm=TRUE)
median_steps <- median(daily_steps$steps, na.rm=TRUE)
head(mydata)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
str(mydata)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

## What is mean total number of steps taken per day?
#### 2.Histogram of the total number of steps taken each day

```r
hist(daily_steps$steps,
     main = "Histogram of the total number of steps taken each day",
     xlab = "total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

#### 3.Mean and median number of steps taken each day
- The mean of the total number of steps taken per days is 1.0766189\times 10^{4}.
- The median of the total number of steps taken per days is 10765.


## What is the average daily activity pattern?
#### 4. Time series plot of the average number of steps taken

```r
day_avg <- mydata %>%
            group_by(interval) %>%
            summarise(avg_steps = mean(steps, na.rm = T))
ggplot(day_avg, aes(x = interval, y = avg_steps, group = 1)) + 
      geom_line() + scale_x_discrete(breaks = seq(0, 2500, 500))
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

#### 5. The 5-minute interval that, on average, contains the maximum number of steps

```r
max_interval <- day_avg[which.max(day_avg$avg_steps), ]
```
- The 5-minute interval with max number of steps is **835** with 206.1698113 steps

## Imputing missing values
#### 6. Code to describe and show a strategy for imputing missing data

```r
sum(is.na(mydata))
```

```
## [1] 2304
```
- The total number of missing values in the dataset is 2304

The strategy we are going implement for imputing missing values is to use the mean for the 5-minute interval.

```r
newdata = mydata
length1 = nrow(newdata)
length2 = nrow(day_avg)
for (i in 1:length1) {
  if (is.na(newdata$steps[i])) {
    for (j in 1:length2) {
      if (newdata$interval[i] == day_avg[j, 1]) {
        newdata$steps[i] = day_avg[j, 2]
      }
    } 
  }    
}
```

#### 7. Histogram of the total number of steps taken each day after missing values are imputed


## Are there differences in activity patterns between weekdays and weekends?
#### 8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
- Introduce new factor weekday//weekend Sat/Sub German

```r
mydata$weekday = TRUE
weekday = weekdays(as.POSIXct(mydata$date, format = "%Y-%m-%d" ))
for (i in 1:length(weekday)) {
  if (weekday[i] == "Samstag" | weekday[i] == "Sonntag") {
    mydata$weekday[i] = FALSE
  }
}
dataWeekday = mydata[which(mydata$weekday == TRUE), ]
dataWeekend = mydata[which(mydata$weekday == FALSE), ]

avgWeekdayPattern = aggregate(dataWeekday$steps, 
                               by = list(dataWeekday$interval), 
                               FUN = mean)
names(avgWeekdayPattern) = c("interval", "steps")
avgWeekdayPattern$dayTag = "weekday"
avgWeekendPattern = aggregate(dataWeekend$steps, 
                              by = list(dataWeekend$interval), 
                              FUN = mean)
names(avgWeekendPattern)= c("interval", "steps")
avgWeekendPattern$dayTag = "weekend"

avgPattern = rbind(avgWeekdayPattern, avgWeekendPattern)
```

Now let's plot

```r
library(lattice)
xyplot(steps ~ interval | dayTag, data = avgPattern, 
       type = "l", layout = c(1, 2))
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

#### 9. All of the R code needed to reproduce the results (numbers, plots, etc.) in the report

