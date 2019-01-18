---
title: "Exercise - Aroma-profiling in Cheese"
output:
  html_document:
    number_sections: true
    toc: TRUE
    keep_md: true
editor_options: 
  chunk_output_type: console
---




Cheese has been produced with, and without maturation culture to accelerate/differentiate maturation.
Aroma profiling is obtained from cheeses of age _4_, _7_ and _10_ weeks. 


# Set libraries and Import data

It is a good idea to start the script with setting the libraries you are going to use and then import data


```r
library(rio)
library(ggplot2)
X <- import('data/cheese_aromas.xlsx')
```

# Summary statistics

Use table() to figure out how many samples there are in the combination of time and maturation culture

Produce two sets of summary stats: 
First, descriptive stats for one variable for each design cell and 
Second, the mean value for all aroma compounds across all design cells. 

Use aggregate() for both. Here are some inspiration


```r
aggregate(X$`1Butanol`, list(X$ti...,X$...),function(x) c(length(x), mean(x),...))
```


```r
Xm <- aggregate(X, list(X$...),...)
```





# Plot of data

## Boxplot with points on top

We wish to make some plots of the raw data emphasizing the design. So make a plot using ggplot() (from the ggplot2 package) to plot the design on the x-axis and some aroma comound on the y-axis, with a boxplot in the background and the individual observations as points. 

Should look something like this

![](Exercises_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

and here are some code for inspiration


```r
ggplot(data = X, aes(time_culture,...)) + 
  geom_boxplot() + ...
```

## Scatter plot of two aroma compounds

Try to make the following, and see what is going on.


```r
ggplot(data = X, aes(`1Butanol`, `1Propanol`, 
                     color = factor(time_weeks), 
                     shape = maturation_culture)) + 
  geom_point() + 
  stat_ellipse() 
```

* Try to make a scatter plot as above, but now add a (_straight_) line through the points by adding stat_smooth(). 
If you only want one line, then the coloring and shapes should be removed. 

* __Difficult__: can you figure out if it is possible to have a plot with colors, shapes and ellipses BUT only one straight line? 

## Lineplot with errorbars

Instead of plotting the raw data, lets plot the mean value and put on some error-bars. Still with the x-axis and y-axis being the same. 

We need a bit tricky version of aggregate() to make it work. And furhter some renaming.


```r
Xag <- do.call(data.frame,
               aggregate(X$`1Butanol`, 
                         list(X$time_weeks,X$maturation_culture), 
                         function(x) c(mean(x), sd(x)))
)

colnames(Xag) <- c('time','culture','mn','sd')

ggplot(data = Xag, aes(time,mn, color = culture, ymin = mn - sd, ymax = mn+sd)) + 
  geom_point() + 
  geom_errorbar() + 
  geom_line()
```

![](Exercises_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

# T-test
In the figure above it is not clear if week 7 and 10 for samples with added culture are significantly different. To find out we can perform a perform a T-test. T-tests can be done by using the function _t.test_ - try to use the commands ?t.test or help(t.test) for help. Try to make a t-test on the difference in 1-butanol between week 7 and 10 for samples with culture.

You could consider to subset the original data.


```r
Xtemp <- subset(X,...)
w7 <- subset(Xtemp,...)
w10 <- ...

t.test(w7$`1Butanol`...)
```

Or in a more comprehensive way, simply _only_ include the data which you want to use. 


```r
t.test(data = X[X$time_weeks %in% c(7,10) & X$maturation_culture=='culture',], 
       `1Butanol` ~ time_weeks)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  1Butanol by time_weeks
## t = 2.177, df = 8.5567, p-value = 0.05898
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -0.003492978  0.150944876
## sample estimates:
##  mean in group 7 mean in group 10 
##         7.391145         7.317419
```

In any case you should get exactly the same answer. 



# Linear models and ANOVA
In the above example there were three time points and not only the two used for the t.test. If we want to analyse the effect of time we should to an analysis of variance ANOVA. This can be done by making a linear model with lm(y~x). This model can yield the ANOVA table by being passed to the anova function.

* Try to do a ANOVA on the culture samples with all time points (should time points be continuous or discrete?)
    + subset(X...)
    + lm(1-butanol~time)
    + anova(mdl)

* Do the model assumptions hold? (Normally distributed residuals, try plot(mdl))
* Extract the coefficients and compare them to the line plot above. 



This could also be analysed as a two-way anova as there are in fact two design (experimental) factors!



# PCA

Here we want to make a PCA model on the multivariate data ( _22 GC features_ ). 
That is a low-rank, or _simple_ representation of what goes on in all 22 directions in (here) two dimentions. 

The PCA model it self can be calculated using _build-in_ functionality in R, but the plotting requires some additional packages. 

## Install the ggbiplot package

There are several places, where packages are made available for download and use. One of the is _github_. 
The package that we are going to use is appearing here. One really nice thing in R, is that the download and installation of packages is done directly from R. So no need to use a browser, point-and-click. However, you will need another package for getting the package. Namely the _devtools_ package. 



```r
install.packages('devtools')
```

Then using this library we can get ggbiplot


```r
library(devtools)
install_github('vqv/ggbiplot')
```

## Make the PCA model

Use the prcomp() function to make a PCA model on _the response variables_ DO NOT include design!
Further, remember to pre-treat the input data using centering and scaling. 


```r
PCAmdl <- prcomp(X[,4:...], scale. = T, center = T)
```




## Plot the model

A vanilla version plot of the PCA model looks as follows: 


```r
library(ggbiplot)
ggbiplot(PCAmdl)
```

![](Exercises_files/figure-html/unnamed-chunk-19-1.png)<!-- -->

As this function is engined by ggplot() most of the modification that can be used by ggplot() can also be used here. Specifically, you might want to extend the axes in order to be able to see the labels on the loadings. That is done by adding xlim(c(low_limit,high_limit)) to the plot (similarly for ylim()). Try to do this. 

The ggbiplot() function can included classes for the scores (that is if the samples are from different groups). In this case we a design, so try to incorporate this via the groups =  argument in the function. Further, set the argument ellipse = TRUE to get some even nicer representation.

![](Exercises_files/figure-html/unnamed-chunk-20-1.png)<!-- -->

## Costumize the plot from scratch. 

Although ggbiplot() fullfills most needs in terms of PCA vizualisation, you might want to make the plot even specific. In that case, the way to go, is to 1) export the scores or loadings and put them into a data.frame and 2) use ggplot() to make exactly the version you want. 

