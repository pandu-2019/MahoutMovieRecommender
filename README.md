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
<table> 
          <thead> 
           <tr> 
            <th style="text-align: right">User ID</th> 
            <th style="text-align: center">(Movie ID : Recommendation Strength) Tuples</th> 
           </tr> 
          </thead> 
          <tbody> 
           <tr> 
            <td style="text-align: right">35</td> 
            <td>[ 2067:5.0, 17:5.0, 1041:5.0, 2068:5.0, 2087:5.0, 1036:5.0, 900:5.0, 1:5.0, 2081:5.0, 3135:5.0 ]</td> 
           </tr> 
           <tr> 
            <td style="text-align: right">70</td> 
            <td>[ 1682:5.0, 551:5.0, 1676:5.0, 1678:5.0, 2797:5.0, 17:5.0, 1:5.0, 1673:5.0, 2791:5.0, 2804:5.0 ]</td> 
           </tr> 
           <tr> 
            <td style="text-align: right">105</td> 
            <td>[ 21:5.0, 3147:5.0, 6:5.0, 1019:5.0, 2100:5.0, 2105:5.0, 50:5.0, 1:5.0, 10:5.0, 32:5.0 ]</td> 
           </tr> 
           <tr> 
            <td style="text-align: right">140</td> 
            <td>[ 3134:5.0, 1066:5.0, 2080:5.0, 1028:5.0, 21:5.0, 2100:5.0, 318:5.0, 1:5.0, 1035:5.0, 28:5.0 ]</td> 
           </tr> 
           <tr> 
            <td style="text-align: right">175</td> 
            <td>[ 1916:5.0, 1921:5.0, 1912:5.0, 1914:5.0, 10:5.0, 11:5.0, 1200:5.0, 2:5.0, 6:5.0, 16:5.0 ]</td> 
           </tr> 
           <tr> 
            <td style="text-align: right">210</td> 
            <td>[ 19:5.0, 22:5.0, 2:5.0, 16:5.0, 20:5.0, 21:5.0, 50:5.0, 1:5.0, 6:5.0, 25:5.0 ]</td> 
           </tr> 
           <tr> 
            <td style="text-align: right">245</td> 
            <td>[ 2797:5.0, 3359:5.0, 1674:5.0, 2791:5.0, 1127:5.0, 1129:5.0, 356:5.0, 1:5.0, 1676:5.0, 3361:5.0 ]</td> 
           </tr> 
           <tr> 
            <td style="text-align: right">280</td> 
            <td>[ 562:5.0, 1127:5.0, 1673:5.0, 1663:5.0, 551:5.0, 2797:5.0, 223:5.0, 1:5.0, 1674:5.0, 2243:5.0 ]</td> 
           </tr> 
          </tbody> 
         </table>
         
# Building a Service
Next, we’ll use this lookup file in a simple web service that returns movie recommendations for any given user.

<ol> 
        <li>Get Twisted, and Klein and Redis modules for Python. <pre class="brush: shell">        sudo easy_install twisted
        sudo easy_install klein
        sudo easy_install redis
</pre> </li> 
        <li>Install Redis and start up the server. <pre class="brush: shell">        wget http://download.redis.io/releases/redis-2.8.7.tar.gz
        tar xzf redis-2.8.7.tar.gz
        cd redis-2.8.7
        make
        ./src/redis-server &amp;
</pre> </li> 
        <li>Build a web service that pulls the recommendations into Redis and responds to queries.<br> Put the following into a file, e.g., “hello.py”<p></p> <pre class="brush: python">from klein import run, route
import redis
import os

#Start up a Redis instance
r = redis.StrictRedis(host='localhost', port=6379, db=0)

#Pull out all the recommendations from HDFS
p = os.popen("hadoop fs -cat recommendations/part*")

#Load the recommendations into Redis
for i in p:

  #Split recommendations into key of user id 
  #and value of recommendations
  #E.g., 35^I[2067:5.0,17:5.0,1041:5.0,2068:5.0,2087:5.0,
  #1036:5.0,900:5.0,1:5.0,081:5.0,3135:5.0]$
  k,v = i.split('\t')

  #Put key, value into Redis
  r.set(k,v)

#Establish an endpoint that takes in user id in the path
@route('/&lt;string:id&gt;')

def recs(request, id):
  #Get recommendations for this user
  v = r.get(id)
  return 'The recommendations for user '+id+' are '+v.decode("utf-8")


#Make a default endpoint
@route('/')

def home(request):
  return 'Please add a user id to the URL, e.g. http://localhost:8087/1234n'

#Start up a listener on port 8087
run("localhost", 8087)
   

</pre> </li> 
        <li>Start the web service.<br> <code>twistd -noy hello.py &amp;</code> </li> 
        <li>Test the web service with user id “37”:<br> <code>curl localhost:8080/37</code> </li> 
        <li>You should see a response like this (again, your recommendations will differ):<br> The recommendations for user 37 are 
       [7:5.0,2088:5.0,2080:5.0,1043:5.0,3107:5.0,2087:5.0,2078:5.0,3108:5.0,1042:5.0,1028:5.0] 
</li> 
        <li>When you’re finished, don’t forget to shut down the cluster:<br> <code>./elastic-mapreduce --list</code><p></p> <pre>j-UNIQUEJOBID      WAITING        ec2-AA-BB-CC-DD.compute-1.amazonaws.com         mahout-tutorial</pre> <p><code>./elastic-mapreduce --terminate j-UNIQUEJOBID</code> </p></li> 
        <li>Confirm shutdown:<br> <code>./elastic-mapreduce --list</code><p></p> <pre>j-UNIQUEJOBID     SHUTTING_DOWN     ec2-AA-BB-CC-DD.compute-1.amazonaws.com         mahout-tutorial</pre> <p>After a few minutes:<br> <code>./elastic-mapreduce --list</code></p> <pre>j-UNIQUEJOBID     TERMINATED     ec2-AA-BB-CC-DD.compute-1.amazonaws.com         mahout-tutorial</pre> </li> 
       </ol>
