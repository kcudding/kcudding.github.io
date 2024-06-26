---
title: "DFO training: Logistic Regression"
date: "2024-06-28"
output:
  html_document:
    toc: true
    toc_depth: 3
    number_sections: true
    keep_md: true
    toc_float:
      collapsed: false
      smooth_scroll: false
  pdf_document:
    toc: true
    toc_depth: '3'
  word_document:
    toc: true
    toc_depth: '3'
---



# **Logistic regression**

## Objectives

* Learn main uses of logistic regressions  
* Describe logistic regression assumptions    
* Apply models to your own data using R programming language  
* Interpret model fits and results   

## What is a logistic regression?

* Used to understand association between binary response variable and predictors 
* Example: species distribution models 
    + Binary response variable: presence (1) or absence (0) of species in area
    + Results in predictions of habitat suitability

## How are logistic regressions different from linear regressions?

*Linear regression*  
- Quantitative outcomes are predicted based on value of predictors using straight line  
- We calculate correlation and test for significance of regression  
- We compare different types of model, from more simple, with single predictors, to more complicated, with several predictors and interactions, and then find the ones that provide a best fit to our data  

*Logistic regression*  
- We can also do all of that!  
- Main difference: our outcomes are *binary* as opposed to continuous measurements  
- Examples of binary outcomes: presence versus absence; positive to a disease versus negative; dead versus alive  
- Outcomes can be categorized as 1 (e.g., success) and 0 (e.g., failure)
- In the case of Hamilton Harbour dataset, let's suppose that it becomes hard for zooplankton to eat when there are more "less edible" algae than "edible" algae in the environment (that was the criteria for populating the column "Easy to eat":)


| Nitrate/nitrite|  Edible| Less edible|Easy to eat |
|---------------:|-------:|-----------:|:-----------|
|            2.23|  884.60|      1645.7|yes         |
|            2.50|  900.00|      1133.3|yes         |
|            2.50|  923.20|       939.9|yes         |
|            2.13| 1546.40|      1312.0|no          |
|            2.13|  811.90|       454.1|no          |
|            2.62|  339.16|       308.2|no          |
|            2.67| 1376.80|       552.2|no          |

When we plot this type of binary data, we see that observations are either of the two outcome possibilities.



