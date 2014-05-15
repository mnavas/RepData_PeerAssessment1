Assignment
==========================================================

## Loading and preprocessing the data

Show any code that is needed to

### 1. Load the data (i.e. read.csv())


```r
data <- read.table("activity.csv", sep = ",", header = TRUE)
```


### 2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
data$date <- as.Date(data$date, "%Y-%m-%d")
```


## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

### 1. Make a histogram of the total number of steps taken each day


```r
library(ggplot2)
qplot(date, data = data, weight = steps, geom = "histogram")
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 


### 2. Calculate and report the mean and median total number of steps taken per day


```r
total.per.day <- with(data, {
    aggregate(cbind(steps = steps), by = list(date = date), FUN = sum)
})
```


The mean of the total number of steps taken per day is as follows


```r
mean(total.per.day$steps, na.rm = TRUE)
```

```
## [1] 10766
```


The median of the total number of steps taken per day is as follows


```r
median(total.per.day$steps, na.rm = TRUE)
```

```
## [1] 10765
```


## What is the average daily activity pattern?

### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
average.per.interval <- with(data, {
    aggregate(cbind(steps = steps), by = list(interval = interval), FUN = mean, 
        na.rm = TRUE)
})
qplot(interval, steps, data = average.per.interval, geom = "line", ylab = "Average Steps")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 


### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
average.per.interval$interval[which.max(average.per.interval$steps)]
```

```
## [1] 835
```


## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum(is.na(data$steps))
```

```
## [1] 2304
```


### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Use the mean for that 5-minute interval

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
new.data <- data
for (i in which(is.na(data$steps))) {
    new.data$steps[i] <- average.per.interval$steps[average.per.interval$interval == 
        data$interval[i]]
}
```


### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
qplot(date, data = new.data, weight = steps, geom = "histogram")
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 



```r
new.total.per.day <- with(new.data, {
    aggregate(cbind(steps = steps), by = list(date = date), FUN = sum)
})
```


The mean of the total number of steps taken per day is as follows


```r
mean(new.total.per.day$steps, na.rm = TRUE)
```

```
## [1] 10766
```


The median of the total number of steps taken per day is as follows


```r
median(new.total.per.day$steps, na.rm = TRUE)
```

```
## [1] 10766
```


The values of the mean don't differ and the median differ from the original values of the data.

For the mean the difference is

```r
mean(total.per.day$steps, na.rm = TRUE) - mean(new.total.per.day$steps, na.rm = TRUE)
```

```
## [1] 0
```


For the median the difference is 

```r
median(total.per.day$steps, na.rm = TRUE) - median(new.total.per.day$steps, 
    na.rm = TRUE)
```

```
## [1] -1.189
```


## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

### 1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
data[as.POSIXlt(data$date)$wday == 0 | as.POSIXlt(data$date)$wday == 6, "day"] <- "weekend"
data[as.POSIXlt(data$date)$wday != 0 & as.POSIXlt(data$date)$wday != 6, "day"] <- "week day"
data$day <- factor(data$day)
```


### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis):


```r
interval.day <- with(data, {
    aggregate(cbind(steps = steps), by = list(interval = interval, day = day), 
        FUN = mean, na.rm = TRUE)
})
qplot(interval, steps, data = interval.day, geom = "line", ylab = "Average Steps", 
    facets = day ~ .)
```

![plot of chunk unnamed-chunk-18](figure/unnamed-chunk-18.png) 

