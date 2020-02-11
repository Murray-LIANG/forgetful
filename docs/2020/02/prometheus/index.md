# Prometheus


https://prometheus.io/docs/instrumenting/exporters/

## Unity Exporter Ideas

We could develop an exporter for Unity. An prometheus exporter works as a server and prometheus will scrape the metrics from it in a specified interval.

1. Develop an exporter for Unity. 
2. Register some real time queries for some common resource (or supporting configured via conf file)
3. When prometheus scrapes the metrics, `Collect` method of Unity exporter will be called. It will get the latest query result from Unity in the logic of `Collect`.

**NOTE: The minimal interval of real time query supported by Unity is 60 sec (1 min). We support to configure the prometheus scape interval. But if the prometheus scape interval is less than 60 sec (i.e. 5 sec), there could be 12 metrics at most collected by prometheus are the same.**

