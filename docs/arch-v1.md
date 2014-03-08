# Stucco Architecture

![Architecture diagram](../diagrams/arch.png)


## Guidelines

* Default configuration files should be defined in [yaml](http://yaml.org/) 
* Component configuration should be read from the [configuration service](#configuration-service)
* Messages between services should be formatted as [JSON](http://json.org/) and sent via HTTP
* Logs should be sent to [logstash](http://logstash.net/) as JSON via the [TCP Input](http://logstash.net/docs/1.2.1/inputs/tcp)
* Messages into Storm should be via AMQP using [RabbitMQ](http://www.rabbitmq.com/)


---------------------------------------------------------------------

## Prerequisites

* [RabbitMQ](http://www.rabbitmq.com/): the message queue that interfaces between the collectors and the [rt processing pipeline](#rt)
* [Riak](http://basho.com/riak/): the distributed key-value store that stores the raw documents, accessible through the [document-service](#document-service)
* [Titan](http://thinkaurelius.github.io/titan/): the distributed graph database that stores the knowledge graph, accessible through the [query-service](#query-service)
* [Logstash](http://logstash.net/): tool for collecting and managing log files from stucco components using [elasticsearch](http://www.elasticsearch.org/overview/elasticsearch/) and [kibana](http://www.elasticsearch.org/overview/kibana/)
* [etcd](https://github.com/coreos/etcd): tool for sharing configuration and service discovery from stucco components

---------------------------------------------------------------------

## Collection

### Description

The collectors pull data or process data streams and push the collected data (documents) into the message queue. Each type of collector is independent of others. The collectors can be implemented in any language. 

Collectors can either send messages with data or messages without data. For messages without data, the collector will add the document to the document store and attach the returned `id` to the message.

Collectors can either be stand-alone and run on any host, or be host-based and designed to collect data specific to that host.

### Collector Types

#### Web collector

Web collectors pull a document via HTTP/HTTPS given a URL. Documents will be decompressed, but no other processing will occur. 

##### Content format

Various (e.g. HTML, XML, CSV). 

#### Scraping collector

Scrapers pull data embedded within a web page via HTTP/HTTPS given a URL and an HTML pattern. 

##### Content format

HTML.

#### RSS collector

RSS collectors pull an RSS/ATOM feed via HTTP/HTTPS given a URL.

##### Content format

XML.

#### Twitter collector

Twitter collectors pull Tweet data via HTTP from the Twitter Search REST API given a user (@username), hashtag (#keyword), or search term.

##### Content format

JSON.

#### Netflow collector

Netflow collectors will collect from [Argus](http://www.qosient.com/argus/). The collector will listen for argus streams using `ra` tool and convert to XML and pipe to send the flow data to the message queue as a string.

##### Content format

**TODO.**

#### Host-based collectors

Host-based collectors collect data from an individual host using agents.

Host-based collectors should be able to collect and forward:

* System logs
* [Hone](https://github.com/HoneProject/) data
* Installed packages

##### Content format

If we are writing the collector, JSON. If not, whatever format the agent uses.

### State

Stand-alone collectors may require state state (state should be stored with the scheduler, such as the last time a site was downloaded). Host-based collectors may need to store state (e.g. when the last collection was run).

### Input Transport Protocol

Input transport protocol will depend on the type of collector.

### Input Format

Input format will depend on the type of collector.

### Output Transport Protocol

[Advanced Message Queuing Protocol (AMQP)](http://www.amqp.org/), as implemented in RabbitMQ. See the [concepts documentation](http://www.rabbitmq.com/tutorials/amqp-concepts.html) for information about AMQP and RabbitMQ concepts. See the [protocol documentation](http://www.rabbitmq.com/amqp-0-9-1-reference.html) for more on AMQP. Examples below are in [Go](http://golang.org/) using the [amqp package](http://godoc.org/github.com/streadway/amqp). [Other libraries](http://www.rabbitmq.com/devtools.html) should implement similar interfaces.

The RabbitMQ [exchange](http://www.rabbitmq.com/amqp-0-9-1-quickref.html#class.exchange) is exchange-type of `topic` with the exchange-name of `stucco`. 

The exchange declaration options should be:

    "topic",    // type
    true,       // durable
    false,      // auto-deleted
    false,      // internal
    false,      // noWait
    nil,        // arguments

The [publish](http://www.rabbitmq.com/amqp-0-9-1-reference.html#basic.publish) options should be:

    stucco,     // publish to an exchange named stucco
    <routingKey>, // routing to 0 or more queues
    false,      // mandatory
    false,      // immediate

The `<routingKeys>` format should be: `stucco.in.<data-type>.<data-name (optional)>`, where:

* data-type (required): the type or type of data, such as cve, nvd, maxmind, cpe, argus, hone.
* data-name (optional): the name of the data, such as the hostname of the sensor.

The [message options](http://www.rabbitmq.com/amqp-0-9-1-reference.html#class.basic) should be:

        DeliveryMode:    1,    // 1=non-persistent, 2=persistent
        Timestamp:       time.Now(),
        ContentType:     "text/plain",
        ContentEncoding: "",
        Priority:        1,    // 0-9
        HasContent:      true, // boolean
        Body:            <payload>,

`DeliveryMode` should be 'persistent'.

`Timestamp` should be automatically filled out by your amqp client library. If not, the publisher should specify.

`ContentType` should be "text/xml" or "text/csv" or "application/json" or "text/plain" (i.e. collectorType from the output format). This is dependent on the data source.

`ContentEncoding` may be required if things are, for example, gzipped.

`Priority` is optional.

`HasContent` is an application-specific part of the message header that defines whether or not there is content as part of the message. It should be defined in the message header field table using a boolean: `HasContent: true` (if there is data content) or `HasContent: false` (if the document service has the content). The spout will use the document service accordingly. This is the only application-specific data needed.

`Body` is the payload, either the document itself or the id if `HasContent` is false.

The corresponding binding keys for the [queue](http://www.rabbitmq.com/amqp-0-9-1-quickref.html#class.queue) defined in the spout will can use the wildcards to determine which spout should handle which messages:

* * (star) can substitute for exactly one word.
* # (hash) can substitute for zero or more words.

For example, `stucco.in.#` would listen for all input.

### Output Format

There are two types of output messages: (1) messages with data and (2) messages without data that reference an ID in the document store.



---------------------------------------------------------------------

## Scheduler

### Description

### State


---------------------------------------------------------------------

## Message Queue (MQ)

### Description

The message queue accepts input (documents) from the collectors and pushes the documents into the processing pipeline.
The message queue is implemented with [RabbitMQ](http://www.rabbitmq.com/), which implements the AMQP standard.

### Configuration

The queue should hold messages until they have been processed by the Storm Spout.

### Protocol

Input and output protocol is [AMQP 0-9-1](http://www.rabbitmq.com/protocol.html).

### Format

The message queue should pass on the data as is from collectors.

---------------------------------------------------------------------

## RT

### Description

[RT](https://github.com/stucco/rt) is the Real-time processing component of Stucco implemented as a [Storm](http://storm-project.net/) cluster.

This diagram shows how all the RT components are connected. The diagram at the very top shows how RT connects to the other components described in this document.

If the data it receives is not already included in the document store, it will be added.

The data it receives will be transformed into a graph, consistent with the [ontology definition](https://github.com/stucco/ontology), and then added into graph store.

### Input Transport Protocol
[See AMQP Spout](https://github.com/stucco/docs/blob/master/docs/arch-v1.md#amqp-spout)

### Input Format
[See AMQP Spout](https://github.com/stucco/docs/blob/master/docs/arch-v1.md#amqp-spout)

The spout will send an acknowledgement to the queue when the messages are received, so that the queue can release these resources.

### Output Transport Protocol
[See Alignment Bolt](https://github.com/stucco/docs/blob/master/docs/arch-v1.md#graph-bolt)

### Output Format
[See Alignment Bolt](https://github.com/stucco/docs/blob/master/docs/arch-v1.md#graph-bolt)

RT will also add the raw documents it receives to the document store if needed.

RT may add additional intermediate output (eg. partially labeled text documents) to the document store if needed.

### RT Components

#### AMQP Spout

##### Description
The AMQP spout pulls messages off the queue and pushes them to the UUID Bolt. 

##### Input Transport Protocol
[Advanced Message Queuing Protocol (AMQP)](http://www.amqp.org/)

##### Input Format

[See Collector's Output Format](#output-format)

##### Output Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Output Format
JSON object with the following fields:

* `json` (string) - the JSON message

#### UUID Bolt

##### Description
The UUID bolt generates and appends a universally unique identifier (UUID) to the message tuples.

The UUID is generated by computing a SHA-512 hash of the input string so that the UUID is deterministic based on the input.

##### Input Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Input Format
JSON object with the following fields:

* `json` (string) - the JSON message

##### Output Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Output Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `json` (string) - the JSON message

#### Route Bolt

##### Description
The Route bolt sends a tuple to the appropriate pipeline, depending on whether the tuple contains a structured document or unstructured document, based on the data source it was collected from.

##### Input Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Input Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `json` (string) - the JSON message

##### Output Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Output Format
JSON object with the following fields are submitted to either the "structured" or "unstructured" stream:

* `uuid` (string) - the hash of the JSON message
* `json` (string) - the JSON message

#### Parse Bolt

##### Description
The Parse bolt parses a structured document and produces its corresponding subgraph.

##### Input Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Input Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `json` (string) - the JSON message

##### Output Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Output Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `graph` (string) - the GraphSON version of the structured document

#### Extract Bolt

##### Description
The Extract bolt extracts an unstructured document's content either from the message, or by requesting the document from the document-service. The document content is then passed to bolts that can find domain-specific concepts.

##### Input Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Input Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `json` (string) - the JSON message

##### Output Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Output Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `text` (string) - the unstructured text from the document

#### Concept Bolt

##### Description
The Concept bolt finds domain-specific concepts within unstructured text.

##### Input Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Input Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `text` (string) - the unstructured text from the document

##### Output Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Output Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `text` (string) - the unstructured text from the document
* `concepts` (string) - JSON object representing the domain-specific concepts

#### Relation Bolt

##### Description
The Relation bolt discovers relationships between the concepts and constructs a subgraph of this knowledge.

##### Input Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Input Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `text` (string) - the unstructured text from the document
* `concepts` (string) - JSON object representing the domain-specific concepts

##### Output Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Output Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `graph` (string) - the GraphSON representation of the unstructured document's concepts (nodes) and relationships (edges)

#### Alignment Bolt

##### Description
The Alignment bolt aligns and merges the new subgraph into the full knowledge graph.

##### Input Transport Protocol
[Storm's Multilang Protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)

##### Input Format
JSON object with the following fields:

* `uuid` (string) - the hash of the JSON message
* `graph` (string) - the GraphSON subgraph representing a document

##### Output Transport Protocol
HTTP/REST

##### Output Format
GraphSON subgraph representing a document

---------------------------------------------------------------------

## Document Service

### Description

The [document-service](https://github.com/stucco/document-service) stores and makes available the raw documents. The backend storage for the document service is implemented in [Riak](http://basho.com/riak/).

### Commands

#### Add Document

Be sure to set the `content-type` of the HTTP header when adding documents to the appropriate type (e.g. `content-type: application/json` for JSON data or `content-type: application/pdf` for PDF files.

Routes:

* PUT `server:port/document` - add a document and autogenerate an id
* PUT `server:port/document/id` - add a document with a specific id

#### Get Document

The `accept-encoding` can be set to `gzip` to compress the communication (i.e., `accept-encoding: application/gzip`).

The `accept` command can be one of the following: `application/json`, `text/plain`, or `application/octet-stream`. Use `application/octet-stream` for PDF files and other binary data.

Routes:

* GET `server:port/document/id` - retrieve a document based on the specific id

### Input Transport Protocol

HTTP.

### Input format

[See Collector's Output Format](#output-format)

### Output Transport Protocol

HTTP.

### Output format

JSON.


---------------------------------------------------------------------

## Query Service

### Description

The Query Service provides the API for the Graph Store to allow the Alignment bolt, the Visualization/UI, and any third-party applications to interface with the graph database, implemented in [Titan](http://thinkaurelius.github.io/titan/).

This API provides a [GraphSON](https://github.com/tinkerpop/blueprints/wiki/GraphSON-Reader-and-Writer-Library) interface over HTTP.

The API will provide functions that facilitate common operations (eg. get a node by ID) and also allow arbitrary [Gremlin](https://github.com/tinkerpop/gremlin/wiki) queries.  (As the API matures, the use of arbitrary Gremlin queries will be removed or restricted to the Alignment bolt only.)

The API will be implemented with Rexter and a set of [Rexter Extensions.](https://github.com/tinkerpop/rexster/wiki/Extensions)

### Routes

* `host:port/graphs/graph/type/<typename>`  
  Returns a list of all nodes of type `<typename>`
* `host:port/graphs/graph/node/<nodename>`  
  Returns the node with the specified `<nodename>`
* `host:port/graphs/graph/tp/gremlin?<gremlinquery>`  
  Runs the given `<gremlinquery>` and returns any results

### Transport Protocol

HTTP.

### Transport Format

GraphSON.


---------------------------------------------------------------------

## Configuration Service

### Description

The configuration service hosts configuration information for all services. It is implemented in [etcd](http://coreos.com/using-coreos/etcd/).

The `etcd` configuration is [described here](https://github.com/coreos/etcd/blob/master/Documentation/configuration.md#configuration-file). The default stucco configuration is loaded into `etcd` when instantiated by the [config-loader](https://github.com/stucco/config-loader).  The default stucco configuration, [`stucco.yml`](https://github.com/stucco/config/blob/master/stucco.yml) is in the [config repo](https://github.com/stucco/config); edit that file to have the configuration changes loaded. The nested configuration in the `stucco.yml` gets translated to the url path, so, 

    stucco
      document-service
        port

Would be accessible using the following URL: `http://127.0.0.1:4001/v2/keys/stucco/document-service/port`.

### Usage

To communicate with the configuration service, either use HTTP or use [a client library](https://github.com/coreos/etcd/blob/master/Documentation/libraries-and-tools.md) that supports version 2. There is also a [command line client](https://github.com/coreos/etcdctl/). To test with HTTP, use `curl`:

    curl -L http://127.0.0.1:4001/v2/keys/mykey -XPUT -d value="this is awesome"
    curl -L http://127.0.0.1:4001/v2/keys/mykey


### Input Transport Protocol

HTTP/REST or using client library.

### Input Format

JSON (for HTTP), or specific to the client library used.

### Output Transport Protocol

JSON (for HTTP), or specific to the client library used.

### Output Format

HTTP/REST or using client library.


---------------------------------------------------------------------

## Logging Service

### Description

The logging service collects and aggregates logs from each of the components. The log server is implemented as a [logstash](http://logstash.net/) server with an [elasticsearch](http://www.elasticsearch.org/) backend. 

### Usage

Sending logs to the server should be done via the [TCP input plugin](http://logstash.net/docs/1.3.2/inputs/tcp). The port number for the development environment can be found in the `logstash` configuration section of the [Vagrantfile](https://github.com/stucco/dev-setup/blob/master/Vagrantfile).

The Vagrant VM is set up with a logstash server to aggregate log files. Log files can be input in multiple ways. There is a simple configuration set up in the Vagrantfile. The easiest way to send logs is to send them over a TCP connection. For an example in node.js and python, see https://gist.github.com/jgoodall/6323951

### Input Transport Protocol

TCP.

### Input Format

JSON. The log message should be a stringified JSON object with the log message in the @message field. Optionally, an array of tags can be added to the @tags field. Additional fields can be added as an object to the @fields field as key-value pairs.

### Output

To view the logs, use HTTP with a web browser. The web interface to logstash is implemented by [kibana (v3)](http://www.elasticsearch.org/overview/kibana/). For the kibana port number in the development environment, see `webserver_port` in the `kibana` configuration section of the [Vagrantfile](https://github.com/stucco/dev-setup/blob/master/Vagrantfile).