### On Scores

As an example of something which might could be useful, is three individual score plots; one for each timepoint. 

In order to make that we do: 


```r
scores <- data.frame(X, PCAmdl$x) # extract scores and put into data.frame combined with design (and original data)
head(scores,2)
```

```
##   maturation_culture time_weeks   time_culture X1Butanol X1Propanol
## 1            culture          4 04uger_culture  7.542316   4.108593
## 2            culture          4 04uger_culture  7.492835   4.142102
##   X2Butanol Isopropyl_Alcohol X1Pentanol X2Pentanol  Acetone  Butanal
## 1  7.025338          6.479134   6.602876   4.872258 6.689937 4.867165
## 2  6.951893          6.625942   6.578138   4.792159 6.994308 4.957335
##   Butanal_2methyl Butanal_3methyl Propanal_2methyl X2Heptanone
## 1        6.572955        7.125086         6.394694    7.078801
## 2        6.870942        7.375913         6.678770    7.091987
##   Butanoic_acid_3methyl Butanoic_acid Acetic_acid Pentanoic_acid
## 1              3.490007      2.826384    5.223277       1.725756
## 2              3.169435      2.123829    4.905353       1.546119
##   Formic_acid_butyl_ester Propanoic_acid_2hydroxy_ethyl_ester
## 1                5.922506                            1.886282
## 2                5.915372                            1.762319
##   Acetic_acid_ethenyl_ester Pentanal X2Butanone   Nonane       PC1
## 1                  7.008109 6.839849   8.404732 5.446106 -3.199387
## 2                  6.839216 6.920548   8.410900 5.771083 -2.209928
##          PC2      PC3       PC4        PC5       PC6         PC7       PC8
## 1 -0.1222409 2.193389 -1.689694 -0.2408070 0.6711684 -0.01894841 0.7477768
## 2  0.2324992 3.124559 -2.220143 -0.5838099 0.7608341  0.61886464 0.5762888
##           PC9         PC10        PC11       PC12       PC13        PC14
## 1 -0.01172872 -0.005058009 -0.08848177 0.06731906 0.04453961  0.07121666
## 2  0.72343630 -0.742602661  0.19935638 0.05851943 0.38476215 -0.37115335
##            PC15       PC16        PC17        PC18         PC19
## 1 -0.0464973288 0.09346433 -0.02116017 -0.04722489 -0.008143801
## 2  0.0006669017 0.25174015 -0.07465636  0.06286470 -0.029978391
##          PC20        PC21        PC22
## 1 -0.03668081 -0.07363705  0.04123002
## 2 -0.03161958  0.07170486 -0.02150698
```

