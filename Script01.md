Activity assay of 3D HPLC fractions RNAseAlert assay for James
================
Jan Sklenar
22/5/2021

## Goals

Compare the activity profiles of purified aphid extract.Fractions are
from 3D LC fractionation.

## Details

-   
-   
-   

<!-- -->

    ## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --

    ## v ggplot2 3.3.3     v purrr   0.3.4
    ## v tibble  3.1.2     v dplyr   1.0.6
    ## v tidyr   1.1.3     v stringr 1.4.0
    ## v readr   1.4.0     v forcats 0.5.1

    ## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

## Data preparation

### Read data into R

``` r
## read in raw data and some annotation
setwd("C:/0_R/210522_James_activity/")
raw <- read.csv("data/summary_18-05-21_edit_ACTIVITY.csv", skip = 7, stringsAsFactors = FALSE)
str(raw)
colnames(raw)
dim(raw)
head(raw)
tail(raw)
# get rid of NAs
#raw <- raw[complete.cases(raw),]
dim(raw)
```

### Rename columns

``` r
#mynames <- c("Reading","MeasTimeSec",
#             paste(rep("reaction_1",4),"-",seq(1:4)),
#             paste(rep("reaction_2",4),"-",seq(1:4)),
#             paste(rep("reaction_3",4),"-",seq(1:4)),
#             paste(rep("reaction_4",4),"-",seq(1:4)),
#             paste(rep("reaction_5",4),"-",seq(1:4)),
#             paste(rep("reaction_6",4),"-",seq(1:4)))
#mynames
#names(raw) <- mynames

colnames(raw)
# data I need
S <- colnames(raw)[grep("S\\d", colnames(raw))]
min <- colnames(raw)[grep("min", colnames(raw))]
mydata=c(min,S)
mydata

df <- raw[,mydata]
dim(df)
dim(df[complete.cases(df),])
```

### Common annotations to be used in plots

Sample names and conditions for plot legends.

``` r
title  <- "3D-LC fractions"
mysubtitle <- "May 2021"
name <- "replicates:"
IDs <- as.character(seq(1:4))
mycolours <- c("black","red","blue","green")#,"lightblue2","lightblue3")
mylabels <-  c("one",
             "two",
             "three",
             "four")
mylabels <- paste(IDs, mylabels, sep=": ")
names(mylabels) <- IDs
```

<!---
### Calculations of RNA concentration
-->

### Conversion to a long data format

``` r
df_long <- gather(df, key = "Sample", value = "RFU", -"min")
head(df_long)
table(is.na(df_long))
#df[is.na(df),]
```

### Parse the sample names to create variables

``` r
#' remove columns if necessary
colnames(df_long)

#' parse 'Sample' to obtain sample number 'SampleNo'
df_long$Sample[1]

df_long$SampleNo <- substr(df_long$Sample,2,regexpr('_', df_long$Sample)-1)

df_long$RepNo <- substr(df_long$Sample,regexpr('_', df_long$Sample)+1,regexpr('F', df_long$Sample)-2)

df_long$FracNo <- substr(df_long$Sample,regexpr('F', df_long$Sample), length(df_long$Sample))

head(df_long)

#' in the same way extract replicate number - 'RepNo'
#df$RepNo <- substr(df$Sample,regexpr(' - ', df$Sample)+3, regexpr('$', df$Sample))
#head(df$RepNo)

# #' Change sec to min
# table(is.na(df$MeasTimeSec))
# df$MeasTimeMin <- df$MeasTimeSec/60
# head(df$MeasTimeMin)
# #' remove white space from 'Well' sting
# #df$Well<- substr(df$Well, 2, regexpr('$', df$Well))

#' convert SampleNo to factor in order it to sort nicely in ggplots
df_long$SampleNo <- as.factor(df_long$SampleNo)
df_long$SampleNo <- factor(df_long$SampleNo, levels=IDs) 
unique(df_long$SampleNo)
head(df_long)
```

### Subset of data for plotting if necessary

