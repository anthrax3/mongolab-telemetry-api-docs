# Metrics Glossary

This document summarizes the metrics most commonly seen on MongoLab Telemetry dashboards, and for which alert definitions are most commonly defined.
The string in column 1 represents the string that would be used as the `METRIC_ID` in an alert definition.

| Metric ID | Units | Description |
| :--------- | :----- | :----------- |
| commands | ops/millisecond | Rate of commands |
| deletes | ops/millisecond | Rate of deletes |
| getmores | ops/millisecond | Rate of getmores |
| inserts | ops/millisecond | Rate of inserts |
| queries | ops/millisecond | Rate of queries |
| updates | ops/millisecond | Rates of updates |
| currentConnections | - | Number of open connections |
| cpuIOWait* | percent | CPU IOWait
| cpuSystem* | percent | System CPU
| cpuUser* | percent | User CPU
| pageFaults | faults/millisecond | Rate of page faults |
| effectiveLockPercentage | percent | Effective lock |
| operationsWaitingForGlobalLock | mumber of operations | Total queued
| readsWaitingForGlobalLock | number of operations | Readers queued
| writesWaitingForGlobalLock | number of operations | Writers queued
| nonMappedVirtualMemory | megabytes | Non-mapped virtual memory
| totalDataPlusIndexSizeWithoutLocal | bytes | Total size of data plus indexes (without 'local' database)
| totalFileSizeWithoutLocal | bytes | Total file size (without 'local' database)
| totalFileSize | bytes | Total file size (includes 'local' database)
| timeFlushingToDisk | milliseconds | Background flush average |
| networkBytesOut | bytes/millisecond | Rate of network bytes out
| networkBytesIn | bytes/millisecond | Rate of network bytes in
| replicationLag | Milliseconds | Time that secondary is behind primary
| replicationOplogWindow | Hours | Time between newest and oldest entries in the oplog

* These CPU metrics are currently not normalized to the number of CPU cores on the VM. We will soon be publishing new, normalized CPU metrics that always sum to 100%.

