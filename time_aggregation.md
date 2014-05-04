# my little reminder for (just some) time aggregation methods
### obviously many other methods are also possible but not here contemplated (known)!

*****

## generate some fake data to play with

#### time sequence using POSIXct class (storing date-time and many other features like tz, dst)


```r
# start and end of time seq
start <- as.POSIXct("2014-05-01 00:00:00")
end <- as.POSIXct("2014-05-02 00:00:00")

# by sec
bysec <- as.POSIXct(seq(start, end, by = "sec"))
# alternatively
bysec <- as.POSIXct(seq(start, end, by = 1))

# by min
bymin <- as.POSIXct(seq(start, end, by = "min"))
# alternatively
bymin <- as.POSIXct(seq(start, end, by = 60))

# by hour
byhour <- as.POSIXct(seq(start, end, by = "hour"))
# alternatively
byhour <- as.POSIXct(seq(start, end, by = 60 * 60))

# and so on for other time intervals...

# eg. by 15 minutes
by15min <- as.POSIXct(seq(start, end, by = "15 mins"))
# alternatively
by15min <- as.POSIXct(seq(start, end, by = 60 * 15))
```


#### df with 3 vars 


```r
set.seed(123)
v1 <- rnorm(length(bymin), 5)

set.seed(456)
v2 <- rnorm(length(bymin), 10)

# here using bymin as time seq
df <- data.frame(date = bymin, v1, v2)

# introduce some random NAs
df[sample(nrow(df), 20), "v1"] <- NA
df[sample(nrow(df), 10), "v2"] <- NA

summary(df)
```

```
##       date                           v1             v2       
##  Min.   :2014-05-01 00:00:00   Min.   :2.19   Min.   : 7.04  
##  1st Qu.:2014-05-01 06:00:00   1st Qu.:4.35   1st Qu.: 9.38  
##  Median :2014-05-01 12:00:00   Median :5.01   Median :10.05  
##  Mean   :2014-05-01 12:00:00   Mean   :5.01   Mean   :10.06  
##  3rd Qu.:2014-05-01 18:00:00   3rd Qu.:5.65   3rd Qu.:10.76  
##  Max.   :2014-05-02 00:00:00   Max.   :8.39   Max.   :13.06  
##                                NA's   :20     NA's   :10
```

```r
str(df)
```

```
## 'data.frame':	1441 obs. of  3 variables:
##  $ date: POSIXct, format: "2014-05-01 00:00:00" "2014-05-01 00:01:00" ...
##  $ v1  : num  4.44 4.77 6.56 5.07 5.13 ...
##  $ v2  : num  8.66 10.62 10.8 8.61 9.29 ...
```


## aggregation through base methods (no packages)
****

### mean by hour using format

```r
head(aggregate(df[c("v1", "v2")], format(df["date"], "%Y-%m-%d %H"), mean, na.rm = TRUE))
```

```
##            date    v1     v2
## 1 2014-05-01 00 5.057 10.194
## 2 2014-05-01 01 4.965  9.970
## 3 2014-05-01 02 5.012  9.787
## 4 2014-05-01 03 4.963 10.198
## 5 2014-05-01 04 5.194 10.023
## 6 2014-05-01 05 4.987 10.263
```

```r
# alternatively
head(aggregate(df[c(2, 3)], format(df[1], "%Y-%m-%d %H"), mean, na.rm = TRUE))
```

```
##            date    v1     v2
## 1 2014-05-01 00 5.057 10.194
## 2 2014-05-01 01 4.965  9.970
## 3 2014-05-01 02 5.012  9.787
## 4 2014-05-01 03 4.963 10.198
## 5 2014-05-01 04 5.194 10.023
## 6 2014-05-01 05 4.987 10.263
```


### mean by hour using cut

```r
# see cut.POSIXt for refs
head(aggregate(df[c("v1", "v2")], list(date = cut(df$date, breaks = "hour")), 
    mean, na.rm = TRUE))
```

```
##                  date    v1     v2
## 1 2014-05-01 00:00:00 5.057 10.194
## 2 2014-05-01 01:00:00 4.965  9.970
## 3 2014-05-01 02:00:00 5.012  9.787
## 4 2014-05-01 03:00:00 4.963 10.198
## 5 2014-05-01 04:00:00 5.194 10.023
## 6 2014-05-01 05:00:00 4.987 10.263
```


