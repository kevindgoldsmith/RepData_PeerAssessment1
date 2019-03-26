Reproducible Research Project 1
-------------------------------

#### 1) Code for reading in the dataset and/or processing the data

Load the data

    fitdata<-read.csv("activity.csv")

Process/transform the data into a suitable format for analysis

    fitdata$date<-as.Date(as.character(fitdata$date), 
                       format = "%Y-%m-%d")
    fitdataNB<-na.omit(fitdata)

#### 2) Histogram of the total number of steps taken each day

Calculate the total number of steps taken per day

    library(plyr)
    daydata<- ddply(fitdataNB,.(date),summarize,TotalSteps=sum(steps))

Make a histogram of the total number of steps taken each day

    hist(daydata$TotalSteps, main = "Histogram of steps per day", xlab = "Total Steps per Day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-4-1.png)

#### 3) Mean and median number of steps taken each day

Calculate and report the mean and median of the total number of steps
taken per day

    meansteps = mean(daydata$TotalSteps)
    mediansteps = median(daydata$TotalSteps)
    cat("Mean: ",  meansteps, "\nMedian: ", mediansteps)

    ## Mean:  10766.19 
    ## Median:  10765

#### 4) Time series plot of the average number of steps taken

Make a time series plot of the 5-minute interval and the average number
of steps taken, averaged across all days

    plot(daydata, type = "l", main = "Total Steps by Day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-6-1.png)

#### 5)The 5-minute interval that, on average, contains the maximum number of steps

Which 5-minute interval, on average across all the days in the dataset,
contains the maximum number of steps?

    intervalavg<-ddply(fitdataNB, .(interval), summarize, AvgSteps = mean(steps))
    maxsteps = which.max(intervalavg$AvgSteps)
    cat("Higest average interval: ", intervalavg$interval[maxsteps])

    ## Higest average interval:  835

#### 6) Code to describe and show a strategy for imputing missing data

Calculate and report the total number of missing values in the dataset

    NAcount = sum(is.na(fitdata$steps))
    print(NAcount)

    ## [1] 2304

Devise a strategy for filling in all of the missing values in the
dataset. Use the mean for the 5-minute interval. Create a new dataset
that is equal to the original dataset but with the missing data filled
in.

    library(dplyr)
    fitdata2<-fitdata %>% group_by(interval) %>% mutate(steps = ifelse(is.na(steps), mean(steps, na.rm = TRUE), steps)) %>% ungroup
    daydata2<- ddply(fitdata2,.(date),summarize,TotalSteps=sum(steps))

#### 7) Histogram of the total number of steps taken each day after missing values are imputed

Make a histogram of the total number of steps taken each day

    hist(daydata2$TotalSteps, main = "Histogram of steps per day", xlab = "Total Steps per Day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-10-1.png)

Calculate and report the mean and median total number of steps taken per
day

    meansteps = mean(daydata2$TotalSteps)
    mediansteps = median(daydata2$TotalSteps)
    cat("Mean: ",  meansteps, "\nMedian: ", mediansteps)

    ## Mean:  10766.19 
    ## Median:  10766.19

#### 8) Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

Create a new factor variable in the dataset with two levels - "weekday"
and "weekend"

    byweekday<-mutate(fitdata2,WD = factor(ifelse(weekdays(fitdata2$date) %in% c("Saturday", "Sunday"),"Weekend","Weekday")))

Make a panel plot containing a time series plot of the 5-minute interval
and the average number of steps taken, averaged acrossed all weekdays or
weekend days

    library(lattice)
    intervaldata<- ddply(byweekday,c(.(interval),.(WD)),summarize,AvgSteps=mean(steps))
    xyplot(AvgSteps ~ interval | WD, data = intervaldata, layout = c(1,2), type = "l")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-13-1.png)
