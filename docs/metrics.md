## Purpose

Metrics will primarily be used to resolve conflicts during the alignment process.

The following may also make use of these metrics: 

* setting priority of the data collection queue
* determining whether to process or ignore potentially-relevant documents
* determine which extraction algorithms to run?
* setting the edge weight that can be used for setting thresholds when querying the database (eg. ignore edges with weight under, say, 0.40)


## Types of metrics

1. Source metrics ("Domain-driven")
  * credibility of source - statically defined in config file
  * credibility of author - statically defined in config file
  * relevance - dynamically determined by aggregating the document relevance scores.  This could be done either directly, eg. by averaging the scores of all articles, or indirectly, eg. by finding the proportion of articles which pass the relevance filter.
1. Document metrics ("Document-driven")
  * relevance - dynamically determined by the article classification algorithm that will run before the entity extraction.
  * uniqueness (novelty) -  dynamically determined based on the new knowledge inserted into the graph.  This will require comparing the extracted graph with the complete knowledge graph.  This must be done prior to aligning the graphs.  This may use similar features of our API as aligning does.
1. Extraction metrics ("Algorithm-driven")
  * confidence - dynamically determined by the NLP algorithms operating on unstructured text.  The extraction algorithms will produce a score for each node and edge.  The scores will require aggregation of the property scores.
