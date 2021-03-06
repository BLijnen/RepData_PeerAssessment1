PA1\_template
================

Reproducible research - course project 1
========================================

Import dataset into R Studio and load packages
----------------------------------------------

``` r
library(readr)
activity <- read.csv("C:/Users/blijnen/Documents/activity.csv")
```

library(knitr) library(lattice)

Data preprocessing
------------------

``` r
activity$date <- as.Date(activity$date, "%Y-%m-%d")
```

Data analysis
-------------

### What is mean total number of steps taken per day?

1.  Calculate the total number of steps taken per day

``` r
steps_per_day<-tapply(activity$steps, activity$date, sum)
```

1.  Make a histogram of the total number of steps taken each day

``` r
hist(steps_per_day, main="Histogram of number of steps per day", ylim=c(0,30), xlab="total number of steps", col="coral")
```

![](PA1_template_files/figure-markdown_github-ascii_identifiers/graph1-1.png)

1.  Calculate and report the mean and median of the total number of steps taken per day

``` r
mean_day<- mean(steps_per_day, na.rm = TRUE)
median_day<-median(steps_per_day, na.rm=TRUE)
```

The average number of steps each day is 10766.19 and the median is 10765.

### What is the average daily activity pattern?

1.  Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

``` r
steps_interval <- tapply(activity$steps, activity$interval, mean, na.rm = TRUE)
plot(as.numeric(names(steps_interval)),
     steps_interval, 
     xlab = "5-minute interval", 
     ylab = "Average number of steps",
     main = "Average daily activity pattern", 
     type = "l", 
     lwd=2, 
     col="grey")
```

![](PA1_template_files/figure-markdown_github-ascii_identifiers/graph2-1.png)

1.  Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

``` r
max_steps <- sort(steps_interval, decreasing = TRUE)[1]
```

The interval \[835, 840\] contains the maximum average number of steps (206 steps)

### Imputing missing values

1.  Calculate and report the total number of missing values in the dataset

``` r
total_NA <- sum(is.na(activity$steps))
```

The dataset contains 2304 missing values.

1.  Devise a strategy for filling in all of the missing values in the dataset.

I will replace the missing values by the mean value for each interval

1.  Create a new dataset that is equal to the original dataset but with the missing data filled in.

``` r
x <- split(activity, activity$interval)
for(i in 1:length(x)){
    x[[i]]$steps[is.na(x[[i]]$steps)] <- steps_interval[i]
}
impute_NA <- do.call("rbind",x)
impute_NA <- impute_NA[order(impute_NA$date),]
```

1.  Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

``` r
steps_day_impute <- tapply(impute_NA$steps, impute_NA$date, sum)
hist(steps_day_impute, main="Histogram of number of steps per day (imputed data)", ylim=c(0,30),
     xlab="total number of steps", col="coral")
```

![](PA1_template_files/figure-markdown_github-ascii_identifiers/graph3-1.png)

``` r
mean_days_impute<- mean(steps_day_impute, na.rm = TRUE)
median_days_impute<- median(steps_day_impute, na.rm = TRUE)
```

Both the mean and the median are 10766.19. Compared to the data without imputation, the mean remained unchanged wheras the median has slightly increased.

### Are there differences in activity patterns between weekdays and weekends?

1.  Create a new factor variable in the dataset with two levels "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

``` r
impute_NA$day <- ifelse(weekdays(as.Date(impute_NA$date)) == "Saturday" | 
                                weekdays(as.Date(impute_NA$date)) == "Sunday", "weekend", "weekday")   
```

1.  Make a panel plot containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days.

``` r
day <- weekdays(activity$date)
day_factor <- vector()
for (i in 1:nrow(activity)) {
    if (day[i] == "Saturday") {
        day_factor[i] <- "Weekend"
    } else if (day[i] == "Sunday") {
        day_factor[i] <- "Weekend"
    } else {
        day_factor[i] <- "Weekday"
    }
}
activity$day_factor <- day_factor
activity$day_factor <- factor(activity$day_factor)
steps_days <- aggregate(steps ~ interval + day_factor, data = activity, mean)
names(steps_days) <- c("interval", "day_factor", "steps")
library(lattice)
xyplot(steps ~ interval | day_factor, steps_days, type = "l", layout = c(1, 2), 
    xlab = "Interval", ylab = "Number of steps")
```

![](PA1_template_files/figure-markdown_github-ascii_identifiers/graph4-1.png)
