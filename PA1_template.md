Reproducible Research - Peer Assessment 1
-----------------------------------------

### Loading and preprocessing the data

1. Load the data (i.e. read.csv())
```
activity<-read.csv("activity.csv")
```
2. Process/transform the data (if necessary) into a format suitable for your analysis 
```
activity$date<-as.Date(activity$date, "%Y-%m-%d")
```

### What is mean total number of steps taken per day?

1. Make a histogram of the total number of steps taken each day

```r
total<-aggregate(steps~date, data=activity, sum, na.rm=TRUE)  
hist(total$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-1-1.png) 

2. Calculate and report the mean and median total number of steps taken per day

```r
mean(total$steps)  
```

```
## [1] 10766.19
```

```r
median(total$steps)
```

```
## [1] 10765
```

###What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
intervals<-aggregate(steps~interval, data=activity, mean, na.rm=TRUE)  
plot(steps~interval, data=intervals, type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
intervals[which.max(intervals$steps), ]$interval
```

```
## [1] 835
```

###Inputting Missing Values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(activity))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
library(sqldf)
```

```
## Loading required package: gsubfn
## Loading required package: proto
## Loading required package: RSQLite
## Loading required package: DBI
```

```r
activity_complete<-activity[complete.cases(activity),]
activity_NA<-activity[is.na(activity$steps),]
activity_extrapolated<-sqldf("select intervals.steps, activity_NA.date, activity_NA.interval from activity_NA, intervals where activity_NA.interval=intervals.interval")
```

```
## Loading required package: tcltk
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
activity_recombined<-rbind(activity_complete, activity_extrapolated)
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps? 


```r
total_extrapolated<-aggregate(steps~date, data=activity_recombined, sum, na.rm=TRUE)  
hist(total_extrapolated$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png) 

```r
mean(total_extrapolated$steps)  
```

```
## [1] 10766.19
```

```r
median(total_extrapolated$steps)
```

```
## [1] 10766.19
```
Conclusion: replacing NA values with the mean values for those intervals has a negligible affect
on the overall mean and median. Using a different value, such as 0, would probably have had a bigger affect.

###Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day. 

```r
activity_recombined$weekday<-weekdays(as.POSIXct(activity_recombined$date, tz="", format="%Y-%m-%d"))
activity_recombined$daytype<-ifelse(activity_recombined$weekday == "Saturday" | activity_recombined$weekday == "Sunday", "weekend", "weekday")
head(activity_recombined)
```

```
##     steps       date interval weekday daytype
## 289     0 2012-10-02        0 Tuesday weekday
## 290     0 2012-10-02        5 Tuesday weekday
## 291     0 2012-10-02       10 Tuesday weekday
## 292     0 2012-10-02       15 Tuesday weekday
## 293     0 2012-10-02       20 Tuesday weekday
## 294     0 2012-10-02       25 Tuesday weekday
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data. 

```r
library(lattice)
meansteps<-aggregate(activity_recombined$steps, by=list(activity_recombined$interval, activity_recombined$daytype), mean)
names(meansteps) <- c("interval","daytype","steps")
xyplot(steps~interval | daytype, data=meansteps, type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 