![](https://raw.github.com/kcudding/kcudding.github.io/main/teach/RIntro/logistic1.png)

This type of data is best fit by an s-shaped curve instead of a line. And this is another difference between linear and logistic regressions.
 
## What does the logistic curve mean?



![](https://raw.github.com/kcudding/kcudding.github.io/main/teach/RIntro/logistic2.png)

* It represents the *probability* of positive outcomes depending on predictors  
* Hover your mouse over the logistic curve: as we move along the curve and our predictor values change, we go from 0 to 100% probability of our outcome  

## Mathematical representation

* Before we can correlate variables in our models: the code we use to run logistic regressions transforms the response variable to get a linear relationship between variables  
* This transformation is called logit (log of probability of success/probability of failure)
* Our explanatory variable is in log(odds)  

This is the equation for the logistic regression:

$$ Log (p/1-p) = b0+b1*x1+e $$
*Log (p/1-p)*: response variable  
*b0*: y intercept  
*x*: predictor variable  
*b1*: slope for explanatory variable 1  
*e*: error  

If we had more explanatory variables, we could keep adding them to the equation above as b2*x2 and so forth.
 
## What kind of predictors can we have in a logistic regression?

Just like in a linear regression, we can use continuous and/or discrete variables to make predictions. Here are some examples:

**Continuous variables**

+ Temperature  

+ Precipitation  

+ Water depth  

+ Nitrogen concentration  

**Discrete variables**  

+ Levels of aquatic vegetation  

+ Soil type 

+ Water body type  

## Key binomial logistic model assumptions  

- Dependent variable has 2 levels or is a proportion of successes  

- Observations are independent  

- Normal distribution of data or residuals is not needed  

## Aquatic ecology studies using logistic regression  

- The distribution of gammarid species was predicted using logistic regressions, where current velocity was the most important factor explaining their distribution [(Peeters & Gardeniers, 1998)](https://onlinelibrary.wiley.com/doi/epdf/10.1046/j.1365-2427.1998.00304.x)  

- Foraging shift behaviour from the benthos to the water surface in brown trout (1 = surface prey consumed, 0 = no surface prey consumed) was predicted using fish length as a predictor in a logistic regression [(Sánchez-Hernández & Cobo, 2017)](https://cdnsciencepub.com/doi/full/10.1139/cjfas-2017-0021])

![[Amphipod Gammaridae](https://commons.wikimedia.org/wiki/File:Amphipod_Gammaridae_%288741971996%29.jpg); [Brown Trout, USFWS Mountain-Prairie](https://www.flickr.com/photos/usfwsmtnprairie/49860328703)](https://raw.github.com/kcudding/kcudding.github.io/main/teach/RIntro/logistic_studies_example.jpg)

## Practice time: run your own logistic model

### Steps to run a logistic model in R

1. Select potentially interesting predictors
2. Format predictors to correspond to binomial levels
3. Select time period and location
4. Run model
5. Interpret model
6. Plot results

### Let's create this first model together

First, we are going to have a look at some potentially interesting variables:


```r
knitr::kable(ham[c(25:30), c(2,8,26,47,98)],row.names = FALSE, digits=2, align=rep('c', 3),
             col.names = c("Location", "Season", "Total phosphorus", "Filamentous diatom biomass", "Epilimnion temperat."))
```



| Location | Season | Total phosphorus | Filamentous diatom biomass | Epilimnion temperat. |
|:--------:|:------:|:----------------:|:--------------------------:|:--------------------:|
|   deep   |   2    |       0.01       |            0.00            |        21.79         |
|   deep   |   2    |       0.01       |            3.10            |        21.83         |
|    NE    |   1    |       0.02       |             NA             |        19.89         |
|   deep   |   1    |       0.02       |             NA             |        19.71         |
|    NE    |   1    |        NA        |             NA             |        13.90         |
|   deep   |   1    |        NA        |           287.18           |        16.05         |

Let's consider that filamentous diatom is our response variable of interest, as this food source is hard for zooplankton to consume. We will look at epilimnion temperature as a potential explanatory variable.

First, let's create a copy of the original dataset so that we can introduce modifications but keep the original data in case we need it.


```r
filam_diatom <- ham
```

Before we can get started with the analysis, we need to remove NA data. We will now do that for the potential response variable filamentous diatom column, and for the potential explanatory variable epilimnion temperature:


```r
filam_diatom <- filam_diatom[!is.na(filam_diatom$filamentous_Diatom), ]
filam_diatom <- filam_diatom[!is.na(filam_diatom$mean_mixing_depth_temp), ]
```


*Practice time*: Now you can do this last step (removing NA data) for total phosphorus ("TP dissolved_ECCC1m"), as we will consider this as another potential explanatory variable later on. 


```r
# Use the original dataset "ham" to subset this data, and give this new  dataset a name, like "filam_diatom_P"

# Don't forget to remove NA data from both the response and the explanatory variables
```



Let's go back to our "filam_diatom" dataset. Now we will create a new column to describe presence or absence of filamentous diatoms.

Ensure the reference group ("absent") is the first to be shown:


```r
filam_diatom$filam_presence <- ifelse(filam_diatom$filamentous_Diatom > 0, "present", "absent")
filam_diatom$filam_presence <- factor(filam_diatom$filam_presence)
levels(filam_diatom$filam_presence)
```

```
## [1] "absent"  "present"
```

Subset to analyse at specific times of the year and at specific locations
Here I selected summer conditions. Ideally we should also subset by station (column "Station_Acronym"), but because there's not enough data to do that, let's subset by depth, by removing records in deep locations:


```r
filam_diatom <- subset(filam_diatom, (season==2 | season==3) & (!area_group=="deep"))
```

Run model using glm function and family binomial


```r
model <- glm(filam_presence ~ mean_mixing_depth_temp,
             data = filam_diatom, family = binomial)
```

Now we can check the model results


```r
summary(model)
```

```
## 
## Call:
## glm(formula = filam_presence ~ mean_mixing_depth_temp, family = binomial, 
##     data = filam_diatom)
## 
## Coefficients:
##                        Estimate Std. Error z value Pr(>|z|)   
## (Intercept)             10.1581     3.8402   2.645  0.00816 **
## mean_mixing_depth_temp  -0.4202     0.1736  -2.420  0.01551 * 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 79.807  on 69  degrees of freedom
## Residual deviance: 72.195  on 68  degrees of freedom
## AIC: 76.195
## 
## Number of Fisher Scoring iterations: 5
```

So we can see that our p-value for the epilimnion temperature predictor is smaller than 0.05, which means that the presence of filamentous diatoms can be predicted by temperatures. 

Let's see what each component of the model result summary means:

![](https://raw.github.com/kcudding/kcudding.github.io/main/teach/RIntro/model_output_description.jpg)


We are ready for the best part: plotting model predictions  

The dotted curves are confidence intervals, which show us the range in which we are 95% sure about the location of true values, based on our data


```r
# Now, let's calculate predicted probabilities for different values of mean_mixing_depth_temp

# Create a sequence of values for mean_mixing_depth_temp
mean_mixing_depth_temp_values <- seq(min(filam_diatom$mean_mixing_depth_temp), max(filam_diatom$mean_mixing_depth_temp), length.out = length(filam_diatom$mean_mixing_depth_temp))

# Create a data frame with mean_mixing_depth_temp values
newdata <- data.frame(mean_mixing_depth_temp = mean_mixing_depth_temp_values)

# Predict probabilities for each value of mean_mixing_depth_temp
predicted_probabilities_filam <- predict(model, newdata = newdata, type = "response")

# Calculate confidence intervals manually
z <- qnorm(1 - (1 - 0.95) / 2)  # 95% confidence level
se <- sqrt(predicted_probabilities_filam * (1 - predicted_probabilities_filam) / nrow(filam_diatom))
lower_bound <- predicted_probabilities_filam - z * se
upper_bound <- predicted_probabilities_filam + z * se

# Convert "present" and "absent" to 1 and 0
presence_numeric_filam <- ifelse(filam_diatom$filam_presence == "present", 1, 0)

# Plot the predicted probabilities with confidence intervals
plot(mean_mixing_depth_temp_values, predicted_probabilities_filam, type = "l",
     main = "Filamentous diatom presence",
     xlab = "Epilimnion temperature (\u00B0C)", ylab = "Predicted probability", cex.axis = 1.5, ylim = c(0, 1), lwd = 2)
lines(mean_mixing_depth_temp_values, lower_bound, col = "blue", lty = 2)
lines(mean_mixing_depth_temp_values, upper_bound, col = "blue", lty = 2)
points(filam_diatom$mean_mixing_depth_temp, presence_numeric_filam)
```

![](https://raw.github.com/kcudding/kcudding.github.io/main/teach/RIntro/logistic3.png)

*Practice time*:

### Create your own model

Now it's your turn! Run your own logistic model using total phosphorus as a predictor, and filamentous diatoms as the response variable again.

*Formatting* 

Do you remember how you previously labeled your dataset for the phosphorus analysis? You can use that for this exercise. You have already removed NA data, but before proceeding, let's relabel the response variable to remove empty spaces, which may cause errors going forward:


```r
names(filam_diatom_P)[names(filam_diatom_P) == "TP dissolved_ECCC1m"] <- "Total_phosphorus"
```

Now you can create a new column that describes the presence or the absence of filamentous diatoms across the dataset:


```r
# To create a new column, first type the label you chose previously for the dataframe related to the phosphorus analysis (something like "filam_diatom_P"), then create a column name, and use the function "ifelse" to label all observations where measurements were greater than zero as "present", and all observations where measurements were equal to zero as "absent":

your_dataset_name$new_column_name <- ifelse(your_dataset_name$filamentous_Diatom > 0, "present", "absent")
```

Format this new column as a factor


```r
# use the factor function here

# and check how it looks by calling:
levels(your_dataset_name$filamentous_Diatom)
```




*Select time periods and locations*

You can choose to select summer, or another time period. Because we have more data for phosphorus, you can select a single station (column 5: "Station_Acronym")


```r
# Use the subset function to select desired periods of time and locations. You can use "&" for more than one selection at the same time, and "|" for selecting either one or other option:
your_dataset_name <- subset(your_dataset_name, (season=="your selection" | season=="your selection") & (Station_Acronym=="your selection"))
```




*Run model*


```r
# Use glm function and family binomial
your_model_name <- glm("response variable" ~ "explanatory variable",
             data = your_dataset_name, family = binomial)
```




Check model results using the summary function


```r
summary(your_model_name)
```

How well did this predictor do? 

*Plot model predictions*


```r
# Create a sequence of values for Total_phosphorus
Total_phosphorus_values <- seq(min(filam_diatom_P$Total_phosphorus), max(filam_diatom_P$Total_phosphorus), length.out = length(filam_diatom_P$Total_phosphorus))

# Create a data frame with Total_phosphorus values
newdata <- data.frame(Total_phosphorus = Total_phosphorus_values)

# Predict probabilities for each value of Total_phosphorus
predicted_probabilities_P <- predict(model, newdata = newdata, type = "response")

# Calculate confidence intervals manually
z <- qnorm(1 - (1 - 0.95) / 2)  # 95% confidence level
se <- sqrt(predicted_probabilities_P * (1 - predicted_probabilities_P) / nrow(filam_diatom))
lower_bound <- predicted_probabilities_P - z * se
upper_bound <- predicted_probabilities_P + z * se

# Convert "present" and "absent" to 1 and 0
presence_numeric_P <- ifelse(filam_diatom_P$filam_presence == "present", 1, 0)

# Plot the predicted probabilities with confidence intervals
plot(Total_phosphorus_values, predicted_probabilities_P, type = "l", 
     main = "Filamentous diatom presence",
     xlab = "Total phosphorous (dissolved fraction) (mg/L)", ylab = "Predicted probability", cex.axis = 1.5, ylim = c(0, 1), lwd = 2)
lines(Total_phosphorus_values, lower_bound, col = "blue", lty = 2)
lines(Total_phosphorus_values, upper_bound, col = "blue", lty = 2)
points(filam_diatom_P$Total_phosphorus, presence_numeric_P)
```

![](https://raw.github.com/kcudding/kcudding.github.io/main/teach/RIntro/logistic4.png)


## What could go wrong?

- *Mixing up predictors and response variables in the logistic model equation*
    + If you get a warning when running your logistic model, you may have mixed up the predictors and the response variables. The response variable should be the first one to appear after the opening parenthesis 
    + In addition to the variable mix-up, this warning tells us that we may be using quantitative measures as our response variable, which is not appropriate for a logistic regression. See how this error looks like:


```r
model <- glm(Total_phosphorus ~ filam_presence,
             data = filam_diatom_P, family = binomial)
```

```
## Warning in eval(family$initialize): non-integer #successes in a binomial glm!
```

- *Incorrectly coding data so that it is not independent*

Imagine you have counts of living and dead organisms in your dataset:


```r
example_independence <- data.frame(date=rep("Jan-1-2024", times=3), location=c("station-1","station-2", "station-3"), living_daphnia=floor(runif(3, min=0, max=101)),dead_daphnia=floor(runif(3, min=0, max=101)))
# runif function generates random numbers within the range established by "min" and "max" values
knitr::kable(head(example_independence))
```



|date       |location  | living_daphnia| dead_daphnia|
|:----------|:---------|--------------:|------------:|
|Jan-1-2024 |station-1 |             98|           79|
|Jan-1-2024 |station-2 |             71|           13|
|Jan-1-2024 |station-3 |             12|           40|

In this case, instead of considering each individual as "living" or "dead", you should calculate the proportion of living organisms *per replicate* like this:
    

```r
example_independence$proportion <- round(example_independence$living_daphnia/(example_independence$living_daphnia+example_independence$dead_daphnia),2)
# the round function ensures we have 2 decimals in our proportion values

knitr::kable(head(example_independence))
```



|date       |location  | living_daphnia| dead_daphnia| proportion|
|:----------|:---------|--------------:|------------:|----------:|
|Jan-1-2024 |station-1 |             98|           79|       0.55|
|Jan-1-2024 |station-2 |             71|           13|       0.85|
|Jan-1-2024 |station-3 |             12|           40|       0.23|

This proportion will be your response variable for the logistic model. When using proportions, you should also provide the "weights" information in the glm formula (i.e., a dataset with total number of trials per replicate, or the sum of events where we got success + events where we got failure).

- *Categorical response variable is formatted as character instead of factor* 
    + Formatting data as factors allows for defining order or reference group for statistical testing  


```r
# Here we are selecting and formatting the binomial data as we did before, but now we accidentally forget to format the new column as "factor" 
example <- ham

example$filam_presence <- ifelse(example$filamentous_Diatom > 0, "present", "absent")

model <- glm(filam_presence ~ mean_mixing_depth_temp,
             data = example, family = binomial)
```

```
## Error in eval(family$initialize): y values must be 0 <= y <= 1
```

Check the data in the problematic column:
    

```r
str(example$filam_presence)
```

```
##  chr [1:742] "present" NA NA "present" NA NA NA NA NA NA NA NA "present" NA ...
```

See how we need to remove NA data and format the column as factor for our model to run nicely:


```r
example <- example[!is.na(example$filamentous_Diatom), ]

example$filam_presence <- factor(example$filam_presence)

levels(example$filam_presence)
```

```
## [1] "absent"  "present"
```

```r
model <- glm(filam_presence ~ mean_mixing_depth_temp,
             data = example, family = binomial)
```

No more errors now!

That's it for now, but if you are interested in more complex logistic models, here are some resources:

## R resources 

*Multiple logistic regression*

- [Building Skills in Quantitative Biology (Cuddington, Edwards, & Ingalls, 2022)](https://www.quantitative-biology.ca/machine-learning-and-classification.html#multiple-logistic-regression)  

- [Getting started with Multivariate Multiple Regression (Ford, 2024)](https://library.virginia.edu/data/articles/getting-started-with-multivariate-multiple-regression)  


*Multinomial logistic regression*

- [Multinomial logistic regression (Russell, 2022)](https://ladal.edu.au/regression.html#Mixed-Effects_Binomial_Logistic_Regression)  

*Mixed-effects logistic regression*

- [Mixed-Effects Binomial Logistic Regression (Schweinberger, 2022 )](https://stats4nr.com/logistic-regression#modeling-more-than-two-responses)   

- [Mixed-effects logistic regression (Sonderegger, Wagner, & Torreira, 2018)](https://people.linguistics.mcgill.ca/~morgan/qmld-book/mixed-effects-logistic-regression.html)   
