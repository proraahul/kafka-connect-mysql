# kafka-connect-mysql


## What is kafka connect
- kafka connect is a framework that helps move data between external systems and apache kafka 
It acts like an automated data pipeline 


## download kafka_2.13-3.9.0 

##for setting 3 controllers and 3 brokers 
paste it || kafka_2.13-3.9.0/config || here 3 controller and 3 broker file from here 
| -- https://github.com/learnwithme-here/Learn-Kafka/blob/main/Kraft_Controller_Broker_config.zip | 

## Genereate cluster UUID ##
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"

now one by one
## Format Log Directories ##
bin/kafka-storage.sh format  -t $KAFKA_CLUSTER_ID -c config/controller1.properties
bin/kafka-storage.sh format  -t $KAFKA_CLUSTER_ID -c config/controller2.properties
bin/kafka-storage.sh format  -t $KAFKA_CLUSTER_ID -c config/controller3.properties
bin/kafka-storage.sh format  -t $KAFKA_CLUSTER_ID -c config/broker1.properties
bin/kafka-storage.sh format  -t $KAFKA_CLUSTER_ID -c config/broker2.properties
bin/kafka-storage.sh format  -t $KAFKA_CLUSTER_ID -c config/broker3.properties

## Start Controllers and brokers ##

nohup bin/kafka-server-start.sh config/controller1.properties > controller1.nohup.log &
nohup bin/kafka-server-start.sh config/controller2.properties > controller2.nohup.log &
nohup bin/kafka-server-start.sh config/controller3.properties > controller3.nohup.log &

nohup bin/kafka-server-start.sh config/broker1.properties > broker1.nohup.log &
nohup bin/kafka-server-start.sh config/broker2.properties > broker2.nohup.log &
nohup bin/kafka-server-start.sh config/broker3.properties > broker3.nohup.log &


## Producer ##
bin/kafka-console-producer.sh --topic test1 --bootstrap-server localhost:9092

## Consumer ##
bin/kafka-console-consumer.sh --topic test1 --from-beginning --bootstrap-server localhost:9092


## download kafka connect jdbc connector self hosted 
| https://www.confluent.io/hub/confluentinc/kafka-connect-jdbc

STEP : 
    - go to kafka inside kafka_2.13-.3.9.0/libs
 and here extract the jdbc zip file 
 wget "zip link" (you can copy when you download) and ENTER

and remove the zip file === rm "zip file" ENTER

STEP : setup distributed properties 
 cd ..
 to kafka_2.13-3.9.0/
 and vi config/connect-distributed-properties

at the end in connect-distributed-properties set property plugin.path="kafka-demo/kafka_2.13-3.9.0/libs"
and bootstrap.server=localhost:9092,localhost:9093,localhost:9094
and save it esc :wq! enter

## and now Starting Kafka Connect Distributed
bin/connect-distributed.sh config/connect-distributed.properties


## add kafka ui 
copy link addresss from https://github.com/provectus/kafka-ui/releases 

inside kafka_2.13-3.9.0/ wget "jar link" enter

now your kafka ui downloaded 

STEP:
 
for setup kafka-ui
open vi application.yaml file

## set this 

kafka:
  clusters:
    - name: local
      bootstrapServers:  "localhost:9092,localhost:9093,localhost:9094"
      kafkaConnect:
                  - name: first
                    address: http://localhost:8083


## now run 

##Kafka UI
$JAVA_HOME/bin/java -Dspring.config.additional-location=application.yaml -Dserver.port=8080 --add-opens java.rmi/javax.rm
i.ssl=ALL-UNNAMED -jar kafka-ui-api-v0.7.2.jar


##in kafka ui set jdbc source and sink

###source config 

{
	"connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
	"timestamp.column.name": "<enter timestamp column for incremental capture eg; column last_updated>", //"update_at"
	"incrementing.column.name": "<enter_incremental_column_name, eg; primary key column id> ", //"id"
	"validate.non.null": "true",
	"tasks.max": "1",
	"connection.attempts": "3",
	"transforms": "ValueToKey",
	"batch.max.rows": "500",
	"connection.backoff.ms": "3000",
	"table.types": "TABLE",
	"transforms.ValueToKey.fields": "id",
	"table.whitelist": "<Enter_table_name>", //employees
	"mode": "incrementing",
	"connection.loginTimeout": "10",
	"topic.prefix": "<enter_topic_prefix eg; source_>", //mysql_
	"poll.interval.ms": "60000",
	"name": "postgres_source", //mysql_source
	"connection.url": "jdbc:mysql://<ip_address/hostname>:<port>/<database_name>",
  "connection.user":"<Enter_user_name>",
  "connection.password":"<Enter_Password>",
	"transforms.ValueToKey.type": "org.apache.kafka.connect.transforms.ValueToKey"
}


###sink config

{
	"connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
	"table.name.format": "<Enter_target_table_name>", //"sink_employees"
	"tasks.max": "1",
	"topics": "<Enter_topic_to_consume_data_from>", //mysql_employees
	"fields.whitelist": "list of fields to whitelist", //"id,name,department,update_at" <== your table fields
	"auto.evolve": "true",
	"name": "postgres_sink",
	"value.converter.schemas.enable": "true",
	"auto.create": "true", 
	"connection.url": "jdbc:mysql://<ip_address/hostname>:<port>/<database_name>",
  	"connection.user":"<Enter_username>,
  	"connection.password":"<Enter_Password>",
	"value.converter": "org.apache.kafka.connect.json.JsonConverter",
	"insert.mode": "upsert",
	"pk.mode": "record_key", 
	"pk.fields": "<Enter primary key columns eg; column id" //"id"
}



 
