---
title: "PA1_template.Rmd"
author: "KirbyArinder"
date: "Sunday, February 15, 2015"
output: html_document
---
This is a document intended to fulfill the requirements for the first peer assessment for the "Reproducible Research" class.  It is intended to be read in conjunction with the relevant assignment page for that class.  

## Loading and preprocessing the data

The first step in fulfilling this assignment was to load the data, as follows:     

```{r}
basedata <- read.csv("RepResPA1data.csv")
```

The column intended to represent the date was stored in such a way as to cause sorting errors, so the data were processed by converting that column into date format as follows:  

```{r}
basedata$date <- as.Date(basedata$date, format ="%m/%d/%Y")
```

No further preprocessing was necessary.  

## What is mean total number of steps taken per day?

This question was answered using tools from the plyr package, as follows:  

```{r}
library(plyr)
library(dplyr)

stepsperday <- ddply(basedata, c("date"), summarise, steps = sum(steps))
```

The mean and median steps per day were calculated as follows:  

```{r}
meansteps <- mean(stepsperday$steps, na.rm = TRUE)
medsteps <- median(stepsperday$steps, na.rm = TRUE)
```

The mean number of steps per day, ignoring missing values, is 10766.19;the median is 10765.  

A histogram of the total number of steps taken per day, created using the appended code, appears below.  

```{r}
hist(stepsperday$steps, xlab = "total steps per day", main = "Steps per day, missing values ignored")
```


## What is the average daily activity pattern?

In order to plot the number of steps per interval, averaged over days of the week, it was first necessary to calculate the mean steps per interval.  This was done as follows:  

```{r}
avstepsperinterval <- ddply(basedata, c("interval"), summarise, steps = mean(steps, na.rm = TRUE))
```

The largest number of mean steps per interval was calculated as follows:  

```{r}
moststepsperinterval <- avstepsperinterval[which(avstepsperinterval$steps == max(avstepsperinterval$steps)), ]
```

Interval 835 had the largest mean number of steps:  206.1698.  

The code below creates the plot also included below, of average daily activity pattern in terms of steps per interval.  

```{r}
plot(avstepsperinterval$interval, avstepsperinterval$steps, type="l", ylab = "steps", xlab = "interval", main = "Mean steps per interval")
```

## Imputing missing values

Of the two simple imputation strategies suggested on the assignment page, the strategy of replacing NA values with the average value of the interval in question over all days was the more appropriate.  Replacing NA values with the average value of the day in question over all intervals fails to remove all NA values, as some days have no measurements at all.  No intervals have no measurements at all, however, even if some intervals have only measurements whose value is zero.  As such, for purposes of this assignment, all NA values were replaced with the average value of the appropriate interval over all days.  

###IMPORTANT GRADING NOTE

As of this writing, I have been unable to implement the above imputation strategy in R.  The logic seems clear, but I have been unable to write functional code for it.  I don't have any reference books with me, my search-fu is apparently weak, and there's a deadline; nonetheless, I am deeply ashamed, and intend to solve the problem on my own time.  However, in this fallen world, we must sometimes achieve results by imperfect mechanisms.  In order to partially complete the assignment, I read the base data into Excel, implemented the strategy in Excel -- adding a column containing imputed values, rather than replacing the original -- saved the resulting file as a csv, and read it back into R.  

The following R code read my modified csv.  I fear that, without access to this modified csv, you will have only my word, the summary statistics, and the shape of the plot to assure you that the imputation algorithm was implemented in a logically correct fashion.  It was, and I could include the Excel function I used, but really I wouldn't give this method any better grade for doing so, so I'm not going to worry about it.  


```{r}
basedata2 <- read.csv("RepResPA1dataMOD2.csv")
```

Since "RepResPA1dataMOD2.csv" differs from the csv provided with the assignment only by virtue of its extra column, "interpolated", containing imputed values in place of NAs, it was necessary to convert the "date" column into date format, just as is in the first section of this document.  The following code accomplished the date conversion: 

```{r}
basedata2$date <- as.Date(basedata2$date, format ="%m/%d/%Y")
```

Here's a look inside the preprocessed csv I'm using for this section, just to assure you of the bona fides of the modifications I made, and so the following code is a little clearer.    

```{r}
head(basedata2)
```

The following code summarizes the total number of steps per day on the imputed-values column of the new file:  

```{r}
imputedstepsperday <- ddply(basedata2, c("date"), summarise, interpolated = sum(interpolated))
```

And the following code calculates the mean and median of steps per day, with imputed values included.  

```{r}
meanimputed <- mean(imputedstepsperday$interpolated, na.rm = TRUE)
medimputed <- median(imputedstepsperday$interpolated, na.rm = TRUE)
```

The mean and median values, post-imputation, are both 10766.19.  Thus, this imputation strategy leaves the mean number of steps per day unchanged, and brings the median into alignment with the mean, as expected.  

The code below creates the appended histogram:  

```{r}
hist(imputedstepsperday$interpolated, xlab = "total steps per day", main = "Steps/day, missing values imputed")
```

Once again, though the post-imputation mean, median, and histogram are correct, I wouldn't give full credit for this solution if I were grading it; I'd count off one point for failing to provide an imputation strategy AND code for that strategy.  I gave you a strategy, but not code.  I might, if I were feeling particularly strict, also count off one point for failing to provide R code for all results; strictly speaking, if you have the right csvs, the r code provided WILL produce all results given, but you have to have a csv not provided with the assignment that implements a proper imputation strategy as described.  Anyway, alas, I'd count off one or two points.  Sic transit gloria mundi.  

## Are there differences in activity patterns between weekdays and weekends?

The following code finds the days of the week and classifies them by a factor variable with levels "Weekend" and "Weekday".  


```{r}
#This part finds the days of the week.
basedata2$day <- weekdays(basedata2$date)

#This part classifies the days of the week.  
basedata2$weekday <- ifelse(basedata2$day %in% c("Saturday", "Sunday"), "Weekend", "Weekday")

#This part changes the weekday column into a factor.    
basedata2$weekday <- as.factor(basedata2$weekday)
```

And the following code creates subsets by that factor. 

```{r}  
WEdata <- subset(basedata2, weekday == "Weekend", select = c(steps, interpolated, date, interval, day, weekday))
WDdata <- subset(basedata2, weekday == "Weekday", select = c(steps, interpolated, date, interval, day, weekday))
```

The following summarizes the subsets:  

```{r}
WEavstepsperinterval <- ddply(WEdata, c("interval"), summarise, interpolated = mean(interpolated))
WDavstepsperinterval <- ddply(WDdata, c("interval"), summarise, interpolated = mean(interpolated))
```

And finally, the following code plots the subsets.  In both plots, the Y axis maximum was altered to a consistent 300 in order to allow for less misleading visual comparison. The answer to the question posed is that people walk less overall on the weekends, but with a spike at early intervals.  

```{r}
par(mfrow = c(2,1))
with(WEavstepsperinterval, (plot(interval, interpolated, type = "l", ylab = "mean steps", ylim = c(0, 300), main = "Weekend mean steps per interval")))
with(WDavstepsperinterval, (plot(interval, interpolated, type = "l", ylab = "mean steps", ylim = c(0, 300), main = "Weekday mean steps per interval")))
```