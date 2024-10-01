# opensearch-logstash

## Logstash Service Architecture
Logstash processes logs from different servers and data sources and it behaves as the shipper. The shippers are used to collect the logs and these are installed in every input source. Brokers like Redis, Kafka or RabbitMQ are buffers to hold the data for indexers, there may be more than one brokers as failed over instances.

Indexers like Lucene are used to index the logs for better search performance and then the output is stored in Elasticsearch or other output destination. The data in output storage is available for Kibana and other visualization software.

![alt text](https://github.com/hosseinpanahii/opensearch-logstash/blob/main/logstash_service_architecture.png)

## Logstash Internal Architecture

The Logstash pipeline consists of three components Input, Filters and Output. The input part is responsible to specify and access the input data source such as the log folder of the Apache Tomcat Server.
![alt text](https://github.com/hosseinpanahii/opensearch-logstash/blob/main/logstash_internal_architecture.png)

## Example to Explain the Logstash Pipeline
The Logstash configuration file contains the details about the three components of Logstash. In this case, we are creating a file name called Logstash.conf.

The following configuration captures data from an input log “inlog.log” and writes it to an output log “outlog.log” without any filters.

## Logstash.conf
The Logstash configuration file just copies the data from the inlog.log file using the input plugin and flushes the log data to outlog.log file using the output plugin.
```bash
input {
   file {
      path => "C:/tpwork/logstash/bin/log/inlog.log"
   }
}
output {
   file {
      path => "C:/tpwork/logstash/bin/log/outlog.log"
   }
}
```

## Logstash.conf
In this Logstash configuration, we add a filter named grok to filter out the input data. The input log event, which matches the pattern sequence input log, only get to the output destination with error. Logstash adds a tag named "_grokparsefailure" in the output events, which does not match the grok filter pattern sequence.

Logstash offers many inbuilt regex patterns for parsing popular server logs like Apache. The pattern used here expects a verb like get, post, etc., followed by a uniform resource identifier.

```bash
input {
   file {
      path => "C:/tpwork/logstash/bin/log/inlog2.log"
   }
}
filter {
   grok {
      match => {"message" => "%{WORD:verb} %{URIPATHPARAM:uri}"}
   }
}
output {
   file {
      path => "C:/tpwork/logstash/bin/log/outlog2.log"
   }
}

```
## Logstash - Collecting Logs
Logs from different servers or data sources are collected using shippers. A shipper is an instance of Logstash installed in the server, which accesses the server logs and sends to specific output location.

It mainly sends the output to the Elasticsearch for storage. Logstash takes input from the following sources :

    STDIN
    Syslog
    Files
    TCP/UDP
    Microsoft windows Eventlogs
    Websocket
    Zeromq
    Customized extensions
## Collecting Logs Using Apache Tomcat 7 Server
In this example, we are collecting logs of Apache Tomcat 7 Server installed in windows using the file input plugin and sending them to the other log.
### logstash.conf
Here, Logstash is configured to access the access log of Apache Tomcat 7 installed locally. A regex pattern is used in path setting of the file plugin to get the data from the log file. This contains “access” in its name and it adds an apache type, which helps in differentiating the apache events from the other in a centralized destination source. Finally, the output events will be shown in the output.log.
```bash
input {
   file {
      path => "C:/Program Files/Apache Software Foundation/Tomcat 7.0/logs/*access*"
      type => "apache"
   }
} 
output {
   file {
      path => "C:/tpwork/logstash/bin/log/output.log"
   }
}
```
## Collecting Logs Using STDIN Plugin
In this section, we will discuss another example of collecting logs using the STDIN Plugin.
### logstash.conf
```bash
input {
   stdin{}
}
output {
   file {
      path => "C:/tpwork/logstash/bin/log/output.log"
   }
}
```
## Logstash - Parsing the Logs
Logstash receives the logs using input plugins and then uses the filter plugins to parse and transform the data. The parsing and transformation of logs are performed according to the systems present in the output destination. Logstash parses the logging data and forwards only the required fields. Later, these fields are transformed into the destination system’s compatible and understandable form.
### How to Parse the Logs?
Parsing of the logs is performed my using the GROK (Graphical Representation of Knowledge) patterns and you can find them in Github −

https://github.com/elastic/logstash/tree/v1.4.2/patterns.

Logstash matches the data of logs with a specified GROK Pattern or a pattern sequence for parsing the logs like "%{COMBINEDAPACHELOG}", which is commonly used for apache logs.

The parsed data is more structured and easy to search and for performing queries. Logstash searches for the specified GROK patterns in the input logs and extracts the matching lines from the logs. You can use GROK debugger to test your GROK patterns.

The syntax for a GROK pattern is %{SYNTAX:SEMANTIC}. Logstash GROK filter is written in the following form −

%{PATTERN:FieldName}

Here, PATTERN represents the GROK pattern and the fieldname is the name of the field, which represents the parsed data in the output.

For example, using online GROK debugger https://grokdebugger.com/

#### Input
```log
A sample error line in a log −
[Wed Dec 07 21:54:54.048805 2016] [:error] [pid 1234:tid 3456829102]
   [client 192.168.1.1:25007] JSP Notice:  Undefined index: abc in
   /home/manu/tpworks/tutorialspoint.com/index.jsp on line 11
```
#### GROK Pattern Sequence
This GROK pattern sequence matches to the log event, which comprises of a timestamp followed by Log Level, Process Id, Transaction Id and an Error Message.
```log
\[(%{DAY:day} %{MONTH:month} %{MONTHDAY} %{TIME} %{YEAR})\] \[.*:%{LOGLEVEL:loglevel}\]
   \[pid %{NUMBER:pid}:tid %{NUMBER:tid}\] \[client %{IP:clientip}:.*\]
   %{GREEDYDATA:errormsg}
```

#### output
The output is in JSON format.

```jsonc
{
   "day": [
      "Wed"
   ],
   "month": [
      "Dec"
   ],
   "loglevel": [
      "error"
   ],
   "pid": [
      "1234"
   ],
   "tid": [
      "3456829102"
   ],
   "clientip": [
      "192.168.1.1"
   ],
   "errormsg": [
      "JSP Notice:  Undefined index: abc in
      /home/manu/tpworks/tutorialspoint.com/index.jsp on line 11"
   ]
}
```
## Logstash - Filters
Logstash uses filters in the middle of the pipeline between input and output. The filters of Logstash measures manipulate and create events like Apache-Access. Many filter plugins used to manage the events in Logstash. Here, in an example of the Logstash Aggregate Filter, we are filtering the duration every SQL transaction in a database and computing the total time.

### Installing the Aggregate Filter Plugin
Installing the Aggregate Filter Plugin using the Logstash-plugin utility. The Logstash-plugin is a batch file for windows in bin folder in Logstash.
```cmd
>logstash-plugin install logstash-filter-aggregate
```
### logstash.conf
In this configuration, you can see three ‘if’ statements for Initializing, Incrementing, and generating the total duration of transaction, i.e., the sql_duration. The aggregate plugin is used to add the sql_duration, present in every event of the input log.
```jsonc
input {
   file {
      path => "C:/tpwork/logstash/bin/log/input.log"
   }
} 
filter {
   grok {
      match => [
         "message", "%{LOGLEVEL:loglevel} - 
            %{NOTSPACE:taskid} - %{NOTSPACE:logger} - 
            %{WORD:label}( - %{INT:duration:int})?" 
      ]
   }
   if [logger] == "TRANSACTION_START" {
      aggregate {
         task_id => "%{taskid}"
         code => "map['sql_duration'] = 0"
         map_action => "create"
      }
   }
   if [logger] == "SQL" {
      aggregate {
         task_id => "%{taskid}"
         code => "map['sql_duration'] ||= 0 ;
            map['sql_duration'] += event.get('duration')"
      }
   }
   if [logger] == "TRANSACTION_END" {
      aggregate {
         task_id => "%{taskid}"
         code => "event.set('sql_duration', map['sql_duration'])"
         end_of_task => true
         timeout => 120
      }
   }
}
output {
   file {
      path => "C:/tpwork/logstash/bin/log/output.log"    
   }
}
```
### input.log
The following code block shows the input log data.

```log
INFO - 48566 - TRANSACTION_START - start
INFO - 48566 - SQL - transaction1 - 320
INFO - 48566 - SQL - transaction1 - 200
INFO - 48566 - TRANSACTION_END - end
```
### output.log
As specified in the configuration file, the last ‘if’ statement where the logger is – TRANSACTION_END, which prints the total transaction time or sql_duration. This has been highlighted in yellow color in the output.log.

```log
{
   "path":"C:/tpwork/logstash/bin/log/input.log","@timestamp": "2016-12-22T19:04:37.214Z",
   "loglevel":"INFO","logger":"TRANSACTION_START","@version": "1","host":"wcnlab-PC",
   "message":"8566 - TRANSACTION_START - start\r","tags":[]
}
{
   "duration":320,"path":"C:/tpwork/logstash/bin/log/input.log",
   "@timestamp":"2016-12-22T19:04:38.366Z","loglevel":"INFO","logger":"SQL",
   "@version":"1","host":"wcnlab-PC","label":"transaction1",
   "message":" INFO - 48566 - SQL - transaction1 - 320\r","taskid":"48566","tags":[]
}
{
   "duration":200,"path":"C:/tpwork/logstash/bin/log/input.log",
   "@timestamp":"2016-12-22T19:04:38.373Z","loglevel":"INFO","logger":"SQL",
   "@version":"1","host":"wcnlab-PC","label":"transaction1",
   "message":" INFO - 48566 - SQL - transaction1 - 200\r","taskid":"48566","tags":[]
}
{
   "sql_duration":520,"path":"C:/tpwork/logstash/bin/log/input.log",
   "@timestamp":"2016-12-22T19:04:38.380Z","loglevel":"INFO","logger":"TRANSACTION_END",
   "@version":"1","host":"wcnlab-PC","label":"end",
   "message":" INFO - 48566 - TRANSACTION_END - end\r","taskid":"48566","tags":[]
```

## Logstash - Transforming the Logs
Logstash offers various plugins to transform the parsed log. These plugins can Add, Delete, and Update fields in the logs for better understanding and querying in the output systems.

We are using the Mutate Plugin to add a field name user in every line of the input log.
### Install the Mutate Filter Plugin
To install the mutate filter plugin; we can use the following command.
```cmd
>Logstash-plugin install Logstash-filter-mutate
```

### logstash.conf
In this config file, the Mutate Plugin is added after the Aggregate Plugin to add a new field.

```jsonc
input {
   file {
      path => "C:/tpwork/logstash/bin/log/input.log"
   }
} 
filter {
   grok {
      match => [ "message", "%{LOGLEVEL:loglevel} -
         %{NOTSPACE:taskid} - %{NOTSPACE:logger} -
         %{WORD:label}( - %{INT:duration:int})?" ]
   }
   if [logger] == "TRANSACTION_START" {
      aggregate {
         task_id => "%{taskid}"
         code => "map['sql_duration'] = 0"
         map_action => "create"
      }
   }
   if [logger] == "SQL" {
      aggregate {
         task_id => "%{taskid}"
         code => "map['sql_duration'] ||= 0 ; 
            map['sql_duration'] += event.get('duration')"
      }
   }
   if [logger] == "TRANSACTION_END" {
      aggregate {
         task_id => "%{taskid}"
         code => "event.set('sql_duration', map['sql_duration'])"
         end_of_task => true
         timeout => 120
      }
   }
   mutate {
      add_field => {"user" => "tutorialspoint.com"}
   }
}
output {
   file {
      path => "C:/tpwork/logstash/bin/log/output.log"
   }
}
```
### output.log
You can see that there is a new field named “user” in the output events.

```jsonc
{
   "path":"C:/tpwork/logstash/bin/log/input.log",
   "@timestamp":"2016-12-25T19:55:37.383Z",
   "@version":"1",
   "host":"wcnlab-PC",
   "message":"NFO - 48566 - TRANSACTION_START - start\r",
   "user":"tutorialspoint.com","tags":["_grokparsefailure"]
}
{
   "duration":320,"path":"C:/tpwork/logstash/bin/log/input.log",
   "@timestamp":"2016-12-25T19:55:37.383Z","loglevel":"INFO","logger":"SQL",
   "@version":"1","host":"wcnlab-PC","label":"transaction1",
   "message":" INFO - 48566 - SQL - transaction1 - 320\r",
   "user":"tutorialspoint.com","taskid":"48566","tags":[]
}
{
   "duration":200,"path":"C:/tpwork/logstash/bin/log/input.log",
   "@timestamp":"2016-12-25T19:55:37.399Z","loglevel":"INFO",
   "logger":"SQL","@version":"1","host":"wcnlab-PC","label":"transaction1",
   "message":" INFO - 48566 - SQL - transaction1 - 200\r",
   "user":"tutorialspoint.com","taskid":"48566","tags":[]
}
{
   "sql_duration":520,"path":"C:/tpwork/logstash/bin/log/input.log",
   "@timestamp":"2016-12-25T19:55:37.399Z","loglevel":"INFO",
   "logger":"TRANSACTION_END","@version":"1","host":"wcnlab-PC","label":"end",
   "message":" INFO - 48566 - TRANSACTION_END - end\r",
   "user":"tutorialspoint.com","taskid":"48566","tags":[]
}
```

