## Reproducible Research: Peer Assessment 1

### Loading and preprocessing the data

```r
fname = "activity.zip"
source_url = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
if(!file.exists(fname)) {
    download.file(source_url, destfile=fname, method="curl")
}
con <- unz(fname, "activity.csv")
data <- read.csv(con, colClasses = c("numeric", "Date", "numeric"))
```

### What is mean total number of steps taken per day?
#### Make a histogram of the total number of steps taken each day

```r
split <- split(data, data$date)
total_steps <- sapply(X=split, FUN=function(x) sum(na.omit(x$steps)))
hist(total_steps, col='blue', xlab = "Total Number of Steps")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

#### Calculate and report the **mean** and **median** total number of steps taken per day

```r
steps <- sapply(X=split, FUN=function(x) sum(na.omit(x$steps)))
mean <- mean(steps, na.rm=TRUE)
median <- median(steps, na.rm=TRUE)
```
The mean is 9354.2295 and the median is 1.0395 &times; 10<sup>4</sup>.

### What is the average daily activity pattern?
1. Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
intervals <- unique(data$interval)
meanInterval <- sapply(X = intervals, function(x) mean(data[data$interval==x,'steps'], na.rm = TRUE))
names(meanInterval) <- intervals
plot(names(meanInterval), meanInterval, type="l", ylab="Average number of steps", xlab="Interval")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max <- unique(data$interval)[which(meanInterval==max(meanInterval))]
```
Answer: 835.

### Imputing missing values

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
missing <- nrow(data[is.na(data),])
```

My strategy for filling in all of the missing values in the dataset: the mean for that 5-minute interval.

```r
filling_NA <- sapply(data[is.na(data$steps), 'interval'], function(x) meanInterval[which(names(meanInterval) == x)])
```

Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
data[is.na(data$steps), 'steps'] <- filling_NA
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
split <- split(data, data$date)
total_steps <- sapply(X=split, FUN=function(x) sum(na.omit(x$steps)))
hist(total_steps, col='blue', xlab = "Total Number of Steps")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

```r
steps <- sapply(X=split, FUN=function(x) sum(na.omit(x$steps)))
mean <- mean(steps, na.rm=TRUE)
median <- median(steps, na.rm=TRUE)
```

The mean is 1.0766 &times; 10<sup>4</sup> and the median is 1.0766 &times; 10<sup>4</sup>. This values differ from the first part ones because there are about 13% of NA values, and when you replace them you are duplicating some of the other values. The impact on the estimates is that it equals them and also it increases them both, because you are adding the mean value. If I have added a random value according to the interval instead of its mean, then the numbers shouldn't have changed much. The overall impact is that it normalizes the sample.


#### Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
data$day = factor(labels = c("weekday", "weekend"), weekdays(data$date) == "Sunday" | weekdays(data$date) == "Saturday")
```
Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was creating using simulated data:


```r
dataWday <- data[data$day=="weekday",]
intervals <- unique(dataWday$interval)
meanInterval <- sapply(X = intervals, function(x) mean(dataWday[dataWday$interval==x,'steps'], na.rm = TRUE))
names(meanInterval) <- intervals
data2 = data.frame(mean = meanInterval, interval = intervals, day='weekday')

dataWend <- data[data$day=="weekend",]
intervals <- unique(dataWend$interval)
meanInterval <- sapply(X = intervals, function(x) mean(dataWend[dataWend$interval==x,'steps'], na.rm = TRUE))
names(meanInterval) <- intervals
data2 = rbind(data2, data.frame(mean = meanInterval, interval = intervals, day='weekend'))
require(lattice)
xyplot(data2$mean ~ data2$interval | data2$day, layout = c(1,2), type = "l", ylab="Number of steps", xlab = "Interval")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12.png) 
