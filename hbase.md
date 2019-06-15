- https://dzone.com/refcardz/hbase?chapter=1

# HBase
- distributed, column-oriented persistent multidimensional sorted map
- storing column-family into memory/disc
- disc = hdfs or filesystem
- column family has 'veracity' - version of the record based on timestamp
- Value = Table + RowKey + *Family* + Column + *Timestamp*

```
data is sparse - a lot of column has null values
fast retrieving data by 'key of the row' + 'column name'
contains from: (HBase HMaster) *---> (HBase Region Server)
```
SQL for Hbase - [Phoenix SQL](https://phoenix.apache.org/)


## Why HBase
![hbase-why.png](https://s19.postimg.cc/43tj55w4j/hbase-why.png)

## HBase Architecture
![hbase-architecture.jpg](https://s19.postimg.cc/uq5zufaoz/hbase-architecture.jpg)

## HBase ACID
![hbase-acid.png](https://s19.postimg.cc/7ao2orlpf/hbase-acid.png)

## logical view
![hbase-record-logical-view.png](https://s19.postimg.cc/bjssr74gz/hbase-record-logical-view.png)

## phisical view
![hbase-record-phisical-view.png](https://s19.postimg.cc/rjbgafnkj/hbase-record-phisical-view.png)

## logical to phisical view
![hbase-logical-to-phisical-view.png](https://s19.postimg.cc/dpn3lfsf7/hbase-logical-to-phisical-view.png)

## Table characteristics
![hbase-table-characteristics.png](https://s19.postimg.cc/jruqcabtv/hbase-table-characteristics.png)

## Column-Family
![hbase-column-family.png](https://s19.postimg.cc/z1ulj2mn7/hbase-column-family.png)

## manage HBase
start/stop hbase
```
$HBASE_HOME/bin/start-hbase.sh
$HBASE_HOME/bin/stop-hbase.sh
```

## interactive shell
[cheat sheet](https://learnhbase.wordpress.com/2013/03/02/hbase-shell-commands/)
```
$HBASE_HOME/bin/hbase shell
```

## path to jars from hbase classpath
```
hbase classpath
```

## commands
* list of the tables
```
list
```

* create table 
```
create table 'mytable1'
```

* description of the table
```
descibe 'my_space:mytable1'
```

* count records
```
count 'my_space:mytable1'
```

* delete table
```
drop table 'mytable1'
disable table 'mytable1'
```

* iterate through a table, iterate with range
```
scan 'my_space:mytable1'
scan 'my_space:mytable1', {STARTROW=>"00223cfd-8b50-979d29164e72:1220", STOPROW=>"00223cfd-8b50-979d29164e72:1520"}
```
* save results into a file
```
echo " scan 'my_space:mytable1', {STARTROW=>"00223cfd-8b50-979d29164e72:1220", STOPROW=>"00223cfd-8b50-979d29164e72:1520"} " | hbase shell > out.txt
```

* insert data
```
put 'mytable1', 'row0015', 'cf:MyColumnFamily2', 'my value 01'
```

* read data
```
get 'mytable1', 'row0015'
```

# Java
## java app 
```
java \
    -cp /opt/cloudera/parcels/SPARK2/lib/spark2/jars/*:`hbase classpath`:{{ deploy_dir }}/lib/ingest-pipeline-orchestrator-jar-with-dependencies.jar \
    -Djava.security.auth.login.config={{ deploy_dir }}/res/deploy.jaas \
    com.bmw.ad.ingest.pipeline.orchestrator.admin.TruncateSessionEntriesHelper \
    --hbase-zookeeper {{ hbase_zookeeper }} \
    --ingest-tracking-table-name {{ ingest_tracking_table }} \
    --file-meta-table-name {{ file_meta_table }} \
    --component-state-table-name {{ component_state_table }} \
    --session-id $1
```


------------------------------------------------

https://github.com/tspannhw/phoenix


--------------------------------------------------

```

package com.khan.vaquar;

import java.math.BigDecimal;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;
import java.util.Set;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Repository;
import com.khan.vaquar.domain.TestPOJO;

@Repository
public class HbasePhonixRepository {
	
	@Value("${phoenix.table-name}")
	private String tableName;
	
	@Autowired
	private NamedParameterJdbcTemplate phoenixJdbcTemplate;
	
	public List<PosAccount> findRecords(String str, Set<String> dates) {
		
		String sql = "SELECT * FROM " + this.tableName + " WHERE CURRENT_ACCOUNT_NBR = :str AND DATE IN (:dates)";
		MapSqlParameterSource parameters = new MapSqlParameterSource();
		parameters.addValue("str", str);
		parameters.addValue("dates", dates);
		
		return this.phoenixJdbcTemplate.query(sql, parameters, new MYMapper());
	}
	
	public static final class MYMapper implements RowMapper<PosAccount> {

		@Override
	    public TestPOJO mapRow(ResultSet rs, int rowNum) throws SQLException {
	    	TestPOJO testPOJO = new TestPOJO();
			
	    	testPOJO.setA(rs.getString("A_NBR"));
	    	testPOJO.setT(rs.getString("T_ID"));
	    				
			return testPOJO;
	    }	
	    
	}
	
}

```
-------------------
Dependency 
-------------------

```
	<dependency>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		
		<dependency>
    		<groupId>org.apache.hbase</groupId>
    		<artifactId>hbase-client</artifactId>
    		<version>1.4.0</version>
    		<exclusions>
    			<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.slf4j</groupId>
					<artifactId>slf4j-log4j12</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		
		<dependency>
    		<groupId>org.apache.phoenix</groupId>
    		<artifactId>phoenix-core</artifactId>
    		<!-- This is the version of our Phoenix jar install -->
   			<version>4.7.0-HBase-1.1</version>
   			<exclusions>
   				<exclusion>
					<groupId>org.apache.hbase</groupId>
    				<artifactId>hbase-client</artifactId>
				</exclusion>
   				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.slf4j</groupId>
					<artifactId>slf4j-log4j12</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.mortbay.jetty</groupId>
					<artifactId>servlet-api-2.5</artifactId>
				</exclusion>
				<exclusion>
					<artifactId>servlet-api</artifactId>
					<groupId>javax.servlet</groupId>
				</exclusion>
				<exclusion>
					<groupId>jdk.tools</groupId>
					<artifactId>jdk.tools</artifactId>
				</exclusion>
				<exclusion>
					<groupId>sqlline</groupId>
    				<artifactId>sqlline</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		
		<!--  Maven repo is missing sqlline jar for 1.1.8 for phoenix-core, so include alt version manually -->
		<dependency>
    		<groupId>sqlline</groupId>
    		<artifactId>sqlline</artifactId>
    		<version>1.3.0</version>
		</dependency>
```		
------------
Command
------------
```
upsert into test1 values (1,'Hello');
upsert into test1 values (2,'World');

```

Create table:

```
create 'customer', {NAME=>'addr'}, {NAME=>'order'}

Create 't2',{'NAME=addr'},{'NAME=order;}

put 'customers', 'jsmith', 'addr:city', 'nashville'

```
----------------------------------------------------------
Create Table: insert data
----------------------------------------------------------

```

create 'asteroids', 'object', 'craft'

put 'asteroids', 'row1', 'object:location', '124212'
put 'asteroids', 'row2', 'object:location', '124212'
put 'asteroids', 'row3', 'object:location', '124213'
put 'asteroids', 'row4', 'object:location', '124214'

put 'asteroids', 'row1', 'object:location', '124215'

```

----------------------------------------------------------
connecting Phoenix using following commands on Hadoop node

•	 cd /usr/hdp/2.5.3.0-37/phoenix/bin

•	./sqlline-thin.py <ServerName>:<PORT>

-------------------------------------------------------
- http://hadooptutorial.info/apache-phoenix-hbase-an-sql-layer-on-hbase/

- https://www.youtube.com/watch?v=_HLoH_PgrLk

- https://stackoverflow.com/questions/46331734/how-to-mask-columns-using-spark-2

- https://www.youtube.com/watch?v=_HLoH_PgrLk

- https://www.slideshare.net/HadoopSummit/hbase-in-practice-74890983


- https://github.com/tspannhw/phoenix

- https://community.hortonworks.com/articles/56642/creating-a-spring-boot-java-8-microservice-to-read.html

https://phoenix.apache.org/faq.html#I_want_to_get_started_Is_there_a_Phoenix_Hello_World

### Hotspot

- https://sematext.com/blog/hbasewd-avoid-regionserver-hotspotting-despite-writing-records-with-sequential-keys/
- http://dwgeek.com/avoid-hbase-hotspotting.html/

###

- https://github.com/larsgeorge/hbase-book

### HBase Apache Phonix Spring Boot POC

- https://github.com/vaquarkhan/Apache-Kafka-poc-and-notes/wiki/HBase--Apache-Phonix-Spring-Boot--POC
- https://www.ibm.com/support/knowledgecenter/en/SSWSR9_11.5.0/com.ibm.swg.im.mdmhs.pmebi.doc/topics/bestpractice_loadhbase.html




