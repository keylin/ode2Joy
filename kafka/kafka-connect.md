\8. KAFKA CONNECT

Kafka CONNECT

8.1 Overview

8.1 概览

Kafka Connect is a tool for scalably and reliably streaming data between Apache Kafka and other systems. It makes it simple to quickly define connectors that move large collections of data into and out of Kafka. Kafka Connect can ingest entire databases or collect metrics from all your application servers into Kafka topics, making the data available for stream processing with low latency. An export job can deliver data from Kafka topics into secondary storage and query systems or into batch systems for offline analysis. Kafka Connect features include:

Kafka connect是一个可扩展、高可用的流式数据处理工具，连接Apache Kafka和其他系统。基于Kafka connect，可以快速的定义和实现一个连接器（connector），完成大批量数据导入Kafka或导出Kafka。Kafka connect 可以拉取整个数据库或者你所有的应用服务器数据到Kafka topics，以供低延迟的流式处理。一个导出任务可以把Kafka topics数据投递到辅助存储做查询系统或者批处理系统以供离线分析。Kafka connect的功能如下：

A common framework for Kafka connectors - Kafka Connect standardizes integration of other data systems with Kafka, simplifying connector development, deployment, and management

Kafka连接器通用框架 - Kafka connect 使Kafka和其他数据系统的整合标准化，简化了连接器的开发、部署和管理。

Distributed and standalone modes - scale up to a large, centrally managed service supporting an entire organization or scale down to development, testing, and small production deployments

分布式和单点式 - 分布式可扩展，适用于大规模处理，作为集中化管理的服务支持整个组织；单点式适合开发、测试和小规模的生产环境部署。

REST interface - submit and manage connectors to your Kafka Connect cluster via an easy to use REST API

REST接口 - 通过rest api可以很方便提交和管理kafka connect cluster 里的连接器。

Automatic offset management - with just a little information from connectors, Kafka Connect can manage the offset commit process automatically so connector developers do not need to worry about this error prone part of connector development

自动化的offset管理 - kafka connect 能根据连接器的配置自动管理offsete的提交，所以开发者开发连接器的时候，无需担心这容易出错的offset管理问题。

Distributed and scalable by default - Kafka Connect builds on the existing group management protocol. More workers can be added to scale up a Kafka Connect cluster.

默认支持分布式和可伸缩 - Kafka connect 实现了已有的组管理协议，可以灵活的向Kafka connect cluster中添加更多worker。

Streaming/batch integration - leveraging Kafka's existing capabilities, Kafka Connect is an ideal solution for bridging streaming and batch data systems

整合流处理和批处理 - 基于Kafka已有的功能，用 Kafka connect 作为流处理和批处理数据系统的连接桥梁，是一个理想的解决方案。

8.2 User Guide

8.2 用户指南

The quickstart provides a brief example of how to run a standalone version of Kafka Connect. This section describes how to configure, run, and manage Kafka Connect in more detail.

这个快速入门提供了一个简单的示例，演示如何运行一个单点模式的Kafka connect。这一节内容包括如何配置、运行和管理Kafka connect的细节。

Running Kafka Connect

运行Kafka connect

Kafka Connect currently supports two modes of execution: standalone (single process) and distributed. In standalone mode all work is performed in a single process. This configuration is simpler to setup and get started with and may be useful in situations where only one worker makes sense (e.g. collecting log files), but it does not benefit from some of the features of Kafka Connect such as fault tolerance. You can start a standalone process with the following command:

Kafka connect 目前支持两种模式运行：单点模式（单进程）和分布式模式。在单点模式中，所有的work都运行在同一个进程中。这种模式的配置和启动都比较简单，适用于只有单个worker的情况（e.g 收集日志文件），但是缺少容错等Kafka connect的特性。你可以通过以下命令来启动一个单点模式的Kafka connect：

​    > bin/connect-standalone.sh config/connect-standalone.properties connector1.properties [connector2.properties ...]

​        

​    The first parameter is the configuration for the worker. This includes settings such as the Kafka connection parameters, serialization format, and how frequently to commit offsets. The provided example should work well with a local cluster running with the default configuration provided by config/server.properties. It will require tweaking to use with a different configuration or production deployment. All workers (both standalone and distributed) require a few configs:

第一个参数是指定worker的配置。这个配置包括Kafka的连接参数，序列化格式和提交offset的频率。本文中的示例在默认配置（config/server.properties）下的本机集群中运行良好。对于不同的配置或部署生产环境需要稍微调整配置。所有的worker（包括单点模式和分布模式），都需要如下配置：

​    bootstrap.servers - List of Kafka servers used to bootstrap connections to Kafka

bootstrap.servers - Kafka服务器列表，用于启动时连接Kafka

​    key.converter - Converter class used to convert between Kafka Connect format and the serialized form that is written to Kafka. This controls the format of the keys in messages written to or read from Kafka, and since this is independent of connectors it allows any connector to work with any serialization format. Examples of common formats include JSON and Avro.

  key.converter - 转换器，用于 Kafka connect 格式和要序列化到Kafka topic 中的格式的互相转换。这个控制了消息key写入Kafka或者从Kafka读取的格式，并且转换器是独立于连接器的，所以不同连接器可以配置不同的转换器处理各种序列化格式。例如通用的json和Avro格式。

value.converter - Converter class used to convert between Kafka Connect format and the serialized form that is written to Kafka. This controls the format of the values in messages written to or read from Kafka, and since this is independent of connectors it allows any connector to work with any serialization format. Examples of common formats include JSON and Avro.

