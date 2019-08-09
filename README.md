# charity_recommender

## Business Objective:
Pair donors to the classroom requests that will most motivate them to make an additional gift. Donors must have donated at least once. 

Context:
To date, 3 million people and partners have funded 1.1 million DonorsChoose.org projects. Teachers still spend more than a billion dollars of their own money on classroom materials.
In the second Kaggle Data Science for Good challenge, DonorsChoose.org invites the Kaggle community to help them pair up donors to the classroom requests that will most motivate donors to make an additional gift. 
To support this challenge, DonorsChoose.org has supplied anonymized data on donor giving from the past five years. The winning methods will be implemented in DonorsChoose.org email marketing campaigns.

### NYC Charity Recommender Objective:
A donor has a chance to choose a classroom project they would like to donate to from a list of projects. After selecting the project they would donate to, charity_recommender uses collaborative filtering to recommend 10 new projects the user may also want to donate to. 

## Data Acquisition:
This dataset started out with 4.6 million donations generated by 2 million donors, going towards 901,300 individual projects from the USA between 2012 and 2018. 
The graph below shows the top 10 states with the highest frequency of donations. 
Libraries: SurPRISE

![](https://github.com/Chris-Manna/charity_recommender/blob/master/top_ten_donating_states.png)

# Data Preparation

*Which was the highest city?*
New York, New York has the highest of donation frequency for cities. 
The advantage to localizing the donations dataset to New York City is addressing the coldstart when using collaborative filter recommender system is that most people donating choose to donate to projects they think they know about or are local. By trimming it to a major local area like New York City, this collaborative recommender system focuses on donors in New York City and the donations a New Yorker might want to donate to. More effectively deliver recommendations to donors located in the same area.

##### addressing cold start for recommender systems
+ Localize USA dataset to New York City
+ Total Amount Received over a 5 year period: $8,015,266.39
+ 76,257 donations Donations Received from NYC
+ 26,009 Donors
+ Most Recent Date Donation Received: 2018-05-09
+ Earliest Date Donation Received: 2012-10-27
+ The most common donor donated 2,152 times
+ Max Single Donation: $19,588.38
+ Min Single Donation: $0.21

![](https://github.com/Chris-Manna/charity_recommender/blob/master/donors%20hist.png)

The graph is limited on the X axis to $1,000 to show there is a distribution. The graph is positively skewed past $1,000,000.

### Measuring Donor Enthusiasm for Projects
Donation amount is shown on a logarithmic scale. See graph below:

![](https://github.com/Chris-Manna/charity_recommender/blob/master/log%20donation.png)
In the future I will need to substitute the logarithmic values for dollar values. 

For Frequency: The dataset contains many individual donors that donated one time, while it contains donors that have donated once per month over a multi-year span. Many individual donors donated $1 while others donated millions of dollars over time. While scaling these values in each dimension would be possible, the data collected may likely have skewed the results towards those that are already being donated to frequently and with higher donation quantity. 

##### Instead of scaling donation frequency and quantity
Each donor's choice to donate any amount and any number of times to a single project is converted to a binary value. 
If a user had donated any amount, and any amount of times to a project it would be converted to a 1 or a 0. By converting the donation values to binary, it meant this dataset had become an implicit feedback dataset. 

### Comparing Similarity Matrices on an Implicit Dataset
In this section I applied the SurPRISE library to compare each donors contribution history. The SurPRISE library uses the DonorID in the first column, ProjectID in the second column, and the implicit donation history in third column. By matching the Donor ID's and Project ID's and the donation history, SurPRISE creates similarity matrix full of binary values. With this matrix we can calculate their given donation history as a "distance" from another user's donation history. 

## Modeling and Evaluation
##### The intution behind using a collaborative filtering recommender system on an implicit dataset
![](http://datameetsmedia.com/wp-content/uploads/2018/05/2ebah6c-1.png)
http://datameetsmedia.com/an-overview-of-recommendation-systems/

When donor A and donor B have the exact same donation history, we can say that the distance between these two matrices is essentially 0. 
If B suddenly develops a preference for something new, A may also develop a preference for that new item as well. 
When B likes something new we are using the similarity matrix comparison to guess that A will also like the new item. 
Through a process called collaborative filtering, we can begin to project a possible pattern of donations in the future for donors that have not yet donated by comparing their history to another user's history. 
With collaborative filtering recommender systems A's preference matrix may match many other user preference matrices and we will want to offer A an array of options, called recommendations. 

By comparing the distance between donors A's donation history to donor B's donation history, we can fill the donor A's donation history in with B's donation history. Using B's filled out donor history as a reference for possible projects subsequent donations may be on a list of recommendations.  For example, if A and B both choose to eat pizza and ride a bike, then chances are if B likes sports drinks that A will also like sports drinks, so why not recommend A sports drinks as well? 

## Deploy
##### In the real world
Donor preferences do not always match exactly and so instead we want to use their similarity distances. By applying the SurPRISE library we don't have to do all the Matrix Factorization mathematics that goes into generating the distances between donor preference matrices and instead return a list of projects they may choose to donate to. 
 
In this project, five algorithms were used to test which recommendations would best suit the donor based on implicit donation history comparing to other donors. 
These five algorithms use the Grid Search Cross Validation (GridSearchCV) to compare Mean Absolute Error and Root Mean Squared Error between which yields the best error metrics and time to compute recommendations. 

Using the three algorithms with best results are listed here: Singular Value Decomposition for Implicit Feedback (SVD++), Singular Value Decomposition (SVD), and Non-Negative Matrix Factorization (NNMF). 

Used GridSearchCV on Matrix Factorization techniques to calculate error rates. It turns out SVD++ has the lowest Mean Absolute error rate. A boxplot visualization is below: 

![](https://github.com/Chris-Manna/charity_recommender/blob/master/Boxplot%20MAE.png)

We did not use the CoClustering or Normal Predictor. Because the matrix was so sparse, and the CoClustering Algorithm works best by comparing a more heavily populated matrix, the resulting recommendation error metrics didn't make sense. Because the implicit dataset is not normally distributed, the NormalPredictor algo didn't work well here either.

Error metrics don't quite work when using recommender systems because error metrics are used for predictions. Recommendations, are not exactly predictions. When you give someone a list of recommendations and they choose one of them, it does not necessarily mean they didn't like the other ones. They may like them all the same amount, they may not see the item or project they would have most wanted to donate to. The challenge in interpreting error metrics comes where the algorithms are predicting based on history to project into the future what a person may like, but without many more datapoints and feedback, a recommendation list will have to be tested in live trials. 

According to the Error metrics SVD++ yielded the best scores for error metrics results and it is most geared towards the implicit feedback DataSet. The NNMF is a close second in my opinion. The Google Colab GUI returns a list of 10 projects that a user may like to choose from next. It will keep track of the new users preferences and compare the new user DataSet to the rest of the New York Donor DataSet, too. 

## My next steps
- Use PySpark to aggregate all unique donation entries from all over the country. 
- Incorporating Meta Data from Donors and Projects to create a hybrid filter
- Compare Content Filter Recommender System against a Collaborative Recommender System that has more meta data
- Include a Time Series to see where donations are coming from, more expansive overview and EDA

