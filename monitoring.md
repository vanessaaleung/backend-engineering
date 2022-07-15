# Monitoring
_The systematic process of collecting data over time_

- [Modeling](#modeling)
- [Time](#time)
- [Making a Graph](#making-a-graph)

- make sure our system behaves within predefined limits
- detect anomalies
- correlate consequences to causes
- Data/Metrics
  - example: # of requests hitting your server
  - Meter: a rate of events such or # requests per seconds
  - Latency: mean/median/XX percentile, histogram
  - Gauge: a single value, such as the current memory or CPU

## Modeling
- Use tags to describe what the metric is measuring
```json
{
  "role": "database",
  "host": "foo.guc3.net",
  "what": "cpu-idel-percentage", // name of the metric
  "type": "meter",
  "value": 1.0,
  "timestamp": 1234567
}
```

## Time
- Metrics are collected via Java
- Emitted at regular interval like 30 seconds


## Making a Graph
- Use MMA: have the dashboards declared in code
- Grafana
- Aggregation: transform and combine raw data points
  - ForEach: apply the aggregation for each time series
  - GroupBy: apply the aggregation for each subset of series (e.g. finding the max datapoint for each site)
  - Collapse: apply the aggregation for everything
  - Filters: filter out results, useful when we have many time series
- Resolution: e.g. show one datapoint per one hour
- Tips
  - Use ForEach + GroupBy: think horizontally then vertically
  - Latencies: never sum(), use max() or avg()
  - Ratios: same as above never sum()