You see that in addition to the original data, coloumns with PC1, PC2,..., PC22 is added. 

Lets plot them:


```r
ggplot(data = scores,aes(PC1,PC2, color = maturation_culture)) + 
  geom_point() + 
  geom_hline(yintercept = 0) + 
  geom_vline(xintercept = 0)
```

![](Exercises_files/figure-html/unnamed-chunk-22-1.png)<!-- -->

Now try to make modifications to the code to get something like this. 

HINT: you will need to play around with facet_wrap() or facet_grid(), stat_ellipse(), theme_bw() and some general settings in theme(). 

![](Exercises_files/figure-html/unnamed-chunk-23-1.png)<!-- -->


### On Loadings

Here, we only have _22_ variables, but we might have situations where there are way more. In such situations it would be nice to make the plot intepretable without having to scrutinize to much. One such way, is to add information on the points. Here in the form of classes: 

These variables are aroma compounds and can be subdivided into the chemical categories: 

* Alchohols
* Aldehydes 
* Ketones 
* Acids

We would like to infer this information in different ways, but first of all we need to set up the loadings in a data.frame() and define class from the name of the variable. 


```r
loadings <- data.frame(PCAmdl$rotation)
loadings$name <- rownames(loadings)

loadings$class <- 'No Class'
# Alchohols
loadings$class[grepl('ol',loadings$name)] <- 'Alcohols'
# Aldehydes 
loadings$class[grepl('al',loadings$name)] <- 'Aldehydes'
# Ketones 
loadings$class[grepl('one',loadings$name)] <- 'Ketones'
# Acids
loadings$class[grepl('acid',loadings$name)] <- 'Acids'

table(loadings$class)
```

```
## 
##     Acids  Alcohols Aldehydes   Ketones  No Class 
##         7         6         5         3         1
```

```r
loadings$name[loadings$class=='No Class'] 
```

```
## [1] "Nonane"
```

Lets plot the loadings, color- and shape according to molecular class and put on labels. 


```r
ggplot(data = loadings, aes(PC1, PC2, color = class, label = name, shape = class)) + 
  geom_point(size = 10) + 
  geom_text() + 
  geom_hline(yintercept = 0) + 
  geom_vline(xintercept = 0)  
```

![](Exercises_files/figure-html/unnamed-chunk-25-1.png)<!-- -->

You are pretty happy with this plot, but not totally satisfied!

Try to play around with __label size__, __coloring__, __theme__ and __label position__ to get things more clear. 

Something like below is doable but how? 

HINT: For the displacement of the labels you'll need a package called ggrepel. 

A HARD one: The de-selection of some labels is a pretty hard task, but try to google and see if you can figure this one out!

![](Exercises_files/figure-html/unnamed-chunk-26-1.png)<!-- -->




