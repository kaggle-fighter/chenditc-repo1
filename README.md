# BNP kaggle competition actvity log

#### Competition site: https://www.kaggle.com/c/bnp-paribas-cardif-claims-management


#### 2016/02/17:
 1. Tested AWS Machine Learning with default arguments. Tested result stored in aws_result:
   - aws_result_1.csv ( Result set downloaded from aws )
   - sample_submission.csv ( Sample format )
   - merge_result.py ( Merge aws_result_1.csv and sameple_submission.csv, get a valid submission )
   - aws_submit_1.csv ( Submitted to kaggle )
     - Rank: 1169
     - Log loss score: 0.50312
   The result is almost same as "Random Forest Benchmark".
   
 2. Tested Azure Machine Learning with following pipeline. Since a lot of fields have more than 20% of missing value, a proper way to process them can make a lot of difference.
   1. Use PCA to replace numerical missing value (10 iteration)
   2. Use NA to replace categorical missing value
   3. Use Boosted Decision Tree to make prediction
   
   The result is better:
    - Rank: 980
    - Log loss score: 0.47278
   
#### 2016/02/18:
 1. Read Missing_Data_Our_View_of_the_State_of_art.pdf:
   1. Missing data might be missing for different reason. 
     - It might randomly missing due to the flaw during data collection. 
       - Eg. Survey identity not reachable after first round of survey. 
     - It might randomly missing due to the nature of the data. 
       - Eg. Field is "The marriage year", but survey person is too young.
     - It might missing with a probability correlate to specific property of the event. 
       - Eg. HIV patient don't want to disclose the sexual orientation. 
   2. Traditional way to recover missing data:
     - Delete missing row: Reduced the power of traning size.
     - Replace with average: preserved average, but disturb the covariance and the standard deviation. 
