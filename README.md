
# Amazon Product Review Analysis

This project demonstrates large-scale data analysis using **Hadoop MapReduce** (in Java) and **Python** for preprocessing and visualization. The dataset contains Amazon product reviews, and the system extracts insights like:

-  **Average rating per product**
-  **Sentiment polarity counts (Positive, Neutral, Negative)**
-  **Number of reviews per year**

---

## Project Structure

```
Amazon-Review-Insights/
├── AmazonReviewAnalysis/       # Java MapReduce code
│   ├── src/                    # Java classes (Mappers, Reducers, and Drivers)
|      ├── AvgRatingMapper.java
│      ├── AvgRatingReducer.java
│      ├── AvgRatingDriver.java
│      ├── ReviewsPerYearMapper.java
│      ├── ReviewsPerYearReducer.java
│      ├── ReviewsPerYearDriver.java
│      ├── SentimentMapper.java
│      ├── SentimentReducer.java
│      ├── SentimentDriver.java            
│      ├── ReviewYearMapper.java            
│      ├── ReviewYearReducer.java
│      ├── ReviewYearDriver.java
├── preprocess/                 # Python data cleaning
│   ├── preprocess.ipynb
│   └── cleaned_reviews.tsv
├── visualization/              # Python visualization
│   ├── visualization.ipynb
├── data/
│   └── Reviews.csv
├── output/
│   └── AmazonReviewAnalysis2.0.jar
├── README.md
└── requirements.txt
```


## How to Run the Project

### 1. Preprocessing (Python)

```bash
cd preprocess
pip install -r requirements.txt
```

This cleans the raw dataset and outputs `cleaned_reviews.tsv`.

---

### 2. Install Hadoop
1. Install java jdk 8
    sudo apt install openjdk-8-jdk
2. Add configuration on bash file
    nano .bashrc
    ```bash
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
    export PATH=$PATH:/usr/lib/jvm/java-8-openjdk-amd64/bin 
    export HADOOP_HOME=~/hadoop-3.4.1/ 
    export PATH=$PATH:$HADOOP_HOME/bin 
    export PATH=$PATH:$HADOOP_HOME/sbin 
    export HADOOP_MAPRED_HOME=$HADOOP_HOME 
    export YARN_HOME=$HADOOP_HOME 
    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop 
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native 
    export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native" 
    export HADOOP_STREAMING=$HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.4.1.jar
    export HADOOP_LOG_DIR=$HADOOP_HOME/logs 
    export PDSH_RCMD_TYPE=ssh
    ```
3. Install ssh
```bash
    sudo apt-get install ssh
```
4. Download hadoop tar file from official website and extract it.
```bash
    wget https://downloads.apache.org/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz
    tar -zxvf ~/Downloads/hadoop-3.2.3.tar.gz 
```
5. Configure Hadoop
```bash
    cd hadoop-3.4.1/etc/hadoop
```
core-site.xml
```bash
<configuration> 
 <property> 
 <name>fs.defaultFS</name> 
 <value>hdfs://localhost:9000</value>  </property> 
 <property> 
<name>hadoop.proxyuser.dataflair.groups</name> <value>*</value> 
 </property> 
 <property> 
<name>hadoop.proxyuser.dataflair.hosts</name> <value>*</value> 
 </property> 
 <property> 
<name>hadoop.proxyuser.server.hosts</name> <value>*</value> 
 </property> 
 <property> 
<name>hadoop.proxyuser.server.groups</name> <value>*</value> 
 </property> 
</configuration>
```

hdfs-site.xml
```bash
<configuration> 
 <property> 
 <name>dfs.replication</name> 
 <value>1</value> 
 </property> 
</configuration>
```

mapred-site.xmll
```bash
<configuration> 
 <property> 
 <name>mapreduce.framework.name</name>  <value>yarn</value> 
 </property> 
 <property>
 <name>mapreduce.application.classpath</name> 
  
<value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value> 
 </property> 
</configuration>
```

yarn-site.xml
```bash
<configuration> 
 <property> 
 <name>yarn.nodemanager.aux-services</name> 
 <value>mapreduce_shuffle</value> 
 </property> 
 <property> 
 <name>yarn.nodemanager.env-whitelist</name> 
  
<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREP END_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value> 
 </property> 
</configuration>
```
6. Format the Namenode for the first time
```bash
hdfs namenode -format
```

7. Start hadoop service
```bash
start-dfs.sh
start-yar.sh
```
to stop
```bash
stop-dfs.sh
stop-yarn.sh
```

8. Check web interface
Namenode - http://localhost:9870
ResourceManager - http://localhost:8088


### 3. Hadoop MapReduce (Java in Eclipse)

1. Import `AmazonReviewAnalysis/` as a Java Project in Eclipse.
2. Compile all mapper, reducer, and driver classes.
3. Export the project as a `JAR` file into `/output` folder.
4. Upload the JAR and cleaned input to HDFS:

```bash
hdfs dfs -mkdir -p /review_analysis/input
hdfs dfs -put cleaned_reviews.tsv /review_analysis/input
hadoop jar AmazonReviewAnalysis2.0.jar main.AvgRatingDriver /review_analysis/input /review_analysis/output_avg_rating
hadoop jar AmazonReviewAnalysis2.0.jar main.SentimentDriver /review_analysis/input /review_analysis/output_review_sentiment
hadoop jar AmazonReviewAnalysis2.0.jar main.ReviewYearDriver /review_analysis/input /review_analysis/output_reviews_year
hadoop jar AmazonReviewAnalysis2.0.jar main.ReviewsPerYearDriver /review_analysis/input /review_analysis/output_reviews_per_year

```
5. Preview outputs:
```bash
hdfs dfs -cat /review_analysis/output_avg_rating/part-r-00000
hdfs dfs -cat /review_analysis/output_review_sentiment/part-r-00000
hdfs dfs -cat /review_analysis/output_reviews_year/part-r-00000
hdfs dfs -cat /review_analysis/output_reviews_per_year/part-r-00000

```

6. Download output from HDFS:

```bash
hdfs dfs -get /review_analysis/output_avg_rating ./output/avg_rating
hdfs dfs -get /review_analysis/output_review_sentiment ./output/review_sentiment
hdfs dfs -get /review_analysis/output_reviews_year ./output/reviews_year
hdfs dfs -get /review_analysis/output_reviews_per_year ./output/reviews_per_year

```

### 3. Visualization (Python)

Install required libraries:

```bash
pip install -r requirements.txt
```

Generate plots:

```bash
cd visualization
python avg_rating_chart.py
python sentiment_pie_chart.py
python reviews_per_year.py
python wordcloud_generator.py
```

---

## Requirements

- Java 8+
- Hadoop (configured on a Linux VM or cluster)
- Python 3.8+
- Python Libraries:
  - pandas
  - matplotlib
  - seaborn
  - wordcloud

## Dataset

- **Name**: Amazon Product Reviews 
- **Source**: [Kaggle](https://www.kaggle.com/datasets/arhamrumi/amazon-product-reviews)


