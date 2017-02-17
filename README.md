# Spark-Pratice
Spark core practice

Running python script as a spark job
#Save this to a file with py extension and copy below contents in it

from pyspark import SparkContext, SparkConf
conf = SparkConf().setAppName("pyspark")
sc = SparkContext(conf=conf)
dataRDD = sc.textFile("/user/cloudera/departments")
for line in dataRDD.collect():
    print(line)
   
-----------------------------------------------------------------------------------------
# To run the python file above enter command below
spark-submit --master local yourfileName.py

==========================================================================================

#Count number of records in a file
# Load data from HDFS and storing results back to HDFS using Spark
from pyspark import SparkContext
dataRDD = sc.textFile("/user/cloudera/departments")
print(dataRDD.count())

==========================================================================================

# Developing word count program
# Create a file and type few lines and save it as wordcount.txt and copy to HDFS
# to /user/cloudera/wordcount.txt

data = sc.textFile("/user/cloudera/wordcount.txt")
dataFlatMap = data.flatMap(lambda x: x.split(" "))
dataMap = dataFlatMap.map(lambda x: (x, 1))
dataReduceByKey = dataMap.reduceByKey(lambda x,y: x + y)

dataReduceByKey.saveAsTextFile("/user/cloudera/wordcountoutput")

for i in dataReduceByKey.collect():
  print(i)
  

==========================================================================================

# Join disparate datasets together using Spark
# Problem statement, get the revenue and number of orders from order_items on daily basis
ordersRDD = sc.textFile("/user/cloudera/orders")
orderItemsRDD = sc.textFile("/user/cloudera/order_items")

ordersParsedRDD = ordersRDD.map(lambda rec: (int(rec.split(",")[0]), rec))
orderItemsParsedRDD = orderItemsRDD.map(lambda rec: (int(rec.split(",")[1]), rec))

ordersJoinOrderItems = orderItemsParsedRDD.join(ordersParsedRDD)
revenuePerOrderPerDay = ordersJoinOrderItems.map(lambda t: (t[1][1].split(",")[1], float(t[1][0].split(",")[4])))

# Get order count per day
ordersPerDay = ordersJoinOrderItems.map(lambda rec: rec[1][1].split(",")[1] + "," + str(rec[0])).distinct()
ordersPerDayParsedRDD = ordersPerDay.map(lambda rec: (rec.split(",")[0], 1))
totalOrdersPerDay = ordersPerDayParsedRDD.reduceByKey(lambda x, y: x + y)

# Get revenue per day from joined data
totalRevenuePerDay = revenuePerOrderPerDay.reduceByKey( \
lambda total1, total2: total1 + total2 \
)

for data in totalRevenuePerDay.collect():
  print(data)

# Joining order count per day and revenue per day
finalJoinRDD = totalOrdersPerDay.join(totalRevenuePerDay)
for data in finalJoinRDD.take(5):
  print(data)


==========================================================================================

