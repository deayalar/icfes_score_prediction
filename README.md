ICFES Score Prediction using regression models

By: David Ernesto Ayala Russi
Mat: 212622

# Introduction

The ICFES (Colombian Institute for the Promotion of Higher Education) is the public organization in charge of managing and evaluating the educational institutes like schools and universities in Colombia. [Official Site(in Spanish)](https://www.icfes.gov.co/)
Specifically, the ICFES applies a standardized test to the students from last year of high school. The exam is known as "Saber 11" and evaluates different subjects such as critical reading, mathematics, social studies, science, and english. The ICFES exam is recognized as the most important test because it assesses the academic skills of the students and its results are taken into account in university admissions.
Every year the ICFES publishes an anonymized dataset with the results of each student but also includes socio-economic and demographic data collected in the enrolment form. 

The goal of this project is to build **regression models to predict the total score** of the student in the test, using the socio-economic and demographic variables as predictors. Information about the schools will be utilized as well.

# Dataset construction
The final dataset utilized in this project is the result of the combination and cleansing of various datasets. The exam is applied twice per year, usually in April and October. Therefore, the ICFES publishes one results dataset for each exam date on the Colombian government open data platform known as ["Datos Abiertos"](https://www.datos.gov.co/en/). 
This project is focused on the results of 2019. So, the original datasets can be found at\
*[Saber 11 2019-1 (April)](https://www.datos.gov.co/en/Educaci-n/Saber-11-2019-1/tkn6-e4ic): One version with adjusted column names in englis can be found [here](https://www.amazon.com/clouddrive/share/sz3ZHNZ8JjlWbXJdM0HPHz4wOwxaSY5hoRuZz55kwsT) (15.5Mb)\
*[Saber 11 2019-2 (October)](https://www.datos.gov.co/en/Educaci-n/Saber-11-2019-2/ynam-yc42): One version with adjusted column names in englis can be found [here](https://www.amazon.com/clouddrive/share/sz3ZHNZ8JjlWbXJdM0HPHz4wOwxaSY5hoRuZz55kwsT) (416.0Mb)

Combining both datasets, it can be seen that the exam was taken by 546.212 people and contain 82 variables. The difference of applicants between the first and the second semester is large because typically the first session is taken by students which school is calendar B, but most of the schools in Colombia are calendar A, whose normally take the exam in the second session.

On the other hand, the ICFES also publishes the classifiaction of schools. It is useful information for school directors because it gives them an overview of the general performance of their students in the test, but most importantly the ICFES gives a quality classification of the school. Data for 2019 is not publicly available yet, but data from 2018 is still useful.
This data is open but requires prior registration in the "DataIcfes" [site](https://www.icfes.gov.co/investigadores-y-estudiantes-posgrado/acceso-a-bases-de-datos). However, the school datasets can be downloaded here\
*[Schools 2018-1](https://www.amazon.com/clouddrive/share/tcug7IAJTVxwR5gfUl9pJJnK2CANK4cI0VnZpRkMCTl) (40.3Kb)\
*[Schools 2018-2](https://www.amazon.com/clouddrive/share/iZGpSPST9VVKs9uV420AutLSd8bAir8RfG6KbW14gYr) (1.4Mb)

A sequence of transformations and combinations are applied on these datasets to get the one used in this notebook. Concretely, in the `preprocessing.Rmd` notebook the datasets are combined and the irrelevant variables are removed, for example identifications, codes, specific subject scores, etc. And finally, since the dataset is in spanish, translations are made in `translating.Rmd`

The following image presents a flow of the transformations made to build the final dataset
![Transformations and combinations](https://sl-project.s3-us-west-2.amazonaws.com/sl_dataset_preparation.png)

An intermediate dataset without translations is found [here](https://www.amazon.com/clouddrive/share/CNv90sKCwIBd2U1QLo7N3CAnR4Pu3GNABXyvkR03pgd) (124.1Mb)
The final dataset con be downloaded from. **This is the one that should be downloaded to run this notebook** [here](https://www.amazon.com/clouddrive/share/04s9ceeIHtdRkGOuk6Vihfh30OFzJq0guEUKNNA8XCQ) (88.3Mb)

## Data loading
The models in this notebook are intended to predict the SCORE_GLOBAL
```{r}
df <- read.csv('Saber_11_2019_translated.csv', stringsAsFactors = T)
df$SCHO_CATEGORY = as.factor(df$SCHO_CATEGORY)
df <- df[!is.na(df$SCORE_GLOBAL),]
colnames(df)
```
This dataset contains 248.487 observations, it is composed by 37 predictors, and the target variable is `SCORE_GLOBAL`, The variables with the `SCHO_` prefix are variables related to the school, `STU_` corresponds to the student and `FAMI_` are variables that defines the familiar conditions useful to describe the socio-economic situation of a given applicant.
A brief description of each variable is given below

"SCHO_TOTAL_SCORE": A number that corresponds to the school calification, it is given by the ICFES\
"SCHO_CATEGORY": categorical variable that express the quality of the school, goes from 1(low quality) to 5(high quality)\
"STU_GENDER": Gender of the student\
"STU_HASETNICITY": True/False that shows if the applicant belongs to any etnical group\
"FAMI_ESTRATO": Categorical variable that reflects socio-economic conditions based on the access to public services\
"FAMI_PEOPLEATHOME": Number of people at home\
"FAMI_ROOMSHOUSE": NUmber of rooms in the house\
"FAMI_FATHEREDUCATION": Level of education achieved by the father of the applicant\
"FAMI_MOTHEREDUCATION": Level of education achieved by the mother of the applicant\
"FAMI_JOBFATHER": Categorical variable that express the job of the father\
"FAMI_JOBMOTHER": Categorical variable that express the job of the mother\
"FAMI_HASINTERNET": Shows if the family has access to the internet\
"FAMI_HASTV": Shows if the family has a tv\
"FAMI_HASCOMPUTER": Shows if the family  has access to a computer\
"FAMI_HASWHASHINGMACHINE": Shows if the family has washing machine\
"FAMI_HASOVEN": Shows if the family has oven\
"FAMI_HASCAR": Shows if the family has car\
"FAMI_HASMOTO": Shows if the family has motorbike\
"FAMI_HASVIDEOGAMESCONSOLE": Shows if the family has any video games console\
"FAMI_NUMBOOKS": Categorical variable that defines the number of books in the house\
"FAMI_EATMILKDERIVATIVES_WEEK": Categorical that describes how often the applicant eat milk derivates in one week\
"FAMI_EATMEATFISHEGGS_WEEK": Categorical that describes how often the applicant eat meat, fish or eggs in one week\
"FAMI_EATCEREALFRUITS_WEEK": Categorical that describes how often the applicant eat cereals and fruits in one week\
"STU_DAILYREADING": Categorical to define the daily number of hours dedicated to read for reasons different from studying\
"STU_DAILYINTERNET": Categorical to define the daily number of hours dedicated to the internet for reasons different from studying\
"STU_WEEKLYWORKHOURS":Number of weekly hours that the student works (if he/she works)\
"SCHO_GENDER": A school may be mixed, but it may also have only female or only male students\
"SCHO_NATURE": Private of Public school\
"SCHO_CALENDAR": Calendar of the school can be A or B depending on when the academic year starts\
"SCHO_BILINGUAL": Shows if the school is bilingual or not\
"SCHO_TYPE": Describes if the school is academic, technical, etc.\
"SCHO_AREA_LOCATION": Urban or Rural school\
"SCHO_TIME": The school can give lessons in the morning, afternoon, night, saturdays, etc.\
"STU_INSE_INDIVIDUAL": Socio-economic indicator of the student computed by the ICFES. It goes from 0 up to 100\
"STU_NSE_INDIVIDUAL": Socio-economic group of the student computed by the ICFES\
"STU_AGE": Age off the student\
**"SCORE_GLOBAL": Target variable, this is the global score that goes from 0 up to 500**

```{r}
hist(df$SCORE_GLOBAL)
boxplot(df$SCORE_GLOBAL)
```
```{r}
summary(df$SCORE_GLOBAL)
```

## First Question
Is it possible to wrap the familiar demographic variables in one variable? Answering that question it will be possible to remove most of the categorical variables in the model.

Usually, "Estrato" is used to describe the economic situation of a person or a group of people in the same territory such as neighbourhood, district and so on. It is defined taking into account variables like access to public services, road infrastructure, etc. So, this is the first candidate. "Estrato" is a discrete variable that can take values from 1 to 6 or "No Estrato" (usually for rural areas), being 1 the lowest economic situation and 6 the highest

`SCHO_CATEGORY` is a variable that describes the quality if the school in a categorical variable in which 1 is low quality and 5 is high quality. Here the relationship betwween these two variables will be explored.
```{r}
require(dplyr)
library(ggplot2)
agg_estrato_df <- df %>%
  group_by(FAMI_ESTRATO,SCHO_CATEGORY) %>% 
  summarise(Percentage=n()) %>% 
  group_by(FAMI_ESTRATO) %>% 
  mutate(Percentage=Percentage/sum(Percentage)*100) %>% 
  as.data.frame()
ggplot(agg_estrato_df, aes(x=FAMI_ESTRATO,y=SCHO_CATEGORY,fill=Percentage))+
      geom_tile()+
      scale_fill_gradientn(colours = c("white","blue"), values = c(0,1))
```

Most of the high quality schools in Colombia are private and accessing these institutions not only can be very expensive, but also are reserved to wealthy families. In this graph it can be seen a clear sign of inequality between school quality and the "estrato". It is noticeable that people from lowest estratos cannot access to high quality schools and most of the people from higher estratos can afford the best schools. But it is even more worrying to see that people with no estrato, usually rural areas can barely reach the middle quality schools.

A stratified sample by "estrato" is going to be extracted from the data to fit a classification model and see if the family variables can be explained by `FAMI_ESTRATO`
```{r}
set.seed(1)
sample_by_estrato <- df %>%
  group_by(FAMI_ESTRATO) %>%
  sample_n(500)
```

The classification model will be a Support Vector Machine with a linear kernel
```{r}
library(e1071)
library(caret)
svmfit = svm (FAMI_ESTRATO ~ FAMI_PEOPLEATHOME+FAMI_ROOMSHOUSE+FAMI_FATHEREDUCATION+FAMI_MOTHEREDUCATION+FAMI_JOBFATHER+FAMI_JOBMOTHER+FAMI_HASINTERNET+FAMI_HASTV+FAMI_HASCOMPUTER+FAMI_HASWHASHINGMACHINE+FAMI_HASOVEN+FAMI_HASCAR+FAMI_HASMOTO+FAMI_HASVIDEOGAMESCONSOLE+FAMI_NUMBOOKS+FAMI_EATMILKDERIVATIVES_WEEK+FAMI_EATMEATFISHEGGS_WEEK+FAMI_EATCEREALFRUITS_WEEK, data=sample_by_estrato ,cost =0.1, kernel = "linear")
confusionMatrix(predict(svmfit, sample_by_estrato[, c("FAMI_PEOPLEATHOME", "FAMI_ROOMSHOUSE", "FAMI_FATHEREDUCATION", "FAMI_MOTHEREDUCATION",  "FAMI_JOBFATHER", "FAMI_JOBMOTHER", "FAMI_HASINTERNET", "FAMI_HASTV", "FAMI_HASCOMPUTER", "FAMI_HASWHASHINGMACHINE", "FAMI_HASOVEN", "FAMI_HASCAR", "FAMI_HASMOTO", "FAMI_HASVIDEOGAMESCONSOLE", "FAMI_NUMBOOKS", "FAMI_EATMILKDERIVATIVES_WEEK", "FAMI_EATMEATFISHEGGS_WEEK", "FAMI_EATCEREALFRUITS_WEEK")]), sample_by_estrato$FAMI_ESTRATO)
```

The training accuracy is too low, and there are many misclassified observations, So, it is better spend time exploring other variables such as `STU_INSE_INDIVIDUAL`
Let's explore the relation ship between the `STU_INSE_INDIVIDUAL` and `SCORE_GLOBAL`, Is it possible to say that the performance on the exam is explainable by the socio-economic situation?
```{r}
cor(df$SCORE_GLOBAL, df$STU_INSE_INDIVIDUAL, method = "pearson")
model.linear <- lm(SCORE_GLOBAL ~ STU_INSE_INDIVIDUAL, data=df)
summary(model.linear)
```
From these results it can be said that is an important variable but it's not enough, which means that the performance not only depends on the socio-economic indicator.

Then relationship between familiar `FAMI_` variables and `STU_INSE_INDIVIDUAL` is explored below. A linear model will be fitted.
```{r}
model.fami <- lm(STU_INSE_INDIVIDUAL ~ FAMI_ESTRATO+FAMI_PEOPLEATHOME+FAMI_ROOMSHOUSE+FAMI_FATHEREDUCATION+FAMI_MOTHEREDUCATION+
FAMI_JOBFATHER+FAMI_JOBMOTHER+FAMI_HASINTERNET+FAMI_HASTV+FAMI_HASCOMPUTER+FAMI_HASWHASHINGMACHINE+FAMI_HASOVEN+FAMI_HASCAR+FAMI_HASMOTO+FAMI_HASVIDEOGAMESCONSOLE+FAMI_NUMBOOKS+FAMI_EATMILKDERIVATIVES_WEEK+FAMI_EATMEATFISHEGGS_WEEK+FAMI_EATCEREALFRUITS_WEEK, data=df)
summary(model.fami)
```
Free space
```{r}
rm(model.fami)
rm(model.linear)
```

The Adjusted R-squared:  0.9856 is high and most of the variables are statistically significant, So it is possible to say that **Familiar variables can be wrapped by the socio-economic indicator `STU_INSE_INDIVIDUAL`**. In practice for a new model that predicts the `SCORE_GLOBAL` it will be a nested model. Since `STU_INSE_INDIVIDUAL` is computed by the ICFES, it will be possible to predict on this linear model and then predict on the global score model.

# Regression Modeling

This dataframe contains all columns except those related to familiar conditions
```{r}
df_nofamily <- df[, c("SCHO_TOTAL_SCORE","SCHO_CATEGORY","STU_GENDER","STU_HASETNICITY","STU_DAILYREADING", "STU_DAILYINTERNET","STU_WEEKLYWORKHOURS","SCHO_GENDER","SCHO_NATURE","SCHO_CALENDAR","SCHO_BILINGUAL","SCHO_TYPE","SCHO_AREA_LOCATION","SCHO_TIME","STU_INSE_INDIVIDUAL","STU_NSE_INDIVIDUAL","STU_NSE_ESTABLISHMENT","STU_AGE","SCORE_GLOBAL")]
```

Test/train split, train set will contain 75% of the data, test set wil contain the remaining 25%
The resulting dataset contains 18 predictors
```{r}
set.seed (1)
x = model.matrix(SCORE_GLOBAL ~ ., df_nofamily)[ , -1]
y = df_nofamily$SCORE_GLOBAL
train = sample(1:nrow(x), nrow(x) * 0.75)
test = (-train)
y.test = y[test]
```

## Linear regression
The goal of this project is to find out regression models that can predict the `SCORE_GLOBAL` the first model will be a linear regression model

Baseline Linear Regression model with all the variables to see how it performs with the actual data size in terms of memory this is as a first exploratory analysis and a starting point

Model without family variables
```{r}
model.linear <- lm(SCORE_GLOBAL ~ ., data=df_nofamily, subset = train)
summary(model.linear)
```

```{r}
linear.pred = predict(model.linear, newdata = df_nofamily[test,])
linear.pred.train = predict(model.linear, newdata = df_nofamily[train, ])
print(paste0("Test MSE = ", mean (( linear.pred - y.test ) ^2)))
print(paste0("Train MSE = ", mean (( linear.pred.train - y[train] ) ^2)))
```

The previous model includes many predictors that are not statistically significant, therefore It is worthy to explore the efects of regularization in this model, Is it possible to get a simpler model?

## Lasso regression
First of all, it is noticeable that in this dataset the number of observations is much larger than the number of predictors. So it would be preferred keeping all the coefficient estimates, but using a shrinkage method. Here, Lasso is preferred because I want the coefficient estimates approach zero faster. In fact, some of them could be zero

Fit a null model and a one model without shrinking
```{r}
#install.packages("glmnet")
library(glmnet)
grid =10^ seq (10 , -2 , length =100)
lasso.mod = glmnet ( x [ train ,] , y [ train ] , alpha =1 , lambda = grid , thresh =1e-12)
```

The Null model can be obtained using a large value of lambda
```{r}
lasso.pred = predict (lasso.mod, s =1e10 , newx = x [ test ,])
mean (( lasso.pred - y.test ) ^2)
```

Least squares model with lambda = 0 will give the same performance of a linear regression with all the predictors
```{r}
lasso.pred = predict (lasso.mod, s =0 , newx = x [test ,])
mean (( lasso.pred - y.test ) ^2)
```

Use cross-validation to find the best lambda, by default it uses k = 10
```{r}
cv.out = cv.glmnet( x [ train ,] , y [ train ] , alpha =1)
bestlambda = cv.out$lambda.min
print(paste0("Best lambda: ", bestlambda))
lasso.pred = predict ( lasso.mod , s=bestlambda , newx=x[test ,])
mean((lasso.pred - y.test ) ^2)
```

Mean square error is quite similar to the model with all the variables.
Test performance with the chosen lambda is quite similar. However, let's examine the coefficients, 
```{r}
lasso.out = glmnet (x ,y , alpha =1 , lambda = grid)
lasso.coef = predict(lasso.out, type = "coefficients" , s = bestlambda ) [1:49,]
lasso.coef[lasso.coef ==0]
```
Certainly it was possible to get a simpler model since some of the coefficients were zero

## Partial least squares
Although the lasso regression model is simpler, removing 2 out of 49 predictors (Including categorical variables) is not a huge improvement, here partial least squares method will be explored to get a model with less variables reducing the dimensionality
```{r}
#install.packages("pls")
library(pls)
pls.fit = plsr(SCORE_GLOBAL ~ ., data=df_nofamily , subset = train , scale = TRUE , validation ="CV")
summary ( pls.fit )
#validationplot(pls.fit, val.type ="MSEP")
```

It can be seen that lowest cv error is when the number of components is 12, so getting the MSE in the test set, it is very similar than the model with all paremeters and lasso regression.
```{r}
pls.pred = predict(pls.fit, x[ test ,] , ncomp =12)
mean (( pls.pred - y.test ) ^2)
```
Partial least squares model consumes a lot of memory
```{r}
rm(pls.fit)
```
So far the models have been linear. Let's explore what is beyond that

## Polynomial Regressions
So far, it has been showed that the `STU_INSE_INDIVIDUAL` is an important variable, in this model a polynomial regression of grade 2 will be fitted
```{r}
polynomial_model <- lm(SCORE_GLOBAL ~ poly(STU_INSE_INDIVIDUAL,2)+SCHO_TOTAL_SCORE+SCHO_CATEGORY+STU_GENDER+STU_HASETNICITY+STU_DAILYREADING+STU_DAILYINTERNET+STU_WEEKLYWORKHOURS+SCHO_GENDER+SCHO_NATURE+SCHO_CALENDAR+SCHO_BILINGUAL+SCHO_TYPE+SCHO_AREA_LOCATION+SCHO_TIME+STU_NSE_ESTABLISHMENT+STU_AGE+STU_NSE_INDIVIDUAL, data=df_nofamily, subset = train)
summary(polynomial_model)
```
```{r}
polynomial_model.predict = predict(polynomial_model, newdata = df_nofamily[test, ] )
mean (( polynomial_model.predict - y.test ) ^2)
```
The mean squared error is sligthy better than the linear model, however the next model will be a combination of partial least squared and the polynomial approach.

Applying dimensionality reduction to the polynomial model can improve the performance
```{r}
library(pls)
pls.fit = plsr(SCORE_GLOBAL ~ poly(STU_INSE_INDIVIDUAL,2)+SCHO_TOTAL_SCORE+SCHO_CATEGORY+STU_GENDER+STU_HASETNICITY+STU_DAILYREADING+STU_DAILYINTERNET+STU_WEEKLYWORKHOURS+SCHO_GENDER+SCHO_NATURE+SCHO_CALENDAR+SCHO_BILINGUAL+SCHO_TYPE+SCHO_AREA_LOCATION+SCHO_TIME+STU_NSE_ESTABLISHMENT+STU_AGE+STU_NSE_INDIVIDUAL, data=df_nofamily , subset = train , scale = TRUE , validation ="CV")
summary ( pls.fit )
```

Test mean squared error of the partials least sqaured uncluding polynomial features
```{r}
pls.pred.test = predict(pls.fit, cbind(poly(df_nofamily$STU_INSE_INDIVIDUAL[test],2), df_nofamily[test, ]) , ncomp =12)
pls.pred.train = predict(pls.fit, cbind(poly(df_nofamily$STU_INSE_INDIVIDUAL[train],2), df_nofamily[train, ]) , ncomp =12)
print(paste0("Test MSE = ", mean (( pls.pred.test - y.test ) ^2)))
print(paste0("Train MSE = ", mean (( pls.pred.train - y[train] ) ^2)))
```
The MSE is better in this model

```{r}
rm(pls.fit)
```


## Tree based regression
This section will explore the use of tree regression models, the first model will be a single tree regression
```{r}
#install.packages("tree")
library (tree)
tree.regression = tree(SCORE_GLOBAL ~ . , df_nofamily , subset = train )
summary(tree.regression)
```

```{r}
plot (tree.regression)
text (tree.regression, pretty =0)
```

```{r}
cv.tree.regression = cv.tree(tree.regression)
plot (cv.tree.regression$size, cv.tree.regression$dev, type = 'b')
```

```{r}
tree.pred.test = predict(tree.regression, df_nofamily[test,])
tree.pred.train = predict(tree.regression, df_nofamily[train,])
print(paste0("Test MSE = ", mean (( tree.pred.test - y.test ) ^2)))
print(paste0("Train MSE = ", mean (( tree.pred.train - y[train] ) ^2)))
```
The tree based regression has a poor performance compared to the previous models, however it is worthy to explore other methods

## Boosting
In this project, the boosting will be applied to regression trees
```{r}
#install.packages("gbm")
library ( gbm )
model.boost = gbm(SCORE_GLOBAL ~ ., data = df_nofamily [ train ,] , distribution="gaussian" , n.trees =100, interaction.depth =5)
summary(model.boost)
```

Running the boosting regression, the most important predictors can be devealed, in this model `SCHO_CATEGORY` and `STU_INSE_INDIVIDUAL` are by far the most important.
So, below the partial dependence plot are displayed
```{r}
par ( mfrow = c (1 ,2) )
plot ( model.boost , i ="SCHO_CATEGORY")
plot ( model.boost , i ="STU_INSE_INDIVIDUAL")
```

```{r}
yhat.boost = predict (  model.boost , newdata = df_nofamily[test,] , n.trees =100)
yhat.boost.train = predict (  model.boost , newdata = df_nofamily[ train ,] , n.trees =100)
print(paste0("Test MSE = ", mean (( yhat.boost - y.test ) ^2)))
print(paste0("Train MSE = ", mean (( yhat.boost.train - y[train] ) ^2)))
```

It was a huge improvement compared to the previous models, it shows the impact of techniques like boosting. The next model will also use a shrinkage factor

```{r}
model.boost = gbm(SCORE_GLOBAL ~ ., data = df_nofamily [ train ,] , distribution="gaussian" , n.trees =100, interaction.depth =7,  shrinkage =0.5, verbose=F)
summary(model.boost)
```

```{r}
yhat.boost = predict (  model.boost , newdata = df_nofamily[test,] , n.trees =100)
yhat.boost.train = predict (  model.boost , newdata = df_nofamily[ train ,] , n.trees =100)
print(paste0("Test MSE = ", mean (( yhat.boost - y.test ) ^2)))
print(paste0("Train MSE = ", mean (( yhat.boost.train - y[train] ) ^2)))
```

The shrinkage factor had a huge impact in the model performance

# Conclusion
This work shows different regression models intended to predict the global score achieved by a student in the ICFES exam. On one side, there is a linear model that can wrap the familiar predictors in a socio-economic variable and, on the other hand, the best model to predict the global score was a boosting technique of regression trees.
