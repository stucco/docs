### Types of metrics

1. Source metrics ("Domain-driven")
  * credibility of source  
    Based on survey data?  and/or config file?
  * credibility of author  
    Based on survey data?  and/or config file?
  * relevance of data source  
    This can be found by combining the per-document relevance scores below.  This could be done either directly, eg. by averaging the scores of all articles, or indirectly, eg. by finding the proportion of articles which pass the relevance filter.
1. Document metrics ("Document-driven")
  * relevance  
    This can be the score determined by the article classification algorithm that will run before the entity extraction.
  * uniqueness (novelty)  
    This will require comparing the extracted graph with the complete knowledge graph.  This must be done prior to merging the graphs.  This may use similar features of our API as merging does.
1. Extraction metrics ("Algorithm-driven")
  * confidence of extraction algorithms  
    The extraction algorithms will produce a score for all entities and relations found.  The node scores will require some aggregation of the entity and edge scores for its properties.
  * further details TBD

### Combining metrics
1. Node scores ...
1. Edge scores ...

### Using metrics

Metrics will be used for:

* setting priority of the data collection queue
* determining whether to process or ignore potentially-relevant documents
* determine which extraction algorithms to run?
* providing input to the alignment process
* setting thresholds when querying the database (eg. ignore edges with weight under, say, 0.40)