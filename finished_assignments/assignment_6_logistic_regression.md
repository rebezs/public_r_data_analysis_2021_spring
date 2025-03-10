---
title: "Assignment 4: Logistic regression"
author: "Marton Kovacs"
output: 
  html_document: 
    toc: yes
    keep_md: yes
editor_options: 
  chunk_output_type: console
---



# Background story

In this lab assignment you are going to work with data related to the survival of passengers of the RMS Titanic. “The sinking of the Titanic is one of the most infamous shipwrecks in history. On April 15, 1912, during her maiden voyage, the widely considered “unsinkable” RMS Titanic sank after colliding with an iceberg. Unfortunately, there weren’t enough lifeboats for everyone onboard, resulting in the death of 1502 out of 2224 passengers and crew. While there was some element of luck involved in surviving, it seems some groups of people were more likely to survive than others.” (Quote from the Kaggle Titanic Challenge).

For the sake of this assignment, let’s imagine that you are called as an expert to a court case: Kate, one of the survivors of the Titanic accident is suing her __father, Leonardo, for not accompanying Kate and her mother Sue on the trip__ and this way decreasing their chances of survival. The family planned to move to the US back in 1912. __They bought 3rd class tickets for the three of them for 8 British Pounds each. (They did not get cabins with their 3rd class tickets.)__ The plan was that they embark in Southampton and all of them got on board, but Leonardo got separated from them in the rush of passengers during boarding. Later it turned out that Leonardo deliberately got separated from them and got off the boat before it’s departure, to run away and live with his mistress. __Kate was only 4 at the time, and Sue was 20.__ During the accident __Kate got on one of the last lifeboats and was later rescued, but there was no room for Sue on the lifeboat, and she did not survive the disaster.__

Now 20 years later Kate is suing her father for leaving them on the boat, because she thinks that this eventually led to Sue’s death, as the absence of Leonardo decreased their chances of survival.

You are called in as an expert to this court case. Your task is to present a report about whether the presence of Leonardo statistically could have led to an improved chance of survival.

# Dataset

Use the data file called ‘assignment_4_dataset’, from the 'data/' folder.

