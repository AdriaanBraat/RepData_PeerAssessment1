# Reproducible Research: Peer Assessment 1


#### Reproducible research week 2 - assignment 1

## Introduction and research questions
This report describes an analysis of data from a personal activity monitoring device. This device collects data at 5 minute intervals trough out the day. The report contains the following steps:

1. Code for reading the data and/or processing the data
2. How are the total number of steps each day distribituted? 
      + Histogram of the total number of steps
      + Mean and median number of steps taken each day
      + Time series plot of the average number of steps taken.


### The data
The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The variables included in the dataset are:

- __steps__: Number of steps taken in a 5-minute interval (missing values code as NA)
- **date**: The date on which the measurement was taken in YYYY-MM-DD format
- **interval**: Identifier for the 5-minute interval in which the measurement was taken.


### Reading the data
We read the data from my folder "Reproducible research"

```r
setwd("~/R/Coursera/Reproducible research")
list.files()
```

```
##  [1] "activity.csv"              "Markdown_example.docx"    
##  [3] "Markdown_example.html"     "Markdown_example.Rmd"     
##  [5] "PA1_template.Rmd"          "repdata_data_activity.zip"
##  [7] "spambase.txt"              "Week 1 lecture notes.R"   
##  [9] "Week 2 Assignment 1.Rmd"   "Week 2 lecture notes.R"   
## [11] "Week_2_Assignment_1.html"
```

```r
unzip("repdata_data_activity.zip")
df_Activity <- read.csv("activity.csv")
```

### What is the mean total number of steps taken per day? 
This research question is answered by following a number of steps:

#### A Histogram of the total number of steps

The following code gives a histogram on the total number of steps per day.

First we make a vector with the total number of steps per day using aggregate, then we make a histogram:


```r
activity_per_day <- aggregate(steps ~ date, df_Activity, FUN = sum)
hist(activity_per_day$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

#### Mean and median of the total number of steps
The mean and median are easily calculated using the activity_per_day vector. On some days there were no measurements. We neglect the missing values in the calculation of both the mean and the median.


```r
mean(activity_per_day$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(activity_per_day$steps, na.rm = TRUE)
```

```
## [1] 10765
```

The mean and median are more or less the same

### What is the average daily activity pattern?
The data contains step measures on 5-minute intervals. By calculating the average of each 5-minute interval accross the days, the average daily pattern can be produced. The following code generates an average for each interval and plots the time series:


```r
activity_per_interval <- aggregate(steps ~ interval, df_Activity, FUN = mean)
plot(activity_per_interval$steps, type = "l", xlab = "5-minute interval", ylab = "average number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

The highest activity is between the 100st and 120st 5-minute time interval. This means most actvitiy is between 8 and 9 in the morning.

### Imputing missing values
The data contains a lot of missing data:


```r
sum(is.na(df_Activity$steps))
```

```
## [1] 2304
```

```r
sum(is.na(df_Activity$steps))/dim(df_Activity)
```

```
## [1]   0.1311475 768.0000000
```
As we see from the second calculation this is about 13% of the total number of lines.

We can impute the data with the mean on the five minute interval. 


```r
df_Activity_imputed <- merge(df_Activity, activity_per_interval, by = "interval")
df_Activity_imputed$steps.x[is.na(df_Activity_imputed$steps.x)] <- df_Activity_imputed$steps.y[is.na(df_Activity_imputed$steps.x)]
```

And create a histogram and calculate the mean and the median of the total number of steps


```r
activity_per_day_imputed <- aggregate(steps.x ~ date, df_Activity_imputed, FUN = sum)
hist(activity_per_day_imputed$steps.x)
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

```r
mean(activity_per_day_imputed$steps.x)
```

```
## [1] 10766.19
```

```r
median(activity_per_day_imputed$steps.x)
```

```
## [1] 10766.19
```
The mean and median are not much affected by imputing the missing data.

### Are there differences in activity patterns between weekdays and weekends
We go back to the original dataset and add a factorvariable with two levels: weekday and weekend.


```r
df_Activity$date <- as.Date(df_Activity$date)
Weekdays <- c('maandag', 'dinsdag', 'woensdag', 'donderdag', 'vrijdag')
df_Activity$WeekDay <- factor((weekdays(df_Activity$date) %in% Weekdays), 
         levels=c(FALSE, TRUE), labels=c('weekend', 'weekday'))
table(df_Activity$WeekDay)
```

```
## 
## weekend weekday 
##    4608   12960
```
2/7th of the lines are weekend lines and 5/7th of the lines are weekday lines. 

Now we investigate if there are differences in activity patterns by calculating the average number of steps per time interval averaged over weekends and weekdays and display it in a timeseries panel plot.


```r
activity_per_interval_weekdays <- aggregate(df_Activity$steps, list(interval = df_Activity$interval, weekday = df_Activity$WeekDay), mean, na.rm =TRUE)
library(lattice)
xyplot(activity_per_interval_weekdays$x ~ activity_per_interval_weekdays$interval | activity_per_interval_weekdays$weekday, type = "l", layout = c(1,2), xlab = "5-minute interval", ylab = "Average steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->
