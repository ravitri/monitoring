

# PromQL

## Aggregation Basics

### Gauge

* Gauges are a snapshot of state, and usually when aggregating them you want to take a sum, average, minimum or maximum.
    ```
    sum without(device,fstype,mountpoint,instance)(node_filesystem_size_bytes)
    ```

* Can be `sum`, `min`, `max`, `avg` 

### Counter

* Counter tracks the number or size of events. Mostly, what needs to be known is how quickly the counter is increasing over time. This is usually done using the `rate` function, though the `increase` and `irate` functions also operate on the counter values.
    ```
    rate(node_network_receive_bytes_total[5m])
    ```

* The `5m` says to provide with 5 minutes of data, so the returned value will be an average over the last 5 minutes.

* The output of `rate` is a gauge, so the same aggregations apply as for gauges. The `node_network_receive_bytes_total` metric has a `device` label, so if you aggregate it away you will get the total bytes received per machine per second.
    ```
    sum without (device)(rate(node_network_receive_bytes_total[5m]))
    ```

### Summary

* A Summary metric will usually contain both a `_sum` and `_count`, and somethimes a time series with no suffix with a `quantile` label. The `_sum` and `_count` are both counters.

* Example:
    ```
    sum without(handler)(rate(prometheus_http_response_size_bytes_sum[5m]))
    /
    sum without(handler)(rate(prometheus_http_response_size_bytes_count[5m]))
    ```

* The summary allows to calculate the average size of an event where we divide the `_sum` by `_count` (after taking a rate) to get an average over a time period.

### Histogram

* Histogram metrics allow you to track the distribution of the size of events, allowing you to calculate quantiles from them.

### Matchers

There are four kinds of matchers:

|Pattern|       Description     |
|:-----:|:---------------------:|
|  `=`  | Equals                |
|  `!=` | Does not Equal        |
|  `=~` | Matches Pattern       |
|  `!~` | Doesn't match Pattern |

Example:
```
node_filesystem_size_bytes{job="node",mountpoint=~"/run/.*",mountpoint!~"/run/user/.*"}
```

### Instant Vector

* An instant vector selector returns an **instant vector** of the most recent samples before the query evaluation time, which is to say a list of zero or more time series. Each of these time series will have one sample, and a sample contains both a value and a timestamp.

### Range Vector

* Unlike an instant vector selector which returns one sample per time series, a **range vector** selector can return many samples for each time series. Example,
    ```
    rate(process_cpu_seconds_total[1m])
    ```
* The `[1m]` turns the instant vector selector into a range vector selector, and instructs PromQL to return all time series matching the selector all samples for the minute up to the query evaluation time.

### Offset

* Offset allows you to take the evaluation time for a query, and on a per-selector basis put it back further back in time. Example, following query would get memory usage an hour before the query evaluation time.
    ```
    process_resident_memory_bytes{job="node"} offset 1h
    ```
* The usefulness of offset is in the following scenario where change in memory is to be known only in the past hour.
    ```
    process_resident_memory_bytes{job="node"}
    -
    process_resident_memory_bytes{job="node"} offset 1h
    ```

## Aggregation Operators

### Grouping

Aggregation operators only work on instant vectors.

#### without

* When aggregating metrics you should usually try to preserve such target labels (Eg. `job`, `instance`) and should use the `without` clause when aggregating to specify the specific labels you want to remove. Example,
    ```
    sum without(fstype, mountpoint)(node_filesystem_size_bytes)
    ```

#### by

* In addition to `without` there is a slo the `by` clause. Where `without` specifies the labels to remove, `by` specifies the labels to keep. Example,
    ```
    sum by(job, instance, device)(node_filesystem_size_bytes)
    ```
* You cannot use both `by` and `without` in the same aggregation.
* General preference is to use `without` than `by`.
* Two cases where `by` is more useful:
    1. Unlike `without`, `by` does not automatically drop **`__name__`** label.
        ```
        sort_desc(count by(__name__)({__name__=~".+"}))
        ```
    2. Where we want to remove any labels we don't know about.
        ```
        count by(release)(node_uname_info)
        ```


