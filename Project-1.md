---
output:
  html_document: 
    keep_md: yes
  pdf_document: default
---
Reproducible research_Peer-graded Assignment: Course Project 1
========================================

## Code for reading in the dataset and processing the data
1. Read the csv data into R using read.csv.
2. convert the "date" column as date class.


```r
activity<-read.csv("activity.csv")
activity$date <- as.Date(activity$date, "%Y-%m-%d")
```

## Histogram of the total number of steps taken each day
1. Use functions the dplyr package.
2. Group the data frame using "date" and calculate the sum of "steps" for each date, with NA value removed.
3. Plot the total number of steps taken each day using histogram.

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
total_step<-activity %>% 
    group_by(date) %>%
    summarize(total=sum(steps, na.rm = TRUE))
hist(total_step$total, xlab="Total steps per day", main = "Total number of steps taken each day")
```

![](Project-1_files/figure-html/Histogram-1.png)<!-- -->

## Mean and median number of steps taken each day

```r
# Mean number of steps taken each day
mean(total_step$total)
```

```
## [1] 9354.23
```

```r
# Median number of steps taken each day
median(total_step$total)
```

```
## [1] 10395
```

## Time series plot of the average number of steps taken
1. Group the data frame using "interval" and calculate the sum of "steps" for each interval, with NA value removed.
2. Plot time series plot of the average number of steps taken.


```r
Daily_average_step<-activity %>% 
    group_by(interval) %>%
    summarize(average=mean(steps,na.rm=TRUE))
with(Daily_average_step, plot(interval, average, type="l", ylab="Average steps"))
```

![](Project-1_files/figure-html/TimeSeriesPlot-1.png)<!-- -->

## The 5-minute interval that, on average, contains the maximum number of steps


```r
Daily_average_step$interval[which(Daily_average_step$average==max(Daily_average_step$average))]
```

```
## [1] 835
```

## Code to describe and show a strategy for imputing missing data
1. Describe the total number of missing values in the data frame.
2. Set the NA value to the mean for that 5-minute interval using a "for" loop.

```r
# The total number of missing values in the dataset
sum(is.na(activity$steps))
```

```
## [1] 2304
```

```r
# Set the NA value to the mean for that 5-minute interval.
activity_2<-activity
for (i in 1:nrow(activity_2)){
    if(is.na(activity_2$steps[i])){
        activity_2$steps[i] <- Daily_average_step$average[which(Daily_average_step$interval==activity_2$interval[i])]
    }
}
```

## Histogram of the total number of steps taken each day after missing values are imputed
1. Group the data frame using "date" and calculate the sum of "steps" for each date, using the dataset after imputing the NA values.
2. Plot the total number of steps taken each day using histogram.


```r
total_step_NARemoved <- activity_2 %>%
    group_by(date) %>%
    summarize(total=sum(steps))
hist(total_step_NARemoved$total, xlab="Total steps per day", main = "Total number of steps taken each day (NA imputated)")
```

![](Project-1_files/figure-html/histogram-1.png)<!-- -->

## The mean and median of the total number of steps taken per day (NA imputated)

```r
# Mean
mean(total_step_NARemoved$total)
```

```
## [1] 10766.19
```

```r
# Median
median(total_step_NARemoved$total)
```

```
## [1] 10766.19
```

## Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
1. Using a "for" loop to create a new factor variable called "Week" in the dataset with two levels (“weekday” and “weekend”) to indicate whether a given date is a weekday or weekend day.
2. Using ggplot2 package to plot the average number of steps taken per 5-minute interval across weekdays and weekends.


```r
#Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
for (i in 1:nrow(activity_2)){
    if(weekdays(activity_2$date[i]) %in% c("Saturday", "Sunday")) {
        activity_2$Week[i] <- "weekend"
    } else{
        activity_2$Week[i] <- "weekday"
    }
}
activity_2$Week <- factor(activity_2$Week)

#Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
library(ggplot2)
step_week<-activity_2 %>%
    group_by(interval,Week) %>%
    summarize(average=mean(steps))
```

```
## `summarise()` has grouped output by 'interval'. You can override using the
## `.groups` argument.
```

```r
g <- ggplot(step_week, aes(interval, average))
g+geom_line()+facet_grid(Week~.)+labs(y="Average steps", x="Time interval")
```

![](Project-1_files/figure-html/weekPlot-1.png)<!-- -->








