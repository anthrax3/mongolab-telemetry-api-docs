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
| pageFaults | faults/millisecond | Rate of page faults |
| timeFlushingToDisk | milliseconds | Background flush average |
| currentConnections | - | Number of open connections |
| effectiveLockPercentage | percent | Effective lock |
| operationsWaitingForGlobalLock | Number of operations | Total queued
| readsWaitingForGlobalLock | Number of operations | Readers queued
| writesWaitingForGlobalLock | Number of operations | Writers queued
| networkBytesOut | bytes/millisecond | Rate of network bytes out
| networkBytesIn | bytes/millisecond | Rate of network bytes in
| cpuIOWait | Percent | CPU IOWait
| cpuSystem | Percent | System CPU
| cpuUser | Percent | User CPU


