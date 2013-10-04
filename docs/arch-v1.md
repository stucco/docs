# Stucco Architecture

In general, the following guidelines should be followed:

* Configuration files should be defined in [yaml](http://yaml.org/)
* Communication outside of Storm should use AMQP via [RabbitMQ](http://www.rabbitmq.com/)
* Messages should be formatted as [JSON](http://json.org/)
* Logs should be sent to [logstash](http://logstash.net/) as JSON via the [TCP Input](http://logstash.net/docs/1.2.1/inputs/tcp)


---------------------------------------------------------------------

## Collection

### Description

The collectors pull data or process data streams and push the collected data (documents) into the message queue. Each type of collector is independent of others. The collectors can be implemented in any language. 

Collectors can either send messages with data or messages without data. For messages without data, the collector will add the document to the document store and attach the returned `id` to the message.

Collectors can either be stand-alone and run on any host, or be host-based and designed to collect data specific to that host.

### Collector Types

#### Web collector

Web collectors pull a document via HTTP/HTTPS given a URL. Documents will be decompressed, but no other processing will occur. 

##### Configuration

##### Content format

Various (e.g. HTML, XML, CSV). 

#### Scraping collector

Scrapers pull data embedded within a web page via HTTP/HTTPS given a URL and an HTML pattern. 

##### Configuration

##### Content format

HTML.

#### RSS collector

RSS collectors pull an RSS/ATOM feed via HTTP/HTTPS given a URL.

##### Configuration

##### Content format

XML.

#### Twitter collector

Twitter collectors pull Tweet data via HTTP from the Twitter Search REST API given a user (@username), hashtag (#keyword), or search term.

##### Configuration

Twitter API Key

##### Content format

JSON.

#### Netflow collector

Netflow collectors will collect from [Argus](http://www.qosient.com/argus/). The collector will listen for argus streams using `ra` tool and convert to XML and pipe to send the flow data to the message queue as a string.

#### Host-based collectors

Host-based collectors collect data from an individual host using agents.

Host-based collectors should be able to collect and forward:

* System logs
* [Hone](https://github.com/HoneProject/) data
* Installed packages

##### Configuration

// TODO

##### Content format

If we are writing the collector, JSON. If not, whatever format the agent uses.

### State

Stand-alone collectors should not store or require any state (state should be stored with the scheduler). Host-based collectors may need to store state (e.g. when the last collection was run).

### Input Transport Protocol

Input transport protocol will depend on the type of collector.

### Input Format

Input format will depend on the type of collector.

### Output Transport Protocol

[Advanced Message Queuing Protocol (AMQP)](http://www.amqp.org/)

### Output Format

There are two types of output messages: (1) messages with data and (2) messages without data that reference an ID in the document store.

All messages should be formatted as a JSON object.
All JSON messages should be converted to strings (i.e. using [JSON.stringify()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)).

All messages should contain the following fields. Either `document-id` or `content` must be included.

* `sourceName` (string) - the name of the source of the message; e.g. ‘cve’ (required)
* `sourceUrl` (string) - the url of the document – this should be a link to the document itself; e.g. http://cve.mitre.org/data/downloads/allitems.xml, Note: in some cases this URL will point ot a generic URL that requires an API.
In those cases we will provide the URL and then describe whether we have an API call with it and what metadata was used to get the content from the site. (optional)
* `sourceMetadataUsed` (string) - will contain context specific metadata for the site being accessed.
For example if the URL requires a set of query terms or terms for access those are listed (optional)
* `dateCollected` (number) - the unix timestamp (i.e. date +%s) when the document was collected; e.g. 1377114034 (required)
* `collectorType` (string) - this defines the type content, whether exogenous or endogenous, enumerated types: WEBContent, hone, assets, netflow  (optional)
* `documentName` (string) - the name of the document represented by the message; e.g. ‘allitems’ (optional)
* `documentCreation` (number) - the unix timestamp (i.e. date +%s) of the document creation; e.g. 1365012038 (optional)
* `documentModification` (number) - the unix timestamp (i.e. date +%s) of the document creation; e.g. 1366135032 (optional)
* `contentType` (string) - the MIME type of the message; e.g. text/xml @see http://www.iana.org/assignments/media-types (required)
* `content` (base64 encoded string) - the JSON object to send to Storm, currently BASE64 encoded  (optional)
* `documentId` (string) - the ID provided by the document store; only provide an ID if content is not provided (optional)

An example output would be:

// TODO - ADD EXAMPLE

---------------------------------------------------------------------

## Scheduler

### Description

### Configuration

### State


---------------------------------------------------------------------

## Message Queue (MQ)

### Description

The message queue accepts input (documents) from the collectors and pushes the documents into the processing pipeline.
The collectors should send data into the default exchange into the queue named `stucco`.

### State

The queue should hold messages until they have been processed by the Storm Spout.

### Input format

The message queue should pass on the data as is from collectors.

### Output format

The message queue should pass on the data as is from collectors.

---------------------------------------------------------------------

## RT (Storm)

[RT is the Real-time processing component of Stucco.](https://github.com/stucco/rt)

![overview of architecture](https://raw.github.com/stucco/docs/master/docs/arch.png) This diagram shows how RT connects to the other components described here.

It will accept messages through AMQP, formatted as described in the "output format" subsection of "collection," above.

If the data is not already included in the document store, it will be added.

The data it receives will be transformed into a graph, consistant with the [ontology definition](https://github.com/stucco/ontology), and then added into the neo4j instance.

---------------------------------------------------------------------

## Document Service

### Description

The document store contains the raw documents. The backend is implemented in Riak, but the document-service API abstracts that.

### Commands

#### Add Document

Be sure to set the `content-type` of the HTTP header when adding documents to the appropriate type (e.g. `content-type: application/json` for JSON data or `content-type: application/pdf` for PDF files.

Routes:

* `server:port/add` - add a document and autogenerate an id
* `server:port/add/id` - add a document with a specific id

#### Get Document

The `accept-encoding` can be set to `gzip` to compress the communication (i.e., `accept-encoding: application/gzip`).

The `accept` command can be one of the following: `application/json`, `text/plain`, or `application/octet-stream`. Use `application/octet-stream` for PDF files and other binary data.

Routes:

* `server:port/get/id` - get a document based on the specific id

### Output Transport Protocol

HTTP.

### Input format

JSON.

### Output Transport Protocol

HTTP.

### Output format

JSON.


---------------------------------------------------------------------

## Graph Store API


---------------------------------------------------------------------

## Log Server

### Description

A log server collects and aggregates logs from each of the components. The log server is implemented as a logstash server. Sending logs to the server should be done via the TCP input plugin. 

The Vagrant VM is set up with a logstash server to aggregate log files. Log files can be input in multiple ways. There is a simple configuration set up in the Vagrantfile. The easiest way to send logs is to send them over a TCP connection. For an example in node.js and python, see https://gist.github.com/jgoodall/6323951

### Configuration

The logstash configuration for the TCP input should look like this:

input {
  tcp {
    type => "stucco-tcp"
    port => 9563
    charset => "UTF-8"
    format => "json_event"
  }
}

### Input Transport Protocol

TCP, by default on port 9563.

### Input Format

JSON. The log message should be a stringified JSON object with the log message in the @message field. Optionally, an array of tags can be added to the @tags field. Additional fields can be added as an object to the @fields field as key-value pairs.

### Output

To view the logs, use HTTP with a web browser. The web interface to logstash is implemented by [kibana (v3)](http://www.elasticsearch.org/overview/kibana/).
