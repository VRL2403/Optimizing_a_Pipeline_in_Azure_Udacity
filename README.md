# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.
In this project pipeline was build using HyperDrive and AutoML and performance of both the approach was compared.

---

## Summary

**Data Set Information:**

The data is related with direct marketing campaigns of a Portuguese banking institution. The marketing campaigns were based on phone calls. Often, more than one contact to the same client was required, in order to access if the product (bank term deposit) would be ('yes') or not ('no') subscribed.

The classification goal is to predict if the client will subscribe (yes/no) a term deposit (variable y).

### Attribute Information:

Input variables:
#### bank client data:
1. age (numeric)
2. job : type of job (categorical: 'admin.','blue-collar','entrepreneur','housemaid','management','retired','self-employed','services','student','technician','unemployed','unknown')
3. marital : marital status (categorical: 'divorced','married','single','unknown'; note: 'divorced' means divorced or widowed)
4. education (categorical: 'basic.4y','basic.6y','basic.9y','high.school','illiterate','professional.course','university.degree','unknown')
5. default: has credit in default? (categorical: 'no','yes','unknown')
6. housing: has housing loan? (categorical: 'no','yes','unknown')
7. loan: has personal loan? (categorical: 'no','yes','unknown')
#### related with the last contact of the current campaign:
8. contact: contact communication type (categorical: 'cellular','telephone')
9. month: last contact month of year (categorical: 'jan', 'feb', 'mar', ..., 'nov', 'dec')
10. day_of_week: last contact day of the week (categorical: 'mon','tue','wed','thu','fri')
11. duration: last contact duration, in seconds (numeric). Important note: this attribute highly affects the output target (e.g., if duration=0 then y='no'). Yet, the duration is not known before a call is performed. Also, after the end of the call y is obviously known. Thus, this input should only be included for benchmark purposes and should be discarded if the intention is to have a realistic predictive model.
#### other attributes:
12. campaign: number of contacts performed during this campaign and for this client (numeric, includes last contact)
13. pdays: number of days that passed by after the client was last contacted from a previous campaign (numeric; 999 means client was not previously contacted)
14. previous: number of contacts performed before this campaign and for this client (numeric)
15. poutcome: outcome of the previous marketing campaign (categorical: 'failure','nonexistent','success')
#### social and economic context attributes
16. emp.var.rate: employment variation rate - quarterly indicator (numeric)
17. cons.price.idx: consumer price index - monthly indicator (numeric)
18. cons.conf.idx: consumer confidence index - monthly indicator (numeric)
19. euribor3m: euribor 3 month rate - daily indicator (numeric)
20. nr.employed: number of employees - quarterly indicator (numeric)

#### Output variable (desired target):
21. y - has the client subscribed a term deposit? (binary: 'yes','no')
    
---

**The best performing model**

There were two approach taken to solve the problem by creating pipelines: 
- Using a LogisticRegression model with scikit-learn, and tuning the model's hyperparameters using Azure's HyperDrive. 
- Using Azure AutoML for training several kinds of models such as LightGBM, XGBoost, Logistic Regression, VotingEnsemble, among others algorithms and find the best one based on accuracy. 

*The best performing model was VotingEnsemble with accuracy of 0.9172 which was build by second approach i.e. Azure's AutoML.*

---

## Scikit-learn Pipeline
**Pipeline architecture, including data, hyperparameter tuning, and classification algorithm.**

- Steps involved in the entry script(train.py):
1. Importing data using URL using TabularDatasetFactory.
[Link for dataset](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv) 
2. Cleaning the data - dropping columns, removing rows with missing entries, one hot encoding the categorical data, feature engineering etc.
3. Splitting the data into train and test sets.
4. Training the logistic regression model using arguments from the HyperDrive runs.
5. Calculating the accuracy score.
6. Creating best run model output file.

- Steps involved in the project notebook (udacity-project.ipynb):
1. Assigning a compute cluster to be used as the target.
2. Specifying the parameter sampler (RandomSampling).
3. Specifying an Early Stopping Policy (BanditPolicy).
4. Creating a SKLearn Estimator.
5. Creating a HyperDriveConfig using the estimator, hyperparameter sampler, and policy.
6. Getting the best run model and its metrics.
7. Saving the model.
   
![HyperDrive Runs](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-Run.PNG)

![HyperDrive Child Runs](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-Child-Runs.PNG)

![HyperDrive Child Runs](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-Child-Runs-Primary-Metrics.PNG)

**Benefits of parameter sampler**

- Parameter sampler is used to define the different hyperparameters as well as different range of values of that hyperparameters that one wants to tune the model traing process.
- In this project, Discrete search sampler was used for both hyperparemeters (C and max_iter) along with Random sampling method. 
- In random sampling, hyperparameter values are randomly selected from the defined search space.
- In case of hyperparameters, C represents the inverse regularization parameter and max_iter represents the maximum number of iterations.


**Benefits of Early Stopping Policy**

- Early Stopping policy is used to prevent model training process from running for a long time and consuming up resources.
- Early stopping policy used in this project is ***BanditPolicy***. 
- The benefit of using this early stopping policy is that it terminates training process as soon as it finds that the primary metric (accuray in this project) is not within the pre-defined threshold.

---

## AutoML

![Auto ML run](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-Auto-ML.PNG)

- Azure AutoML is a no code environment, capable of training many different models in a short period of time like RandomForests, BoostedTrees, XGBoost, LightGBM, SGDClassifier, VotingEnsemble, etc.
- Following config parameters are set for AutoML:
  * experiment_timeout_minutes=30 : By default set in simulator for reducing excessive use of computational resources. 
  * task='classification' : As the problem was classification based to check *the client subscribed a term deposit or not*.
  * compute_target : The compute target with specific compute i.e. "Standard_D2_V2" and max_nodes.
  * training_data : The data on which the algorithm will be trained.
  * label_column_name : The name of the column that used for prediction (target column).
  * n_cross_validations=3 : It is how many cross validations to perform when user validation data is not specified.
  * primary_metric = 'accuracy' : The primary metric for optimizing the model selection.
  * enable_early_stopping = True : Parameter to enable early termination policy.

*The best performing model was VotingEnsemble with accuracy of 0.9172*

![Auto ML Best Run](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-Auto-ML-Best-Model.PNG)

---

## Explanations

![Auto ML Data Exploration](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-Data-Exploration.PNG)

![Auto ML Data Importance](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-Data-Importance.PNG)

![Auto ML Data Importance](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-Summary_importance.PNG)

---

## Performace Metrics

![Auto ML Performace Metrics](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-AutoML-Metrics-1.PNG)

![Auto ML Performace Metrics](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-AutoML-Metrics-2.PNG)

---

## Pipeline comparison

- Accuracy acheived using PythonSDK HyperDrive is 0.9096611026808296 whereas incase of Azure AutoML is 0.9173.
- There is much difference in accuracy of AutoML and HyperDrive but the advantage of AutoML is one doesn't require to code much and using little efforts one can train various models and also get its performance metrics and visualizations.  
- And as compared to HyperDrive, AutoML architecture is quite superior, which enables to training 'n' number of models quitely and efficiently.

---

## Future work

1. Handling implance data. 
2. Bayesian Parameter Sampler might improve model accuracy.
3. Implementing neural network or trees to implement model using HyperDrive. 

---

## Proof of cluster clean up

- Following is proof for custer clean up


![Cluster Clean Up Proof](https://github.com/VRL2403/Optimizing_a_Pipeline_in_Azure_Udacity/blob/master/Images/Udacity-Project-Cluster-CleanUp-Proof.PNG)
