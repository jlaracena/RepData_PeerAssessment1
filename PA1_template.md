# Reproducible Research: Peer Assessment 1

## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement -- a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## Data

The data for this assignment can be downloaded from the course web site:

Dataset: Activity monitoring data [52K]
The variables included in this dataset are:

- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

- date: The date on which the measurement was taken in YYYY-MM-DD format

- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.


## Loading and preprocessing the data

1. Loading de data:
```{r}
data1 <- read.csv("activity.csv")
head(data1)
```

2. Process/transform the data (if necessary) into a format suitable for your analysis
```{r}
# Doesn't apply
```

## What is mean total number of steps taken per day?

1. Calculate the total number of steps taken per day

```{r}
data1.splt.day <- split(data1, as.factor(data1$date))
steps_day <- sapply(data1.splt.day, function(x) sum(x$steps, na.rm=T))
```

2. Make a histogram of the total number of steps taken each day
```{r}
hist(steps_day, breaks=20)
```

3. Calculate and report the mean and median of the total number of steps taken per day

```{r}
mean(sapply(data1.splt.day, function(x) sum(x$steps, na.rm=T)))

median(sapply(data1.splt.day, function(x) sum(x$steps, na.rm=T)))
```

## What is the average daily activity pattern?

1. Make a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```{r}
data1.splt.interv <- split(data1, as.factor(data1$interval))
out <- sapply(data1.splt.interv, function(x) mean(x$steps, na.rm=T))

grap1 <- plot(as.numeric(names(out)), out, type="l", lwd=2,
     xlab="Interval", ylab="Steps (averaged over days)")
```


2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r}
c(interval=names(which.max(out)), round(max_steps=max(out),))
```


## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with 𝙽𝙰s)

```{r}
sum(is.na(data1$steps))
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```{r}
missing <- is.na(data1$steps)
mean_interv <- sapply(data1.splt.interv, function(x) mean(x$steps, na.rm=T))

n_day <- length(levels(as.factor(data1$date)))
n_day
```


3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```{r}
data1.imputed <- data1
data1.imputed[missing,]$steps <- rep(mean_interv, n_day)[missing]
cbind(head(data1[missing,]), head(data1.imputed[missing,]))
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```{r}
steps_day.imputed <- tapply(data1.imputed$steps, as.factor(data1$date), sum)
hist(steps_day.imputed)
```


## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```{r}
data1.imputed$day <- weekdays(as.Date(data1.imputed$date))
daylevel <- vector()

for (i in 1:nrow(data1)) {
        if (data1.imputed$day[i] == "Saturday") {
                daylevel[i] <- "Weekend"
        } else if (data1.imputed$day[i] == "Sunday") {
                daylevel[i] <- "Weekend"
        } else {
                daylevel[i] <- "Weekday"
        }
}

data1.imputed$daylevel <- daylevel
data1.imputed$daylevel <- factor(data1.imputed$daylevel)

stepsbyday <- aggregate(steps ~ interval + daylevel, data = data1.imputed, mean)
names(stepsbyday) <- c("interval", "daylevel", "steps")
```

2. Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```{r}
library(lattice)
xyplot(steps ~ interval | factor(daylevel),
       data=stepsbyday,
       type = 'l',
       layout = c(1, 2),
       xlab="5-Minute Intervals",
       ylab="Average Steps Taken")
```
