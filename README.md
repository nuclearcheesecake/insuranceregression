# Predicting health insurance costs using linear regression (linear algebra approach + R)

<p align="center">
  <img width="525" src="https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/national-cancer-institute-NFvdKIhxYlU-unsplash.jpg">
</p>

## Table of Contents

1. [Scenario explanation](#1)
2. [Obtaining and cleaning the data](#2)
3. [Creating the LRM](#3)
4. [Validation and prediction](#4)

<a name="1"></a>
## 1. Scenario explanation

<ins> What is the point of this project? </ins>

After completing a theoretical module on _linear regression models_ at university, I wanted to test what I learned, and see if I can use my knowledge to make predictions given raw data. I will thus be creating the linear regression model by using the linear algebra and statistics that I studied during the semester (using R for computations), and then verify my findings using the linear regression functions in R, such as lm() and automatic selection functions.

The goal of this project is to test the waters to see if I can create a model that reasonably predicts medical costs billed by health insurers, given a few attributes of the patient. I will firstly clean the data, because dirty data won't provide good predictions. Then I will proceed to create the LRM, with a process that will be discussed in [section 3](#3).

After this, I will hopefully have constructed a useful model, which I can then use to test on new data. Because I obtained the data from an external source, and thus have no control over the assignment of variables, this will be an **exploratory observational study**.

<ins> How will I validate my model? </ins>

In order to test whether the model I build can make good decisions, I will be using a number of "goodness of fit" measures. But I will also use _cross-validation_, where I split my data 80/20, and use 80% of the data to build the model and 20% as "new" data to make predictions off of, and see how accurate the model predicts new observations.

<a name="2"></a>
## 2. Obtaining and cleaning the data

* **Description of data source**

For this analysis, I will be using data supplied by the Kaggle user [Miri Choi](https://www.kaggle.com/mirichoi0218) called _Medical Cost Personal Datasets_. This dataset can be found [on Kaggle](https://www.kaggle.com/mirichoi0218/insurance/), or in my [repository](https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/insurance.csv).

This data was obtained in 2018, but as I am aiming to test out my LRM creation skills, the data does not have to be timely. The data does however contain enough attributes, inluding the amount charged, per patient, and is thus comprehensive and relevant. Thus I consider the data to have integrity, for my purposes. I would have liked to see the severity of the affliction, or a comparision between insurers, but this dataset is enough to just test the waters with.

The data is considered [open data](https://opendatacommons.org/licenses/dbcl/1-0/).

* **Loading the data**

I load the data into RStudio as follows:

```
insurance_df = read.csv(file = "C:\\Users\\wicka\\Desktop\\Insurance LRM project\\insurance.csv", header = T)
```

This creates a data frame. We can explore the composition of the columns as follows:

```
> str(insurance_df)
'data.frame':	1337 obs. of  7 variables:
 $ age     : int  19 18 28 33 32 31 46 37 37 60 ...
 $ sex     : chr  "female" "male" "male" "male" ...
 $ bmi     : num  27.9 33.8 33 22.7 28.9 ...
 $ children: int  0 1 3 0 0 0 1 3 2 0 ...
 $ smoker  : chr  "yes" "no" "no" "no" ...
 $ region  : chr  "southwest" "southeast" "southeast" "northwest" ...
 $ charges : num  16885 1726 4449 21984 3867 ...
```

The data types of all of the columns check out, thus we do not have to change anything there. Since they have a small number of discrete levels, I note that _sex_, _children_, _smoker_ and _region_ will have to be dealt with as qualitative variables.

* **Cleaning the data**

First, we can check the qualitative variables to see if any of the pre-defined levels have been entered incorrectly:

```
> unique(insurance_df$sex)
[1] "female" "male"  
> unique(insurance_df$children)
[1] 0 1 3 2 5 4
> unique(insurance_df$smoker)
[1] "yes" "no" 
> unique(insurance_df$region)
[1] "southwest" "southeast" "northwest"
[4] "northeast"
```

Everything seems to be in order. Now we can check to see if any of the values for the continuous quantitative variables are _null_/_not applicable_:

```
> which(as.vector(is.na(insurance_df$age)) == TRUE)
integer(0)
> which(as.vector(is.na(insurance_df$charges)) == TRUE)
integer(0)
> which(as.vector(is.na(insurance_df$bmi)) == TRUE)
integer(0)
```

I entered the following just to test if this line of code worked, and it outputted the every possible index:

```
which(as.vector(is.na(insurance_df$age)) == FALSE) # just to see if logic checks out
```

Thus there are no _null_ values in either the qualitative or the quantitative variables. Next, we check for duplicates:

```
> which(duplicated(insurance_df) == TRUE)
[1] 582
> insurance_df[582,]
    age  sex   bmi children smoker    region  charges
582  19 male 30.59        0     no northwest 1639.563
```

And indeed there seems to be a duplicated row. Let's see which row it is a duplicate of:

```
> which(insurance_df[,1] == insurance_df[582,1] & insurance_df[,2] == insurance_df[582,2] & insurance_df[,3] == insurance_df[582,3] & insurance_df[,4] == insurance_df[582,4] & insurance_df[,5] == insurance_df[582,5] & insurance_df[,6] == insurance_df[582,6] & insurance_df[,7] == insurance_df[582,7])
[1] 196 582
> insurance_df[196,]
    age  sex   bmi children smoker    region  charges
196  19 male 30.59        0     no northwest 1639.563
```

So now we can delete row 582 as being a duplicate, since it is unlikely that 2 people with exactly the same age, sex, BMI, children, smoker- and region-classification would pay the exact same amount, since that would mean that these two equitable people suffered the same afflication in a short amount of time. First we check how many entries are in the data frame, then delete the duplicate, look at the new values in that row compared with row 196 and then check how many entries we have now:

```
> insurance_df = insurance_df[-582,]
> insurance_df[196,]
    age  sex   bmi children smoker    region  charges
196  19 male 30.59        0     no northwest 1639.563
> insurance_df[582,]
    age  sex   bmi children smoker    region  charges
583  39 male 45.43        2     no southeast 6356.271
> nrow(insurance_df)
[1] 1337
```

Now our data can be considered clean, and we can proceed to working with it.

* **Preliminary exploration**

Let's look at the relationships between the various attributes and the insurance charge, just for interest's sake, and to get an idea of what is going on in the data. This will not be used to make decisions off of, since the data has not been split yet, but just so that I can get a feel for what lies in the dataset.

```
par(mfcol = c(2,3))
plot(insurance_df$age, insurance_df$charges, main = "Age vs Charges", ylab = "Charges", xlab = "Age")
plot(factor(insurance_df$sex), insurance_df$charges, main = "Sex vs Charges", col = c("pink", "lightblue"), ylab = "Charges", xlab = "Sex")
plot(insurance_df$bmi, insurance_df$charges, main = "BMI vs Charges", ylab = "Charges", xlab = "BMI")
plot(factor(insurance_df$children), insurance_df$charges, main = "Children vs Charges", ylab = "Charges", xlab = "Number of children", col = rainbow(5))
plot(factor(insurance_df$smoker), insurance_df$charges, main = "Smoker vs Charges", col = c("green","yellow"), ylab = "Charges", xlab = "Smoker?")
plot(factor(insurance_df$region), insurance_df$charges, main = "region vs Charges", col = c("red","blue","green","yellow"), names = c("NE", "NW", "SE", "SW"), ylab = "Charges", xlab = "Region")

```

<p align="center">
  <img width="725" src="https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/plot.png">
</p>

We can see that there is something interesting going on in the _age_ graph, and that the _smoker_ graph provides quite a big difference between the two categories. The other graphs are relatively uninteresting, but we will conduct more formal tests in our analysis.

* **Splitting the data 80/20**

Firstly, we have to define the sample size that we want to gather from the data frame. I am also setting a seed so that the results I obtain can be reproducible if the same seed is used on someone else's computer. The sample taken will then still be random, but always be the same sample for this data frame and seed.

```
> samplesize = floor(0.2*nrow(insurance_df))
> samplesize
[1] 267
> set.seed(123)
```

Thus I will be holding out 267 entries to test my data on. Now we can randomly sample  indices from 1 to the size of our data frame, picking 267 values. Then we use these indices to create two new data frames, one to be used for the model and one for testing.

```
sample.index = sample(1:nrow(insurance_df), size = samplesize)
data.model = insurance_df[-sample.index,]
data.testing = insurance_df[sample.index,]
```

To test that we assigned the right amount of variables to the right data frame, we can look at the length of each:

```
> nrow(data.model)
[1] 1070
> nrow(data.testing)
[1] 267
```

Great! Now we can start working on building a model with the data in _data.model_.

<a name="3"></a>
## 3. Creating the LRM

* **Predictors vs Response - any relationship?**

Let's use a graphical approach to see if there is any relationship in the sample data:

```
par(mfcol = c(2,3))
plot(data.model$age, data.model$charges, main = "Age vs Charges", ylab = "Charges", xlab = "Age")
plot(factor(data.model$sex), data.model$charges, main = "Sex vs Charges", col = c("pink", "lightblue"), ylab = "Charges", xlab = "Sex")
plot(data.model$bmi, data.model$charges, main = "BMI vs Charges", ylab = "Charges", xlab = "BMI")
plot(factor(data.model$children), data.model$charges, main = "Children vs Charges", ylab = "Charges", xlab = "Number of children", col = rainbow(5))
plot(factor(data.model$smoker), data.model$charges, main = "Smoker vs Charges", col = rainbow(2), ylab = "Charges", xlab = "Smoker?")
plot(factor(data.model$region), data.model$charges, main = "Region vs Charges", col = rainbow(4), names = c("NE", "NW", "SE", "SW"), ylab = "Charges", xlab = "Region")
```

<p align="center">
  <img width="725" src="https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/sampleplot.png">
</p>

I will thus have to explore the _age_ attribute more thoroughly. At this stage, I am reasonably certain that your sex, children and region will not have a large effect, but we will see. Being a smoker, on the other hand, is expected to have an effect on your charges. There is also something going on with the BMI, where it looks like we have one group staying close to the bottom of the graph, and another group that tends higher as BMI increases.

Now we check mathematically by calculating the correlation:

```
> cor(insurance_df[,c(1,3,4,7)])
                age        bmi   children    charges
age      1.00000000 0.10934361 0.04153621 0.29830821
bmi      0.10934361 1.00000000 0.01275466 0.19840083
children 0.04153621 0.01275466 1.00000000 0.06738935
charges  0.29830821 0.19840083 0.06738935 1.00000000
```

Age and BMI has a slight positive correlation with charges, and children can be seen as insignificant. The reason for age and BMI not having higher correlations can be due to the interactions present - evident in the "three strokes" in age and the "lower and upper clouds" in BMI. Let's explore that further.

* **Checking for interactions and multicollinearity**

<ins> Interactions between predictors </ins>

Let's look at the interaction between age with the qualitative variables, as well as BMI:

```
par(mfcol = c(2,2))
interaction.plot(data.model$age, data.model$smoker, data.model$charges, col = rainbow(2))
interaction.plot(data.model$age, data.model$sex, data.model$charges, col = rainbow(2))
interaction.plot(data.model$age, data.model$children, data.model$charges, col = rainbow(6))
interaction.plot(data.model$age, data.model$region, data.model$charges, col = rainbow(4))
```
<p align="center">
  <img width="525" src="https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/ageint.png">
</p>

```
par(mfcol = c(2,2))
interaction.plot(data.model$bmi, data.model$smoker, data.model$charges, col = rainbow(2))
interaction.plot(data.model$bmi, data.model$sex, data.model$charges, col = rainbow(2))
interaction.plot(data.model$bmi, data.model$children, data.model$charges, col = rainbow(6))
interaction.plot(data.model$bmi, data.model$region, data.model$charges, col = rainbow(4))
```
<p align="center">
  <img width="525" src="https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/bmiint.png">
</p>

From these collection of graphs, we can see that there is mostly no interaction, except with the _smoker_ attribute. Let's compare our scatterplots with the interaction plots:

```
par(mfcol = c(1,2))
plot(data.model$age, data.model$charges, main = "Age vs Charges", ylab = "Charges", xlab = "Age")
interaction.plot(data.model$age, data.model$smoker, data.model$charges, col = rainbow(2))
```

<p align="center">
  <img width="525" src="https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/compareage.png">
</p>

```
par(mfcol = c(1,2))
plot(data.model$bmi, data.model$charges, main = "BMI vs Charges", ylab = "Charges", xlab = "BMI")
interaction.plot(data.model$bmi, data.model$smoker, data.model$charges, col = rainbow(2))

```

<p align="center">
  <img width="525" src="https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/comparebmi.png">
</p>

So the upper two "strokes" of data in the age scatterplot can be attributed to smokers, and the lower to non-smokers. Similarly, the lower "cloud" of data in BMI are non-smokers, and the higher is smokers. So there is some explanation for the trends in age and BMI. But what these two comparisions fail to solve, is the fact that the smoker data still varies so wildly in the interaction plot, to such a degree that it causes smokers to split into two "strokes" in the age scatterplot. Similarly, looking at the BMI scatterplot, the upper "cloud" seems to be segmented in two - a glance at the BMI interaction plot confirms this notion, that for smokers, BMI is segmented.

If we look at the BMI/smoker graphs, we see that this segmentation is seperated at about the 30 mark. Let's create a new vector of data that is TRUE if a person's BMI is equal to or lower than 30, and FALSE if higher. Having a high or low BMI if you are a smoker might then have a correlation with charges.

```
lowbmi = (data.model$bmi <= 30)
par(mfcol = c(1,2))
interaction.plot(data.model$age, lowbmi, data.model$charges, col = rainbow(2))
interaction.plot(lowbmi, data.model$smoker, data.model$charges, col = rainbow(2))
```

<p align="center">
  <img width="525" src="https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/lowbmi.png">
</p>

Indeed we can see on the left that the FALSE group, where the BMI is above 30, has greater fluctuations into higher charges thatn those with low BMI. On the right we also see that if you are a non-smoker, your charge is not influenced greatly by your BMI, wheares for smokers, the mean charge of people with higher BMIs are much higher.

To further see the combined effect of smoking and BMI, let's create a new vector that measures whether a person smokes and has a BMI lower or equal to 30. If we draw the interaction plot of that with age, with the effect on charges, it all becomes more clear:

```
lowbmiandsmoke = paste(data.model$smoker, as.character(data.model$bmi <= 30))
interaction.plot(data.model$age, lowbmiandsmoke, data.model$charges, col = c("red","blue","purple","green"))
```

<p align="center">
  <img width="525" src="https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/combined.png">
</p>

> Where:
> **no TRUE** = non-smokers with a low BMI
> **no FALSE** = non-smokers with a high BMI
> **yes TRUE** = smokers with a low BMI
> **yes TRUE** = smokers with a high BMI


This explains the three "strokes" in the age data perfectly! The lower "stroke" is all the non-smokers, whereas the upper "strokes" are the smokers with differing BMIs.

This means that we should add a new column in our data frame, that classifies whether a person has a BMI lower than 30, or not.

```
data.model$lowbmi = data.model$bmi <= 30
plot(factor(data.model$lowbmi), data.model$charges, main = "BMI classification vs Charges", col = c("purple","green"), ylab = "Charges", xlab = "BMI Classification (TRUE = 30 or lower, FALSE = 31 or higher)")
```

<p align="center">
  <img width="525" src="https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/lowbmibox.png">
</p>


<ins> Multicollinearity </ins>

* **Reduction of predictor variables**

* **Model refinement and selection**

* **Automatic selection procedures in R**

* **Final model selection based on objective measures**

<a name="4"></a>
## 4. Validation and prediction

* **Model selection measures - why the model chosen is the best possible model**

* **Testing the model using cross-validation**