value.converter - 转换器，用于 Kafka connect 格式和要序列化到Kafka topic 中的格式的互相转换。这个控制了消息value写入Kafka或者从Kafka读取的格式，并且转换器是独立于连接器的，所以不同连接器可以配置不同的转换器处理各种序列化格式。例如通用的json和Avro格式。

The important configuration options specific to standalone mode are:

下面是一些单点模式特有的重要配置：

offset.storage.file.filename - File to store offset data in

offset.storage.file.filename - 存储 offset 数据的文件。

The remaining parameters are connector configuration files. You may include as many as you want, but all will execute within the same process (on different threads). Distributed mode handles automatic balancing of work, allows you to scale up (or down) dynamically, and offers fault tolerance both in the active tasks and for configuration and offset commit data. Execution is very similar to standalone mode:

剩下的参数都是连接器的配置文件。你可以提供多个连接器配置，但这些连接器都作为不同线程运行在同一个进程（worker）中。分布式模式自动平衡work，允许你动态的增减work数量，并且为活跃任务、连接器配置和offset数据提交提供容错机制。分布式模式的运行命令和单点模式非常相似：

​    > bin/connect-distributed.sh config/connect-distributed.properties

​        

   The difference is in the class which is started and the configuration parameters which change how the Kafka Connect process decides where to store configurations, how to assign work, and where to store offsets and task statues. In the distributed mode, Kafka Connect stores the offsets, configs and task statuses in Kafka topics. It is recommended to manually create the topics for offset, configs and statuses in order to achieve the desired the number of partitions and replication factors. If the topics are not yet created when starting Kafka Connect, the topics will be auto created with default number of partitions and replication factor, which may not be best suited for its usage. In particular, the following configuration parameters, in addition to the common settings mentioned above, are critical to set before starting your cluster:

区别在于启动的类和配置参数，这些配置参数决定了kafka connect 配置存储地址、work分配算法、、offset和任务状态存储地址。在分布式模式中，kafka connect 的offsets、配置和任务状态都存储在kafka topic中。推荐手动的创建这些存储offset、配置和状态的topic，为这些topic配置需要的分片数和副分片数。如果这些topic在kafka connect启动时没有创建好，会自动创建默认配置的topic，这些配置不一定是最合适的配置。特别指出，除了上面提到的一些共用设置，下面的这些配置在kafka connect cluster 启动前也是至关重要的：

​    group.id (default connect-cluster) - unique name for the cluster, used in forming the Connect cluster group; note that this must not conflict with consumer group IDs

group.id（默认值是 connect cluster）- 集群的名字，组建connect cluster 组的时候用到；注意，这个 group id 不能和consumer 的 group ID 冲突。

​    config.storage.topic (default connect-configs) - topic to use for storing connector and task configurations; note that this should be a single partition, highly replicated, compacted topic. You may need to manually create the topic to ensure the correct configuration as auto created topics may have multiple partitions or be automatically configured for deletion rather than compaction

config.storage.topic (默认值connect-configs) - 用于存储连机器和任务配置的topic；注意，这个topic应该是单分片、多副本且可压缩的。你需要以正确的配置手动创建这个topic，因为默认方式创建的topic会是多分片或者以deletion方式清除log。

offset.storage.topic (default connect-offsets) - topic to use for storing offsets; this topic should have many partitions, be replicated, and be configured for compaction

offset.storage.topic (默认 connect-offsets) - 存储 offset 的topic；这个topic需要多分片、有副本，并且是压缩日志。

status.storage.topic (default connect-status) - topic to use for storing statuses; this topic can have multiple partitions, and should be replicated and configured for compaction

status.storage.topic (默认是connect-status)

存储状态的topic；这个topic需要多分片、有副本，并且是压缩日志。

Note that in distributed mode the connector configurations are not passed on the command line. Instead, use the REST API described below to create, modify, and destroy connectors.

注意，分布式模式下，连接器的配置不是通过命令行传入的，而是通过下面讲到的rest API来创建、修改和销毁连接器。

Configuring Connectors

配置连接器

Connector configurations are simple key-value mappings. For standalone mode these are defined in a properties file and passed to the Connect process on the command line. In distributed mode, they will be included in the JSON payload for the request that creates (or modifies) the connector. Most configurations are connector dependent, so they can't be outlined here. However, there are a few common options:

连接器的配置是简单的KV格式的。对于单点模式，这些配置放在配置文件中，通过命令参数传递给connect进程。在分布式模式下，这些配置作为一个json，通过http请求来创建或修改连接器。大部分配置都是连接器的特有配置，所以这里不罗列了。然而，连接器也有一些共有的配置：

mapping

name - Unique name for the connector. Attempting to register again with the same name will fail.

name - 连机器的名称，需保证唯一性，不允许注册相同名称的连接器。

connector.class - The Java class for the connector

connector.class - 配置连接器对应的java 类

tasks.max - The maximum number of tasks that should be created for this connector. The connector may create fewer tasks if it cannot achieve this level of parallelism.

tasks.max - 当前连接器创建的可创建任务数。如果达不到这个并行级别，连接器可能会创建比配置更少的任务数。

key.converter - (optional) Override the default key converter set by the worker.

key.converter - （可选）覆盖worker设置的key格式转换器。

value.converter - (optional) Override the default value converter set by the worker.

value.converter - （可选）覆盖worker设置的value格式转换器。

The connector.class config supports several formats: the full name or alias of the class for this connector. If the connector is org.apache.kafka.connect.file.FileStreamSinkConnector, you can either specify this full name or use FileStreamSink or FileStreamSinkConnector to make the configuration a bit shorter. Sink connectors also have one additional option to control their input:

connector.class 配置支持两种格式，类的完整名称和类的快捷方式。例如连接器是org.apache.kafka.connect.file.FileStreamSinkConnector，你可以配置为这个全名或者是FileStreamSink 、FileStreamSinkConnector，使配置看起来更简短一点。sink 连接器有额外配置来控制输入：

topics - A list of topics to use as input for this connector

topics - 推送数据到连接器的topic（可配置多个）

For any other options, you should consult the documentation for the connector.

对于其他的选项，你可以查看连接器的文档。

Transformations

Connectors can be configured with transformations to make lightweight message-at-a-time modifications. They can be convenient for data massaging and event routing. A transformation chain can be specified in the connector configuration.

连接器内可以配置一些变换器做一些轻量的消息修改。对于数据消息和事件路由，这个功能很方便。转换链可配置在连接器的配置中。

transforms - List of aliases for the transformation, specifying the order in which the transformations will be applied.

transforms - 配置一个或多个变换器的别名，同时也确定变换器链上的顺序。

transforms.$alias.type - Fully qualified class name for the transformation.

transforms.$alias.type - 确定变换器类型的完整类名。

transforms.$alias.$transformationSpecificConfig Configuration properties for the transformation

transforms.$alias.$transformationSpecificConfig - 相应变换器使用到的配置。

For example, lets take the built-in file source connector and use a transformation to add a static field.

举个例子，创建一个file source connector ，配置一个添加静态字段的变换器。

Throughout the example we'll use schemaless JSON data format. To use schemaless format, we changed the following two lines in connect-standalone.properties from true to false:

在这个例子中，我们使用的是无模式的json数据格式。为了使用无模式的数据格式，我们需要把connect-standalone.properties文件中的以下两行配置从true改为false：

​        key.converter.schemas.enable

value.converter.schemas.enable

​            

​    The file source connector reads each line as a String. We will wrap each line in a Map and then add a second field to identify the origin of the event. To do this, we use two transformations:

file source connector 以字符串的形式逐行读取文件。我们把每行数据放到作为一个字段map中，同时新增一个字段来标记这个事件。要达到这个目的，我们用到了两个变换器：

​    HoistField to place the input line inside a Map

HoistField 可以把一行输入当作一个字段写入map

​    InsertField to add the static field. In this example we'll indicate that the record came from a file connector

HoistField 可以新增一个静态字段。在本例中，

After adding the transformations, 们用于标示这个纪录来源是一个 file connect。

connect-file-source.properties file looks as following:

connect-file-source.properties 文件如下所示：

​        name=local-file-source

​      connector.class=FileStreamSource

​          tasks.max=1

​          file=test.txt

​          topic=connect-test

​          transforms=MakeMap, InsertSource

​          transforms.MakeMap.type=org.apache.kafka.connect.transforms.HoistField$Value

​          transforms.MakeMap.field=line

​          transforms.InsertSource.type=org.apache.kafka.connect.transforms.InsertField$Value

​          transforms.InsertSource.static.field=data_source

​          transforms.InsertSource.static.value=test-file-source

​            

​    All the lines starting with transforms were added for the transformations. You can see the two transformations we created: "InsertSource" and "MakeMap" are aliases that we chose to give the transformations. The transformation types are based on the list of built-in transformations you can see below. Each transformation type has additional configuration: HoistField requires a configuration called "field", which is the name of the field in the map that will include the original String from the file. InsertField transformation lets us specify the field name and the value that we are adding.

所有的行输入都会依次经过转换器的转换处理。你会看到转换链中我们创建的转换器：“InsertSource”和“MakeMap”。转换器类型是接下来看到的一系列内置转换器。每种类型的转换器都需要额外的配置。HoistField 需要一个“field”配置，指定从文件中读取的数据存入map时的字段名。InsertField 允许我们指定要添加到map中的字段名和字段值。

When we ran the file source connector on my sample file without the transformations, and then read them using kafka-console-consumer.sh, the results were:

现在我们对样例文件运行 file source connector，并且不做转换处理，然后用 kafka-console-consumer.sh，结果如下：

​        "foo"

​          "bar"

​          "hello world"

​           

   We then create a new file connector, this time after adding the transformations to the configuration file. This time, the results will be:

现在我们创建一个新的file connector，这次我们在配置文件中添加了转换器。这次的结果如下所示：

​        {"line":"foo","data_source":"test-file-source"}

​                {"line":"bar","data_source":"test-file-source"}

​                {"line":"hello world","data_source":"test-file-source"}

​            

​    You can see that the lines we've read are now part of a JSON map, and there is an extra field with the static value we specified. This is just one example of what you can do with transformations. Several widely-applicable data and routing transformations are included with Kafka Connect:

你会看到读取的那一行是json map的一部分，并且添加了我们指定的新字段和静态值。你能用转换器做很多事情，这只是其中的一个例子。Kafka connect已经实现了几个常用的数据处理转换器和路由转换器：

​    InsertField - Add a field using either static data or record metadata

​    ReplaceField - Filter or rename fields

MaskField - Replace field with valid null value for the type (0, empty string, etc)

ValueToKey

HoistField - Wrap the entire event as a single field inside a Struct or a Map

ExtractField - Extract a specific field from Struct and Map and include only this field in results

SetSchemaMetadata - modify the schema name or version

TimestampRouter - Modify the topic of a record based on original topic and timestamp. Useful when using a sink that needs to write to different tables or indexes based on timestamps

RegexpRouter - modify the topic of a record based on original topic, replacement string and a regular expression