This is the training dataset of the Titanic dataset from the Kaggle Titanic Challenge (https://www.kaggle.com/c/titanic/overview), a prediction challenge for people who are just starting to learn about machine learning and other statistical prediction techniques. The following description is available for the dataset:

## Metadata


|Variable |Definition                                                        |Notes                                                                                                              |
|:--------|:-----------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------|
|survival |Survival                                                          |0 = No, 1 = Yes                                                                                                    |
|pclass   |Ticket class                                                      |1 = 1st, 2 = 2nd, 3 = 3rd                                                                                          |
|sex      |Sex                                                               |male or female                                                                                                     |
|Age      |Age in years                                                      |NA                                                                                                                 |
|sibsp    |Number of siblings or spouses accompanying the passenger on board |(for example if the passenger was traveling with 2 of his/her siblings and his/her spouse, this number would be 3) |
|parch    |Number of parents or children accompanying the passenger on board |(for example if the passenger was traveling just with his/her mother, this number would be 1)                      |
|ticket   |The ID number of the ticket                                       |NA                                                                                                                 |
|fare     |Passenger fare (in Pounds)                                        |NA                                                                                                                 |
|cabin    |Cabin number (if any)                                             |NA                                                                                                                 |
|embarked |Port of Embarkation                                               |C = Cherbourg, Q = Queenstown, S = Southampton                                                                     |

# Task

As usual, start with exploring your dataset. Do descriptive and exploratory analysis including visualization to understand the data and to see what type of data you are dealing with. 

You should buin challenges on Kaggle, so there is plenty of discussion and guides on the web about different models and features. If you get stuck, you can look these up to improve your prediction performance.
ld a statistical model with which you can accurately estimate Kate’s and Sue’s chances of survival. First you should fit a statistical model (for example a logistic regression model) on the dataset, calculate the regression equation, and use that equation to compute the survival probability for Kate and Sue separately with and without having Leonardo on board the ship with them.

You can use whichever predictor you would like, but you need to build a model that is at least as accurate so that it can correctly predict the outcome value within the sample with at least 72% accuracy for BOTH those who actually survived and who actually died in the disaster. You need to check this in the Classification table. So it is not enough to have 72% overall correct percentage! In order to be able to reach this prediction accuracy you might have to use some special predictors or to do some feature engineering. A comprehensive exploratory analysis including the visualisation of the relationship of different predictors might help in this. Keep in mind that this is one of the most popular predictio
You do not need to check model assumptions in this assignment (but you can do so if you want to and this might help you improve your prediction performance). 

# What to report

When you have arrived at a satisfactory model describe the final model to the reader so that it is clear how is the model built up, and that based on the description the reader could reproduce your model.

Report about the goodness of fit of the model, whether it is significantly better than the null model (based on the AIC and chi^2 test statistics), and how effective is your model at predicting the outcome (based on McFadden R^2, and the correct prediction percentages in the classification table of the final model). Be sure to report the total correct prediction percentage of the final model and also the correct prediction percentages separately for those who actually died, and those who actually survived.

Also, report the statistics describing the coefficients of the predictors in a table format (for each predictor, this table should include the following: logit regression coefficients, Odds ratios, and 95% confidence intervals for the Odds ratios, Chi^2 test statistics and p values, and AIC values for the reduced models). 

Report which were the most influential predictors in the model, and which were the predictors which did not seem to have unique added value to the model.

Write up the regression equation of the model in the form of 𝑌 = 𝑏0 + 𝑏1 ∗ X1 + 𝑏2 ∗ X2 +…+ bn * Xn, in which you use the actual regression coefficients of your models. (b0 stands for the intercept and b1, b2 … bn stand for the model coefficients for each of the predictors, and X1, X2, … Xn denote the predictors).

Finally, report the predicted probability of survival for Kate and Sue separately with and without having Leonardo on board the ship with them. (So you will have to estimate 4 probabilities in total, two for Kate and two for Sue). It is important that this is in the probability scale (since the jury does not know what logit means and how to interpret it).

# What to discuss

In your discussion of the findings, briefly interpret the results of the above analyses in light of the court case. Based on your results do you find it likely that the presence of Leonardo (spouse to Sue and parent to Kate) would have improved the survival chances of Sue and Kate? What is the best predictor of survival in the model and how does the presence of a spouse and presence of a parent compare to its influence?

# Solution

## Read the data

Read the dataset used in this assignment. Pay attention to the extension of the datafile.


```r
titanic_dataset <- haven::read_sav(here::here("~/Dokumentumok/3_phd_courses/r/assignment/public_r_data_analysis_2021_spring/data/assignment_6_dataset.sav")) %>%
  mutate(Survived = as.factor(Survived),
         Pclass = as.factor(Pclass),
         Sex = as.factor(Sex),
         Embarked = as.factor(Embarked)
        )

label_class = c ("1" = "1st Class", "2" = "2nd Class", "3" = "3rd Class")
label_sex = c("female" = "Females", "male" = "Males")
label_embarked = c("C" = "Cherbourg", "Q" = "Queenstown", "S" = "Southampton")
label_survived = c("0" = "Not survived", "1" = "Survived")
```

## EDA



```r
port_freq <- ggplot(titanic_dataset, aes(Age, fill = Embarked)) +
  geom_histogram(binwidth = 1) +
  labs(title = "The travellers embarked in each port") +
  theme(legend.position = "bottom", legend.title = element_blank()) +
  scale_fill_viridis_d(labels = label_embarked)

port_freq
```

```
## Warning: Removed 177 rows containing non-finite values (stat_bin).
```

![](assignment_6_logistic_regression_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
survival_age <- ggplot(titanic_dataset, aes(Sex, Age, color = Survived)) +
  geom_point(alpha = 0.2, size = 7) +
  coord_flip() +
  labs(title = "The age of passengers") +
  xlab(NULL) +
  ylab(NULL) +
  theme(legend.title = element_blank(), legend.position = c(0.1, 0.9)) +
  scale_color_viridis(discrete = TRUE, labels = label_survived)

survival_age
```

```
## Warning: Removed 177 rows containing missing values (geom_point).
```

![](assignment_6_logistic_regression_files/figure-html/unnamed-chunk-3-2.png)<!-- -->

```r
class_freq <- ggplot(titanic_dataset, aes(Age, fill = Sex)) +
  geom_histogram(binwidth = 1) +
  facet_wrap(vars(Pclass), labeller = labeller(Pclass = label_class)) +
  labs(title = "The travellers in each class") +
  theme(legend.position = "bottom", legend.title = element_blank()) +
  scale_fill_viridis_d(labels = label_sex)

class_freq
```

```
## Warning: Removed 177 rows containing non-finite values (stat_bin).
```

![](assignment_6_logistic_regression_files/figure-html/unnamed-chunk-3-3.png)<!-- -->

```r
survival_freq <- ggplot(titanic_dataset, aes(Age, fill = Survived)) +
  geom_histogram(binwidth = 1) +
  facet_grid(cols = vars(Pclass), rows = vars(Sex), labeller = labeller(Pclass = label_class, Sex = label_sex)) +
  labs(title = "Age of survivers faceted by sex and class") +
  theme(legend.position = "bottom", legend.title = element_blank()) +
  scale_fill_viridis_d(labels = label_survived)

survival_freq
```

```
## Warning: Removed 177 rows containing non-finite values (stat_bin).
```

![](assignment_6_logistic_regression_files/figure-html/unnamed-chunk-3-4.png)<!-- -->

```r
survival_prop <- ggplot(titanic_dataset, aes(Pclass, fill = Survived)) +
  geom_bar(position = "fill") +
  facet_wrap(vars(Sex), labeller = labeller(Sex = label_sex)) +
  labs(title = "The Proportion of survivers") +
  xlab("Passenger class") +
  ylab(NULL) +
  theme(legend.position = "bottom", legend.title = element_blank()) +
  scale_fill_viridis_d(labels = label_survived)

survival_prop
```

![](assignment_6_logistic_regression_files/figure-html/unnamed-chunk-3-5.png)<!-- -->

```r
titanic_dataset_log <- titanic_dataset %>%
mutate(Survived = as.numeric(as.character(Survived)))

survival_line <- ggplot(titanic_dataset_log, aes(Age, Survived)) +
   facet_grid(cols = vars(Pclass), rows = vars(Sex), labeller = labeller(Pclass = label_class, Sex = label_sex)) +
   geom_point() +
   geom_smooth(
     method = "glm", 
     color = "blue",
     se = FALSE, 
     method.args = list(family = binomial)
   )
 
survival_line
```

```
## `geom_smooth()` using formula 'y ~ x'
```

```
## Warning: Removed 177 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 177 rows containing missing values (geom_point).
```

![](assignment_6_logistic_regression_files/figure-html/unnamed-chunk-3-6.png)<!-- -->


## Clean the data


```r
clean_dataset <- titanic_dataset %>%
  dplyr::select(Survived, Age, Sex, Pclass, SibSp, Parch) %>%
  mutate(Pclass = as.numeric(as.character(Pclass))) %>%
  na.omit()
```

## Creating a datatable for Sue, Kate, and Leonardo


```r
Name <- c("Kate with Leo", "Sue with Leo", "Kate without Leo", "Sue without Leo")
Age <- c(4, 22, 4, 22)
SibSp <- c(0, 1, 0, 0)
Parch <- c(2, 1, 1, 1)
Sex <- c("female","female","female","female")
Pclass <- c(3,3,3,3)

family_data <- data.frame(Name, Age, SibSp, Parch, Pclass)
```

## Building the null model


```r
null_model <- glm(Survived ~ 1, data = clean_dataset, family = binomial)
summary(null_model)
```

```
## 
## Call:
## glm(formula = Survived ~ 1, family = binomial, data = clean_dataset)
## 
## Deviance Residuals: 
##    Min      1Q  Median      3Q     Max  
## -1.021  -1.021  -1.021   1.342   1.342  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)    
## (Intercept)  -0.3799     0.0762  -4.985  6.2e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 964.52  on 713  degrees of freedom
## Residual deviance: 964.52  on 713  degrees of freedom
## AIC: 966.52
## 
## Number of Fisher Scoring iterations: 4
```

## Building the model


```r
dataset_model <- clean_dataset %>%
  filter(Sex == "female")

model <- glm(Survived ~ Age + Pclass + SibSp + Parch, data = dataset_model, family = binomial)
model <- step(model, Survived ~ Age + Pclass + SibSp + Parch, data = dataset_model, family = binomial)
```

```
## Start:  AIC=210.53
## Survived ~ Age + Pclass + SibSp + Parch
## 
##          Df Deviance    AIC
## - Parch   1   201.14 209.14
## <none>        200.53 210.53
## - SibSp   1   205.63 213.63
## - Age     1   205.68 213.68
## - Pclass  1   274.75 282.75
## 
## Step:  AIC=209.14
## Survived ~ Age + Pclass + SibSp
## 
##          Df Deviance    AIC
## <none>        201.14 209.14
## + Parch   1   200.53 210.53
## - Age     1   207.03 213.03
## - SibSp   1   208.05 214.05
## - Pclass  1   279.88 285.88
```

```r
summary(model)
```

```
## 
## Call:
## glm(formula = Survived ~ Age + Pclass + SibSp, family = binomial, 
##     data = dataset_model)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -3.1892   0.1137   0.2362   0.5423   1.8384  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)    
## (Intercept)  7.88879    1.17643   6.706 2.00e-11 ***
## Age         -0.03506    0.01488  -2.356   0.0185 *  
## Pclass      -2.29018    0.34565  -6.626 3.45e-11 ***
## SibSp       -0.44912    0.17899  -2.509   0.0121 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 290.76  on 260  degrees of freedom
## Residual deviance: 201.14  on 257  degrees of freedom
## AIC: 209.14
## 
## Number of Fisher Scoring iterations: 6
```

# Check the assumptions



```r
Survived_prob <- predict(model, family_data, type = "response")
Survived_odds <- Survived_prob / (1 - Survived_prob)


Family_data_assumption <- data.frame(family_data, Survived_prob)
```

# Compare the models


```r
McFaden_R2 <- 1-(logLik(model) / logLik(null_model))

classDF <- data.frame(response = dataset_model$Survived, predicted = round(fitted(model),0))
class_table <- xtabs(~ predicted + response, data = classDF)
sensitivity <- class_table[2,2] / (class_table[2,1] + class_table[2,2])
specificity <- class_table[1,1] / (class_table[1,2] + class_table[1,1])

res <- chisq.test(dataset_model$Survived, round(fitted(model),0))

models <- list(null_model, model)
modelnames <- c("null model", "model")
aic <- aictab(cand.set = models, modnames = modelnames)
```

# Calculate odds ratio and confidence interval


```r
odds_and_conf <- logistic.display(model, simplified=TRUE)
```

# Report the results

The null model was fitted to all the passengers.
The build the model with higher accuracy I filtered out the female passengers. The age (Age), the passenger class (Pclass), the number of siblings or spouses accompanying the passenger (SibSp), and the number of parents or children accompanying the passenger (Parch) were entered as predictor values. Stepwise regression analysis was performed to find the significant predictor variables.
The regression equation of the model is: Survived = 7.793 - 0.033 ∗ Age - 0.248 ∗ Pclass - 0.406 . The best predictor of the survival is the passengers' class (b =  - 0.406, p < 0.001). The presence of spouse or siblings was significant (p = 0.03), while the presence of parent or child was not significant, thus it was not included in the final model.


```r
summary(model)
```

```
## 
## Call:
## glm(formula = Survived ~ Age + Pclass + SibSp, family = binomial, 
##     data = dataset_model)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -3.1892   0.1137   0.2362   0.5423   1.8384  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)    
## (Intercept)  7.88879    1.17643   6.706 2.00e-11 ***
## Age         -0.03506    0.01488  -2.356   0.0185 *  
## Pclass      -2.29018    0.34565  -6.626 3.45e-11 ***
## SibSp       -0.44912    0.17899  -2.509   0.0121 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 290.76  on 260  degrees of freedom
## Residual deviance: 201.14  on 257  degrees of freedom
## AIC: 209.14
## 
## Number of Fisher Scoring iterations: 6
```

```r
res
```

```
## 
## 	Pearson's Chi-squared test with Yates' continuity correction
## 
## data:  dataset_model$Survived and round(fitted(model), 0)
## X-squared = 78.536, df = 1, p-value < 2.2e-16
```

```r
class_table
```

```
##          response
## predicted   0   1
##         0  37  13
##         1  27 184
```

```r
sensitivity
```

```
## [1] 0.8720379
```

```r
specificity
```

```
## [1] 0.74
```

```r
odds_and_conf
```

```
##  
##               OR  lower95ci upper95ci     Pr(>|Z|)
## Age    0.9655512 0.93779340 0.9941306 1.849758e-02
## Pclass 0.1012483 0.05142473 0.1993443 3.454665e-11
## SibSp  0.6381910 0.44936014 0.9063727 1.210103e-02
```

```r
aic
```

```
## 
## Model selection based on AICc:
## 
##            K   AICc Delta_AICc AICcWt Cum.Wt      LL
## model      4 209.30       0.00      1      1 -100.57
## null model 1 966.52     757.22      0      1 -482.26
```

```r
McFaden_R2
```

```
## 'log Lik.' 0.7914559 (df=4)
```

```r
Family_data_assumption
```

```
##               Name Age SibSp Parch Pclass Survived_prob
## 1    Kate with Leo   4     0     2      3     0.7064139
## 2     Sue with Leo  22     1     1      3     0.4496468
## 3 Kate without Leo   4     0     1      3     0.7064139
## 4  Sue without Leo  22     0     1      3     0.5614428
```

# Discussion

Sue's survival probability without Leo on the board was 0.5614428, while with Leo, it was 0.4496468.

Kate's survival probability with or without Leo on the board was 0.7064139, as having a parent on the board was not entered as a predictor variable to the model.

The results shows that Kate had a higher probability to survive without Leo on the board, and that Leo's absence had nothing to do with her death.




