# Machine Learning

Table of Contents
[Amazon Machine Learning (AML) vs. Amazon SageMaker](#amazon-machine-learning-aml-vs-amazon-sagemaker)

## Amazon Machine Learning (AML) vs. Amazon SageMaker

1. SageMaker is a platform used to build, train and deploy machine learning models at scale.
   1. Manage entire end-to-end process
   2. Include most common ML algorithms and frameworks and optimize them to deliver performance.
   3. Dive into more complicated models (with Jupyter Notebooks with different algorithms to change different models).
   4. Connect directly to S3 data.
   5. Use AWS Glue to move data from RDS, DynamoDB, Redshift into S3.

2. AML has more limitation
   1. Limited tooling behind it
   2. Data retrieval:  S3, Redshift, RDS
   3. More limitation on supported models - only
      1. Binary classification
      2. Multiclass classification
      3. Regression

## Amazon Machine Learning AML vs. Mahout vs. Spark/SparkMLlib

1. AML is limited to 100 categorical/multiclass recommendations.
1. AML supports only supervised learning - AML requires data to be labelled.