Details on how to configure each transformation are listed below:

org.apache.kafka.connect.transforms.InsertField

Insert field(s) using attributes from the record metadata or a configured static value.

Use the concrete transformation type designed for the record key (org.apache.kafka.connect.transforms.InsertField$Key) or value (org.apache.kafka.connect.transforms.InsertField$Value).

NAME	DESCRIPTION	TYPE	DEFAULT	VALID VALUES	IMPORTANCE

offset.field	Field name for Kafka offset - only applicable to sink connectors.

Suffix with ! to make this a required field, or ? to keep it optional (the default).	string	null		medium

partition.field	Field name for Kafka partition. Suffix with ! to make this a required field, or ? to keep it optional (the default).	string	null		medium

static.field	Field name for static data field. Suffix with ! to make this a required field, or ? to keep it optional (the default).	string	null		medium

static.value	Static field value, if field name configured.	string	null		medium

timestamp.field	Field name for record timestamp. Suffix with ! to make this a required field, or ? to keep it optional (the default).	string	null		medium

topic.field	Field name for Kafka topic. Suffix with ! to make this a required field, or ? to keep it optional (the default).	string	null		medium

org.apache.kafka.connect.transforms.ReplaceField

Filter or rename fields.

Use the concrete transformation type designed for the record key (org.apache.kafka.connect.transforms.ReplaceField$Key) or value (org.apache.kafka.connect.transforms.ReplaceField$Value).

NAME	DESCRIPTION	TYPE	DEFAULT	VALID VALUES	IMPORTANCE

blacklist	Fields to exclude. This takes precedence over the whitelist.	list	""		medium

renames	Field rename mappings.	list	""	list of colon-delimited pairs, e.g. foo:bar,abc:xyz	medium

whitelist	Fields to include. If specified, only these fields will be used.	list	""		medium

org.apache.kafka.connect.transforms.MaskField

Mask specified fields with a valid null value for the field type (i.e. 0, false, empty string, and so on).

Use the concrete transformation type designed for the record key (org.apache.kafka.connect.transforms.MaskField$Key) or value (org.apache.kafka.connect.transforms.MaskField$Value).

NAME	DESCRIPTION	TYPE	DEFAULT	VALID VALUES	IMPORTANCE

fields	Names of fields to mask.	list		non-empty list	high

org.apache.kafka.connect.transforms.ValueToKey

Replace the record key with a new key formed from a subset of fields in the record value.

NAME	DESCRIPTION	TYPE	DEFAULT	VALID VALUES	IMPORTANCE

fields	Field names on the record value to extract as the record key.	list		non-empty list	high

org.apache.kafka.connect.transforms.HoistField

Wrap data using the specified field name in a Struct when schema present, or a Map in the case of schemaless data.

Use the concrete transformation type designed for the record key (org.apache.kafka.connect.transforms.HoistField$Key) or value (org.apache.kafka.connect.transforms.HoistField$Value).

NAME	DESCRIPTION	TYPE	DEFAULT	VALID VALUES	IMPORTANCE

field	Field name for the single field that will be created in the resulting Struct or Map.	string			medium

org.apache.kafka.connect.transforms.ExtractField

Extract the specified field from a Struct when schema present, or a Map in the case of schemaless data.

Use the concrete transformation type designed for the record key (org.apache.kafka.connect.transforms.ExtractField$Key) or value (org.apache.kafka.connect.transforms.ExtractField$Value).

NAME	DESCRIPTION	TYPE	DEFAULT	VALID VALUES	IMPORTANCE

field	Field name to extract.	string			medium

org.apache.kafka.connect.transforms.SetSchemaMetadata

Set the schema name, version or both on the record's key (org.apache.kafka.connect.transforms.SetSchemaMetadata$Key) or value (org.apache.kafka.connect.transforms.SetSchemaMetadata$Value) schema.

NAME	DESCRIPTION	TYPE	DEFAULT	VALID VALUES	IMPORTANCE

schema.name	Schema name to set.	string	null		high

schema.version	Schema version to set.	int	null		high

org.apache.kafka.connect.transforms.TimestampRouter

Update the record's topic field as a function of the original topic value and the record timestamp.

This is mainly useful for sink connectors, since the topic field is often used to determine the equivalent entity name in the destination system(e.g. database table or search index name).

NAME	DESCRIPTION	TYPE	DEFAULT	VALID VALUES	IMPORTANCE

timestamp.format	Format string for the timestamp that is compatible with java.text.SimpleDateFormat.	string	yyyyMMdd		high

topic.format	Format string which can contain ${topic} and ${timestamp} as placeholders for the topic and timestamp, respectively.	string	${topic}-${timestamp}		high

org.apache.kafka.connect.transforms.RegexRouter

Update the record topic using the configured regular expression and replacement string.

Under the hood, the regex is compiled to a java.util.regex.Pattern. If the pattern matches the input topic, java.util.regex.Matcher#replaceFirst() is used with the replacement string to obtain the new topic.

NAME	DESCRIPTION	TYPE	DEFAULT	VALID VALUES	IMPORTANCE

regex	Regular expression to use for matching.	string		valid regex	high

replacement	Replacement string.	string			high

REST API

Since Kafka Connect is intended to be run as a service, it also provides a REST API for managing connectors. By default, this service runs on port 8083. The following are the currently supported endpoints:

GET /connectors - return a list of active connectors

POST /connectors - create a new connector; the request body should be a JSON object containing a string name field and an object config field with the connector configuration parameters

GET /connectors/{name} - get information about a specific connector

