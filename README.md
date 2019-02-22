# charity_recommender

# NYC Charity Recommender
Using a Google Colabs GUI, a donor chooses a classroom project they would like to donate to from a list of 48,296 New York based projects. After selecting the project they would donate to, the charity_recommender program uses collaborative filtering to recommend 10 projects they may also want to donate to. 

# Exploring and Scrubbing the DataSet
### Localizing USA dataset to New York City
This dataset started out with 24 million individual donations generated by 4.6 million individual donors, going towards 850,000 individual projects from all over the USA. Working with the 'Big Data' original dataset fell outside of the scope of the project so I decided to trim it down. 

In order to maintain integrity of the dataset while trimming the datapoints, I chose to keep donations going towards projects in New York City. There are advantages to localizing the donations dataset to New York City. Most people donating chose to donate to projects that were local anyway. By trimming it to a local area like New York City, our recommender_system was not missing that much in variation from donors outside of the town. I chose New York City because it is the most densly populated city in the United States.

##### Localizing USA dataset to New York City
Trimming 4.6 million donations to 75 thousand
- Total DataSet before cleaning
+ 24 million individual donations, 
+ 4.6 million individual donors, 
+ 850,000 projects in total

And after cleaning… and localizing the dataset to New York City
+ 75,676 individual Donations
+ 25,876 individual Donors
+ 48,296 Projects 

![]()

### Measuring Donor Enthusiasm for Projects
In this section we translate donation quantity and frequency from donors to projects into a enthusiam/satisfaction metric looking at the donation history of the whole dataset. The question we want to answer is: how enthusiastic is a single donor about a single project? The challenge in answering this question lies in the way one may want to interpret enthusiasm for a given project. In this DataSet donation patterns vary on a logarithmic scale. 

For Frequency: The dataset contains many individual donors that donated one time, while it also contains donors that have donated once per month over a multi-year span. Many individual donors donated $1 while others donated in the tens of thousands, and in some instances, millions of dollars. While scaling these values in each dimension would be possible, the data collected may likely have skewed the results towards those that are already being donated to frequently and with higher donation quantity. 

##### Instead of scaling donation frequency and quantity
I interpreted each donor's choice to donate any amount and any number of times to a single project as a binary value. If a user had donated any amount, and any amount of times to a project  it would be converted to a 1 or a 0. By converting the donation values to binary, it meant this dataset had become an implicit feedback dataset. 

### Comparing Similarity Matrices on an Implicit Dataset
In this section I applied the SurPRISE library to compare each donors contribution history. The SurPRISE library uses the DonorID in the first column, ProjectID in the second column, and the implicit donation history in third column. By matching the Donor ID's and Project ID's and the donation history, SurPRISE creates similarity matrix full of binary values. With this matrix we can calculate their given donation history as a "distance" from another user's donation history. 

##### The intution behind using a collaborative filtering recommender system on an implicit dataset
When donor A and donor B have the exact same donation history, we can say that the distance between these two matrices is essentially 0. If B suddenly develops a preference for something new, A may also develop a preference for that new item as well. When B likes something new we are using the similarity matrix comparison to guess that A will also like the new item. Through a process called collaborative filtering, we can begin to project a possible pattern of donations in the future for donors that have not yet donated by comparing their history to another user's history. With collaborative filtering recommender systems A's preference matrix may match many other user preference matrices and we will want to offer A an array of options, called recommendations. 

By comparing the distance between donors A's donation history to donor B's donation history, we can fill the donor A's donation history in with B's donation history. Using B's filled out donor history as a reference for possible projects subsequent donations may be on a list of recommendations.  For example, if A and B both choose to eat pizza and ride a bike, then chances are if B likes sports drinks that A will also like sports drinks, so why not recommend A sports drinks as well? 

##### In the real world
Donor preferences do not always match exactly and so instead we want to use their similarity distances. By applying the SurPRISE library we don't have to do all the Matrix Factorization mathematics that goes into generating the distances between donor preference matrices and instead return a list of projects they may choose to donate to. 
 
In this project we used five algorithms to test which recommendations would best suit the donor based on implicit donation history in comparison to other donors. We use five algorithms using the Grid Search Cross Validation (GridSearchCV) to compare Mean Absolute Error and Root Mean Squared Error between which yields the best error metrics and time to compute. 

Using the three algorithms with best results are listed here: Singular Value Decomposition for implicit feedback (SVD++), Singular Value Decomposition (SVD), and Non-Negative Matrix Factorization (NNMF). 

We did not use the CoClustering or Normal Predictor. Because the matrix was so sparse, and the CoClustering Algorithm works best by comparing a more heavily populated matrix, the resulting recommendation error metrics didn't make sense. Because the implicit dataset is not normally distributed, the NormalPredictor algo didn't work well here either.

It can be challenging to interpret error metrics when using recommender systems. When you give someone a list of recommendations and they choose one of them, it does not necessarily mean they didn't like the other ones. They may like them all the same amount, they may not see the item or project they would have most wanted to donate to. The challenge in interpreting error metrics comes where the algorithms are predicting based on history to project into the future what a person may like, but without many more datapoints and feedback, a recommendation list will have to be tested in live trials. 

According to the Error metrics SVD++ yielded the best scores for error metrics results and it is most geared towards the implicit feedback DataSet. The NNMF is a close second in my opinion. The Google Colab GUI returns a list of 10 projects that a user may like to choose from next. It will keep track of the new users preferences and compare the new user DataSet to the rest of the New York Donor DataSet, too. 

# My next steps
- Use PySpark to aggregate all unique donation entries from all over the country. 
- Incorporating Meta Data from Donors and Projects to create a hybrid filter
- Compare Content Filter Recommender System against a Collaborative Recommender System that has more meta data
- Include a Time Series to see where donations are coming from, more expansive overview and EDA

