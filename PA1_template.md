# Reproducible Research: Peer Assessment 1
M Askari  
Saturday, May 16, 2015  


This is an R Markdown document for peer assignment 1 of coursera course.  

### Data for this assignment

Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

### Variables in input data

- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

- date: The date on which the measurement was taken in YYYY-MM-DD format

- interval: Identifier for the 5-minute interval in which measurement was taken

### Setting Working Directory


```r
# Please set the working directory to location you saved input data
# Commented for final commit. 
# setwd("C:/XXXXXX/GitHub/RepData_PeerAssessment1")
```




## Loading and preprocessing the data

```r
# Loading and transforming the data
activityData<-read.table("activity.csv",
              header=TRUE,sep=",",
              colClasses=c('numeric','character','numeric'), 
              na.strings="NA")
activityDataTransformed<-mutate(activityData,NewDate=as.POSIXct(strptime(activityData$date, "%Y-%m-%d")))

# Selecting only 3 column and getting rid of old date column
activityData<-activityDataTransformed[,c("steps","NewDate","interval")]

# Renaming to all lower case names. 
names(activityData)<-c("steps","date","intervals")
```
  
   
   
## What is mean total number of steps taken per day?

Below chart shows a historgam for number of steps taken for each day. 

```r
#drawing the first chart
histData<-activityData %>% group_by(date) %>% summarise(totalSteps=sum(steps,na.rm=TRUE))
hist(histData$totalSteps,xlab="Steps",ylab="Days",col="red" ,breaks=5, main="Steps taken on a Daily basis")
```

![](figures/InitialHistogram-1.png) 

### Daily Mean

Below chart shows daily mean of steps taken (excluding NA days):


```r
groupedActivityData<-group_by(activityData,date)
dailySummary<-summarise(groupedActivityData,Daily_Mean=mean(steps,na.rm = TRUE),Daily_Median=median(steps,na.rm = TRUE))

with(dailySummary[!is.na(dailySummary$Daily_Mean),],
     {
             plot(date,Daily_Mean,type="n",xlab="Days",ylab="Mean Steps",xaxt = "n");
             lines(date,Daily_Mean,col="red")
             axis(1, date, format(date, "%d %b"), cex.axis = .6 ,las=2)
             legend("topleft",pch=NA,col=c("red"),legend=c("Daily Mean (NA excluded)"),lwd=1)
        
        
}
     )
```

![](figures/DailyMeanWithNA-1.png) 


### Daily Median

Below chart shows daily Median of steps taken (excluding NA days):


```r
with(dailySummary[!is.na(dailySummary$Daily_Median),],
     {
             plot(date,Daily_Median,type="n",xlab="Days",ylab="Median Steps",xaxt = "n");
             lines(date,Daily_Median,col="blue")
             axis(1, date, format(date, "%d %b"), cex.axis = .6 ,las=2)
             legend("topleft",pch=NA,col=c("blue"),legend=c("Daily Meidan (NA excluded)"),lwd=1)
        
        
}
     )
```

![](figures/DaiyMedianWithNA-1.png) 

Values of summary is presentd in variable named `dailySummary` . Here is how you can access this variable using head command:



```r
head(dailySummary,n=10)
```

```
## Source: local data frame [10 x 3]
## 
##          date Daily_Mean Daily_Median
## 1  2012-10-01        NaN           NA
## 2  2012-10-02    0.43750            0
## 3  2012-10-03   39.41667            0
## 4  2012-10-04   42.06944            0
## 5  2012-10-05   46.15972            0
## 6  2012-10-06   53.54167            0
## 7  2012-10-07   38.24653            0
## 8  2012-10-08        NaN           NA
## 9  2012-10-09   44.48264            0
## 10 2012-10-10   34.37500            0
```


## What is the average daily activity pattern?



```r
groupedActivityData<-group_by(activityData,intervals)
dailyPattern<-summarise(groupedActivityData, dailyMean=mean(steps,na.rm=TRUE))
with(dailyPattern,
     {
             plot(intervals,dailyMean,type="n",xlab="Minutes",ylab="Mean Steps");
             lines(intervals,dailyMean,col="blue")
             legend("toprigh",pch=NA,col=c("blue"),legend=c("Interval Meidan (NA excluded)"),lwd=1)
        
        
}
     )
```

