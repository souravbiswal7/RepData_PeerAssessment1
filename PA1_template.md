---
title: "Steps Data Assignment"
author: "Hui Yi Lee"
date: "3/25/2020"
output:
  word_document: default
  html_document: default
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Average total number of steps taken per day

Load the data and process/transform into a format suitable for analysis. Then calculate the total number of steps taken per day, and plot a histogram. 

Also calculate the mean and median of the average total number of steps taken per day. 

```{r}
activity <- read.csv("activity.csv", sep = ",", header = TRUE)
activity$date <- as.Date(activity$date)

library(reshape)
meltdate <- melt(activity, id = c("date"))
sumbydate <- cast(meltdate, date ~ variable, sum)
plot1 <- hist(sumbydate$steps, main = "Distribution of total steps per day", xlab = "Total steps per day")

mean(sumbydate$steps, na.rm = TRUE)
median(sumbydate$steps, na.rm = TRUE)

```

![alt text](https://github.com/huiyilee/RepData_PeerAssessment1/blob/master/plot1.png)

The mean is 10766.19, median is 10765.

## Average daily activity pattern

Make a time series plot of the average number of steps taken for each 5-minute interval, averaged across all days in the dataset.

Find the 5-minute interval with maximum number of steps.

```{r}
activity$interval <- as.character(activity$interval)
meltint <- melt(activity, id = c("interval"))
meltint <- na.omit(meltint)

avbyint <- cast(meltint, interval ~ variable, mean)
avbyint$interval <- as.numeric(avbyint$interval)
avbyint <- avbyint[order(avbyint$interval), ]
avbyint$min <- seq( from = 0, to = 1435, by = 5)

plot2 <- plot(avbyint$min, avbyint$steps, type = "l", xlab = "Time interval (in minutes)", ylab = "Average number of steps")

## Sort to get interval with the maximum number of steps, and subset to get the first row.
avbyint <- avbyint[order(-avbyint$steps), ]
avbyint[1, ]

```

![alt text](https://github.com/huiyilee/RepData_PeerAssessment1/blob/master/plot2.png)

The time interval 515 to 520 minutes, or 8.45-8.50am, has the maximum number of steps of 206.17.

## Imputing missing values

Calculate and report the total number of missing values in the dataset:
```{r}
sum(is.na(activity))
```
There are 2304 missing values in the dataset.

Use the average number of steps for the relevant 5-minute interval to fill in missing values.

```{r}
replaceNA <- activity
replaceNA$avperint <- avbyint$steps[match(replaceNA$interval, avbyint$ interval)]
replaceNA <- transform(replaceNA, steps = ifelse(is.na(steps), avperint, steps))
```

Make a histogram of the total number of steps taken each day, based on data set with NA values filled in. 

Also calculate and report the mean and median total number of steps taken per day.

```{r}
meltdate2 <- melt(replaceNA, id = c("date", "interval"))
sumbydate2 <- cast(meltdate2, date ~ variable, sum)
plot3 <- hist(sumbydate2$steps, main = "Distribution of total steps per day", xlab = "Total steps per day")
mean(sumbydate2$steps, na.rm = TRUE)
median(sumbydate2$steps, na.rm = TRUE)
```
![alt text](https://github.com/huiyilee/RepData_PeerAssessment1/blob/master/plot3.png)

Note that the mean of 10766.19 is the same as in the first part of assignment, since we filled in NA values based on average number of steps for the relevant 5-minute interval. However median is slightly higher at 10766.19 compared to 10765 from first part of assignment. 

## Activity patterns, weekdays versus weekends

Create a new factor variable in the dataset with two levels – “weekday” and “weekend”.

```{r}
dayofweek <- replaceNA
dayofweek$interval <- as.character(dayofweek$interval)
dayofweek$daytype <- weekdays(dayofweek$date)

dayofweek <- transform(dayofweek, daytype = ifelse(daytype == "Saturday" | daytype == "Sunday", "weekend", "weekday"))
```

Make a panel plot of average number of steps taken, averaged across all weekday days or weekend days respectively.

```{r}
meltdaytype <- melt(dayofweek, id = c("date", "interval", "daytype"))
avdaytype <- cast(meltdaytype, interval+daytype ~ variable, mean)
avdaytype$interval <- as.numeric(avdaytype$interval)
avdaytype <- avdaytype[order(avdaytype$interval), ]

weekdays <- avdaytype[which(avdaytype$daytype == "weekday"), ]
weekend <- avdaytype[which(avdaytype$daytype == "weekend"), ]
plot4 <- par(mfrow=c(2,1))
plot(weekdays$interval, weekdays$steps, type="l", xlab="Interval on weekdays", ylab="Average steps taken")
plot(weekend$interval, weekend$steps, type="l", xlab="Interval on weekends", ylab="Average steps taken")
```
![alt text](https://github.com/huiyilee/RepData_PeerAssessment1/blob/master/plot4.png)

On weekdays, time interval with a higher number of steps tends to be earlier in the day. On weekends, average number of steps throughout the day tends to be higher than on an average weekday.
