---
title: "Reproducible Research_Proj1_ActivityMonitoringData"
author: "Heather Tippett"
date: "August 2019"
output: 
  html_document: 
    keep_md: true
---



Reproducible Research Coursera Project 1 (All assignment instructions are copied directly from Coursera.)

## Assignment Details
This assignment will be described in multiple parts. You will need to write a report that answers the questions detailed below. Ultimately, you will need to complete the entire assignment in a single R markdown document that can be processed by knitr and be transformed into an HTML file.

Throughout your report make sure you always include the code that you used to generate the output you present. When writing code chunks in the R markdown document, always use echo=TRUE so that someone else will be able to read the code. This assignment will be evaluated via peer assessment so it is essential that your peer evaluators be able to review the code for your analysis.

## Loading and Preprocessing the data
Show any code that is needed to:  
1) Load the data (i.e. read.csv())  
2) Process/transform the data into a format suitable for your analysis

The r code below loads in the data and omits the NA values:


```r
activity <- read.csv("activity.csv")
activityO <- na.omit(activity)
```

## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

1) Calculate the total number of steps taken per day
The code below aggregates all of the steps taken on each day.


```r
dailySteps <- aggregate(activityO$steps, list(Day = activityO$date), sum)
```

2) Make a histogram of the total number of steps taken each day
3) Calculate and report the mean and median of the total number of steps taken per day

```r
meanSteps <- mean(dailySteps$x)
medianSteps <- median(dailySteps$x)
cat("The mean of the steps data set is: ",meanSteps)
```

```
## The mean of the steps data set is:  10766.19
```

```r
cat("The median of the steps data set is: ",medianSteps)
```

```
## The median of the steps data set is:  10765
```

```r
hist(dailySteps$x,  main = "Histogram of Daily Steps",xlab = "Daily Step Count",plot=TRUE)
abline(v=medianSteps, col ="red", lwd=2)
abline(v=meanSteps, col ="blue", lwd=2, lty=2)
legend('topright',legend = c("Median = 10,765 Steps","Mean = 10,766.19 Steps"), col = c("red","blue"), lty=1:2, cex=.8)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

## What is the average daily activity pattern?
1) Make a time series plot (i.e. type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2) Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

The code below plots a time series, finds the x-coordinate of the maximum number of steps and adds a legend to the plot.


```r
library(plyr)
```

```
## Warning: package 'plyr' was built under R version 3.5.3
```

```r
meanIntervalSteps <- ddply(activityO, ~interval, summarise, mean = mean(steps))
plot(meanIntervalSteps$interval, meanIntervalSteps$mean, type = "l", main = "Time Series Plot for Avg # of Steps Taken per 5-Minute Interval", xlab = "5-Minute Intervals", ylab = "Average Number of Steps")
meanIntervalSteps[which.max(meanIntervalSteps$mean),]
```

```
##     interval     mean
## 104      835 206.1698
```

```r
abline(v=835, col = "blue", lwd=2)
legend('topright', legend = "Interval with Max # of Steps = 835", col = "blue", lty=1, cex=.8)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

## Inputing Missing Values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1) Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
missingValues <- nrow(activity[is.na(activity$steps),])
cat("The total number of missing values is: ",missingValues)
```

```
## The total number of missing values is:  2304
```

2) Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Using the mean steps for each NA 5 minute interval, I filled in the missing values. 

3) Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
activityFilled <- activity
for (i in 1:length(activityFilled$steps)) 
{ 
  if (is.na(activityFilled$steps[i] == TRUE)) 
  {
    activityFilled$steps[i] <- meanIntervalSteps$mean[match(activityFilled$interval[i], meanIntervalSteps$interval)]
  }
}
```

4) Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
dailyStepsF <- aggregate(activityFilled$steps, list(Day = activityFilled$date), sum)
meanStepsF <- mean(dailyStepsF$x)
medianStepsF <- median(dailyStepsF$x)
cat("The mean of the filled steps data set is: ",meanStepsF, ". ")
```

```
## The mean of the filled steps data set is:  10766.19 .
```

```r
cat("The median of the filled steps data set is: ",medianStepsF, ".")
```

```
## The median of the filled steps data set is:  10766.19 .
```

```r
hist(dailyStepsF$x,  main = "Histogram of Daily Steps (wo NA)",xlab = "Daily Step Count",plot=TRUE)
abline(v=medianStepsF, col ="red", lwd=2)
abline(v=meanStepsF, col ="blue", lwd=2, lty=2)
legend('topright',legend = c("Median = 10,766.19 Steps","Mean = 10,766.19 Steps"), col = c("red","blue"), lty=1:2, cex=.8)
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

The values shown in the histogram above, indicate that there is a higher frequency of step count in the 10,000 - 15,000 step bucket. This shifts our mean by 1.19 steps to match our median exactly. But at a quick glance, the two plots are identical. 


##Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
activityFilled$wkDay <- weekdays(as.Date(activityFilled$date))
for(i in 1:length(activityFilled$wkDay)) 
{ 
  if(activityFilled$wkDay[i] == "Saturday" || activityFilled$wkDay[i] == "Sunday") 
  {
    activityFilled$dayClass[i] <- "Weekend"
  } 
  else 
  {
      activityFilled$dayClass[i] <- "Weekday"
  }
}
```

Make a panel plot containing a time series plot type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

Step 1: Using ddply, we can seperate the data by dayClass and interval and get the mean for each step.

```r
mergedData <- ddply(activityFilled, .(dayClass,interval), summarize, mean = mean(steps))
```

Step 2: Create the time series plot

```r
library(lattice)
xyplot(mean ~ interval | dayClass, mergedData, type = "l", layout = c(1,2), xlab = "Interval", ylab = "Number of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->