``` r
# # Selected (cleaned) data contains only columns needed for plotting
# # Well, Sample, SampleNo, RepNo, MeasTimeSec, RFU
# colnames(df)
# df <- df[,c(1,2,3,4,5,6,7)] 
# head(df) 
# # there should be no missing values
# table(is.na(df))
# df[!complete.cases(df),]
```

## Plots

### All the data

``` r
# all the data
head(df_long)
#plot(x=df_long$min,y=df$RFU)

# colour by samples
ggplot(data = df_long) +
  aes(x = min, y=RFU) +
  geom_point(aes(color=SampleNo)) +
  scale_color_discrete(name=name, breaks=IDs, labels=mylabels)
```

![](Script01_files/figure-gfm/plots-1.png)<!-- -->

``` r
ggplot(data = df_long) + 
  aes(x = min, y=RFU) +
  geom_smooth(aes(color=SampleNo), method = 'gam',se=T, level=0.95) + 
  geom_point(aes(color=RepNo), alpha=1/50) +
  facet_wrap(~FracNo, scales = "fixed")
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/plots-2.png)<!-- -->

``` r
ggplot(data = df_long) + 
  aes(x = min, y=RFU) +
  geom_smooth(aes(color=SampleNo), method = 'gam',se=T, level=0.95) + scale_color_manual(values = mycolours) + 
  geom_point(aes(fill=RepNo), alpha=1/100) + 
  #scale_color_manual(values = mycolours) + 
  scale_fill_manual(values = mycolours) + 
  facet_grid(SampleNo~FracNo, scales = "fixed")
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/plots-3.png)<!-- -->

### Data adjustments if necessary

Removal of bad replicates and outliers

``` r
## Adjustments - remove outliers if necessary

## Sometimes samples needs to be combined...
#  df$SampleComb <- df$SampleNo 
#  df[df$SampleComb==2,"SampleComb"] <- 1

## Measurement of this sample have many outliers...
#  ggplot(subset(df, SampleNo %in% c(7))) +
#  aes(x = MeasTimeSec, y=RFU) +
#  geom_point(aes(color=SampleNo))

## Use summary statistics (mean) to find and remove obvious outliers...

# #  means of all replicates 
# dfstat <- df %>% filter(SampleNo==1|SampleNo==2|SampleNo==3|SampleNo==4|SampleNo==5|SampleNo==6) %>% group_by(SampleNo,RepNo) %>% summarise(mean=mean(RFU))
# print(dfstat,n=nrow(dfstat))
# #View(dfstat)



# Data that should be removed
#dim(df) 
#dim(df %>% filter(RepNo==1))
#dim(df %>% filter(RepNo!=1))
#df <- df %>% filter(RepNo!=1)

## To invert filter...
#  filter(df,SampleNo == 7 & RepNo == 3)
#  It is the same as leaving out all the measurement from some wells
#  df <- filter(df, Well != "A01" & Well != "B01" & Well != "C01"&  Well != "D01"& Well != "E01"& Well != "F01" & Well != "G01"& Well != "H01")

## Filter can be also inverted using... 
#  df <- filter(df, Well != "G03")

## To remove some data...
#  There are some zeros (negative spikes) in Sample "7 1/3" 
#  Let's removethose  zeros here
#  df[which(df$Sample=="7 1/3"&df$RFU==0),]
#  dfs[which(dfs$Sample=="7 1/3"&dfs$RFU==0),] <- NA

## Checks...
head(df_long)
length(rownames(df_long))
length(complete.cases(df_long))
table(is.na(df_long))
## only complete cases
df[which(is.na(df_long)),]
df_long_clean <- df_long[complete.cases(df_long),]
## check for NAs
dim(df_long)
dim(df_long_clean)
table(is.na(df_long))
table(is.na(df_long_clean))
```

### Panels of plots

