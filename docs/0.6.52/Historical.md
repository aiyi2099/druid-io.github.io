---
layout: doc_page
---
Historical
=======

Historical nodes load up historical segments and expose them for querying.

Quick Start
-----------
Run:

```
io.druid.cli.Main server historical
```

With the following JVM configuration:

```
-server
-Xmx256m
-Duser.timezone=UTC
-Dfile.encoding=UTF-8

druid.host=localhost
druid.service=historical
druid.port=8081

druid.zk.service.host=localhost

druid.server.maxSize=10000000000

# Change these to make Druid faster
druid.processing.buffer.sizeBytes=100000000
druid.processing.numThreads=1

druid.segmentCache.locations=[{"path": "/tmp/druid/indexCache", "maxSize"\: 10000000000}]
```

Note: This will spin up a Historical node with the local filesystem as deep storage.

JVM Configuration
-----------------
The historical module uses several of the default modules in [Configuration](Configuration.html) and has no uniques configs of its own.

Running
-------

```
io.druid.cli.Main server historical
```

Loading and Serving Segments
----------------------------

Each historical node maintains a constant connection to Zookeeper and watches a configurable set of Zookeeper paths for new segment information. Historical nodes do not communicate directly with each other or with the coordinator nodes but instead rely on Zookeeper for coordination.

The [Coordinator](Coordinator.html) node is responsible for assigning new segments to historical nodes. Assignment is done by creating an ephemeral Zookeeper entry under a load queue path associated with a historical node. For more information on how the coordinator assigns segments to historical nodes, please see [Coordinator](Coordinator.html).

When a historical node notices a new load queue entry in its load queue path, it will first check a local disk directory (cache) for the information about segment. If no information about the segment exists in the cache, the historical node will download metadata about the new segment to serve from Zookeeper. This metadata includes specifications about where the segment is located in deep storage and about how to decompress and process the segment. For more information about segment metadata and Druid segments in general, please see [Segments](Segments.html). Once a historical node completes processing a segment, the segment is announced in Zookeeper under a served segments path associated with the node. At this point, the segment is available for querying.

Loading and Serving Segments From Cache
---------------------------------------

Recall that when a historical node notices a new segment entry in its load queue path, the historical node first checks a configurable cache directory on its local disk to see if the segment had been previously downloaded. If a local cache entry already exists, the historical node will directly read the segment binary files from disk and load the segment.

The segment cache is also leveraged when a historical node is first started. On startup, a historical node will search through its cache directory and immediately load and serve all segments that are found. This feature allows historical nodes to be queried as soon they come online.

Querying Segments
-----------------

Please see [Querying](Querying.html) for more information on querying historical nodes.

For every query that a historical node services, it will log the query and report metrics on the time taken to run the query.