---
title: "Peer Assessment"
author: "Seun Daniel"
date: "Saturday, January 17, 2015"
html_document:
    keep_md: true
---
##Peer Assignment 1
## Loading and preprocessing the data
```{r setoptions, echo = TRUE, results='asis'}

```

**Loading the necassary packages**

library(ggplot2)
library(grid)
library(gridExtra)
library(scales)

###First things first, we unzip and read in the csv file 

```{r Data Input}
dataFile <- unz("repdata-data-activity.zip", "activity.csv")
  activity <- read.csv(file = dataFile, header = TRUE, sep = ",")
```

###Then date is modified to suit this analysis
```{r Date and Time}
activity$datetime <- as.POSIXct(
    with(activity, paste(date, 
  paste(interval %/% 100, interval %% 100, sep=":"))), 
	format="%Y-%m-%d %H:%M",tz=""
)

```


###Plotting the histogram
```{r}
dailySteps <- setNames(
      aggregate(
          steps~as.Date(date),
          activity,
          sum,
          na.rm = TRUE),
      c("date","steps")
    )
library(ggplot2)
DailyStepsHist <- ggplot(dailySteps,aes(x=date,y=steps)) + 
  geom_bar(stat="identity") + 
  ggtitle("Total number of steps per day (source data)")

print(DailyStepsHist)

summary(dailySteps$steps)
```
##What is mean total number of steps taken per day?


```{r}
 dsf <- c(mean = mean(dailySteps$steps),median = median(dailySteps$steps))
  print(dsf)
```
##What is the average daily activity pattern?
```{r}
meanDailyPattern <- aggregate(steps~interval,activity,mean,na.rm = TRUE)
meanDailyPattern$time <- as.POSIXct(with(meanDailyPattern,paste(interval %/% 100, interval %% 100, sep=":")),format="%H:%M")
```

###Another plot
```{r}
library(scales)
histA <- ggplot(meanDailyPattern,aes(x=time,y=steps)) + 
          geom_line() + 
          scale_x_datetime(breaks = date_breaks("2 hour"),labels = date_format("%H:%M"))
print(histA)

```

##Inputting missing values
```{r}
with(meanDailyPattern, meanDailyPattern[steps == max(steps),])

csf <- aggregate(cnt~date,cbind(activity[is.na(activity$steps),],cnt=c(1)),sum,na.rm = FALSE)
csf$daysWK <- weekdays(as.Date(csf$date),abbreviate=TRUE)
print(csf[,c(1,3,2)])

unique(csf$daysWK)
```

##Are there differences in activity patterns between weekdays and weekends?

```{r}
quotedData <- aggregate(steps~interval+weekdays(datetime,abbreviate=TRUE),activity,FUN=mean,na.rm=TRUE)
colnames(quotedData) <- c("interval","daysWK","avg_steps")
quotedData$daysWK <- factor(quotedData$daysWK,levels = c("Mon","Tue","Wed","Thu","Fri","Sat","Sun"))
ggplot(quotedData,aes(x=interval,y=avg_steps)) + geom_line() + facet_grid("daysWK ~ .")


```


```{r}
activity$daysWK <- weekdays(activity$datetime,abbreviate=TRUE)
af <- merge(activity,quotedData,by=c("daysWK","interval"),all.x = TRUE)
af <- af[with(af,order(date,interval)),]
af$fixed_steps <- ifelse(is.na(af$steps),af$avg_steps,af$steps)

```

###Plotting another histogram

```{r}

dailySteps2 <- setNames(
      aggregate(
          fixed_steps~as.Date(date),
          af,
          sum,
          na.rm = TRUE),
      c("date","steps")
    )

histB <- ggplot(dailySteps2,aes(x=date,y=steps)) + 
  geom_bar(stat="identity") + 
  ggtitle("Total number of steps per day (fixed data)")

print(histB)
```



###Combining both histograms

```{r}
library(grid)
library(gridExtra)
grid.arrange(histA, histB, nrow=2)

 xsf <- c(mean = mean(dailySteps2$steps),median = median(dailySteps2$steps))
  comparison <- rbind(source = dsf, fixed = xsf, delta = dsf - xsf)
  print(comparison)

week_diff <- aggregate(
  steps~daysWK+interval,  # group steps by weekend/weekday and interval to find average steps 
  with(
    activity,
    data.frame(
      daysWK = factor(
        ifelse(
          weekdays(as.Date(date)) %in% c("Sunday","Saturday"),
          "weekend",  # if sunday or saturday
          "weekday"   # else
        )
      ),
      interval,
      steps
    )
  ),
  FUN = mean,
  rm.na = TRUE
)

```
####Thank you for your time!

