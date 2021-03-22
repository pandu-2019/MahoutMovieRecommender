# Building a Recommender
l. **1.Sign up for an AWS account**
l. **2. Configure the elastic-mapreduce ruby client.**
l. **3. Start up an EMR cluster (note the pricing and make sure to shut the cluster down afterward).**

# 4. Get the MovieLens data

<p>The CSS <code>wget http://files.grouplens.org/datasets/movielens/ml-1m.zip</code></p>
<p>The CSS <code>unzip ml-1m.zip</code></p>

Convert ratings.dat, trade “::” for “,”, and take only the first three columns:
cat ml-1m/ratings.dat | sed 's/::/,/g' | cut -f1-3 -d, > ratings.csv
