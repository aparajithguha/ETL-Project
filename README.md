# ETL Project

A simple ETL project interlinking the various components of Hadoop Ecosystem

<img src="Untitled Diagram.jpg" alt="hi" class="inline"/>

# Creating a simple dataset in Mysql
<ul><li>creating table in MySql</li></ul>
<pre>create table studdetails(id integer(10), firstname varchar(20), lastname varchar(30), age integer(10), <br> phone integer(14), location varchar(30));</pre>
<ul><li>insert some values</li></ul>
<pre>insert into studdetails values(001,"Rajiv","Reddy",21,9848022337,"Hyderabad");
insert into studdetails values(002,"siddarth","Battacharya",22,9848022338,"Kolkata");
insert into studdetails values(003,"Rajesh","Khanna",22,9848022339,"Delhi");
insert into studdetails values(004,"Preethi","Agarwal",21,9848022330,"Pune");
insert into studdetails values(005,"Trupthi","Mohanthy",23,9848022336,"Bhuwaneshwar");
insert into studdetails values(006,"Archana","Mishra",23,9848022335,"Chennai");
insert into studdetails values(007,"Komal","Nayak",24,9848022334,"trivendram");
insert into studdetails values(008,"Bharathi","Nambiayar",24,9848022333,"Chennai");</pre>

# Use sqoop to ingest the data and import to Hive
<pre>
sqoop import --connect  jdbc:mysql://localhost/appu1 --driver com.mysql.jdbc.Driver --username user <br> --password 12345 --table studdetails --hive-import -m 1 --hive-table appu.studdetails1 </pre>

# Create a Hive partition table

<pre>
create table studentpart(id int, firstname String, lastname String, phone bigint, loc String) PARTITIONED BY <br> (age INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE

SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

<b> // we are partitioning based on age</b>
INSERT OVERWRITE TABLE studentpart PARTITION(age) SELECT id,firstname,lastname,phone, <br> loc,age from studdetails1;

<b>// Check your partitions</b>
show partitions studentpart;
<b>// view data based on partitions</b>
select * from studentpart where age='21'; </pre>

# Loading Data to Pig and converting to JSON
<pre>
pig -useHCatalog

A = LOAD 'appu.studentpart' USING org.apache.hive.hcatalog.pig.HCatLoader(); 

B = filter A by age == 24;

store B into 'studentjson' USING JsonStorage();</pre>

# Loading Data to SPARK
<pre>
import org.apache.spark.sql.SparkSession

val session = SparkSession.builder().appName("test").master("local").getOrCreate()

val df = session.read.json("part-m-000000")

df.show() </pre>

# Import the same data to MongoDB
<pre>
<b>//create a database and a collection</b>
use student
db.createCollection('studentdetails')
<b>//now exit the mongo shell</b>
<b>//now use mongo import</b>
mongoimport -d student -c studentdetails part-m-000000
</pre>