``` r
## LOESS regression,

p <- ggplot(data = df_long) + 
  aes(x = min, y=RFU) +
  geom_smooth(aes(color=SampleNo), method = 'gam',se=T, level=0.95) #+geom_point(aes(colour=RepNo))
p
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
## Panel of plots fixed scale
p+facet_wrap(~FracNo, scales = "free_y") + #coord_cartesian(xlim = c(0,500)) +
  labs(title = title, x = "min", y = "RFU") +
  scale_colour_discrete(name=name, breaks=IDs, labels=mylabels)
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-4-2.png)<!-- -->

``` r
## Panel of plots of Fractions - zoomed
p+facet_wrap(~FracNo, scales = "fixed") +
  coord_cartesian(xlim = c(0,600)) +
  labs(title = title, x = "min", y = "RFU") +
  scale_colour_discrete(name=name, breaks=IDs, labels=mylabels)
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-4-3.png)<!-- -->

``` r
## Panel of plots of Fractions - zoomed
# adding the actual points
p + geom_point(aes(colour=SampleNo), size = 1, alpha=1/10) + facet_wrap(~FracNo, scales = "fixed") +  coord_cartesian(xlim = c(0,600)) +
  labs(title = title, x = "min", y = "RFU") +
  scale_colour_discrete(name=name, breaks=IDs, labels=mylabels)
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-4-4.png)<!-- -->

``` r
## Grid of plots of Samples~Fractions - zoomed
p + facet_grid(SampleNo~FracNo, scales = "fixed") +
  coord_cartesian(xlim = c(0,600)) +
  labs(title = title, x = "min", y = "RFU") +
  scale_colour_discrete(name=name, breaks=IDs, labels=mylabels) +
theme(axis.text.x = element_text(angle=90))
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-4-5.png)<!-- -->

``` r
## With data points
q <- ggplot(data = df_long) + #[df_long$FracNo=="F25",]) + 
  aes(x = min, y=RFU) +
  geom_smooth(aes(color=FracNo), method = "gam") + geom_point(aes(colour=SampleNo), alpha=1/30) +
  theme(axis.text.x = element_text(angle=90))
q
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-4-6.png)<!-- -->

``` r
## Panel of plots of Fractions - zoomed, samples combined
q+facet_wrap(~FracNo, scales = "fixed") +
  coord_cartesian(xlim = c(0,600)) +
  labs(title = title, x = "min", y = "RFU")
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-4-7.png)<!-- -->

``` r
q+facet_grid(SampleNo~FracNo, scales = "fixed") +
  coord_cartesian(xlim = c(0,600)) +
  labs(title = title, x = "min", y = "RFU") + 
  theme(axis.text.x = element_text(angle=90))
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-4-8.png)<!-- -->

``` r
## Sums for comparision

ggplot(data = df_long) +aes(x=FracNo, y=RFU)+ geom_col(aes(color=SampleNo))
```

![](Script01_files/figure-gfm/unnamed-chunk-4-9.png)<!-- --> \#\#\#
Plotting activity as measured provides fine details


    ### Observation: 

    - The replicates are not exactly the same as for gene expression profile, despite the signal magnitude is reproducible. For that reason the most of the data do not fall into 95% the confidence intervals. 

     - Summing up averages hides the differences in the data - it is a general problem that many small values can hide a  peak! For that reason, it seems better to sum the replicates, then take average/smooth with the other replicates.  
    This is an MS technique; when we trust our signal we acquire and sum up enough spectra to suppress the noise. Then we can take several of such measurements for error calculation.

    - It is better to show the peaks in an array of plots than the sums.

    - We can also observe and compare the individual replicates. They are not always exactly the same.


    ```r
    #head(df,10)
    head(df_long,10)
    library("dplyr")
    df_long_sum <- df_long %>% group_by(SampleNo,FracNo,min) %>% summarise(RFU=sum(RFU, na.rm = TRUE))

    ## `summarise()` has grouped output by 'SampleNo', 'FracNo'. You can override using the `.groups` argument.

