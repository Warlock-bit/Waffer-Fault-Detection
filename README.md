# Waffer Fault Detection: 

## Table of Content  
  * [Problem Statement](#Problem-Statement)
  * [Data Description](#Data-Description)
  * [Data Validation](#Data-Validation)
  * [Data Insertion in Database](#Data-Insertion-in-Database)
  * [Model Training](#Model-Training)
  * [Prediction Data Description](#Prediction-Data-Description)
  * [Prediction Data Validation](#Prediction-Data-Validation)
  * [Prediction](#Prediction)
  * [Installation](#Installation)
  * [Technologies & Tools Used](#Technologies-&-Tools-Used)

## Problem Statement
The inputs of various sensors for different wafers have been provided. In electronics, a wafer (also called a slice or substrate) is a thin slice of semiconductor used for the fabrication of integrated circuits. The goal is to build a machine learning model which predicts whether a wafer needs to be replaced or not(i.e., whether it is working or not) based on the inputs from various sensors.

## Data Description
The client will send data in multiple sets of files captured from sensors in batches at a given location. Data will contain 590 columns: There are two classes: +1 and -1. 
1. +1 means that the wafer is in a working condition and it doesn’t need to be replaced.
1. -1 means that the wafer is faulty and it needs to be replaced. 


| Column Name | Description|
| ----------- |:-------------|
|Wafer | Random ID for the product|
|Sensor - 1 to 590|Data Captured from various sensors|
|Output|This is the target value.|

Apart from training files, we also require a "schema" file from the client, which contains all the relevant information about the training files such as:
Name of the files, Length of Date value in FileName, Length of Time value in FileName, Number of Columns, Name of the Columns, and their datatype.

## Data Validation 
In this step, we perform different sets of validation on the given set of training files. 
 
1. Name Validation- We validate the name of the files based on the given name in the schema file. We have created a regex pattern as per the name given in the schema file to use for validation. After validating the pattern in the name, we check for the length of date in the file name as well as the length of time in the file name. If all the values are as per requirement, we move such files to "Good_Data_Folder" else we move such files to "Bad_Data_Folder."
1. Number of Columns - We validate the number of columns present in the files, and if it doesn't match with the value given in the schema file, then the file is moved to "Bad_Data_Folder."
1. Name of Columns - The name of the columns is validated and should be the same as given in the schema file. If not, then the file is moved to "Bad_Data_Folder".
1. The datatype of columns - The datatype of columns is given in the schema file. This is validated when we insert the files into Database. If the datatype is wrong, then the file is moved to "Bad_Data_Folder".
1. Null values in columns - If any of the columns in a file have all the values as NULL or missing, we discard such a file and move it to "Bad_Data_Folder".

## Data Insertion in Database 
1. Database Creation and connection - Create a database with the given name passed. If the database is already created, open the connection to the database. 
1. Table creation in the database - Table with name - "Good_Data", is created in the database for inserting the files in the "Good_Data_Folder" based on given column names and datatype in the schema file. If the table is already present, then the new table is not created and new files are inserted in the already present table as we want training to be done on new as well as old training files.     
1. Insertion of files in the table - All the files in the "Good_Data_Folder" are inserted in the above-created table. If any file has invalid data type in any of the columns, the file is not loaded in the table and is moved to "Bad_Data_Folder".

## Model Training 
1. Data Export from Db - The data in a stored database is exported as a CSV file to be used for model training.
1. Data Preprocessing   
   1. Check for null values in the columns. If present, drop the null values.
   1. Check if any column has zero standard deviation, remove such columns as they don't give any information during model training.
   1. Apply scaling and PCA in the columns , remove multi collinearity.
1. Model Selection - After clusters are created, we find the best model for each cluster. We are using two algorithms, `"Random Forest" and "XGBoost"`. For each cluster, both the algorithms are passed with the best parameters derived from GridSearch. We calculate the AUC scores for both models and select the model with the best score. Similarly, the model is selected for each cluster. All the models for every cluster are saved for use in prediction.

## Prediction Data Description 
Client will send the data in multiple set of files in batches at a given location.Data will contain 23 columns and we have to predict whether an order will become back order or not. 
Apart from prediction files, we also require a "schema" file from client which contains all the relevant information about the training files such as:
Name of the files, Length of Date value in FileName, Length of Time value in FileName, Number of Columns, Name of the Columns and their datatype.

## Prediction Data Validation  
In this step, we perform different sets of validation on the given set of training files.  
1. Name Validation- We validate the name of the files on the basis of given Name in the schema file. We have created a regex pattern as per the name given in schema file, to use for validation. After validating the pattern in the name, we check for length of date in the file name as well as length of time in the file name. If all the values are as per requirement, we move such files to "Good_Data_Folder" else we move such files to "Bad_Data_Folder". 
1. Number of Columns - We validate the number of columns present in the files, if it doesn't match with the value given in the schema file then the file is moved to "Bad_Data_Folder". 
1. Name of Columns - The name of the columns is validated and should be same as given in the schema file. If not, then the file is moved to "Bad_Data_Folder". 
1. Datatype of columns - The datatype of columns is given in the schema file. This is validated when we insert the files into Database. If dataype is wrong then the file is moved to "Bad_Data_Folder". 
1. Null values in columns - If any of the columns in a file has all the values as NULL or missing, we discard such file and move it to "Bad_Data_Folder".  

## Data Insertion in Database 
1. Database Creation and connection - Create database with the given name passed. If the database is already created, open the connection to the database. 
1. Table creation in the database - Table with name - "Good_Data", is created in the database for inserting the files in the "Good_Data_Folder" on the basis of given column names and datatype in the schema file. If table is already present then new table is not created, and new files are inserted the already present table as we want training to be done on new as well old training files.     
1. Insertion of files in the table - All the files in the "Good_Data_Folder" are inserted in the above-created table. If any file has invalid data type in any of the columns, the file is not loaded in the table and is moved to "Bad_Data_Folder".

## Prediction  
1. Data Export from Db - The data in the stored database is exported as a CSV file to be used for prediction.
1. Data Preprocessing    
   1. Check for null values in the columns. If present, drop the null values.
   1. Check if any column has zero standard deviation, remove such columns as we did in training.
1. Prediction - model is loaded and is used to prediction.
1. Once the prediction is made, the predictions are saved in a CSV file at a given location and the location is returned to the client.

## Installation
The Code is written in Python 3.7.5. If you don't have Python installed you can find it [here](https://www.python.org/downloads/). If you are using a lower version of Python you can upgrade using the pip package, ensuring you have the latest version of pip. To install the required packages and libraries, run this command in the project directory after [cloning](https://www.howtogeek.com/451360/how-to-clone-a-github-repository/) the repository:
```bash
pip install -r requirements.txt
```
## Technologies & Tools Used
1. Pandas
1. SKlearn
1. Hyperopt
1. Imbalanced-learn
1. Xgboost
1. Postman
1. Autoimpute
1. Kneed
1. Flask
