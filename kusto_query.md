# Kusto
## Useful links
- [databricks-monitoring](https://docs.microsoft.com/en-us/azure/architecture/databricks-monitoring/databricks-observability)
- [spark-monitoring-deploy-json](https://raw.githubusercontent.com/mspnp/spark-monitoring/master/perftools/deployment/loganalytics/logAnalyticsDeploy.json)
- [spark-monitoring-metrics](https://github.com/mspnp/spark-monitoring/blob/main/perftools/deployment/readme.md#step-1-deploy-log-analytics-with-spark-metrics)

## Useful queries
```sh
# Performance testing 
# Show metrics CPU/Memory/Number of records updated every minute
SparkMetric_CL
| where name_s contains "driver.jvm.total."
| where executorId_s == "driver"
| extend memUsed_GB = value_d / 1000000000
| project TimeGenerated, memUsed_GB
| summarize max(memUsed_GB) by bin(TimeGenerated, 1m)
| join kind=inner (
    SparkMetric_CL
| where name_s contains "executor.RunTime"
| project TimeGenerated , runTime=count_d
| join kind=inner (
    SparkMetric_CL
| where name_s contains "executor.cpuTime"
| project TimeGenerated , cpuTime=count_d/1000000, clusterName_s
) on TimeGenerated
| extend cpuUsage=(cpuTime/runTime)*100
| summarize max(cpuUsage) by bin(TimeGenerated, 1m), clusterName_s
| order by TimeGenerated asc nulls last
) on TimeGenerated
|join kind=fullouter (
SparkLoggingEvent_CL
| where Message has 'psr_msg_type' and Message has 'tracking'
| extend _message=parse_json(tostring(parse_json(Message).message))
| extend _cnt=tolong(_message.['cnt'])
| extend _table_name=_message.['table_name']
| extend _tmp=strcat(tostring(_message.['ts']),":00")
| extend _ts=split(_tmp,",")
| extend _timet=strcat(_ts[0],_ts[1])
| extend _time=todatetime(_timet)
| project  _cnt, _table_name, _time
| summarize max(_cnt) by _time,tostring(_table_name)
| order by _time asc nulls last
) on $left.TimeGenerated == $right._time;
```
``` sh
# Databricks errors
SparkLoggingEvent_CL
| where logger_name_s !contains "hon_edm_solution"
| where Level == "ERROR"
| where tostring(traceInfo_s) contains "job" and tostring(traceInfo_s) contains "run"
# regex match job-1324213543-run-245834589
| extend job_id = extract("job.([0-9]+)",1,tostring(traceInfo_s))
| extend task_run_id =extract("run.([0-9]+)",1,tostring(traceInfo_s))
| project TimeGenerated, job_id, task_run_id, tostring(Message), logger_name_s, Level
```
```sh
# Log errors
SparkLoggingEvent_CL
| where Message has 'source_system' and Message has "task_run_id"
| where Level == 'ERROR'
| extend _job_name = tostring(parse_json(Message).job_name)
| extend _job_id = tostring(parse_json(Message).job_id)
| extend _task_run_id = tostring(parse_json(Message).task_run_id)
| extend _message = tostring(parse_json(Message).message)
| project TimeGenerated, _job_name, _job_id, _task_run_id, _message, Level
```
```sh
# Pipeline status 
# Elapsed time/duration for a task
let results = SparkLoggingEvent_CL
    | where Message has 'source_system' and Message has "task_run_id"
    | extend _job_name = tostring(parse_json(Message).job_name)
    | extend _job_id = tostring(parse_json(Message).job_id)
    | extend _task_run_id = tostring(parse_json(Message).task_run_id)
    | extend _message1 = tostring(parse_json(Message).message)
    | summarize endtime1 = arg_max(TimeGenerated, *) by _job_name
    | join kind = inner (
        SparkLoggingEvent_CL
        | where Message has 'source_system' and Message has "task_run_id"
        | extend _job_name = tostring(parse_json(Message).job_name)
        | extend _job_id = tostring(parse_json(Message).job_id)
        | extend _task_run_id = tostring(parse_json(Message).task_run_id)
        | extend _message2 = tostring(parse_json(Message).message)
        )
        on _job_id;
results
| summarize starttime = arg_min(TimeGenerated, *), endtime = arg_max(TimeGenerated, *) by _task_run_id1
| where endtime > starttime and endtime - starttime > 3m
| extend NewAge=replace(@'master_data_source', @'L2R', _job_name)
| extend NewAge=replace(@'master_data_transform', @'R2P', NewAge)
| extend NewAge=replace(@'event_data_transform', @'L2P', NewAge)
| extend NewAge=replace(@'event_data_curation', @'P2C', NewAge)
| extend tmp = split(NewAge, "_")
| extend stage = tmp[-1]
| extend _source_system = tostring(parse_json(Message).source_system)
| extend _customer_id = tostring(parse_json(Message).customer_id)
| project
    _job_name,
    _job_id1,
    _source_system,
    _customer_id,
    _task_run_id1,
    stage,
    starttime,
    endtime,
    duration = datetime_diff('minute', endtime, starttime)
| order by starttime desc nulls last
```

```sh
# Spark streaming jobs
SparkLoggingEvent_CL
| where Message contains "Streaming query made progress"
| extend streaming_progress = parse_json(replace_string(Message, "Streaming query made progress: ", ""))
| extend timestamp = tostring(streaming_progress.timestamp)
| extend id = tostring(streaming_progress.id)
| extend runId = tostring(streaming_progress.runId)
| extend name = tostring(streaming_progress.name)
| extend batchId = tostring(streaming_progress.batchId)
| extend numInputRows = tostring(streaming_progress.numInputRows)
| extend inputRowsPerSecond = tostring(streaming_progress.inputRowsPerSecond)
| extend processedRowsPerSecond = tostring(streaming_progress.processedRowsPerSecond)
| extend durationMs_latestOffset = tostring(streaming_progress.durationMs.latestOffset)
| extend durationMs_triggerExecution = tostring(streaming_progress.durationMs.triggerExecution)
| extend sink_description = tostring(streaming_progress.sink.description)
| extend sink_numOutputRows = tostring(streaming_progress.sink.numOutputRows)
| project
    timestamp,
    id,
    runId,
    name,
    batchId,
    numInputRows,
    inputRowsPerSecond,
    processedRowsPerSecond,
    durationMs_latestOffset,
    durationMs_triggerExecution,
    sink_description,
    sink_numOutputRows 
```
