# Document alignment - merging subgraphs

## A. Merging Nodes:

Two broad categories: merging new nodes with canonical names available, and merging nodes without.  

1. New node has a canonical name:
  * If a matching canonical name is not found in the database, add the node.
  * If a matching canonical name is found in the database, properties should be merged (B) and edges should be merged (C)
2. New node does not have a canonical name:
  * Search for similar nodes, and add "same as" edges, with the appropriate probability (D)

## B. Merging Properties

In cases where the old and new values of a property differ, the new value will be determined by some function that is specified for that property.

These functions may make use of several other node properties, such as:

* value of the old node's property
* value of the new node's property
* old node confidence score
* new node confidence score
* old node source(s) and date(s)
* new node source and date

General process when merging nodes:

* for each conflicting property, update the old node's property as needed, with the appropriate strategy for that property.  eg:  
  old\_node[conflicting\_property] = resolve\_property\_with\_some\_strategy(conflicting\_property, old\_node, new\_node)
* (properties which had "null" for either the old or new value can be handled in the same way.)
* new_node will not be added to database
* edges to/from new_node will be moved to old_node, per section C.

Some general example functions:

    resolve_property_with_newest(const property_name, const old_node, const new_node){
      if( old_node["publishedDate"] < new_node["publishedDate"] ) //publishedDate is an integer unix timestamp.
        return new_node["property_name"]
      else
        return old_node["property_name"]
    }

    resolve_property_by_confidence(const property_name, const old_node, const new_node){
      if( old_node["confidence"] < new_node["confidence"] ) //confidence is a float between 0 and 1.
        return new_node["property_name"]
      else
        return old_node["property_name"]
    }

Other examples could include a weighted average by confidence scores, or functions that may be unique to a specific property, eg. an account's lastLogin property might always take the newest value, or a vulnerability's patchAvailable property might never change to False once a True value has been seen.

Merging node confidence scores - works the same as above, but will always use the function TBD.

## C. Merging Edges

  * If the inV and outV were both merged, and only one pair had an edge, keep that edge.
  * If the inV and outV were both merged, and both had an edge, merge those edge's properties, as described in B.
  * If either the inV or the outV was merged, but not both, the merged node should keep its edge with the un-merged node.
    * It should not duplicate this edge for any nodes sharing a "same as" edge with the un-merged node.

## D. Assigning probability to "same as" edges

  * This property reflects the probability that these two nodes are describing the same entity.
  * It will be a float between 0 and 1.
  * This is calculated by function TBD.
  * If the value is 0, (or below some threshold TBD,) the edge should be excluded.
  * If the value is 1, (or above some threshold TBD,) the two nodes should be merged.