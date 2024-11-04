## Big Data Final Project: "News Sentiment Analysis with Spark"

### Introduction

Welcome to our project on sentiment analysis of news articles. Utilizing the power of Apache Spark and machine learning, we analyze news content to determine underlying sentiments. This guide will walk you through the entire process, from setting up data streaming with Zookeeper and Kafka, to using Spark for data processing and machine learning for sentiment classification. Designed for both beginners and experienced data enthusiasts, this project offers a practical approach to understanding and implementing sentiment analysis in the realm of Big Data. In this guide, the technical details are presented in a way that emphasizes clarity and accessibility. While not delving deeply into every technical nuance, it is crafted to ensure that even those without extensive programming knowledge can grasp the essence of what the project entails and follow along with the processes involved. Let's dive into news sentiment analysis!

#### Questions you may arise

Before moving further, let's address some common questions you might have regarding this project:

- [What is the workflow of this project?](https://gist.github.com/hoz-efa/92fc54ec8168e0f6793962745d835250)
- [Is this process doing any kind of prediction or what?](https://gist.github.com/hoz-efa/8a25f8eb55241b02a32a8d7914ba77e7)
- [What is the role of Zookeeper and Kafka?](https://gist.github.com/hoz-efa/dc05dad2681659d735f32e5c6777a692)
- [Why do we need a producer and a consumer?](https://gist.github.com/hoz-efa/b2038ca4282fd994b5f8dd4a85fdb0a1)
- [What method am I using in this process with producer and consumer?](https://gist.github.com/hoz-efa/a88bd3da7784efd098c8686cceea8e41)
- [How many windows do I need to open, or how many new connections are needed in GCP for this project?](https://gist.github.com/hoz-efa/7f820a945bd8e08fdd74ee57ba02390b)
---
These questions and answers aim to clarify the project's objectives, the technologies used, and the overall setup required. It's structured to help both technical and non-technical readers gain a better understanding of what to expect and how to navigate the project efficiently.

---

### Preliminary Step: Cluster Creation

Before proceeding with the Zookeeper installation, it's essential to set up a cluster as outlined in our "[Nov 19th Class Guide](https://gist.github.com/hoz-efa/5c521fbc5393d6f339949e679a98e215#creating-a-new-cluster)". This guide provides detailed steps for creating a new cluster tailored for our project needs. 

**Note**: If you already have a cluster set up that matches the specifications mentioned in the class guide, you can skip this step and proceed directly with the Zookeeper installation.

Certainly! Here's a paraphrased version of your instructions for installing and configuring Zookeeper, suitable for a README file on GitHub:

---

### Zookeeper Installation Guide

**Step 1: Downloading Zookeeper**
Begin by downloading the Zookeeper package using the following command:
```bash
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
```

**Step 2: Extracting the Package**
Once the download is complete, extract the contents of the package with:
```bash
tar -xzf zookeeper-3.4.14.tar.gz
```

**Step 3: Cleaning Up**
After extraction, remove the now-redundant tar.gz file to save space:
```bash
rm zookeeper-3.4.14.tar.gz
```

**Step 4: Configuring Zookeeper**
Navigate to the Zookeeper configuration directory:
```bash
cd zookeeper-3.4.14/conf/
```
Create a new configuration file by copying the sample file:
```bash
cp zoo_sample.cfg zoo.cfg
```
Edit the configuration file using the vim editor:
```bash
vi zoo.cfg
```
Make the following adjustments in the file:
- Append the line `server.0=127.0.0.1:2888:3888` at the end.
- Modify `dataDir` from `/tmp/zookeeper` to `/var/zookeeper`.

Save and exit vim by pressing `Esc`, typing `:wq`, and hitting `Enter`.

**Step 5: Preparing Data Directory**
Still in the `zookeeper-3.4.14/conf` directory, create a new directory for Zookeeper data:
```bash
sudo mkdir /var/zookeeper
```
Update the directory's ownership to your GCP username (replace `[YOUR_GCP_USERNAME]` with your actual username; use `whoami` to retrieve it if unsure):
```bash
sudo chown [YOUR_GCP_USERNAME]:[YOUR_GCP_USERNAME] /var/zookeeper
```

**Step 6: Setting Up 'myid' File**
Create and open the `myid` file in the vim editor:
```bash
vi /var/zookeeper/myid
```
In the editor, press `Insert`, type `0`, press `Esc`, then save and exit by typing `:wq` and pressing `Enter`.

**Step 7: Starting Zookeeper**
Return to the main Zookeeper directory and start the server:
```bash
cd ..
bin/zkServer.sh start
```
Alternatively, to start Zookeeper from the Home Directory:
```bash
zookeeper-3.4.14/bin/zkServer.sh start
```

**To Stop Zookeeper:**
Use the following command to stop the Zookeeper server:
```bash
bin/zkServer.sh stop
```

### Kafka Installation Guide

**Step 1: Begin in the Home Directory**
Start by navigating to your home directory:
```bash
cd
```

**Step 2: Downloading Kafka**
Download Kafka using the following command:
```bash
wget https://packages.confluent.io/archive/4.1/confluent-4.1.4-2.11.tar.gz
```

**Step 3: Extracting Kafka**
Once downloaded, proceed to extract the Kafka package:
```bash
tar -xzf confluent-4.1.4-2.11.tar.gz
```

**Step 4: Removing the Downloaded Archive**
After extraction, delete the downloaded tar.gz file as it's no longer needed:
```bash
rm confluent-4.1.4-2.11.tar.gz
```

**Step 5: Configuring Kafka**
Navigate to the Kafka directory:
```bash
cd confluent-4.1.4/
```
Edit the Zookeeper properties file to establish a connection with Kafka:
```bash
vi etc/kafka/zookeeper.properties
```
In the properties file, change `dataDir` from `/tmp/zookeeper` to `/var/zookeeper` to match the Zookeeper configuration. Save your changes and exit the editor.

**Step 6: Starting the Kafka Server**
Initiate the Kafka server using the following command:
```bash
bin/kafka-server-start etc/kafka/server.properties
```
This command will start Kafka and display logs.

**Step 7: Running Kafka in the Background**
To start the Kafka server in the background, use this command:
```bash
nohup bin/kafka-server-start etc/kafka/server.properties > /dev/null 2>&1 &
```
Alternatively, if you wish to run Kafka from the home directory, use:
```bash
nohup confluent-4.1.4/bin/kafka-server-start confluent-4.1.4/etc/kafka/server.properties > /dev/null 2>&1 &
```
A message like `[1]` followed by a number indicates that the service has started in the background.

**To Stop Kafka:**
To stop the Kafka server, execute the following command:
```bash
bin/kafka-server-stop
```

### Setting Up HDFS Path and Installing Python Libraries in GCP

**Creating Directories in HDFS for Data Storage**
To store data from the consumer, you'll need to create specific directories in HDFS. Execute the following commands to create these directories:

1. Create the main directory for BigData:
   ```bash
   hadoop fs -mkdir /BigData/
   ```
2. Inside the BigData directory, create a subdirectory for the Final Project:
   ```bash
   hadoop fs -mkdir /BigData/FinalProject/
   ```

**Installing Python Libraries in GCP**
To ensure your environment is equipped with the necessary tools, install these Python libraries:

1. Install `newsapi`:
   ```bash
   pip install newsapi
   ```
2. Install `newsapi-python`:
   ```bash
   pip install newsapi-python
   ```
3. Install `kafka-python`:
   ```bash
   pip install kafka-python
   ```
4. Install `hdfs`:
   ```bash
   pip install hdfs
   ```

**Getting the Producer and Consumer Code**

You can access the code for both the producer and consumer scripts [here](https://gist.github.com/hoz-efa/c5eb5ca8e87cdf366272d03e42e0b044). To use these scripts, follow these steps:

1. Copy and paste the code from the provided link for both "producer.py" and "consumer.py."

2. Save both files with the exact same names, "producer.py" and "consumer.py," on your computer.

3. Open the "producer.py" file and locate the `api_key` variable. You can obtain your API key from [this site](https://newsapi.org/). Add your API key to this variable. Additionally, feel free to modify the `keyword` to search for diverse data, and you can include multiple keywords for an expanded search.

4. In the "consumer.py" file, find the `hdfs_user` variable. Add your GCP username to this variable.

**Uploading Python Scripts to GCP**

Once you've edited and saved both scripts, you'll need to upload them to your Google Cloud Platform (GCP) environment. Here's how:

1. Navigate to the GCP interface and locate the "Upload Button."

2. Select the files you've just edited, "producer.py" and "consumer.py."

3. Initiate the upload process by following the on-screen instructions.

After completing these steps, you can verify that both scripts are available in your GCP environment by using the `ls` command.

### Final Steps for Running Your Project

**Running Zookeeper and Kafka**
Once the setup of tools is complete, you are ready to start the main components and run your scripts.

1. **Running the Producer Script**
   After starting both Zookeeper and Kafka, initiate the producer script in one of the terminal connections:
   ```bash
   python producer.py
   ```

2. **Running the Consumer Script**
   In a new terminal connection, start the consumer script:
   ```bash
   python consumer.py
   ```

**Verifying Data Storage in HDFS**
To ensure that your data is being correctly stored in HDFS, use these commands:

1. **Listing Files in HDFS**
   Open a third connection and check the files stored in HDFS:
   ```bash
   hadoop fs -ls /BigData/FinalProject/
   ```
   This will list all the files in the specified HDFS directory.

2. **Viewing File Contents**
   To inspect the contents of a specific file (e.g., `news_data.txt`), use:
   ```bash
   hdfs dfs -cat /BigData/FinalProject/news_data.txt
   ```

**Optional: Deleting Files from HDFS**
Should you need to delete a file from HDFS in the future, use the following command:
```bash
hdfs dfs -rm /BigData/FinalProject/news_data.txt
```

### Data Analysis and Model Prediction using Spark

**Starting Spark**
Begin by launching Spark with the following command:
```bash
spark-shell --master yarn
```

**Handling Multi-Line Commands in Spark**
Remember, to execute multi-line commands in Spark, use the `:paste` command. After pasting the entire command, execute it by pressing `Ctrl+D` in a new line.

**Step 1: Importing Libraries and Reading Data**
First, import the necessary libraries and read the data from HDFS into a DataFrame:
```scala
import org.apache.spark.sql.{SparkSession, DataFrame}
import org.apache.spark.sql.functions._
val spark = SparkSession.builder()
  .appName("NewsAnalysis")
  .getOrCreate()
val hdfsPath = "hdfs:///BigData/FinalProject/news_data.txt"
val df: DataFrame = spark.read.json(hdfsPath)
```

**Step 2: Exploring the Data**
To understand your data better, start by examining its structure and some initial rows:
- Print the schema of the DataFrame:
  ```scala
  df.printSchema()
  ```
- Display the first few rows:
  ```scala
  df.show()
  ```

**Step 3: Performing Data Analysis**
Now, let's perform some data analysis. You should think of unique queries relevant to your project's objectives. Here are some examples to get you started:

1. **Counting Articles by Source**
   ```scala
   df.groupBy("source_name").count().show()
   ```

2. **Identifying Common Words in Descriptions**
   ```scala
   df.select(explode(split(lower($"description"), "\\s+")).as("word"))
     .groupBy("word")
     .count()
     .orderBy(desc("count"))
     .show()
   ```

3. **Analyzing Article Count by Date**
   ```scala
   df.withColumn("published_date", to_date($"publishedAt"))
     .groupBy("published_date")
     .count()
     .orderBy("published_date")
     .show()
   ```

4. **Determining Top Sources with Most Articles**
   ```scala
   val topSources = df.groupBy("source_name").count().orderBy(desc("count"))
   topSources.show()
   ```

5. **Comparing Article Publication on Weekdays vs. Weekends**
   ```scala
   df.withColumn("day_of_week", date_format($"publishedAt", "E"))
     .groupBy("day_of_week").count().orderBy("day_of_week")
     .show()
   ```

---

**Note:** Please do not copy and paste the example queries provided here in your project, as they were tailored for a specific project. Instead, consider using them as a reference to build your own custom queries or utilize the capabilities of ChatGPT for query assistance.

**Hint:** If you want to explore more combinations of queries or analyze specific aspects of the data, you can leverage the content in the text file. Simply copy a line from the data that describes what's inside the data, and provide it to ChatGPT. Ask ChatGPT what kind of analysis can be performed with this data, and you'll receive additional query ideas and insights.

---

### Sentiment Analysis of News Articles Using Machine Learning

You're now ready to perform sentiment analysis on news articles using Spark. Here's a step-by-step guide based on the structure you've provided and similar to your "[Nov 25th Class Guide](https://gist.github.com/hoz-efa/2ba723be99bd32d5a0d54b10a7664336#step-10-split-data-into-training-and-test-sets)".

**1. Environment Setup and Data Loading**
First, set up your environment and load the data:

```scala
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._
import org.apache.spark.ml.feature.{Tokenizer, Word2Vec, StopWordsRemover}
import org.apache.spark.ml.classification.RandomForestClassifier
import org.apache.spark.ml.evaluation.BinaryClassificationEvaluator
import org.apache.spark.ml.{Pipeline, PipelineModel}
import org.apache.spark.ml.tuning.{CrossValidator, ParamGridBuilder}

// Initialize SparkSession
val spark = SparkSession.builder()
  .appName("NewsSentimentAnalysis")
  .getOrCreate()

// Load data from HDFS
val hdfsPath = "hdfs:///BigData/FinalProject/news_data.txt"
val df = spark.read.json(hdfsPath)
```

You can use `df.printSchema()` to check the schema and `df.show()` to view the data.

**2. Data Preparation: Labeling and Feature Extraction**
Prepare your data by labeling and extracting features:

```scala
// Add labels for sentiment
val labeledData = df.withColumn("label", when($"description".contains("positiveKeyword"), 1).otherwise(0))

val tokenizer = new Tokenizer().setInputCol("description").setOutputCol("words")
val remover = new StopWordsRemover().setInputCol("words").setOutputCol("filteredWords")

val word2Vec = new Word2Vec().setInputCol("filteredWords").setOutputCol("features").setVectorSize(100).setMinCount(0)

val Array(trainingData, testData) = labeledData.randomSplit(Array(0.7, 0.3), seed = 12345)
```

**3. Pipeline Setup and Model Training**
Set up your pipeline and train the model:

```scala
val rf = new RandomForestClassifier()

val pipeline = new Pipeline().setStages(Array(tokenizer, remover, word2Vec, rf))

val paramGrid = new ParamGridBuilder()
  .addGrid(rf.numTrees, Array(20, 50))
  .addGrid(rf.maxDepth, Array(5, 10))
  .build()

val evaluator = new BinaryClassificationEvaluator()
  .setLabelCol("label")
  .setRawPredictionCol("prediction")
  .setMetricName("areaUnderROC")

val cv = new CrossValidator()
  .setEstimator(pipeline)
  .setEvaluator(evaluator)
  .setEstimatorParamMaps(paramGrid)
  .setNumFolds(5)

val cvModel = cv.fit(trainingData)
```

**4. Model Evaluation**
Evaluate the model's performance:

```scala
val predictions = cvModel.transform(testData)
val auc = evaluator.evaluate(predictions)
println(s"Area Under ROC: $auc")
```

**5. Prediction on New Data**
Finally, apply the model to new data for prediction:

```scala
val newPath = "hdfs:///BigData/FinalProject/news_data.txt"
val newData = spark.read.json(newPath)

val newPredictions = cvModel.transform(newData)
newPredictions.select("description", "prediction").show()
```

---

This process provides a complete framework for performing sentiment analysis on news articles using Spark. It includes environment setup, data loading, data preparation, model training and evaluation, and finally, making predictions on new data.

---

> [How to reset the environment?](https://gist.github.com/hoz-efa/b011080381026d61aa5357c5a2b9623e) (Just in case if needed)

[Original Gist Link](https://gist.github.com/hoz-efa/12340737eca1f0b93629b0a110f89dfd)
