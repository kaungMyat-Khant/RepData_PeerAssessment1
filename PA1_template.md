---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

``` r
knitr::opts_chunk$set(echo = TRUE, comment = "",warning = F, message = F)
```


## Loading and preprocessing the data

### Load the data

``` r
if(file.exists("activity.zip")){
    unzip("activity.zip")
    data <- read.csv("activity.csv")
} else {
    print("File not found")
}
```


### Data processing
First of all, we will describe the data.

``` r
dim(data)
```

```
[1] 17568     3
```

``` r
str(data)
```

```
'data.frame':	17568 obs. of  3 variables:
 $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
 $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
 $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

``` r
summary(data)
```

```
     steps            date              interval     
 Min.   :  0.00   Length:17568       Min.   :   0.0  
 1st Qu.:  0.00   Class :character   1st Qu.: 588.8  
 Median :  0.00   Mode  :character   Median :1177.5  
 Mean   : 37.38                      Mean   :1177.5  
 3rd Qu.: 12.00                      3rd Qu.:1766.2  
 Max.   :806.00                      Max.   :2355.0  
 NA's   :2304                                        
```
As shown above, there are *three variables* and *17568 observation* in the data. The data set consists of *integer* and *character* vectors. Since the **date** variable is a *character*, we have to format it as a *date* using the ***as.Date*** function. Additionally, the interval variable is an integer and we need to transform the variable into a factor. We can see five minutes interval for each activities counting the steps. The recorded minutes ranged from zero to 2355 minutes. The step count ranged from **0 to 806** with a mean of 37 steps. There are **2304 missing values**.  


``` r
data$date <- as.Date(data$date, "%Y-%m-%d")
str(data$date)
```

```
 Date[1:17568], format: "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
```

``` r
period <- range(data$date)
```
Now the *date* column becomes *Date* format and the records were from 2012-10-01 to 2012-11-30.


``` r
library(ggplot2)
library(dplyr)
```


## What is mean total number of steps taken per day?

``` r
data %>% 
    group_by(date) %>% 
    summarise(steps = sum(steps,na.rm = T)) %>% 
    ggplot(aes(steps))+
    geom_histogram(binwidth = 1000, fill = "darkgreen", color = "black")+
    labs(title = "Histogram showing total numbers of steps for each day",
         x = "Total number of steps each day",
         y = "Frequency")+
    scale_x_continuous(breaks = seq(0, 22000, 1000))+
    scale_y_continuous(limits = c(0,12), breaks = 1:12)+
    theme_minimal()+
    theme(axis.text.x = element_text(angle = 90))
```

<img src="PA1_template_files/figure-html/histogramSteps-1.png" style="display: block; margin: auto;" />
The histogram showed that the total step counts were spreading more or less normally around 10,000 steps per day.



``` r
total_step <- data %>% 
    group_by(date) %>% 
    summarise(steps = sum(steps,na.rm = T)) %>% 
    ungroup() 
mean1 <- round(mean(total_step$steps),0)
median1 <- median(total_step$steps)
```
The mean of the total step count per day is **9354**.  

The median of the total step count per day is **10395**.

## What is the average daily activity pattern?

``` r
data %>% 
    group_by(interval) %>% 
    summarise(steps = mean(steps, na.rm = T)) %>% 
    ggplot(aes(x = interval, y = steps))+
    geom_line(color = "darkgreen")+
    labs(title = "Time series plot for five minutes intervals for daily activity",
         y = "Average steps taken",
         x = "Intervals(minutes)")+
    scale_x_continuous(limits = c(0,2500))+
    scale_y_continuous(limits = c(0,250))+
    theme_minimal()
```

<img src="PA1_template_files/figure-html/timeSeriesPlot-1.png" style="display: block; margin: auto;" />

According to the line graph, the five-minute intervals between 750 and 1000 minutes is the most active during the day. During a day, participants are inactives before 500 minutes. Then, the participants' step counts increased to maximum after 750 minutes and showed fluctuations after 1000 minutes. After 2000 minute, the steps graually decreased.


``` r
avg_steps <- with(data, tapply(steps, as.factor(interval),mean, na.rm = TRUE))
df_steps <- data.frame(names = names(avg_steps), steps = avg_steps)
max <- df_steps[df_steps$steps == max(df_steps$steps),"names"]
```
Among the 5-minute intervals, on average across all the days in the dataset, **interval 835** contained the maximum number of steps.

## Imputing missing values


``` r
total_NA <- sum(is.na(data$steps))
```
There are **2304** missing values in the step count. 


``` r
interval_median <- data %>%
    group_by(interval) %>% 
    summarise(median = median(steps, na.rm= T))

data_imputed <- data %>% 
    left_join(interval_median, by = join_by(interval)) %>% 
    mutate(steps = if_else(is.na(steps), median, steps)) %>% 
    select(-median) 

sum(is.na(data_imputed$steps))
```

```
[1] 0
```
The missing data were imputed using the **median value for each interval**.


``` r
data_imputed %>% 
    group_by(date) %>% 
    summarise(steps = sum(steps)) %>% 
    ggplot()+
    geom_histogram(aes(steps), binwidth = 1000, fill = "lightgreen", colour = "grey7")+
    labs(title = "Histogram showing total number of step count for each day",
         y = "Frequency",
         x = "Total number of step each day")+
    scale_x_continuous(breaks = seq(0,22000, 1000))+
    scale_y_continuous(limits = c(0,10),
                       breaks = seq(0,10, 1))+
    theme_minimal()+
    theme(axis.text.x = element_text(angle = 90))
```

<img src="PA1_template_files/figure-html/histTotalStepImputed-1.png" style="display: block; margin: auto;" />


``` r
total_step_imputed <- data_imputed %>% 
    group_by(date) %>% 
    summarise(steps = sum(steps))
mean2 <- round(mean(total_step_imputed$steps),0)
median2 <- median(total_step_imputed$steps)
```
For the imputed data, mean of the total number of steps taken per day was **9504**, and the median was **10395**. The median did not change after imputing the missing values but the mean changed. With median imputation the median remained stable but the mean shifted with distribution.

## Are there differences in activity patterns between weekdays and weekends?


``` r
data_imputed <- data_imputed %>% 
    mutate(weekdays = weekdays(date,T)) %>% 
    mutate(weekdays = if_else(weekdays %in% c("Sat","Sun"), 
                              "Weekend", "Weekday")) %>% 
    mutate(weekdays = factor(weekdays, levels = c("Weekday","Weekend"),
                             labels = c("Weekday", "Weekend")))
```


``` r
data_imputed %>% 
    group_by(weekdays, interval) %>% 
    summarise(steps = mean(steps)) %>%
    ungroup() %>% 
    ggplot(aes(x = interval, y = steps))+
    geom_line(color = "navy")+
    scale_x_continuous(limits = c(0,2500))+
    scale_y_continuous(limits = c(0,250))+
    labs(x = "Intervals", y = "Average number of steps",
         title = "Time series plot for five minutes intervals for daily activity")+
    facet_wrap(~ weekdays, nrow = 2)+
    theme_minimal()
```

<img src="PA1_template_files/figure-html/timeSeriesPanelPlot-1.png" style="display: block; margin: auto;" />
The walking activity and average step for each interval showed different patterns for weekday and weekends.