### mean by 15 mins using cut

```r
head(aggregate(df[c("v1", "v2")], list(date = cut(df$date, breaks = "15 mins")), 
    mean, na.rm = TRUE))
```

```
##                  date    v1     v2
## 1 2014-05-01 00:00:00 5.152 10.118
## 2 2014-05-01 00:15:00 4.753 10.345
## 3 2014-05-01 00:30:00 5.277 10.012
## 4 2014-05-01 00:45:00 5.061 10.301
## 5 2014-05-01 01:00:00 4.869  9.825
## 6 2014-05-01 01:15:00 5.180  9.998
```


## aggregation through some dedicated package methods
****

## library zoo (just some relvant examples)


```r
require(zoo)
```

```
## Loading required package: zoo
## 
## Attaching package: 'zoo'
## 
## The following objects are masked from 'package:base':
## 
##     as.Date, as.Date.numeric
```

### myzoo object

```r
myzoo <- zoo(cbind(df$v1, df$v2), df$date)
str(myzoo)
```

```
## 'zoo' series from 2014-05-01 to 2014-05-02
##   Data: num [1:1441, 1:2] 4.44 4.77 6.56 5.07 5.13 ...
##   Index:  POSIXct[1:1441], format: "2014-05-01 00:00:00" "2014-05-01 00:01:00" ...
```

#### note index() or time()

### plot zoo series with lattice

```r
require(lattice)
```

```
## Loading required package: lattice
```

```r
xyplot(myzoo, screens = c("v1", "v2"), scales = list(y = list(relation = "same")))
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-81.png) 

```r
xyplot(myzoo, superpose = TRUE, auto.key = list(text = c("v1", "v2")))
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-82.png) 


### mean by hour using format


```r
head(aggregate(myzoo, format(index(myzoo), "%Y-%m-%d %H"), mean, na.rm = TRUE))
```

```
##                  V1     V2
## 2014-05-01 00 5.057 10.194
## 2014-05-01 01 4.965  9.970
## 2014-05-01 02 5.012  9.787
## 2014-05-01 03 4.963 10.198
## 2014-05-01 04 5.194 10.023
## 2014-05-01 05 4.987 10.263
```



### mean by hour 15 mins using cut


```r
head(aggregate(myzoo, cut(index(myzoo), breaks = "15 mins"), mean, na.rm = TRUE))
```

```
##                        V1     V2
## 2014-05-01 00:00:00 5.152 10.118
## 2014-05-01 00:15:00 4.753 10.345
## 2014-05-01 00:30:00 5.277 10.012
## 2014-05-01 00:45:00 5.061 10.301
## 2014-05-01 01:00:00 4.869  9.825
## 2014-05-01 01:15:00 5.180  9.998
```


### rolling average by 5 mins (trailing average)


```r
head(rollapply(myzoo, width = 5, mean, align = "right", na.rm = TRUE))
```

```
##                                 
## 2014-05-01 00:04:00 5.194  9.595
## 2014-05-01 00:05:00 5.649  9.799
## 2014-05-01 00:06:00 5.787  9.813
## 2014-05-01 00:07:00 5.222  9.703
## 2014-05-01 00:08:00 5.071 10.182
## 2014-05-01 00:09:00 4.956 10.440
```

#### note that rollmean() does not handle NAs

### rolling average by 5 mins (average at every 5 mins)


```r
head(rollapply(myzoo, width = 5, FUN = mean, by = 5, align = "right", na.rm = TRUE))
```

```
##                                 
## 2014-05-01 00:04:00 5.194  9.595
## 2014-05-01 00:09:00 4.956 10.440
## 2014-05-01 00:14:00 5.308 10.319
## 2014-05-01 00:19:00 5.109 11.578
## 2014-05-01 00:24:00 4.267  9.311
## 2014-05-01 00:29:00 4.884 10.148
```






## library xts

```r
require(xts)
```

```
## Loading required package: xts
```

## library chron

```r
require(chron)
```

```
## Loading required package: chron
```

## library lubridate

```r
require(lubridate)
```

```
## Loading required package: lubridate
## 
## Attaching package: 'lubridate'
## 
## The following objects are masked from 'package:chron':
## 
##     days, hours, minutes, seconds, years
```

