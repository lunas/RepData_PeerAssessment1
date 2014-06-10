# Reproducible Research: Peer Assessment 1

## Loading and preprocessing the data

We assume that the current directory contains the zipped data file 'activity.zip'. 

Unzip it and load it:

```r
unzip("activity.zip")
df <- read.csv("activity.csv")
```


## What is mean total number of steps taken per day?

Aggregate the data to get the total of steps per day:


```r
steps.per.day <- aggregate(df$steps, list(df$date), sum, na.rm = T)
names(steps.per.day) <- c("date", "num.steps")
```


Histogram of the steps taken per day


```r
hist(steps.per.day$num.steps, breaks = 10, main = "Histogram: number of steps per day", 
    xlab = "number of steps per day")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 


Calculate mean ond median of the total number of steps per day:

```r
mean(steps.per.day$num.steps, na.rm = T)
```

```
## [1] 9354
```

```r
median(steps.per.day$num.steps, na.rm = T)
```

```
## [1] 10395
```



## What is the average daily activity pattern?

Aggregate the steps over the 5-minutes intervals, taking the mean over all days:


```r
steps.per.interval <- aggregate(df$steps, list(df$interval), mean, na.rm = T)
names(steps.per.interval) <- c("interval", "mean.steps")
```


Plot the time series of the 5-minute interval and the average number of steps taken, averaged across all days.


```r
plot(steps.per.interval$interval, steps.per.interval$mean.steps, t = "l", main = "Average steps per interval", 
    ylab = "Number of steps", xlab = "Interval")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 


Show the 5-minutes intervall with the most steps (averaged over all days):


```r
steps.per.interval[steps.per.interval$mean.steps == max(steps.per.interval$mean.steps), 
    ]
```

```
##     interval mean.steps
## 104      835      206.2
```


## Imputing missing values

### Total number of missing values

The steps-field has 2304 missing values, whereas both the date and interval fields have no missing values:

```r
sum(is.na(df$steps))
```

```
## [1] 2304
```

```r
sum(is.na(df$date))
```

```
## [1] 0
```

```r
sum(is.na(df$interval))
```

```
## [1] 0
```


### Replacing missing values

Copy the datasest to a new data.frame `df.complete` and replace missing values with the mean of the 5-minutes-interval averaged over all days.


```r

df.complete <- df
for (i in 1:nrow(df)) {
    if (is.na(df[i, 1])) {
        df.complete[i, 1] <- steps.per.interval[steps.per.interval$interval == 
            df[i, 3], ][2]
    }
}
```


### Histogram of the steps taken per day, based on replaced missing values


```r
steps.per.day.complete <- aggregate(df.complete$steps, list(df.complete$date), 
    sum, na.rm = T)
names(steps.per.day.complete) <- c("date", "num.steps")

hist(steps.per.day.complete$num.steps, breaks = 10, main = "Histogram: number of steps per day", 
    xlab = "number of steps per day")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 


### Mean and median of steps taken per day, based on replaced missing values

Calculate mean ond median of the total number of steps per day:

```r
mean(steps.per.day.complete$num.steps, na.rm = T)
```

```
## [1] 10766
```

```r
median(steps.per.day.complete$num.steps, na.rm = T)
```

```
## [1] 10766
```


#### Changes introduced by replacing missing values: 
Days without a single steps measurement (i.e. only NAs) show up as having 0 steps in the histogram.   
With missing values replaced by the mean of the interval over the days, such days with seemingly
0 steps disappear.    
With missing values, the mean is 9354 and the median 10395.    
Without missing values, both mean and median are 10766.

## Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
weekend <- weekdays(as.Date(df.complete$date)) == "Saturday" | weekdays(as.Date(df.complete$date)) == 
    "Sunday"
df.complete$weekend <- factor(weekend, levels = c(FALSE, TRUE), labels = c("weekday", 
    "weekend"))
```


Number of steps per interval averaged over weekdays and over weekends:


```r
spiw <- aggregate(df.complete$steps, list(df.complete$weekend, df.complete$interval), 
    mean, na.rm = T)
library(lattice)
names(spiw) <- c("weekend", "interval", "mean.steps")
xyplot(mean.steps ~ interval | weekend, data = spiw, type = "b", layout = c(1, 
    2), xlab = "Interval", ylab = "Number of steps", pch = "")
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13.png) 

