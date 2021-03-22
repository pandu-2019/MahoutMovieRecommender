# Building a Recommender
To demonstrate how to build an analytic job with Mahout on EMR, we’ll build a movie recommender. We will start with ratings given to movie titles by users in the MovieLens data set, which was compiled by the GroupLens team, and will use the “recommenditembased” example to find most-recommended movies for each user.

1. Sign up for an AWS account.
2. Configure the elastic-mapreduce ruby client.
3. Start up an EMR cluster (note the pricing and make sure to shut the cluster down afterward).
4. <p><code>./elastic-mapreduce --create --alive --name mahout-tutorial --num-instances 4 --master-instance-type m1.xlarge --slave-instance-type m2.2xlarge --ami-version 3.1 --ssh</code></p>


4. Get the MovieLens data
        <p><code>wget http://files.grouplens.org/datasets/movielens/ml-1m.zip</code></p>
        <p><code>unzip ml-1m.zip</code></p>

Convert ratings.dat, trade “::” for “,”, and take only the first three columns:
        <p><code>cat ml-1m/ratings.dat | sed 's/::/,/g' | cut -f1-3 -d, > ratings.csv</code></p>

Put ratings file into HDFS:
        <p><code>hadoop fs -put ratings.csv /ratings.csv</code></p>

5. Run the recommender job:
        <p><code>mahout recommenditembased --input /ratings.csv --output recommendations --numRecommendations 10 --outputPathForSimilarityMatrix similarity-matrix --similarityClassname SIMILARITY_COSINE</code></p>

6. Look for the results in the part-files containing the recommendations:
        <p><code>hadoop fs -ls recommendations</code></p>
        <p><code>hadoop fs -cat recommendations/part-r-00000 | head</code></p>
        
        
You should see a lookup file that looks something like this (your recommendations will be different since they are all 5.0-valued and we are only picking ten):

User ID	(Movie ID : Recommendation Strength) Tuples
35	[ 2067:5.0, 17:5.0, 1041:5.0, 2068:5.0, 2087:5.0, 1036:5.0, 900:5.0, 1:5.0, 2081:5.0, 3135:5.0 ]
70	[ 1682:5.0, 551:5.0, 1676:5.0, 1678:5.0, 2797:5.0, 17:5.0, 1:5.0, 1673:5.0, 2791:5.0, 2804:5.0 ]
105	[ 21:5.0, 3147:5.0, 6:5.0, 1019:5.0, 2100:5.0, 2105:5.0, 50:5.0, 1:5.0, 10:5.0, 32:5.0 ]
140	[ 3134:5.0, 1066:5.0, 2080:5.0, 1028:5.0, 21:5.0, 2100:5.0, 318:5.0, 1:5.0, 1035:5.0, 28:5.0 ]
175	[ 1916:5.0, 1921:5.0, 1912:5.0, 1914:5.0, 10:5.0, 11:5.0, 1200:5.0, 2:5.0, 6:5.0, 16:5.0 ]
210	[ 19:5.0, 22:5.0, 2:5.0, 16:5.0, 20:5.0, 21:5.0, 50:5.0, 1:5.0, 6:5.0, 25:5.0 ]
245	[ 2797:5.0, 3359:5.0, 1674:5.0, 2791:5.0, 1127:5.0, 1129:5.0, 356:5.0, 1:5.0, 1676:5.0, 3361:5.0 ]
280	[ 562:5.0, 1127:5.0, 1673:5.0, 1663:5.0, 551:5.0, 2797:5.0, 223:5.0, 1:5.0, 1674:5.0, 2243:5.0 ]
