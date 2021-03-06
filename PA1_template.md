# Reproducible Research: Peer Assessment 1

### Loading and preprocessing the data


```r
activity <- read.csv("activity.csv")
```
***Basic information about the data***

```r
dim(activity)
```

```
## [1] 17568     3
```

```r
summary(activity)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

```r
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

### What is mean total number of steps taken per day?


```r
library(dplyr)
```
***Preparing the data needed for visualisation***

```r
activity_pday <- activity %>%  
        group_by(date) %>% 
        summarise(total_steps = sum(steps))
```
***Plotting histogram***

```r
hist(activity_pday$total_steps, xlab= "Total steps per day",
     main="Histogram of the total number of steps taken each day")  
```

<img src="./PA1_template_files/figure-html/unnamed-chunk-5-1.png" title="" alt="" style="display: block; margin: auto;" />
***Calculating mean and median total number of steps taken per day***

```r
mean_steps1 <- mean(activity_pday$total_steps, na.rm= TRUE)
median_steps1 <- median(activity_pday$total_steps, na.rm= TRUE)

knitr::kable(data.frame(MEAN= mean_steps1, MEDIAN= median_steps1),
             digits=3, align='l')
```



MEAN       MEDIAN 
---------  -------
10766.19   10765  

### What is the average daily activity pattern?

***Preparing data for visualisation***

```r
activity_avg <- activity %>% 
        group_by(interval) %>%
        summarise(mean_steps=mean(steps,na.rm= TRUE))
```
***Plotting the average daily activity pattern***

```r
library(ggplot2)
qplot(x=interval,y=mean_steps,geom="line", data=activity_avg, 
      main='Average daily activity pattern', ylab='avg steps taken')   
```

<img src="./PA1_template_files/figure-html/unnamed-chunk-8-1.png" title="" alt="" style="display: block; margin: auto;" />
***Which 5-minute interval contains the maximum number of steps***

```r
subset(activity_avg, mean_steps == max(mean_steps),
                    select=interval)
```

```
## Source: local data frame [1 x 1]
## 
##   interval
## 1      835
```

### Imputing missing values

***Total number of missing values in the dataset***

```r
sum(apply(is.na(activity), 1, sum))
```

```
## [1] 2304
```
***Filling in all of the missing values in the dataset***

```r
activity_fill<- activity

for (i in 1:nrow(activity)) {
        if (is.na(activity[i,'steps'])) {
                sub <- na.omit(subset(activity,interval == (activity[i,'interval']),
                             select=steps))
                activity_fill[i,'steps']=round(colMeans(sub))
        }
}
```
***Preparing data for visualisation and plotting histogram***

```r
activity_fill_pday <- activity_fill %>%  
        group_by(date) %>% 
        summarise(total_steps=sum(steps))

histogram <- hist(activity_fill_pday$total_steps, xlab="Total steps per day",
             main="Histogram of the total number of steps taken each day")
```

<img src="./PA1_template_files/figure-html/unnamed-chunk-12-1.png" title="" alt="" style="display: block; margin: auto;" />
***Summary of both mean and median statistics with and withouth NAs filled.***

```r
mean_steps <- mean(activity_fill_pday$total_steps)
median_steps <- median(activity_fill_pday$total_steps)

knitr::kable(data.frame(newMean=mean_steps, oldMean=mean_steps1,
                         newMedian=median_steps, oldMedian=median_steps1),
             digits=3, align='l')
```



newMean    oldMean    newMedian   oldMedian 
---------  ---------  ----------  ----------
10765.64   10766.19   10762       10765     

### Are there differences in activity patterns between weekdays and weekends?

***Creating a new factor variable***

```r
activity_fill$week <- NULL

activity_fill$week <- apply(activity_fill, 1, function(row) {
	week5 <- c("Monday","Tuesday","Wednesday", "Thursday","Friday")
        row['week'] <- ifelse(
        	any(weekdays(as.Date(row['date'])) == week5), 
                "weekday",
		"weekend"
	)
})
activity_fill$week <- as.factor(activity_fill$week)
```
***Preparing data for visualisation and plotting***

```r
activity_fill_plot <- activity_fill %>% 
	group_by(interval, week) %>%
        summarise(mean_steps=mean(steps))
        	  
ggplot(activity_fill_plot, aes(x=interval,y=mean_steps))+
	labs(y='avg steps taken') +
	geom_line(color="aquamarine4") + facet_wrap(~week, nrow=2) +
	ggtitle("Differences in activity patterns between weekdays and weekends")
```

<img src="./PA1_template_files/figure-html/unnamed-chunk-15-1.png" title="" alt="" style="display: block; margin: auto;" />