``` r
## With data points
q <- ggplot(data = df_long_sum) + #[df_long$FracNo=="F25",]) + 
  aes(x = min, y=RFU) +
  geom_smooth(aes(color=FracNo), method = "gam") 

#q

## Panel of plots of Fractions - zoomed, samples combined
q+facet_wrap(~FracNo, scales = "fixed") +
  coord_cartesian(xlim = c(0,600)) +
  labs(title = title, x = "min", y = "RFU")
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
q+facet_grid(SampleNo~FracNo, scales = "fixed") +
  coord_cartesian(xlim = c(0,600)) +
  labs(title = title, x = "min", y = "RFU") +
  theme(axis.text.x = element_text(angle = 90)) 
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-5-2.png)<!-- -->

``` r
q+facet_grid(SampleNo~FracNo, scales = "fixed") +
  coord_cartesian(xlim = c(0,600)) +
  labs(title = title, x = "min", y = "RFU") +
  theme(axis.text.x = element_text(angle = 90)) +
  geom_point(aes(colour=SampleNo), alpha=1/30)
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

![](Script01_files/figure-gfm/unnamed-chunk-5-3.png)<!-- -->

<!-- #  -->
<!-- #  -->
<!-- #  -->
<!-- # ## Example for one plot only -->
<!-- # Copied form: [Regression on the plot](https://intellipaat.com/community/7335/adding-regression-line-equation-and-r2-on-graph) -->
<!-- #  -->
<!-- # ```{r include=FALSE, echo=FALSE, eval=FALSE} -->
<!-- # df <- data.frame(x = c(1:100)) -->
<!-- # df$y <- 2 + 3 * df$x + rnorm(100, sd = 40) -->
<!-- #  -->
<!-- # p <- ggplot(data = df, aes(x = x, y = y)) + -->
<!-- #             geom_smooth(method = "lm", se=FALSE, color="black", formula = y ~ x) + -->
<!-- #             geom_point() -->
<!-- #  -->
<!-- # lm_eq <- function(df){ -->
<!-- #     m <- lm(y ~ x, df); -->
<!-- #     eq <- substitute((y) == a + b %.% (x)*","~~(r)^2~"="~r2,  -->
<!-- #          list(a = format(unname(coef(m)[1]), digits = 2), -->
<!-- #               b = format(unname(coef(m)[2]), digits = 2), -->
<!-- #              r2 = format(summary(m)$r.squared, digits = 3))) -->
<!-- #     as.character(as.expression(eq)); -->
<!-- # } -->
<!-- #  -->
<!-- # p1 <- p + geom_text(x = 25, y = 300, label = lm_eq(df), parse = TRUE) -->
<!-- #  -->
<!-- # p1 -->
<!-- # ``` -->
<!-- # Now we have to do it for all the samples. -->
<!-- #  -->
<!-- # ```{r include=FALSE, echo=FALSE, eval=FALSE} -->
<!-- # # Another idea... printing ggplots subsets using a factor -->
<!-- # library("datasets") -->
<!-- # library("dplyr") -->
<!-- # data(iris) -->
<!-- # summary(iris) -->
<!-- # class(iris) -->
<!-- # p <- iris %>% group_by(Species) %>% do(plots=ggplot(data=.) + -->
<!-- #          aes(x=Petal.Width, y=Petal.Length) +  -->
<!-- #            geom_point() +  -->
<!-- #            ggtitle(unique(.$Species))) -->
<!-- # print(p$plots) -->
<!-- #  -->
<!-- # ``` -->
<!-- #  -->
<!-- #  -->
<!-- # ### Now let's put these solutions together -->
<!-- #  -->
<!-- #  -->
<!-- \ print(p$plots) -->
<!-- # this writes regression equation -->
<!-- lm_eq <- function(df,x,y){ -->
<!--     m <- lm(y ~ x, df); -->
<!--     eq <- substitute((y) == a + b %.% (x)*","~~(r)^2~"="~r2,  -->
<!--          list(a = format(unname(coef(m)[1]), digits = 2), -->
<!--               b = format(unname(coef(m)[2]), digits = 2), -->
<!--              r2 = format(summary(m)$r.squared, digits = 3))) -->
<!--     as.expression(eq); # this works for title/subtitle -->
<!--     # as.character(as.expression(eq)); # this works for geom_text -->
<!-- } -->
<!-- p <- df %>% group_by(SampleNo) %>% do(plots=ggplot(data=.) + -->
<!--          aes(x=MeasTimeMin, y=RFU) +  -->
<!--            geom_point() +  -->
<!--            geom_smooth(method = "lm", se=FALSE, color="black", formula = y ~ x) + -->
<!--            ggtitle(mylabels[unique(.$SampleNo)], subtitle = lm_eq(.,.$MeasTimeMin,.$RFU))) -->
<!--           # geom_text(x = 25, y = 300, label = lm_eq(., .$MeasTimeSec, .$RFU), parse = TRUE)) -->
<!-- # print plots held by the object p -->
<!-- #print(p$plots[1]) -->
<!-- print(p$plots) -->
<!-- r <- df %>% group_by(SampleNo, RepNo) %>% do(plots=ggplot(data=.) + aes(x=MeasTimeMin, y=RFU) +  -->
<!-- geom_point() +  -->
<!-- geom_smooth(method = "lm", se=FALSE, color="black", formula = y ~ x) + -->
<!-- ggtitle(mylabels[unique(.$SampleNo)], subtitle = lm_eq(.,.$MeasTimeMin,.$RFU))) -->
<!-- print(r$plots) -->
<!-- ## All together in a panel  -->
<!-- # replicates together -->
<!-- df %>% group_by(SampleNo) %>% ggplot(data=.) + -->
<!--          aes(x=MeasTimeMin, y=RFU) +  -->
<!--            geom_point() +  -->
<!--            geom_smooth(method = "lm", se=FALSE, color="black", formula = y ~ x) + -->
<!-- #           ggtitle(mylabels[unique(.$SampleNo)], subtitle = lm_eq(.,.$MeasTimeMin,.$RFU)) + -->
<!--    facet_wrap(~SampleNo, scales = "free") -->
<!-- # replicates separately -->
<!-- df %>% group_by(SampleNo, RepNo) %>% ggplot(data=.) + -->
<!--          aes(x=MeasTimeMin, y=RFU) +  -->
<!--            geom_point() +  -->
<!--            geom_smooth(method = "lm", se=FALSE, color="black", formula = y ~ x) + -->
<!-- #           ggtitle(mylabels[unique(.$SampleNo)], subtitle = lm_eq(.,.$MeasTimeMin,.$RFU)) + -->
<!--    facet_grid(SampleNo~RepNo, scales = "free") -->
<!-- ``` -->
<!-- ## Gradients -->
<!-- ### So what next? What is the best quantitative value for comparisons? -->
<!-- While it is nice to see the regression, what values are the best to compare for a robust quantitation? -->
<!-- There are two the most obvious  possibilities: -->
<!-- 1. Sum of all RFU values for each over the whole time range. -->
<!-- 2. Fit a linear regression to the time course and use the gradient for quantitative comparison.   -->
<!-- The former method is the simplest. -->
<!-- The latter method shows all the details of the time course; Cas13a kinetics at fixed substrate concentration. The gradient is definitelly more robust than sum. But we have to check linearity RFU ~ time course.  -->
<!-- Moreover, if the assay was well designed, and mesurement is taken above [S]>>Km, close to Vmax, perhaps we could calculate the tangent to plot dP/dt vs t like in: Claro, Enrique. (2000). Understanding initial velocity after derivatives of progress curves. Biochemistry and Molecular Biology Education. 28. 304-306? [link](https://www.sciencedirect.com/science/article/abs/pii/S1470817500000497) -->
<!-- ```{r} -->
<!-- colnames(df) -->
<!-- lin <- df %>% group_by(SampleNo) %>% do(lm(.$RFU ~ .$MeasTimeMin,data=.) %>% coef() %>% as_tibble()) -->
<!-- library("tidyr") -->
<!-- lin$SampleNo <- as.numeric(factor(lin$SampleNo)) -->
<!-- lin -->
<!-- lin$lin_coef <- rep(c("a","b"),times=length(lin$SampleNo)/2) -->
<!-- lin -->
<!-- lin_wide <- lin %>% pivot_wider(., names_from=lin_coef, values_from=value)  -->
<!-- lin_wide -->
<!-- # Why are some x-axis labels hidden? -->
<!-- # How to get x-axis labels replaced by someting more sensible - "mylabels" -->
<!-- # see [link](https://www.tutorialspoint.com/why-scale-fill-manual-does-not-fill-the-bar-with-colors-created-by-using-ggplot2-in-r) -->
<!-- # fill needs a factor -->
<!-- ggplot(lin_wide, aes(x=SampleNo, y=b, fill=as.factor(SampleNo))) +  -->
<!--   geom_bar(stat = "identity") + -->
<!--   scale_fill_manual(name=name,labels=mylabels, values = mycolours) + -->
<!--   labs(title = "The gradient of linear fit of RFU ~ time course", subtitle=mysubtitle) -->
<!-- # The same plot coded in another way (base R)  -->
<!-- op <- par(mar = c(8,4,4,2) + 0.1) ## default is c(5,4,4,2) + 0.1 -->
<!-- op -->
<!-- barplot(lin_wide$b, -->
<!-- main = "The gradient of linear fit of RFU ~ time course", -->
<!-- sub = mysubtitle, -->
<!-- xlab = "Reactions", -->
<!-- ylab = "Gradient", -->
<!-- #names.arg = seq(1:6), las=1, -->
<!-- names.arg = mylabels, las=2, -->
<!-- col = mycolours, -->
<!-- horiz = FALSE) -->
<!-- box() -->
<!-- legend("topright",                                    # Add legend to barplot -->
<!--        legend = mylabels, -->
<!--        fill = mycolours) -->
<!-- ``` -->
<!-- ### When we take replicates individualy... -->
<!-- ```{r} -->
<!-- colnames(df) -->
<!-- lin <- df %>% group_by(SampleNo, RepNo) %>% do(lm(.$RFU ~ .$MeasTimeMin,data=.) %>% coef() %>% as_tibble()) -->
<!-- library("tidyr") -->
<!-- lin$RepNo <- as.numeric(factor(lin$RepNo)) -->
<!-- lin -->
<!-- lin$lin_coef <- rep(c("a","b"),times=length(lin$RepNo)/2) -->
<!-- lin -->
<!-- lin_wide <- lin %>% pivot_wider(., names_from=lin_coef, values_from=value)  -->
<!-- lin_wide -->
<!-- # Why are some x-axis labels hidden? -->
<!-- # How to get x-axis labels replaced by someting more sensible - "mylabels" -->
<!-- # see [link](https://www.tutorialspoint.com/why-scale-fill-manual-does-not-fill-the-bar-with-colors-created-by-using-ggplot2-in-r) -->
<!-- # fill needs a factor -->
<!-- # plot replicate gradient -->
<!-- ggplot(lin_wide, aes(x=SampleNo, y=b)) +  -->
<!--   geom_bar(aes(fill=as.factor(RepNo)), stat = "identity", position = "dodge") + -->
<!--   scale_fill_manual(name=name,labels=mylabels, values = mycolours) + -->
<!-- labs(title = "The gradient of linear fit of RFU ~ time course", subtitle=mysubtitle) -->
<!-- # boxplot is the best -->
<!-- ggplot(lin_wide, aes(x=SampleNo, y=b)) +  -->
<!--   geom_boxplot(aes(fill=SampleNo), outlier.shape = NA) + -->
<!--   geom_jitter(aes(alpha=0.1)) + -->
<!--   scale_fill_manual(name=name,labels=mylabels, values = mycolours) + -->
<!--   labs(title = "The gradient of linear fit of RFU ~ time course", subtitle=mysubtitle) -->
<!-- # And for comparison,  -->
<!-- # what we get when we sum all the data withing replicates... -->
<!-- ggplot(df, aes(x=SampleNo, y=RFU)) + -->
<!--   geom_bar(aes(fill=RepNo), stat = "identity", position = "dodge") + -->
<!--   scale_fill_manual(name=name,labels=mylabels, values = mycolours) + -->
<!--     labs(title = "The sum of all data in RFU ~ time course", -->
<!--          subtitle = mysubtitle) -->
<!-- # box plot of sums -->
<!-- s <- df%>% group_by(SampleNo, RepNo) %>% summarise(SmplSum=sum(RFU)) -->
<!-- ggplot(s, aes(x=SampleNo, y=SmplSum)) +  -->
<!--   geom_boxplot(aes(fill=SampleNo), outlier.shape = NA) + -->
<!--   geom_jitter(aes(alpha=0.1)) + -->
<!--   scale_fill_manual(name=name,labels=mylabels, values = mycolours) + -->
<!--   labs(title = "The sum of all data in RFU ~ time course", subtitle=mysubtitle) -->
<!-- ## What is the answer to what metrics is better, sum or gradient? THere is an obvious oulier in the positive control, that is clearly visible only when we calculate the regressions. But the linearity requirement is clearly more demanding on the data analysis.   -->
<!-- ## The total sum of RFU values shows quite a different picture. It woudl help to analyze more data to see what is correct and practical. -->
<!-- ``` -->
<!-- ```{r include=FALSE, echo=FALSE, eval=FALSE} -->
<!-- ## M-M kinetics is not relevant to this measurement -->
<!-- library("nlstools") -->
<!-- data(vmkm) -->
<!-- nls1 <- nls(michaelis,vmkm,list(Km=1,Vmax=1)) -->
<!-- plotfit(nls1, smooth = TRUE) -->
<!-- ``` -->
<!-- ### Plots of subsets of samples and conditions to compare -->
<!--  Not used this time -->
<!-- ```{r subsets, include=FALSE, echo=FALSE, eval=FALSE} -->
<!-- ## Crude Orf1ab -->
<!-- p1 <- df[df$SampleNo==1|df$SampleNo==2|df$SampleNo==3|df$SampleNo==4,] -->
<!-- ## Crude S -->
<!-- p2 <- df[df$SampleNo==5|df$SampleNo==6,] -->
<!-- ## StrepTrap Orf1ab -->
<!-- #p3 <- df[df$SampleNo==7|df$SampleNo==8,] -->
<!-- ## StrepTrap S -->
<!-- #p4 <- df[df$SampleNo==9|df$SampleNo==10,] -->
<!-- ## CIEX Orf1ab -->
<!-- #p5 <- df[df$SampleNo==11|df$SampleNo==12,] -->
<!-- ## CIEX S -->
<!-- #p6 <- df[df$SampleNo==13|df$SampleNo==14,] -->
<!-- ## substrate and air -->
<!-- #p7 <- df[df$SampleoN==15|df$SampleNo=="air",] -->
<!-- myplots <- list("p1"=p1,"p2"=p2)#,"p3"=p3) -->
<!-- class(myplots) -->
<!-- str(myplots) -->
<!-- ## LOESS regression -->
<!-- for(p in myplots){ -->
<!-- p <- as.data.frame(p) -->
<!-- #  -->
<!-- # print(ggplot(data = p) + -->
<!-- #   aes(x = MeasTimeMin, y=RFU) + -->
<!-- #   geom_smooth(aes(color=SampleNo)) + -->
<!-- #   geom_point(aes(colour=SampleNo)) + -->
<!-- #   labs(title = title, x = "sec",   y = "RFU", color = "Legend Title\n") + -->
<!-- #   #coord_cartesian(xlim = c(0,1000)) + -->
<!-- #   scale_colour_discrete(name=name, breaks=IDs, labels=mylabels)) -->
<!-- # } -->
<!-- # ``` -->
<!-- #  -->
<!-- #  -->
<!-- # ### Notes -->
<!-- #  -->
<!-- # - Confidence intervals when all data points are treated independently are not very good -->
<!-- # - Indeed, correlation coefficients evidence that  -->
<!-- # - It should help to  calculate the gradient of each replicate individually, then calculate summary stats from all the replicates. It is a sensitive method to spot an odd sample, e.g there is an outlier in the positive control -->
<!-- # - It is also possible to use the sum of all RFU values for a replicate, though the numbers are quite different. I think we need more data to be sure what is the best. -->
<!-- # -  -->
<!-- # -  -->
<!-- #  -->
<!-- # -----------------   -->
<!-- # ----------------- -->