![](figures/DailyActivityPattern-1.png) 

Arranging the variable `dailyPattern` by column of dailyMean and selecting the first row will give you answer to which interval has Maximum avg number of steps:


```r
dailyPattern<-arrange(dailyPattern,desc(dailyMean))
head(dailyPattern,n=1)
```

```
## Source: local data frame [1 x 2]
## 
##   intervals dailyMean
## 1       835  206.1698
```

Converting above value to time in a day. Makes 1:55 PM to 2:00 PM holding record of number of steps in our dataset.

## Imputing missing values 



In our data set we have following number as rows without value in "steps" column


```r
nrow(activityData[is.na(activityData$steps),])
```

```
## [1] 2304
```

### FIlling Strategy

I have decided to use mean of each interval as a value for filling up missing items. To rolaborate, if a value of interval "5" is missing for a specific day, I will date average of all available values for interval "5" and use it as indicative value.


```r
groupedActivityData<-group_by(activityData,intervals)
dailySummary<-summarise(groupedActivityData,Daily_Mean=mean(steps,na.rm = TRUE),
                        Daily_Median=median(steps,na.rm = TRUE))

missingdata<-activityData[is.na(activityData$steps),]
# Merging data based on interval
mergedData<-merge(missingdata,dailySummary,by="intervals")
mergedData<-mergedData[,c("Daily_Mean","date","intervals")]
names(mergedData)<-c("steps","date","intervals")

# Getting subset of data which had data from start
nomissingdata<-activityData[!is.na(activityData$steps),]

# Adding filled data back into the dataset
finalData<-rbind(nomissingdata,mergedData)


histData<-finalData %>% group_by(date) %>% summarise(totalSteps=sum(steps,na.rm=TRUE))
hist(histData$totalSteps,xlab="Steps",ylab="Days",col="red" ,breaks=5, main="Steps taken on a Daily basis")
```

![](figures/SecondHistReplacedNA-1.png) 
And now we are showing the daily Mean and Median after chaning the missing values

### Daily Mean ( With NA data replaced)


```r
groupedActivityData<-group_by(finalData,date)
dailySummary<-summarise(groupedActivityData,Daily_Mean=mean(steps,na.rm = TRUE),Daily_Median=median(steps,na.rm = TRUE))

with(dailySummary[!is.na(dailySummary$Daily_Mean),],
     {
             plot(date,Daily_Mean,type="n",xlab="Days",ylab="Mean Steps",xaxt = "n");
             lines(date,Daily_Mean,col="red")
             axis(1, date, format(date, "%d %b"), cex.axis = .6 ,las=2)
             legend("topleft",pch=NA,col=c("red"),legend=c("Daily Mean (NA Replaced)"),lwd=1)
        
        
}
     )
```

![](figures/DailyMeanReplacedNA-1.png) 

### Daily Median ( With NA data replaced)


```r
with(dailySummary[!is.na(dailySummary$Daily_Median),],
     {
             plot(date,Daily_Median,type="n",xlab="Days",ylab="Median Steps",xaxt = "n");
             lines(date,Daily_Median,col="blue")
             axis(1, date, format(date, "%d %b"), cex.axis = .6 ,las=2)
             legend("topleft",pch=NA,col=c("blue"),legend=c("Daily Meidan (NA Replaced)"),lwd=1)
        
        
}
     )
```

![](figures/DailyMedianReplacedNA-1.png) 

### Impact of replacing NA

As you can see and compare two histograms the filling up missing values DID NOT change the sample mean. Histogram is more peaked around the mean.



## Are there differences in activity patterns between weekdays and weekends?



```r
finalData<-mutate(finalData,WeeKDayFlag=ifelse((weekdays(date)=="Saturday"| weekdays(date)=="Sunday"),"weekend","weekday"))
groupedActivityData<-group_by(finalData,intervals,WeeKDayFlag)
dailyPattern<-summarise(groupedActivityData, dailyMean=mean(steps,na.rm=TRUE))

xyplot(dailyPattern$dailyMean~dailyPattern$intervals|dailyPattern$WeeKDayFlag,layout=c(1,2),lwd=3,type="l",xlab="Intervals(minutes)",ylab="Steps(Mean)",main="Weekend vs Weekday pattern comparision")
```

![](figures/weekdaysvsweekend-1.png) 













