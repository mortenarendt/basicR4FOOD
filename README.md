# basicR4FOOD
Course materiale for intensive R course at Dept FOOD UPCH 

## Installation of R and Rstudio
Take a look in the file [installation_guide](https://github.com/mortenarendt/basicR4FOOD/blob/master/Installation_guide.pdf)

## Datasets
Datasets for this course is included the dataset folder /data

Coffee temp (Temperature_Coffee.xlsx): A sensorical panel accessing the sensory properties of coffee brewed at different temperatures. 

GC-Cheese (cheese_aromas.xlsx): Aromaprofiling of chesses produces under different conditions over different storage times. 

Mouse (Mouse_diet_intervention.xlsx): Dietary effects of whey, casein as well as high low fat diet on growth and biomarkers in a mice model. 

## Additional material 

### Notes from the _Data Analysis in Food Science_ course
There is a lot of online available material on statistics using R. The [Notes for Data Analysis in Food Science](https://github.com/mortenarendt/basicR4FOOD/blob/master/Notes_FDA.pdf) covers basic data analysis in eight weeks. Data for most of the content there is to be found [here](https://github.com/mortenarendt/DataAnalysisinFoodScience/) and can be downloaded directly in R using the _devtools_ package: 

```{r}
devtools::install_github('mortenarendt/DataAnalysisinFoodScience')
library(DAinFoodScience)
data("maramaox") 
```

### Cheatsheets

RStudio has made available [Cheatsheets](https://www.rstudio.com/resources/cheatsheets/) covering the utility of different packages, such as _ggplot2_. These are very condensed, but also very usefull for checking out what you can do with a particular package. 

### Videos

For the _Data Analysis in Food Science_ course, there is a bunch of videos available going through theory and _How to_ in R. These can be found on youtube [here](https://www.youtube.com/playlist?list=PL4L59zaizb3FmBdxuDLRdzGsknTrZN6Ys). 


## Import data

Download the excel datafiles to an appropriate location on your computer. Further, install the _rio_ package 
(This you only need to do ones). 

```{r}
install.packages('rio')
```
Make a script containing the stuff below, save the file in the same place as the data, andset the working directory to source file location using Session > Set Working Directory. 

```{r}
library(rio)
X <- import('Temperature_Coffee.xlsx')
```

## Summary stats

Lets calculate the mean for the attribute _Intensity_ for each temperature setting

```{r}
aggregate(X$Intensity, list(X$Sample),mean)
```
... Now lets do it for all attributes. 

```{r}
aggregate(X, list(X$Sample),mean)
```

... And lets get some more summary stats returned as well

```{r}
aggregate(X, list(X$Sample),function(x) c(length(x), mean(x), sd(x)))
```
          
## Simple plots with ggplot2

First, import the package _ggplot2_
```{r}
install.packages('ggplot2')
```

Now lets make a plot of the Intensity (on the y-axis) versus the temperature (on the x-axis) and color all the responses according to the Assessor

```{r}
library(ggplot2)
ggplot(data = X, aes(Sample, Intensity, color = factor(Assessor))) + 
  geom_point()
```
Now, lets join the points within each assessor (and replicate), and furhter, plot each replicate in its own panel. 

```{r}
ggplot(data = X, aes(Sample, Intensity, color = factor(Assessor), group = factor(Assessor))) + 
  geom_point() + geom_line() + 
  facet_wrap(~Replicate)
```

In stead of the raw data you might want to see the means of each design cell (temperature and assessor). That we can calculate as described above with aggregate(). and plot it!

```{r}
Xm <- aggregate(X, list(X$Sample, X$Assessor),mean)
colnames(Xm)[1] <- 'Temp'
ggplot(data = Xm, aes(Temp, Intensity, color = factor(Assessor), group = factor(Assessor))) + 
  geom_point() + geom_line()
```