GET /connectors/{name}/config - get the configuration parameters for a specific connector

PUT /connectors/{name}/config - update the configuration parameters for a specific connector

GET /connectors/{name}/status - get current status of the connector, including if it is running, failed, paused, etc., which worker it is assigned to, error information if it has failed, and the state of all its tasks

GET /connectors/{name}/tasks - get a list of tasks currently running for a connector

GET /connectors/{name}/tasks/{taskid}/status - get current status of the task, including if it is running, failed, paused, etc., which worker it is assigned to, and error information if it has failed

PUT /connectors/{name}/pause - pause the connector and its tasks, which stops message processing until the connector is resumed

PUT /connectors/{name}/resume - resume a paused connector (or do nothing if the connector is not paused)

POST /connectors/{name}/restart - restart a connector (typically because it has failed)

POST /connectors/{name}/tasks/{taskId}/restart - restart an individual task (typically because it has failed)

DELETE /connectors/{name} - delete a connector, halting all tasks and deleting its configuration

Kafka Connect also provides a REST API for getting information about connector plugins:

GET /connector-plugins- return a list of connector plugins installed in the Kafka Connect cluster. Note that the API only checks for connectors on the worker that handles the request, which means you may see inconsistent results, especially during a rolling upgrade if you add new connector jars

PUT /connector-plugins/{connector-type}/config/validate - validate the provided configuration values against the configuration definition. This API performs per config validation, returns suggested values and error messages during validation.

8.3 Connector Development Guide

This guide describes how developers can write new connectors for Kafka Connect to move data between Kafka and other systems. It briefly reviews a few key concepts and then describes how to create a simple connector.

Core Concepts and APIs

Connectors and Tasks

To copy data between Kafka and another system, users create a Connector for the system they want to pull data from or push data to. Connectors come in two flavors: SourceConnectors import data from another system (e.g. JDBCSourceConnector would import a relational database into Kafka) and SinkConnectors export data (e.g. HDFSSinkConnector would export the contents of a Kafka topic to an HDFS file). Connectors do not perform any data copying themselves: their configuration describes the data to be copied, and the Connector is responsible for breaking that job into a set of Tasks that can be distributed to workers. These Tasks also come in two corresponding flavors: SourceTask and SinkTask. With an assignment in hand, each Task must copy its subset of the data to or from Kafka. In Kafka Connect, it should always be possible to frame these assignments as a set of input and output streams consisting of records with consistent schemas. Sometimes this mapping is obvious: each file in a set of log files can be considered a stream with each parsed line forming a record using the same schema and offsets stored as byte offsets in the file. In other cases it may require more effort to map to this model: a JDBC connector can map each table to a stream, but the offset is less clear. One possible mapping uses a timestamp column to generate queries incrementally returning new data, and the last queried timestamp can be used as the offset.

Streams and Records

Each stream should be a sequence of key-value records. Both the keys and values can have complex structure -- many primitive types are provided, but arrays, objects, and nested data structures can be represented as well. The runtime data format does not assume any particular serialization format; this conversion is handled internally by the framework. In addition to the key and value, records (both those generated by sources and those delivered to sinks) have associated stream IDs and offsets. These are used by the framework to periodically commit the offsets of data that have been processed so that in the event of failures, processing can resume from the last committed offsets, avoiding unnecessary reprocessing and duplication of events.

Dynamic Connectors

Not all jobs are static, so Connector implementations are also responsible for monitoring the external system for any changes that might require reconfiguration. For example, in the JDBCSourceConnector example, the Connector might assign a set of tables to each Task. When a new table is created, it must discover this so it can assign the new table to one of the Tasks by updating its configuration. When it notices a change that requires reconfiguration (or a change in the number of Tasks), it notifies the framework and the framework updates any corresponding Tasks.

Developing a Simple Connector

Developing a connector only requires implementing two interfaces, the Connector and Task. A simple example is included with the source code for Kafka in the file package. This connector is meant for use in standalone mode and has implementations of a SourceConnector/SourceTask to read each line of a file and emit it as a record and a SinkConnector/SinkTask that writes each record to a file. The rest of this section will walk through some code to demonstrate the key steps in creating a connector, but developers should also refer to the full example source code as many details are omitted for brevity.

Connector Example

We'll cover the SourceConnector as a simple example. SinkConnector implementations are very similar. Start by creating the class that inherits from SourceConnector and add a couple of fields that will store parsed configuration information (the filename to read from and the topic to send data to):

