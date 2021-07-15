# Predicting health insurance costs using linear regression in R

## Table of Contents

1. [Scenario explanation](#1)
2. [Obtaining and cleaning the data](#2)
3. [Creating the LRM](#3)
4. [Prediction and visualisation](#4)

<a name="1"></a>
## 1. Scenario explanation

After completing a module on linear regression at university, I wanted to test out the theory learned, and see if I can use my knowledge to make predictions given raw data. I will thus be creating the linear regression model by using the linear algebra and statistics that I studied during the semester, and then verify my findings using the linear regression functions in R.

The goal of this project is to test the waters to see if I can create a model that reasonably predicts medical costs billed by health insurers, given a few attributes of the patient. I will firstly clean the data, because dirty data won't provide good predictions. Then I will proceed to create the LRM, with a process that will be discussed in [section 3](#3).

After this, I will hopefully have constructed a useful model, which I can then use to test on new data.

</ins> How will I validate my model? <\ins>

I will be splitting the data in two - cross-validation.

<a name="2"></a>
## 2. Obtaining and cleaning the data

* **Description of data source**

For this analysis, I will be using data supplied by the Kaggle user [Miri Choi](https://www.kaggle.com/mirichoi0218) called _Medical Cost Personal Datasets_. This dataset can be found [on Kaggle](https://www.kaggle.com/mirichoi0218/insurance/), or in my [repository](https://github.com/nuclearcheesecake/insuranceregression/blob/main/misc/insurance.csv).

This data was obtained in 2018, but as I am aiming to test out my LRM creation skills, the data does not have to be timely.

The data is considered [open data](https://opendatacommons.org/licenses/dbcl/1-0/).

* **Loading and exploring the data**


* **Cleaning the data**



<a name="3"></a>
## 3. Creating the LRM

<a name="4"></a>
## 4. Prediction and visualisation
