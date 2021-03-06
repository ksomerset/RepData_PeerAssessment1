---
output: 
  html_document: 
    keep_md: yes
---
##Fit Data Summary - Assignment for Reproducible Research, Week 2

###Preliminary step: Read in the data and load key libraries.
```{r Prelim} 
FitData <- read.csv("activity.csv", header=TRUE)
library(dplyr)
library(ggplot2)
library(xtable)
library(knitr)
library(markdown)
```

###QUESTION 1 - What is the mean total number of steps taken per day?
####PART 1 - Calculate totals by date, and the mean of those totals, excluding nas. Show result.
```{r TotalsAndMean}
SumStepsByDate <- FitData %>% group_by(date) %>% summarize (sum.steps = sum(steps, na.rm=TRUE))
MeanStepsByDate <- mean(SumStepsByDate$sum.steps)
MeanStepsByDate
```
The mean number of steps taken per day is: `r MeanStepsByDate` 

####PART 2 - Create a histogram of the total steps taken per day.
```{r HistogramOfTotalSteps}
qplot(sum.steps, data=SumStepsByDate, binwidth=250, 
      xlab = "Occurrences", ylab = "Total Steps Per Day")

```

####PART 3 - Calculate and report mean and median of total number of steps taken per day.
```{r TotalStepsMeanAndMedian}
StepsMean <- mean(SumStepsByDate$sum.steps)
StepsMedian <- median(SumStepsByDate$sum.steps)
StepsMean
StepsMedian

```
The mean total steps taken per day is `r StepsMean`; the median is `r StepsMedian`.

###QUESTION 2 - What is the average daily activity pattern?
####PART 1 - Make a time series plot of the average steps taken by 5-minute interval.
```{r TimeSeriesAverageStepsByInterval}
StepsByInterval <- group_by(FitData, interval)
AvgStepsByInterval <- summarize(StepsByInterval, avg.steps = mean(steps, na.rm=TRUE))
Plot <- ggplot (AvgStepsByInterval, aes(interval, avg.steps))
Plot + geom_line(size = 1, linetype = 1) + labs(title = "Average Steps By Interval") + labs(x = "Interval", y = "Average Steps Over All Days")

```

####PART 2 - Find the interval with the maximum average steps over all days.
```{r FindIntervalWithMaximumSteps}
MaxAvg <- AvgStepsByInterval[which.max(AvgStepsByInterval$avg.steps),]
MaxAvg

```

The time interval with the highest average steps across all days is interval 835 with 206 steps. 

###QUESTION 3 - Imputing Missing Values
####PART 1 - Calculate and report the total number of missing values in the dataset.
```{r CountNas}
CountNA <- sum(is.na(FitData$steps))
CountNA
meanNA <- mean(is.na(FitData$steps))

```

There are `r CountNA` NA values in the "steps" column of the data; this is `r meanNA` as a percentage of the total rows.

####PART 2 - Devise a strategy for filling in the missing data.

The strategy is to replace each NA with the average of that interval over all days.

####PART 3 - Create a new dataset with the missing values filled in.

```{r FillInMissingData}
## create a column with average steps
FitDataPlus <- cbind (FitData, AvgStepsByInterval) 
colnames(FitDataPlus) <- c("steps", "date", "interval", "interval2", "avg.steps")
## remove the extra intervals column
FitDataPlus$interval2 <- NULL  
## add the imputed values
ind <- is.na(FitDataPlus$steps)
FitDataPlus$steps[ind] <- FitDataPlus$avg.steps[ind]
NewSumStepsByDate <- FitDataPlus %>% group_by(date) %>% 
      summarize (sum.steps = sum(steps))

```

####PART 4 - Use new dataset to re-create the histogram and mean and median total steps per day.

```{r CreateHistogramMeanMedian}
qplot(sum.steps, data=NewSumStepsByDate, binwidth=250, 
      xlab = "Occurrences", ylab = "Total Steps Per Day")
NewStepsMean <- mean(NewSumStepsByDate$sum.steps)
NewStepsMedian <- median(NewSumStepsByDate$sum.steps)

```
With the imputed data, the mean number of steps per day increases from  `r StepsMean` to `r NewStepsMean`. 

The median number of steps per day increases from `r StepsMedian` to `r NewStepsMedian`. 

###QUESTION 4 - Investigating weekday vs. weekend activity
####PART 1 - Create a new factor variable indicating weekend or weekday.
```{r CreateWeekendVsWeekdayVariable}
FitDataPlus$day <- weekdays(as.Date(FitDataPlus$date))
FitDataPlus$weekend <- if_else(FitDataPlus$day %in% c("Saturday", "Sunday"), "Weekend", "Weekday")

```
 
####PART 2 - Make a panel plot of interval vs. average steps - weekend vs. weekday.
```{r GraphDifferenceBetweenWeekendAndWeekday}
## establish which days are weekdays and which are weekends
FitDataPlus$day <- weekdays(as.Date(FitDataPlus$date))
FitDataPlus$weekend <- if_else(FitDataPlus$day %in% c("Saturday", "Sunday"), "Weekend", "Weekday")
## calculate the average steps per interval for each data set, weekend vs. weekday
DataWeekend <- subset(FitDataPlus, weekend == "Weekend")
DataWeekday <- subset(FitDataPlus, weekend == "Weekday")
AvgStepsWeekend <- DataWeekend %>% group_by(interval) %>% 
      summarize (avg.steps = mean(steps))
AvgStepsWeekday <- DataWeekday %>% group_by(interval) %>% 
      summarize (avg.steps = mean(steps))
par(mfrow = c(2, 1), mar = c(4,4,2,1))
plot(AvgStepsWeekend$interval, AvgStepsWeekend$avg.steps, type = "l", 
     xlab = "Interval", ylab = "Average Steps", main = "Weekend")
plot(AvgStepsWeekday$interval, AvgStepsWeekday$avg.steps, type = "l", 
     xlab = "Interval", ylab = "Average Steps", main = "Weekday")

```

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo=TRUE,cache=TRUE)
knitr::opts_chunk$set(fig.path="figs/fig-")

```
