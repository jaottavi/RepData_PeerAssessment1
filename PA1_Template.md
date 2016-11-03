---
title: "RepData_PeerAssign1"
author: "James Ottavi"
date: "10/31/2016"
output: 
  html_document:
    keep_md: TRUE
---



## Step 1: Read in the Activity Data
Download the data from the link and read it in R by using read.csv()

```r
activity <- read.csv("activity.csv")
```

## Step 2: Mean Total Steps per Day
The activity data has 3 variables: steps, date, and interval. In order to get the total amount of steps per day, we will use tapply. After this, the mean and median will be displayed of this variable. Finally, a histogram will be created from the tapply result, showing the distribution of total steps per day.

```r
totalsteps <- with(activity, tapply(steps, date, sum, na.rm = TRUE))
print(c(mean = mean(totalsteps, na.rm = TRUE), median = median(totalsteps, na.rm=TRUE)))
```

```
##     mean   median 
##  9354.23 10395.00
```

```r
hist(totalsteps, main = "Histogram of Total Steps per Day", xlab = "Total Steps")
```

![plot of chunk totalsteps](figure/totalsteps-1.png)

## Step 3: Average Daily Activity Pattern
To look at how the the total steps changed across time intervals, a time series plot has been created below: on the x-axis, the specific interval being looked at; on the y-axis, the average number of total steps taken across all days for that interval.

```r
intervalsteps <- with(activity, tapply(steps, interval, mean, na.rm = TRUE))
plot(intervalsteps, type = "l", ylab = "Average Steps", xlab = "Interval", main = "Average Steps Across Intervals")
```

![plot of chunk timeseries](figure/timeseries-1.png)

## Steps 4: Dealing with Missing Values
In this dataset, there happen to be a good deal of missing values. The below R code: counts the number of rows with missing values; fills in the respective missing values with the mean of that time interval; creates a new, non-NA dataset; and finally crafts a histogram of the new total number of steps taken per day.

```r
sum(!complete.cases(activity)) # Total NAs
```

```
## [1] 2304
```

```r
dfinterval <- data.frame(interval = names(intervalsteps), newsteps = intervalsteps)
nas <- activity[is.na(activity),]
merged <- merge(nas, dfinterval, by = "interval")
merged <- merged[order(merged$date, merged$interval),]
newact <- activity
newact[is.na(newact),"steps"] <- merged$newsteps
sumnewsteps <- with(newact, tapply(steps, date, sum))
print(c(NewMean = mean(sumnewsteps), NewMedian = median(sumnewsteps)))
```

```
##   NewMean NewMedian 
##  10766.19  10766.19
```

```r
hist(sumnewsteps, main="Imputed Histogram Total Steps per Day", xlab= "Total Steps per Day")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png)

## Step Five: Patterns Between Weekdays and Weekends
To see if there is a difference in step pattern between weekdays and weekends, a new factor variable will be created ("weekday", "weekend"). Afterwards, a panel plot will be created to visualize the potential differences.

```r
library(lattice)
library(reshape2)
newact$day <- ifelse(weekdays(as.Date(newact$date)) %in% c("Saturday","Sunday"),"weekend","weekday")
daydata <- aggregate(steps~day+interval, data = newact, mean)
xyplot(steps~interval | factor(day), data = daydata, type = "l", layout = c(1,2))
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)
