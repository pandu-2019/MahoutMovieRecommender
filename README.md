# Building a Recommender
**1.Sign up for an AWS account**
**2. Configure the elastic-mapreduce ruby client.
# 3. Start up an EMR cluster (note the pricing and make sure to shut the cluster down afterward).

# 4. Get the MovieLens data

unzip ml-1m.zip
<p>The HTML <code>button</code>wget http://files.grouplens.org/datasets/movielens/ml-1m.zip</p>

<p>The CSS <code>button</code> property defines the background color of an element.</p>

Convert ratings.dat, trade “::” for “,”, and take only the first three columns:
cat ml-1m/ratings.dat | sed 's/::/,/g' | cut -f1-3 -d, > ratings.csv
