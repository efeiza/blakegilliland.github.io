---
title: " An Assessment of FGCU's Learning Assistant Program"
date: 2020-05-16
tags: [R, ggplot2]
excerpt: "Statistical Analysis of Learning Assistants"
mathjax: "true"
---

### Background

Learning Assistants (LA's) are talented undergraduates who have recently taken a course and remember what it is like to learn the material. They help transform undergraduate courses to include small groups of students articulating, defending, and modifying their ideas about relevant problems or phenomena. Their main role is to support student learning in interactive classroom environments, working with small groups of students as they solve challenging conceptual or mathematical problems.

FGCU’s Learning Assistant program began in 2016 and spanned a wide range of STEM disciplines. In the last year, it has expanded to non-STEM classes as well. We are interested in measuring the effectiveness of the program and determining methods of improvement for the future based on DFW rates.


### Analysis

How can we get a grasp on how to quantify LA impact on student performance? Let's consider our data. We have data from Fall 2016 through Spring 2019 on a by-semester, and by-section granularity level.

```r
library(tidyverse)
library(dplyr)
library(xtable)
library(ggplot2)
library(knitr)

dat = read.csv("LA_DFW_Data.csv")[,1:14]
dat$DFW = dat$DF.Number+dat$W.Number
dat = dat[-1,c(1,4,5,6,7,8,10,12,15)]
attach(dat)
head(dat)
```

    ##     Course                  Course.Name      Term  LA   CRN Avg..DFW.RATE
    ## 1 EVR1001C Intro. Environmental Science Fall 2018  No 82604          0.07
    ## 2 EVR1001C Intro. Environmental Science Fall 2018  No 82605          0.10
    ## 3 EVR1001C Intro. Environmental Science Fall 2018  No 83202          0.08
    ## 4 EVR1001C Intro. Environmental Science Fall 2018 Yes 83350          0.08
    ## 5 COP2006   Introduction to Programming Fall 2016 Yes 80599          0.04
    ## 6 COP2006   Introduction to Programming Fall 2016  No 80600          0.09
    ##   Students DFW
    ## 1       72   5
    ## 2       69   7
    ## 3       75   6
    ## 4       38   3
    ## 5       24   1
    ## 6       32   3

As you can see by the head of our .csv, we know what subject the section was (columns 1 and 2), what term it was taught in, whether there was a LA or not, which section (CRN) it was (and therefore who taught it), what the DFW (D/F/Withdraw) rate was for that section, how many students were in it, and the DFW count, respectively.

```r
tab1 = aggregate(DFW,list(LA,Course.Name),sum)
tab2 = aggregate(Students,list(LA,Course.Name),sum)
tab3 = cbind(tab1,round(tab1[,3]/tab2[,3],2))
colnames(tab3) = c("LA","Course","DFW Count","DFW Rate")
kable(tab3)
```

| LA  | Course                       | DFW Count | DFW Rate |
| :-- | :--------------------------- | --------: | -------: |
| No  | Calculus I                   |       124 |     0.33 |
| Yes | Calculus I                   |        39 |     0.28 |
| No  | Calculus II                  |        26 |     0.20 |
| Yes | Calculus II                  |        12 |     0.34 |
| No  | College Algebra              |       784 |     0.28 |
| Yes | College Algebra              |        88 |     0.31 |
| No  | College Physics w/Lab II     |         5 |     0.16 |
| Yes | College Physics w/Lab II     |         5 |     0.16 |
| No  | Elementary Calculus          |        76 |     0.35 |
| Yes | Elementary Calculus          |        50 |     0.47 |
| No  | Engineering Mechanics        |        14 |     0.27 |
| Yes | Engineering Mechanics        |         6 |     0.16 |
| No  | Finite Mathematics           |        23 |     0.11 |
| Yes | Finite Mathematics           |        18 |     0.17 |
| No  | Gen’l Biology w/Lab I        |       477 |     0.34 |
| Yes | Gen’l Biology w/Lab I        |        40 |     0.35 |
| No  | General Chemistry I          |       682 |     0.37 |
| Yes | General Chemistry I          |       178 |     0.39 |
| No  | General Chemistry II         |       138 |     0.33 |
| Yes | General Chemistry II         |       152 |     0.41 |
| No  | Intermediate Algebra         |       464 |     0.22 |
| Yes | Intermediate Algebra         |        62 |     0.29 |
| No  | Intro Earth Science          |        47 |     0.15 |
| Yes | Intro Earth Science          |        22 |     0.16 |
| No  | Intro to Computer Science    |        36 |     0.22 |
| Yes | Intro to Computer Science    |        31 |     0.17 |
| No  | Intro. Environmental Science |        18 |     0.08 |
| Yes | Intro. Environmental Science |         3 |     0.08 |
| No  | Introduction to Programming  |         6 |     0.12 |
| Yes | Introduction to Programming  |        21 |     0.13 |
| No  | Precalculus                  |       211 |     0.25 |
| Yes | Precalculus                  |        53 |     0.22 |
| No  | Social Science Statistics    |        12 |     0.18 |
| Yes | Social Science Statistics    |         5 |     0.16 |
| No  | Statistical Methods          |       674 |     0.25 |
| Yes | Statistical Methods          |        23 |     0.21 |

### Check Normality before Applying Parametric Test
```r
shapiro.test(tab3$`DFW Rate`[tab3$LA=="Yes"]-tab3$`DFW Rate`[tab3$LA=="No"])

hist(tab3$`DFW Rate`[tab3$LA=="Yes"]-tab3$`DFW Rate`[tab3$LA=="No"],xlab = "Difference between DFW Rates by Course",main=paste("Histogram of the differences between \nDFW Rates by Courses with/without LA's"))
```
    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  tab3$`DFW Rate`[tab3$LA == "Yes"] - tab3$`DFW Rate`[tab3$LA ==     "No"]
    ## W = 0.97282, p-value = 0.8487
    
![](/Users/blakegilliland/Documents/GitHub/blakegilliland.github.io/images/LA_Paper_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->    

### Paired T-test for DFW Rate, pairing by Course. Looking for an LA Effect.

```r
LA_DFW<-tab3$`DFW Rate`[tab3$LA=="Yes"]
NonLA_DFW<-tab3$`DFW Rate`[tab3$LA=="No"]
t.test(LA_DFW,NonLA_DFW,paired=TRUE,alternative="less")

plot(density(tab3$DFW.Rate[tab3$LA=="No"]),xlim=c(0,.7),main = "Distribution of DFW Rates for Courses with LA's or without LA's",xlab = "DFW Rate")
lines(density(tab3$DFW.Rate[tab3$LA=="Yes"]),col="red")
legend(x=.5,y=3.5,col = c("Black", "Red"),legend=c("No","Yes"),lwd=1)
```
    ## 
    ##  Paired t-test
    ## 
    ## data:  LA_DFW and NonLA_DFW
    ## t = 0.93843, df = 17, p-value = 0.8194
    ## alternative hypothesis: true difference in means is less than 0
    ## 95 percent confidence interval:
    ##        -Inf 0.03963536
    ## sample estimates:
    ## mean of the differences 
    ##              0.01388889

![](/Users/blakegilliland/Documents/GitHub/blakegilliland.github.io/images/LA-analysis_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

```r
dat1 = read.csv("PairedData.csv")[,4:15]
dat1$DFW = dat1$DF.Number+dat1$W.Number
attach(dat1)

tab1 = aggregate(DFW,list(LA,Instructors),sum)
tab2 = aggregate(Students,list(LA,Instructors),sum)
tab3 = cbind(tab1,round(tab1[,3]/tab2[,3],2))
colnames(tab3) = c("LA","Instructors","DFW Count","DFW Rate")
kable(tab3)

```
| LA  | Instructors | DFW Count | DFW Rate |
| :-- | ----------: | --------: | -------: |
| No  |           1 |        25 |     0.32 |
| Yes |           1 |        27 |     0.36 |
| No  |           2 |        19 |     0.26 |
| Yes |           2 |        21 |     0.30 |
| No  |           3 |        25 |     0.17 |
| Yes |           3 |        42 |     0.29 |
| No  |           4 |         2 |     0.06 |
| Yes |           4 |         3 |     0.09 |
| No  |           5 |        13 |     0.23 |
| Yes |           5 |        13 |     0.23 |
| No  |           6 |        29 |     0.33 |
| Yes |           6 |        25 |     0.28 |
| No  |           7 |        20 |     0.56 |
| Yes |           7 |        11 |     0.16 |
| No  |           8 |        42 |     0.49 |
| Yes |           8 |        45 |     0.48 |
| No  |           9 |         6 |     0.08 |
| Yes |           9 |         3 |     0.08 |
| No  |          10 |         5 |     0.16 |
| Yes |          10 |         5 |     0.16 |
| No  |          11 |        29 |     0.54 |
| Yes |          11 |        50 |     0.47 |
| No  |          12 |         5 |     0.16 |
| Yes |          12 |         5 |     0.16 |
| No  |          13 |        45 |     0.48 |
| Yes |          13 |        45 |     0.49 |
| No  |          14 |         6 |     0.19 |
| Yes |          14 |        14 |     0.20 |
| No  |          15 |        14 |     0.16 |
| Yes |          15 |        10 |     0.19 |

```r
shapiro.test(tab3$`DFW Rate`[tab3 == "Yes"]-tab3$`DFW Rate`[tab3$LA == "No"])

hist(tab3$`DFW Rate`[tab3$LA == "No"],xlab = "Difference between DFW Rates by Instructor",main=paste("Histogram of the differences between \nDFW Rates by Instructors with/without LA's"))
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  tab3$`DFW Rate`[tab3 == "Yes"] - tab3$`DFW Rate`[tab3$LA == "No"]
    ## W = 0.63002, p-value = 4.869e-05

![](/Users/blakegilliland/Documents/GitHub/blakegilliland.github.io/images/LA_Paper_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->


```r
LA_DFW<-tab3$`DFW Rate`[tab3 == "Yes"]
NonLA_DFW<-tab3$`DFW Rate`[tab3$LA == "No"]
wilcox.test(LA_DFW,NonLA_DFW,alternative = "less",paired = T)

plot(density(Instruct_LA),xlim=c(0,.7),main = paste("Distribution of DFW Rates for Instructors \n that Taught Sections with LA's or without LA's"),xlab = "DFW Rate")
lines(density(Instruct_No_LA),col="red")
legend(x=.5,y=2.5,col = c("Black", "Red"),legend=c("No","Yes"),lwd=1)
```

    ## 
    ##  Wilcoxon signed rank test with continuity correction
    ## 
    ## data:  LA_DFW and NonLA_DFW
    ## V = 36, p-value = 0.6225
    ## alternative hypothesis: true location shift is less than 0
    
![](/Users/blakegilliland/Documents/GitHub/blakegilliland.github.io/images/LA_Paper_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

![](/Users/blakegilliland/Documents/GitHub/blakegilliland.github.io/images/LA-analysis_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

    ##     Group.1     Group.2          x
    ## 1  BSC1010C   Fall 2016 0.35571429
    ## 2  CHM1045    Fall 2016 0.35500000
    ## 3  COP2006    Fall 2016 0.06500000
    ## 4  MAC1105    Fall 2016 0.28812500
    ## 5  MAT1033    Fall 2016 0.20882353
    ## 6  MGF1106    Fall 2016 0.10666667
    ## 7  STA2023    Fall 2016 0.23357143
    ## 8  CHM1045    Fall 2017 0.33571429
    ## 9  COP1500    Fall 2017 0.25000000
    ## 10 ESC1000C   Fall 2017 0.18000000
    ## 11 MAC1105    Fall 2017 0.29866667
    ## 12 MAC1147    Fall 2017 0.26454545
    ## 13 MAC2311    Fall 2017 0.33500000
    ## 14 MAT1033    Fall 2017 0.24529412
    ## 15 CHM1045    Fall 2018 0.41000000
    ## 16 CHM1046    Fall 2018 0.43333333
    ## 17 COP1500    Fall 2018 0.15000000
    ## 18 COP2006    Fall 2018 0.17000000
    ## 19 EVR1001C   Fall 2018 0.08666667
    ## 20 MAC2312    Fall 2018 0.23200000
    ## 21 STA2122    Fall 2018 0.17666667
    ## 22 BSC1010C Spring 2017 0.34375000
    ## 23 CHM1046  Spring 2017 0.34000000
    ## 24 COP1500  Spring 2017 0.19000000
    ## 25 COP2006  Spring 2017 0.07000000
    ## 26 MAC1105  Spring 2017 0.27125000
    ## 27 MAC1147  Spring 2017 0.22000000
    ## 28 MAC2233  Spring 2017 0.38833333
    ## 29 PHY2054C Spring 2017 0.16000000
    ## 30 CHM1045  Spring 2018 0.40200000
    ## 31 CHM1046  Spring 2018 0.31000000
    ## 32 COP1500  Spring 2018 0.19000000
    ## 33 COP2006  Spring 2018 0.17000000
    ## 34 EGM3420C Spring 2018 0.22666667
    ## 35 ESC1000C Spring 2018 0.12500000
    ## 36 MAC1147  Spring 2018 0.23600000
    ## 37 MAC2311  Spring 2018 0.30428571
    ## 38 MGF1106  Spring 2018 0.17666667
    ## 39 STA2023  Spring 2018 0.27230769
