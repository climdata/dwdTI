---
title: "dwdTI"
author: "KMicha71"
date: "17 8 2021"
output:
  html_document: 
    keep_md: true
  pdf_document: default
---



## Work with yearly and moving/rolling 30y normalization


```r
require("ggplot2")
```

```
## Loading required package: ggplot2
```

```r
color.temperature <- c("#0000FF", "#00CCCC", "#FFFFFF", "#EEAA33", "#FF5555")
#install.packages("data.table")
library(data.table)
#install.packages("zoo")
library(zoo)
```

```
## 
## Attaching package: 'zoo'
```

```
## The following objects are masked from 'package:base':
## 
##     as.Date, as.Date.numeric
```

```r
py3 <- read.csv("https://raw.githubusercontent.com/climdata/baur/master/csv/baur_monthly.csv", sep=",")

rollingAVG <- function(data,mon,wid) {
  py4 <-  subset(data, (data$year>1752 & data$month == mon)) 
  avg <- rollapply(py4$temperature, width=wid, by=1, FUN=mean) 
  std <- rollapply(py4$temperature, width=wid, by=1, FUN=sd)
  py4 <-tail(py4, n=1-wid)
  py4$ind <- (py4$temperature - avg)/std

  return(py4)   
}


py5 <- rollingAVG(py3,1,30)
for (mont in c(2,3,4,5,6,7,8,9,10,11,12)) {
  tmp <-rollingAVG(py3,mont,30)
  py5 <- rbind(py5, tmp)
}

norm <- py5

mp <- ggplot(norm, aes(year, month))
mp + geom_raster(aes(fill=ind))+
  scale_y_continuous(breaks=c(1,6,12))+
  theme(panel.background = element_rect(fill = '#EEEEEE', colour = 'white'), legend.position="right", text=element_text(size=14))+
  scale_fill_gradientn(colours=color.temperature)
```

![](README_files/figure-html/rolling-norm-30-1.png)<!-- -->

```r
#hist(norm$temperature)
```


## Work with yearly and moving/rolling 10y normalization


```r
#install.packages("data.table")
library(data.table)
#install.packages("zoo")
library(zoo)

py5 <- rollingAVG(py3,1,10)
for (mont in c(2,3,4,5,6,7,8,9,10,11,12)) {
  tmp <-rollingAVG(py3,mont,10)
  py5 <- rbind(py5, tmp)
}

norm <- py5


mp <- ggplot(norm, aes(year, month))
mp + geom_raster(aes(fill=ind))+
  scale_y_continuous(breaks=c(1,6,12))+
  theme(panel.background = element_rect(fill = '#EEEEEE', colour = 'white'), legend.position="right", text=element_text(size=14))+
  scale_fill_gradientn(colours=color.temperature)
```

![](README_files/figure-html/rolling-norm-10-1.png)<!-- -->

```r
#hist(norm$temperature)
```



## Work with yearly and moving/rolling 10y linearization


```r
#install.packages("data.table")
library(data.table)
#install.packages("zoo")
library(rollRegres)

rollingGLM <- function(data, mon, wid) {
  py4 <-  subset(data, (data$year>1752 & data$month == mon))
  reg <- roll_regres(temperature ~ year, py4, width = wid, do_compute=c('sigmas', '1_step_forecasts')) 
  #reg <- roll_regres(temperature ~ year, py4, width = wid, do_compute=c('sigmas'))
  lapply(reg, tail)
  py4$ind <- (py4$temperature - reg$one_step_forecasts)/reg$sigmas  
  py4$ind2 <- (py4$temperature - py4$year*reg$coefs[,2]+reg$coefs[,1])/reg$sigmas
  
  py4 <-tail(py4, n=-wid)
  return(py4) 
}  

py5 <- rollingGLM(py3,1,10)
for (mont in c(2,3,4,5,6,7,8,9,10,11,12)) {
  tmp <-rollingGLM(py3,mont,10)
  py5 <- rbind(py5, tmp)
}

#py5 <-tail(py5, n=-10)
norm <- py5

mp <- ggplot(norm, aes(year, month))
mp + geom_raster(aes(fill=ind))+
  scale_y_continuous(breaks=c(1,6,12))+
  theme(panel.background = element_rect(fill = '#EEEEEE', colour = 'white'), legend.position="right", text=element_text(size=14))+
  scale_fill_gradientn(colours=color.temperature)
```

![](README_files/figure-html/rolling-glm-10-1.png)<!-- -->

```r
#hist(norm$temperature)
#hist(norm$ind)
```


## Work with yearly and moving/rolling 30y linearization


```r
#install.packages("data.table")
library(data.table)
#install.packages("zoo")
library(rollRegres)

py5 <- rollingGLM(py3,1,30)
for (mont in c(2,3,4,5,6,7,8,9,10,11,12)) {
  tmp <-rollingGLM(py3,mont,30)
  py5 <- rbind(py5, tmp)
}

#py5 <-tail(py5, n=-10)
norm <- py5

mp <- ggplot(norm, aes(year, month))
mp + geom_raster(aes(fill=ind))+
  scale_y_continuous(breaks=c(1,6,12))+
  theme(panel.background = element_rect(fill = '#EEEEEE', colour = 'white'), legend.position="right", text=element_text(size=14))+
  scale_fill_gradientn(colours=color.temperature)
```

![](README_files/figure-html/rolling-glm-30-1.png)<!-- -->

```r
#hist(norm$temperature)
#hist(norm$ind)
```


## Work with yearly and fixed normalization


```r
#install.packages("data.table")
library(data.table)
library(zoo)

py3 <- read.csv("https://raw.githubusercontent.com/climdata/baur/master/csv/baur_monthly.csv", sep=",")


py4 <-  subset(py3, (py3$year>1752 & py3$month == 1)) 

avg <- mean(py4$temperature, na.rm = TRUE)
std <- sd(py4$temperature, na.rm = TRUE)
py4$ind <- (py4$temperature-avg)/std

for (mont in c(2,3,4,5,6,7,8,9,10,11,12)) {
  tmp <-subset(py3, (py3$year>1752 & py3$month == mont)) 
  avg <- mean(tmp$temperature, na.rm = TRUE)
  std <- sd(tmp$temperature, na.rm = TRUE)
  tmp$ind <- (tmp$temperature-avg)/std
  py4 <- rbind(py4, tmp)
}

norm <- py4

mp <- ggplot(norm, aes(year, month))
mp + geom_raster(aes(fill=ind))+
  scale_y_continuous(breaks=c(1,6,12))+
  theme(panel.background = element_rect(fill = '#EEEEEE', colour = 'white'), legend.position="right", text=element_text(size=14))+
  scale_fill_gradientn(colours=color.temperature)
```

![](README_files/figure-html/fixed-norm-1.png)<!-- -->

```r
#hist(norm$temperature)
```