​    public class FileStreamSourceConnector extends SourceConnector {

​            private String filename;

​                private String topic;

​            

​    The easiest method to fill in is getTaskClass(), which defines the class that should be instantiated in worker processes to actually read the data:

​    @Override

​        public Class<? extends Task> getTaskClass() {

​            return FileStreamSourceTask.class;

​            }

​        

​    We will define the FileStreamSourceTask class below. Next, we add some standard lifecycle methods, start() and stop():

​    @Override

​        public void start(Map<String, String> props) {

​            // The complete version includes error handling as well.

​                filename = props.get(FILE_CONFIG);

​                topic = props.get(TOPIC_CONFIG);

​            }

​            

​    @Override

​        public void stop() {

​            // Nothing to do since no background monitoring is required.

​            }

​        

​    Finally, the real core of the implementation is in taskConfigs(). In this case we are only handling a single file, so even though we may be permitted to generate more tasks as per the maxTasks argument, we return a list with only one entry:

​    @Override

​        public List<Map<String, String>> taskConfigs(int maxTasks) {

​            ArrayList<Map<String, String>> configs = new ArrayList<>();

​                // Only one input stream makes sense.

​                Map<String, String> config = new HashMap<>();

​                if (filename != null)

​                    config.put(FILE_CONFIG, filename);

​                    config.put(TOPIC_CONFIG, topic);

​                configs.add(config);

​                return configs;

​            }

​        

​    Although not used in the example, SourceTask also provides two APIs to commit offsets in the source system: commit and commitRecord. The APIs are provided for source systems which have an acknowledgement mechanism for messages. Overriding these methods allows the source connector to acknowledge messages in the source system, either in bulk or individually, once they have been written to Kafka. The commit API stores the offsets in the source system, up to the offsets that have been returned by poll. The implementation of this API should block until the commit is complete. The commitRecord API saves the offset in the source system for each SourceRecord after it is written to Kafka. As Kafka Connect will record offsets automatically, SourceTasks are not required to implement them. In cases where a connector does need to acknowledge messages in the source system, only one of the APIs is typically required. Even with multiple tasks, this method implementation is usually pretty simple. It just has to determine the number of input tasks, which may require contacting the remote service it is pulling data from, and then divvy them up. Because some patterns for splitting work among tasks are so common, some utilities are provided in ConnectorUtils to simplify these cases. Note that this simple example does not include dynamic input. See the discussion in the next section for how to trigger updates to task configs.

​    Task Example - Source Task

​    

Next we'll describe the implementation of the corresponding SourceTask. The implementation is short, but too long to cover completely in this guide. We'll use pseudo-code to describe most of the implementation, but you can refer to the source code for the full example. Just as with the connector, we need to create a class inheriting from the appropriate base Task class. It also has some standard lifecycle methods:

​    public class FileStreamSourceTask extends SourceTask {

​            String filename;

​                InputStream stream;

​                String topic;

​                

​        @Override

​                public void start(Map<String, String> props) {

​                    filename = props.get(FileStreamSourceConnector.FILE_CONFIG);

​                        stream = openOrThrowError(filename);

​                        topic = props.get(FileStreamSourceConnector.TOPIC_CONFIG);

​                    }

​                    

​        @Override

​                public synchronized void stop() {

​                    stream.close();

​                    }

​            

​    These are slightly simplified versions, but show that that these methods should be relatively simple and the only work they should perform is allocating or freeing resources. There are two points to note about this implementation. First, the start() method does not yet handle resuming from a previous offset, which will be addressed in a later section. Second, the stop() method is synchronized. This will be necessary because SourceTasks are given a dedicated thread which they can block indefinitely, so they need to be stopped with a call from a different thread in the Worker. Next, we implement the main functionality of the task, the poll() method which gets events from the input system and returns a List<SourceRecord>:

​    @Override

​        public List<SourceRecord> poll() throws InterruptedException {

​            try {

​                    ArrayList<SourceRecord> records = new ArrayList<>();

​                        while (streamValid(stream) && records.isEmpty()) {

​                            LineAndOffset line = readToNextLine(stream);

​                                if (line != null) {

​                                    Map<String, Object> sourcePartition = Collections.singletonMap("filename", filename);

​                                        Map<String, Object> sourceOffset = Collections.singletonMap("position", streamOffset);

​                                        records.add(new SourceRecord(sourcePartition, sourceOffset, topic, Schema.STRING_SCHEMA, line));

​                                    } else {

​                                    Thread.sleep(1);

​                                    }

​                            }

​                        return records;

​                    } catch (IOException e) {

​                    // Underlying stream was killed, probably as a result of calling stop. Allow to return

​                        // null, and driving thread will handle any shutdown if necessary.

​                    }

​                return null;

​            }

​        

​    Again, we've omitted some details, but we can see the important steps: the poll() method is going to be called repeatedly, and for each call it will loop trying to read records from the file. For each line it reads, it also tracks the file offset. It uses this information to create an output SourceRecord with four pieces of information: the source partition (there is only one, the single file being read), source offset (byte offset in the file), output topic name, and output value (the line, and we include a schema indicating this value will always be a string). Other variants of the SourceRecord constructor can also include a specific output partition and a key. Note that this implementation uses the normal Java InputStream interface and may sleep if data is not available. This is acceptable because Kafka Connect provides each task with a dedicated thread. While task implementations have to conform to the basic poll() interface, they have a lot of flexibility in how they are implemented. In this case, an NIO-based implementation would be more efficient, but this simple approach works, is quick to implement, and is compatible with older versions of Java.

​    Sink Tasks

​    

The previous section described how to implement a simple SourceTask. Unlike SourceConnector and SinkConnector, SourceTask and SinkTask have very different interfaces because SourceTask uses a pull interface and SinkTask uses a push interface. Both share the common lifecycle methods, but the SinkTask interface is quite different:

​    public abstract class SinkTask implements Task {

​            public void initialize(SinkTaskContext context) {

​                    this.context = context;

​                    }

​                    

​        public abstract void put(Collection<SinkRecord> records);

​                

​                public abstract void flush(Map<TopicPartition, Long> offsets);

​            

​    The SinkTask documentation contains full details, but this interface is nearly as simple as the SourceTask. The put() method should contain most of the implementation, accepting sets of SinkRecords, performing any required translation, and storing them in the destination system. This method does not need to ensure the data has been fully written to the destination system before returning. In fact, in many cases internal buffering will be useful so an entire batch of records can be sent at once, reducing the overhead of inserting events into the downstream data store. The SinkRecords contain essentially the same information as SourceRecords: Kafka topic, partition, offset and the event key and value. The flush() method is used during the offset commit process, which allows tasks to recover from failures and resume from a safe point such that no events will be missed. The method should push any outstanding data to the destination system and then block until the write has been acknowledged. The offsets parameter can often be ignored, but is useful in some cases where implementations want to store offset information in the destination store to provide exactly-once delivery. For example, an HDFS connector could do this and use atomic move operations to make sure the flush() operation atomically commits the data and offsets to a final location in HDFS.

​    Resuming from Previous Offsets

​    

The SourceTask implementation included a stream ID (the input filename) and offset (position in the file) with each record. The framework uses this to commit offsets periodically so that in the case of a failure, the task can recover and minimize the number of events that are reprocessed and possibly duplicated (or to resume from the most recent offset if Kafka Connect was stopped gracefully, e.g. in standalone mode or due to a job reconfiguration). This commit process is completely automated by the framework, but only the connector knows how to seek back to the right position in the input stream to resume from that location. To correctly resume upon startup, the task can use the SourceContext passed into its initialize() method to access the offset data. In initialize(), we would add a bit more code to read the offset (if it exists) and seek to that position:

​        stream = new FileInputStream(filename);

​                Map<String, Object> offset = context.offsetStorageReader().offset(Collections.singletonMap(FILENAME_FIELD, filename));

​                if (offset != null) {

​                    Long lastRecordedOffset = (Long) offset.get("position");

​                        if (lastRecordedOffset != null)

​                            seekToOffset(stream, lastRecordedOffset);

​                        }

​            

​    Of course, you might need to read many keys for each of the input streams. The OffsetStorageReader interface also allows you to issue bulk reads to efficiently load all offsets, then apply them by seeking each input stream to the appropriate position.

​    Dynamic Input/Output Streams

​    

Kafka Connect is intended to define bulk data copying jobs, such as copying an entire database rather than creating many jobs to copy each table individually. One consequence of this design is that the set of input or output streams for a connector can vary over time. Source connectors need to monitor the source system for changes, e.g. table additions/deletions in a database. When they pick up changes, they should notify the framework via the ConnectorContext object that reconfiguration is necessary. For example, in a SourceConnector:

​        if (inputsChanged())

​                    this.context.requestTaskReconfiguration();

​                

​    The framework will promptly request new configuration information and update the tasks, allowing them to gracefully commit their progress before reconfiguring them. Note that in the SourceConnector this monitoring is currently left up to the connector implementation. If an extra thread is required to perform this monitoring, the connector must allocate it itself. Ideally this code for monitoring changes would be isolated to the Connector and tasks would not need to worry about them. However, changes can also affect tasks, most commonly when one of their input streams is destroyed in the input system, e.g. if a table is dropped from a database. If the Task encounters the issue before the Connector, which will be common if the Connector needs to poll for changes, the Task will need to handle the subsequent error. Thankfully, this can usually be handled simply by catching and handling the appropriate exception. SinkConnectors usually only have to handle the addition of streams, which may translate to new entries in their outputs (e.g., a new database table). The framework manages any changes to the Kafka input, such as when the set of input topics changes because of a regex subscription. SinkTasks should expect new input streams, which may require creating new resources in the downstream system, such as a new table in a database. The trickiest situation to handle in these cases may be conflicts between multiple SinkTasks seeing a new input stream for the first time and simultaneously trying to create the new resource. SinkConnectors, on the other hand, will generally require no special code for handling a dynamic set of streams.

​    Connect Configuration Validation

​    

Kafka Connect allows you to validate connector configurations before submitting a connector to be executed and can provide feedback about errors and recommended values. To take advantage of this, connector developers need to provide an implementation of config() to expose the configuration definition to the framework. The following code in FileStreamSourceConnector defines the configuration and exposes it to the framework.

​        private static final ConfigDef CONFIG_DEF = new ConfigDef()

​                    .define(FILE_CONFIG, Type.STRING, Importance.HIGH, "Source filename.")

​                        .define(TOPIC_CONFIG, Type.STRING, Importance.HIGH, "The topic to publish data to");

​                        

​        public ConfigDef config() {

​                    return CONFIG_DEF;

​                    }

​            

​    ConfigDef class is used for specifying the set of expected configurations. For each configuration, you can specify the name, the type, the default value, the documentation, the group information, the order in the group, the width of the configuration value and the name suitable for display in the UI. Plus, you can provide special validation logic used for single configuration validation by overriding the Validator class. Moreover, as there may be dependencies between configurations, for example, the valid values and visibility of a configuration may change according to the values of other configurations. To handle this, ConfigDef allows you to specify the dependents of a configuration and to provide an implementation of Recommender to get valid values and set visibility of a configuration given the current configuration values. Also, the validate() method in Connector provides a default validation implementation which returns a list of allowed configurations together with configuration errors and recommended values for each configuration. However, it does not use the recommended values for configuration validation. You may provide an override of the default implementation for customized configuration validation, which may use the recommended values.

​    Working with Schemas

​    

The FileStream connectors are good examples because they are simple, but they also have trivially structured data -- each line is just a string. Almost all practical connectors will need schemas with more complex data formats. To create more complex data, you'll need to work with the Kafka Connect data API. Most structured records will need to interact with two classes in addition to primitive types: Schema and Struct. The API documentation provides a complete reference, but here is a simple example creating a Schema and Struct:

​    Schema schema = SchemaBuilder.struct().name(NAME)

​            .field("name", Schema.STRING_SCHEMA)

​                .field("age", Schema.INT_SCHEMA)

​                .field("admin", new SchemaBuilder.boolean().defaultValue(false).build())

​                .build();

​                

​    Struct struct = new Struct(schema)

​            .put("name", "Barbara Liskov")

​                .put("age", 75);

​            

​    If you are implementing a source connector, you'll need to decide when and how to create schemas. Where possible, you should avoid recomputing them as much as possible. For example, if your connector is guaranteed to have a fixed schema, create it statically and reuse a single instance. However, many connectors will have dynamic schemas. One simple example of this is a database connector. Considering even just a single table, the schema will not be predefined for the entire connector (as it varies from table to table). But it also may not be fixed for a single table over the lifetime of the connector since the user may execute an ALTER TABLE command. The connector must be able to detect these changes and react appropriately. Sink connectors are usually simpler because they are consuming data and therefore do not need to create schemas. However, they should take just as much care to validate that the schemas they receive have the expected format. When the schema does not match -- usually indicating the upstream producer is generating invalid data that cannot be correctly translated to the destination system -- sink connectors should throw an exception to indicate this error to the system.

​    Kafka Connect Administration

​    

Kafka Connect's REST layer provides a set of APIs to enable administration of the cluster. This includes APIs to view the configuration of connectors and the status of their tasks, as well as to alter their current behavior (e.g. changing configuration and restarting tasks).

Kafka connect的REST服务层提供了一些管理Kafka connect 集群的API。这些API的功能包括查看连接器配置及其任务的状态，同时也可以改变连接器当前的运转情况（例如改变配置或者重启任务）。

When a connector is first submitted to the cluster, the workers rebalance the full set of connectors in the cluster and their tasks so that each worker has approximately the same amount of work. This same rebalancing procedure is also used when connectors increase or decrease the number of tasks they require, or when a connector's configuration is changed. You can use the REST API to view the current status of a connector and its tasks, including the id of the worker to which each was assigned. For example, querying the status of a file source (using GET /connectors/file-source/status) might produce output like the following:

​    {

​        "name": "file-source",

​        "connector": {

​            "state": "RUNNING",

​                "worker_id": "192.168.1.208:8083"

​            },

​        "tasks": [

​            {

​                "id": 0,

​                "state": "RUNNING",

​                "worker_id": "192.168.1.209:8083"

​                }

​            ]

​        }

​        

​    Connectors and their tasks publish status updates to a shared topic (configured with status.storage.topic) which all workers in the cluster monitor. Because the workers consume this topic asynchronously, there is typically a (short) delay before a state change is visible through the status API. The following states are possible for a connector or one of its tasks:

连接器和连接器中的任务发布状态到一个Kafka topic中（配置在 status.storage.topic），是整个集群中所有worker的监视器。因为worker是以异步的方式消费这个topic，所有通过 status API 获取的状态信息会有较短时间的延迟。连接器或连接器的任务可能有如下状态：

UNASSIGNED: The connector/task has not yet been assigned to a worker.

RUNNING: The connector/task is running.

PAUSED: The connector/task has been administratively paused.

FAILED: The connector/task has failed (usually by raising an exception, which is reported in the status output).

In most cases, connector and task states will match, though they may be different for short periods of time when changes are occurring or if tasks have failed. For example, when a connector is first started, there may be a noticeable delay before the connector and its tasks have all transitioned to the RUNNING state. States will also diverge when tasks fail since Connect does not automatically restart failed tasks. To restart a connector/task manually, you can use the restart APIs listed above. Note that if you try to restart a task while a rebalance is taking place, Connect will return a 409 (Conflict) status code. You can retry after the rebalance completes, but it might not be necessary since rebalances effectively restart all the connectors and tasks in the cluster.

在大多数情况下，连接器和连接器中的任务状态都是一致的，不过当发生改变或者任务失败时，他们会有短暂时间的状态不一致。例如，当一个连接器起动时，在连接器及其任务状态全部转变为 running 前会有明显的状态更新延迟。当任务失败而连接器没有自动的重启失败的任务，也会出现状态不一致的情况。你可以上面提到的重启API，手动的重启连接器或者任务。注意，当年重启一个任务时，重平衡过程会占用任务位置，connect 会返回409（冲突）状态码。当重平衡结束以后，你可以重试获取状态，但实际也没必要，因为一个实际上重平衡过程会重启kafka connect cluster中的所有连接器和任务。

It's sometimes useful to temporarily stop the message processing of a connector. For example, if the remote system is undergoing maintenance, it would be preferable for source connectors to stop polling it for new data instead of filling logs with exception spam. For this use case, Connect offers a pause/resume API. While a source connector is paused, Connect will stop polling it for additional records. While a sink connector is paused, Connect will stop pushing new messages to it. The pause state is persistent, so even if you restart the cluster, the connector will not begin message processing again until the task has been resumed. Note that there may be a delay before all of a connector's tasks have transitioned to the PAUSED state since it may take time for them to finish whatever processing they were in the middle of when being paused. Additionally, failed tasks will not transition to the PAUSED state until they have been restarted.

临时停止连接器的功能有些场景挺有用的。例如，当远端系统在维护的时候，停止拉取新数据可以避免发大量的日志报警邮件。在这个使用案例中，连接器提供了停止和恢复的接口。当一个source connector停止了，Connect 会停止从上游拉取新数据。当一个sink connector停止了， Connect 会停止向下游推送新数据。停止状态是持久化存储的，所以即使是重启集群，这个连接器也不会又开始处理数据，除非这些任务已经通过接口触发恢复了。注意，停止一个连接器的所有任务会有一定的状态变更延迟，因为在停止前，连接器需要花点时间处理完正在处理中的数据。此外，失败的任务在完成重启之前，也不会转变到停止状态。