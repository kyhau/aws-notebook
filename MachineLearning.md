# Machine Learning

Table of Contents
[Default Limits](#default-limits)
[Pricing](#pricing)


## Default Limits
- Max observation size (target+attributes): 100KB
- Max training data size: 100GB
- Max batch predictions data size: 1TB
- Max batch predictions data records: 100 million
- Max columns in schema: 1000
- Real-time prediction endpoint TPS: 200
- Number of classes for multiclass ML models: 100


## Concepts
Feature
A feature is an individual measurable property of a phenomenon being observed.
Features describe the event that needs to be modelled.
Usually numeric.
Generally more features lead to better predictions
Example when predicting the high temperature for a day: day of the year, pressure, cloud cover, distance to the sun, average sea temperature
Best features are ones that are closely related to the outcome.
- Two main types of machine learning:
Supervised learning
When the historical data is labeled
Labeled data has both features and the outcome that occurred
Example: Emails labeled as spam; contained words "please send money"
Un-supervised learning
When the historical data is not labeled
Data only has features
The machine learning algorithms have to the relationships in the data
Example: Customer demographics with the goal for finding cluster of customers with similar buying habits 
Two main steps:

## Core Concepts
Amazon ML requires the data to be labeled
Dataset has to have both the features and target for each observation/record
A feature is an attribute of a record use to identify patterns; typically there will be multiple features.
A target is the outcome that the patterns are linked to and is the value the ML algorithm is going to predict. 

## Model Types
- Based on the target column data type, Amazon ML recommends the best algorithm to use.
- Regression Model
    - The target/prediction is a numeric value
    - Best used for predicting scores
    - Example: traffic delays - (probability; a number)
    - Evaluate
        - Root Mean Square Error(RMSE)
        - Amazon ML takes the mean of the training target data (RMSE Baseline) and uses that as a baseline and compares
          it to the mean of the predictions (RMSE)
        - A RMSE lower than the RMSE Baseline is better
- Binary Model
    - The target/prediction value is a 0 or 1
    - Best used when the prediction is a Boolean or one of two possible outcomes
    - Example:  Does an email match the spam criteria? 
    - Evaluate
        - Area Under the Curve (AUC)
        - True Positives, True Negatives, False Positives and False Negatives
        - Only model that can be fine tuned by adjusting the score threshold
- Multi-Class Model
    - The target/prediction is from a set of values (categorical)
    - Best used for predicting categories or types
    - Example:  What is the next thing a user will buy based on the history?
    - Evaluate
        - F1 Score
        - Color coded confusion matrix
        - Blues are correct predictions and reds are incorrect predictions

## Three simple steps:
Train
If using the Amazon ML defaults, 70 percent of the dataset is used to train the model.
Machine learning data rule: garbage in, garbage out
Normalize your data
Decide on the Amazon ML model to be used based on the dataset
Supported data sources:
S3 – CSV files
Redshift – SQL Query
RDS – SQL Query
Data Schema
Identify the data schema - the data layout and column data types
Supported data types:
Numeric: any numerical value
Binary: 0/1, yes/no, y/n, true/false, t/f
Categorical: A list of unique string values
Text: Stings, words, long-text
Identify the features and the target
Evaluate
Overall success metric of the model
Visualizations to explore accuracy of model
Alerts to check validity of evaluation
If using the Amazon ML defaults, 30 percent of the dataset is used to evaluate the model
Predict
Batch predictions
Asynchronously generate predictions for multiple input data observations
Large volumes
API or web console
Real-time predictions
Synchronously generate predictions for individual data observations
High throughput, single predictions
API or mobile SDKs


## Use Cases
- Recommendations when "Checking Out" on an e-commerce site
- Spam detection in your email
- Any kind of image, speech or text recognition
- Weather forecasts
- Search engines


## Benefits
- Allows users to create machine learning models based on historical information and generate predictions
- Fully managed service
- No code required for creating models
- Recommends the best ML algorithm to run based on the input data
- Easily integrates into other AWS services for data retrieval
- Deploy within minutes


## Pricing
- Per compute hour for data analysis, training and evaluation
- Predictions
    - Batch
    - Real-time plus endpoint reservation charge


