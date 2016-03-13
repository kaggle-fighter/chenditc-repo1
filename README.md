# BNP kaggle competition actvity log

#### Competition site: https://www.kaggle.com/c/bnp-paribas-cardif-claims-management


#### 2016/02/27:
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
   
#### 2016/02/28:
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
     - Simple hot deck: Randomly select one from existing fields. This preserve the mean and standard deviation.
     - Linear regression: Use the estimate value based on linear regresion over other fields.
 2. Tested on Azure Machine Learning:
   1. Use （ mean / 0 / -1 ) to replace numerical missing value
   2. Use NA to replace categorical missing value
   3. Use Boosted Decision Tree to make prediction ( Tried more leaves per tree, tried more sample per tree )
   
   Use 0 or -1 yield better result. Tuning decision tree hyperparam doesn't improve overall performance.

#### 2016/02/29:
 1. Tried boosted decision tree with 1000 node. The log loss is averaged around 0.50, which is even worse than the default setting (0.47278)
 2. Tried SVM and depp SVM model:
   - SVM has log loss around 0.50 - 0.51, which does not perform better than decision tree.
   - Deep SVM has log loss around 0.51 - 0.53, the log loss vary a lot during 10 fold cross validation. This shows Deep SVM is overfitting the result and does not derive more information.
 3. Kicked off neural network:
   - 100 node in hidden layer. Train with 100 epoch. 10 fold validation. The mean error is still decreasing after 90 epochs, so kick off another job.
   - 100 node in hidden layer. Train with 300 epoch. Generate test result directly.
 
 Conclusion:
  - There might be some magic value that make big difference of some fields, which makes decision tree perform better.
  - The SVM model is restricted by the kernel it use, which might make it hard to accomodate some corner case.
  - Kicked off neural network 

#### 2016/03/01:
 1. Tested neural network performance.
   - 100 node in hidden layer produce log score 1.2
   - 300 node in hidden layer produce log score 1.9
   
   Validated the produced lable and probability in traning set. The prediction error is quite low. The poor performance on test set means this model is overfitting the training set too much.

   Another explaination is the input data format. I didn't normalize the input data, which might cause skew in some way.
   
   Forum discussion suggest it can achieve Leader Board score 0.46 using NN, so might need to dig further.
 2. Forum discussion thread:
   - Replace missing value with out of range value (-1 / -999) improved result. Espcially for tree based algorithm.
   - **Add additional column to indicate missing value can help.**
   - **How to distinguish "Data is missing" and "Data is not applicable"?**
   - **drop v22 seems to help, need to understand why**
   
#### 2016/03/13:
 1. Forum discussion thread: https://www.kaggle.com/c/bnp-paribas-cardif-claims-management/forums/t/19240/analysis-of-duplicate-variables-correlated-variables-large-post
   - Direct conclusion:
     - v91 and v107 are duplicated variable. v71 and v75 are highly correlated. v47 and v110 almost duplicates.
       - This might not improve the overall performance, but in production stream analysis, this can save memory / cpu resource.
     - v31's missing value share same distribution as v31's B.
       - We can try to impute v31's missing value with B.
     - Numerical variable with large correlation:
       - Chained correlation can be discovered and may be able to explain some correlations.
     - Missing value pattern are found to have approximately 50 groups:
       - https://kaggle2.blob.core.windows.net/forum-message-attachments/111103/3873/Missing%20Values.htm?sv=2012-02-12&se=2016-03-16T20%3A00%3A56Z&sr=b&sp=r&sig=q%2F%2FbrrtE6CRPw80a2quyFtbgMH4wj3alU2%2BFLGW7BcA%3D
   - Futher reading and future work:
     - Try to replace some correlated value with the residual to "first-level variables".
     - CHAID model (https://en.wikipedia.org/wiki/CHAID):
       - Used for detection of interaction between variables
     - U-test
     - Add one categorical feature which represents which missing pattern is it.
       - Eg. Non-NA, NA1, NA2 ... NA50
       - This might catch the different type of individual from different data source.
