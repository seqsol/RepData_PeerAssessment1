# Reproducible Research: Peer Assessment 1
Xiaolong He  

## Task 1: Loading and preprocessing the data 
Below are codes for this task:

```r
# Load data into an R object
activity <- read.csv("activity.csv", header=TRUE)

# Transform the data into the format suitable for manipulation by the functions of dplyr package
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
activity <- tbl_df(activity)
```

## Task 2: What is mean total number of steps taken per day?

The first part of the task is to calculate the total number of steps taken per day

```r
# Calculate the number of total steps taken per day
day.steps <- activity %>%
    group_by(date) %>%
    summarize(total.steps=sum(steps, na.rm=TRUE))
```

The second part of the task is  to plot histogram of the total number of steps taken each day:

```r
# Draw histogram of total steps per day
hist(day.steps$total.steps, xlab="Total steps per day", main="Histogram of total steps per day")
```

![](PA1_template_files/figure-html/task2_pt2-1.png)<!-- -->

The third part of the task is to calculate and report the mean and median of the total number of steps taken per day

```r
mean.day.steps <- mean(day.steps$total.steps)
median.day.steps <- median(day.steps$total.steps)
print(paste("The mean of the total number of steps taken per day is", format(mean.day.steps)))
```

```
## [1] "The mean of the total number of steps taken per day is 9354.23"
```

```r
print(paste("The median number of the total steps taken per day is", format(median.day.steps)))
```

```
## [1] "The median number of the total steps taken per day is 10395"
```

## Task 3: What is the average daily activity pattern?

The first part of the task is to make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
# Calculate the average number of steps taken for each 5-minute interval across all days
interval.steps <- activity %>%
    group_by(interval) %>%
    summarise(avg.steps=mean(steps, na.rm=TRUE))

# Make a time series plot of 5-minute interval and average number of steps
with(interval.steps, plot(interval, avg.steps, type="l", xlab="5 minutes intervals", ylab="Average steps"))
```

![](PA1_template_files/figure-html/task3_pt1-1.png)<!-- -->

The second part of the task is to find the 5-minute interval that has the maximum average number of steps

```r
# Find the interval that has the maximum average number of steps
interval.max <- interval.steps$interval[which.max(interval.steps$avg.steps)]
print(paste("The interval that has the maximum average number of steps is", interval.max, ", which is between 8:30 and 8:35"))
```

```
## [1] "The interval that has the maximum average number of steps is 835 , which is between 8:30 and 8:35"
```

## Task 4: Imputing missing values

The first part of the task is to calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
# Find the number of NAs in the dataset
total.NAs <- sum(is.na(activity$steps)) # Total NAs: 2304
print(paste("There are", total.NAs, "missing values in the dataset"))
```

```
## [1] "There are 2304 missing values in the dataset"
```

The second part of the task is to devise a strategy for filling in all of the missing values in the dataset. 
In this practice, I replace NAs in each interval with the average numbers of steps of corresponding intervals

The third part of the task is to create a new dataset that is equal to the original dataset but with the missing data filled in

```r
# Replace the NAs with the average numbers of steps of corresponding intervals, the resulting new dataset is activity.v2
activity.v2 <- activity %>%
    group_by(interval) %>%
    mutate(steps=ifelse(is.na(steps), mean(steps, na.rm=TRUE), steps))
```

The fourth part of the task is to make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day

```r
# Calculate the number of total steps taken per day in the new dataset
new.day.steps <- activity.v2 %>%
    group_by(date) %>%
    summarize(total.steps=sum(steps, na.rm=TRUE))

# Draw histogram
hist(new.day.steps$total.steps, xlab="Total steps per day", main="Histogram of total steps per day")
```

![](PA1_template_files/figure-html/task4_pt4-1.png)<!-- -->

```r
# Calculate and report the mean and median total number of steps taken per day
mean.day.steps <- mean(new.day.steps$total.steps)
median.day.steps <- median(new.day.steps$total.steps)
print(paste("The mean of the total number of steps taken per day is", format(mean.day.steps)))
```

```
## [1] "The mean of the total number of steps taken per day is 10766.19"
```

```r
print(paste("The median number of the total steps taken per day is", format(median.day.steps)))
```

```
## [1] "The median number of the total steps taken per day is 10766.19"
```

The new mean and median values differ from those of the original dataset. Imputing missing values results in higher mean and median value.

## Task 5: Are there differences in activity patterns between weekdays and weekends?

The first part of the  task is to create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day

```r
# Create a new factor variable indicating whether a given date is a weekday or a weekend 

weekday <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
# Function classifying a date into weekday or weekend
whatday <- function(date){
        if (weekdays(date) %in% weekday){
            return("weekday")
        }else {
            return("weekend")
        }
}

# Add a column to activity.v2 to mark the dates in the dataset as weekday or weekend.
activity.v3 <- activity.v2 %>%
    group_by(date) %>%
    mutate(what.day=sapply(as.Date(date), whatday))
```

The second part of the task is to make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)

```r
# Calculate the average number of steps taken for each 5-minute interval across all weekdays and or weekend days
interval.steps.v2 <- activity.v3 %>%
    group_by(what.day, interval) %>%
    summarise(avg.steps=mean(steps, na.rm=TRUE))

# Make a panel time series plot showing the average numbers of steps in every 5-minute intervals across all weekdays or across all weekend days
library(ggplot2)
library(grid)
g<-ggplot(interval.steps.v2, aes(interval, avg.steps))
g+ geom_line(color="blue")+facet_wrap(~what.day,nrow=2, strip.position="top")+labs(x="Interval", y="Number of average steps")+theme(strip.background = element_rect(colour = "black", fill="#F1DABC"), strip.text.x = element_text(size = 12), panel.spacing = unit(0, "lines"))
```

![](PA1_template_files/figure-html/task5_pt2-1.png)<!-- -->


























