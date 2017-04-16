---
title: "Survey Weights in R"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
This example will demonstrate how to create a weighted dataset after a survey has been administered.  By a weighted dataset, we mean a dataset that may have some nonresponse for certain demographics therefore may not be representative of the population of interest.  Therefore, when population statistics are available, we can use them to adjust our data to match those of the population.

Because the purpose of this example, is to explain the methods and code in R for weighting survey data, we developed a fake dataset from which we will conduct the analysis.  Here is a link to another example describing the data set: http://rpubs.com/mhanauer/268260  
```{r}
set.seed(12345)
preYear = c(0:100)
preYear = sample(preYear, 100, replace = TRUE)

income = c(0:100000)
income = sample(income, 100, replace = TRUE)

gender = c("Male", "Female")
gender = sample(gender, 100, replace = TRUE)
gender = as.numeric(factor(gender))

ethnicity = c("White", "African_American", "Mixed_Ethnicity", "Other_Ethnicity")
ethnicity = sample(ethnicity, 100, replace = TRUE)
ethnicity = as.numeric(factor(ethnicity))

postYear = preYear + 10

data = cbind(preYear, income, gender, ethnicity, postYear)
data = as.data.frame(data)
```
Here we create the survey data set that has no weights and assumes a simple random sample (ids =~ 1).  
```{r}
library(survey)
data.svy.unweighted <- svydesign(ids=~1, data=data)
```
Next, we have to get the marginal probabilities for the variables that we want to weight the data by. In this example, we are assuming the population values for female (1) and male (2) are .45 and .55.
```{r}
gender.dist <- data.frame(gender = c("1", "2"),
                       Freq = nrow(data) * c(0.45, 0.55))
```
Here we use the rake function in the survey package to weight the current data by the population values for each of the variables included in the dataset.
```{r}
data.svy.rake <- rake(design = data.svy.unweighted,
                   sample.margins = list(~gender),
                   population.margins = list(gender.dist))
```
Finally, because the weights can become either too large or too small, we can put limits on the weights using the trimWeights function.  In this example, we limit the weights to .3 to 3 (i.e. 3 means a value being counted three times as much as its original value).  

Then we can get the weighted means by using the svymean function.  We can then compare these means with the original means to evaluate the changes the weights played. 
```{r}
data.svy.rake.trim <- trimWeights(data.svy.rake, lower=0.3, upper=3,
                                  strict=TRUE)

svymean(data, data.svy.rake.trim)

mean(data$postYear)
mean(data$income)
```
