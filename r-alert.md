# Alerting

Pigsty uses Prometheus as the primary alerting system.

* [Prometheus](http://p.pigsty.cc/alerts) + [AlertManager](http://a.pigsty.cc/#/alerts) (primary)
* [Grafana](http://demo.pigsty.cc/d/pgsql-alert) (backup), not enabled by default



## Alert

Alerts are critical for daily fault response and improving system availability.

Missing alarms will lead to reduced availability, and false alarms will lead to reduced sensitivity, and it is necessary to design the alarm rules prudently.

* A reasonable definition of alert levels and the corresponding processing flow.
* A reasonable definition of alert indicators, removal of duplicate alert items, and replenishment of missing alert items.
* Scientific config of alert thresholds based on historical monitoring data reduces the false alert rate.
* Reasonably rationalize the special case rules to eliminate false alerts caused by maintenance work, ETL, and offline queries.



## Alert Taxonomy

**Categorized by source module**

* `INFRA`: Infra alerts: alerts generated by infra software such as Prometheus, Grafana, Consul, DNS, Nginx, etc.
* `NODES`: Node alerts, OS, hardware resources, infra software, load balancing, and other alerts, usually handled by DBA.
* `PGSQL`: PostgreSQL alerts, alerts from database/connection pool/load-balancing cluster, usually R&D and DBA concern, DBA handle.
* `REDIS`: Redis alerts, R&D and DBA attention, DBA handle.
* `……`： Application board alerts, with alerts by the business side itself, but DBA will set alerts for business metrics like QPS, TPS, Rollback, and Seasonality.
* Pigsty uses the `category` tag `{infra,pgsql,nodes,redis,....} ` to identify the level of alerts.

**Categorized by urgency**

* P0: CRIT: Incidents with a significant off-site impact require urgent intervention. For example, the primary is down; replication is down. (Incident)
* P1: WARN: Incidents with minor off-site impact, or incidents with redundant processing, requiring response processing at the minute level. (WARN)
* P2: INFO: Impending impact, let loose may worsen at the hourly level, requires a response at the hourly level. (Incident)
* Pigsty uses the `severity` tag `{CRIT, WARN, INFO}` to identify the level of urgency of the alert.

**Category by Indicator Type**

* Errors: PG Down, PGB Down, Exporter Down, Stream Replication Outage, Single Set Cluster Multi-Master.
* Traffic: QPS, TPS, Rollback, Seasonality
* Latency: Average Response Time, Replication Latency
* Saturation: Connection Stacking, Number of Idle Transactions, CPU, Disk, Age (Transaction Number), Buffer.



## Alert Visualization

Pigsty uses **timeline status charts** to present alert information in various monitor dashboards. The horizontal axis represents a period, and a color bar represents an alert event. Only alerts in the **Firing** state are displayed in the alert chart; alerts in the Pending state are usually hidden or shown in gray.



## Alert Rules

Alert rules can be roughly divided into four types: error, delay, saturation, and traffic. 

* Errors: mainly focus on the **aliveness** of each component, as well as network outages, brain fractures, and other abnormalities, and the level is usually high (P0|P1).
* Latency: mainly concerned with query response time, replication latency, slow queries, and long transactions.
* Saturation: mainly focus on CPU, disk (these two belong to system monitoring but are very important for DB), connection pool queue, number of database back-end connections, age (essentially the saturation of available thing numbers), SSD life, etc.
* Traffic: QPS, TPS, Rollback (traffic is usually related to business indicators belonging to business monitoring), seasonality of QPS, and a burst of TPS.



## Prometheus alert rule

The alert rules are defined using Prometheus syntax, and the full alerting rules are detailed in:

* [Infrastructure Alert Rule](https://github.com/Vonng/pigsty/blob/master/roles/prometheus/files/rules/infra.yml)
* [Host Node Alert Rules](https://github.com/Vonng/pigsty/blob/master/roles/prometheus/files/rules/nodes.yml)
* [PostgreSQL Cluster Alert Rules](https://github.com/Vonng/pigsty/blob/master/roles/prometheus/files/rules/pgsql.yml)
* [Redis Cluster Alert Rules](https://github.com/Vonng/pigsty/blob/master/roles/prometheus/files/rules/redis.yml)



## Typical alerts

### **Errors**

A database instance going down will immediately trigger a P0 alert.


```yaml
# database server down
- alert: PostgresDown
  expr: pg_up < 1
  for: 1m
  labels: { level: 0, severity: CRIT, category: pgsql }
  annotations:
    summary: "CRIT PostgresDown {{ $labels.ins }}@{{ $labels.instance }}"
    description: |
      pg_up[ins={{ $labels.ins }}, instance={{ $labels.instance }}] = {{ $value }} < 1
      http://g.pigsty/d/pgsql-instance?var-ins={{ $labels.ins }}
```

When using PostgreSQL in a production environment, the effect of Pgbouncer failure is basically equivalent to Postgres failure, and its survivability alert rule level is unified with Postgres.


```yaml
# database connection pool down
- alert: PgbouncerDown
  expr: pgbouncer_up < 1
  for: 1m
  labels: { level: 0, severity: CRIT, category: pgsql }
  annotations:
    summary: "CRIT PostgresDown {{ $labels.ins }}@{{ $labels.instance }}"
    description: |
      pgbouncer_up[ins={{ $labels.ins }}, instance={{ $labels.instance }}] = {{ $value }} < 1
      http://g.pigsty/d/pgsql-instance?var-ins={{ $labels.ins }}
```

Monitor agent Exporter downtime usually indicates a severe failure: HAProxy and Node Exporter downtime usually means that the LB and database nodes themselves are down and need to be focused on.


```yaml
#==============================================================#
#                       Agent Aliveness                        #
#==============================================================#
# node & haproxy aliveness are determined directly by exporter aliveness
# including: node_exporter, pg_exporter, pgbouncer_exporter, haproxy_exporter
- alert: AgentDown
  expr: agent_up < 1
  for: 1m
  labels: { level: 0, severity: CRIT, category: infra }
  annotations:
    summary: 'CRIT AgentDown {{ $labels.ins }}@{{ $labels.instance }}'
    description: |
      agent_up[ins={{ $labels.ins }}, instance={{ $labels.instance }}] = {{ $value  | printf "%.2f" }} < 1
      http://g.pigsty/d/pgsql-alert?viewPanel=22
```

The duration threshold for all survivability detection is set to 1 minute, which for a 15s acquisition cycle typically means four consecutive failed probes. Regular fast restart operations usually do not trigger survivability alerts.



**Cluster brain fracture partitioning**

The cluster should have only one **partition**. If the number of partitions in the cluster is not 1, it means the cluster has entered an abnormal state: unwritable or brain fractured, which will trigger the P0 alarm immediately. Because the detection threshold is 1 minute, so the regular Failover and Switchover usually do not easily trigger this alert.

```yaml
# cluster partition: split brain
- alert: PostgresPartition
  expr: pg:cls:partition != 1
  for: 1m
  labels: { level: 0, severity: CRIT, category: pgsql }
  annotations:
    summary: "CRIT PostgresPartition {{ $labels.cls }}@{{ $labels.job }} {{ $value }}"
    description: |
      pg:cls:partition[cls={{ $labels.cls }}, job={{ $labels.job }}] = {{ $value }} != 1
```





### **Latency**

There are two alerts related to replication latency: replication outage, and high replication latency, graded as a P1 warning.

* Where **replication break** is an error that is determined using the indicator: `pg_downstream_count{state="streaming"}`. If the number of replicas in the current `streaming` state changes negatively, the broken alert is triggered. `walsender` will determine the replication state, the replica will break directly, and the buffer backlog will go from `streaming` to `catchup` state, which will also trigger this alert. Replication interruptions can cause clients to read stale data, which has some off-site impact and is rated as P1.

* Replication latency can be determined using either latency time or latency bytes. The number of delay bytes is the authoritative indicator. Under normal conditions, replication latency time in 100 milliseconds and replication latency bytes in the order of 100 KB are standard. Based on historical experience data, the time alert threshold of 1MB and 1s is currently used.


```yaml
#==============================================================#
#                         Replication                          #
#==============================================================#
# replication break for 1m triggers a P1 alert (WARN: heal in 5m)
- alert: PostgresReplicationBreak
  expr: changes(pg_downstream_count{state="streaming"}[5m]) > 0
  # for: 1m
  labels: { level: 1, severity: WARN, category: pgsql }
  annotations:
    summary: "WARN PostgresReplicationBreak: {{ $labels.ins }}@{{ $labels.instance }}"
    description: |
      changes(pg_downstream_count{ins={{ $labels.ins }}, instance={{ $labels.instance }}, state="streaming"}[5m]) > 0


# replication lag bytes > 1MiB or lag seconds > 1s
- alert: PostgresReplicationLag
  expr: pg:ins:lag_bytes > 1048576 or pg:ins:lag_seconds > 1
  for: 1m
  labels: { level: 1, severity: WARN, category: pgsql }
  annotations:
    summary: "WARN PostgresReplicationLag: {{ $labels.ins }}@{{ $labels.instance }}"
    description: |
      pg:ins:lag_bytes[ins={{ $labels.ins }}, instance={{ $labels.instance }}] = {{ $value | printf "%.0f" }} > 1048576 or
      pg:ins:lag_seconds[ins={{ $labels.ins }}, instance={{ $labels.instance }}] = {{ $value | printf "%.2f" }} > 1


```

In addition, there are corresponding alert rules for query latency and disk latency.

For example, the average disk read/write response time lasts more than 32ms for one minute, or the average query RT in Pgbouncer exceeds 16ms will trigger a P1 alert.

```yaml

# read latency > 32ms (typical on pci-e ssd: 100µs)
- alert: NodeDiskSlow
  expr: node:dev:disk_read_rt_1m > 0.032 or node:dev:disk_write_rt_1m > 0.032
  for: 1m
  labels: { level: 1, severity: WARN, category: node }
  annotations:
    summary: 'WARN NodeReadSlow {{ $labels.ins }}@{{ $labels.instance }} {{ $value  | printf "%.6f" }}'
    description: |
      node:dev:disk_read_rt_1m[ins={{ $labels.ins }}] = {{ $value  | printf "%.6f" }} > 32ms


# pgbouncer avg response time > 16ms (database level)
- alert: PgbouncerQuerySlow
  expr: pgbouncer:db:query_rt_1m > 0.016
  for: 3m
  labels: { level: 1, severity: WARN, category: pgsql }
  annotations:
    summary: "WARN PgbouncerQuerySlow: {{ $labels.ins }}@{{ $labels.instance }} [{{ $labels.datname }}]"
    description: |
      pgbouncer:db:query_rt_1m[ins={{ $labels.ins }}, instance={{ $labels.instance }}, datname={{ $labels.datname }}] = {{ $value | printf "%.3f" }} > 0.016

```



### Saturation

Saturation metrics primary resources, including many system-level monitoring metrics. Mainly includes CPU, disk, connection pool queue, number of database back-end connections, age (essentially saturation of available thing numbers), SSD life, etc.

**Database Load**

Database load is the combined maximum (percentage, but can exceed 100% when overloaded) of machine CPU usage, Pgbouncer time utilization, Postgres time utilization (14 introduced). Load is the most critical metric in Pigsty, concentrating on the load water level of the database instances and clusters.


```yaml
#==============================================================#
#                        Saturation                            #
#==============================================================#
# instance pressure higher than 70% for 1m triggers a P1 alert
- alert: PostgresPressureHigh
  expr: ins:pressure1 > 0.70
  for: 1m
  labels: { level: 1, severity: WARN, category: pgsql }
  annotations:
    summary: "WARN PostgresPressureHigh: {{ $labels.ins }}@{{ $labels.instance }}"
    description: |
      ins:pressure1[ins={{ $labels.ins }}, instance={{ $labels.instance }}] = {{ $value | printf "%.3f" }} > 0.70

```



**Queueing Detection**

The heap contains two main types of metrics, the number of back-end connections and active connections of the PG on the one hand, and the queuing of the connection pool on the other.

PGB queuing is the decisive metric; it represents that perceptible blocking has occurred on the user side, so the presence of queuing lasting 1 minute triggers a P0 alert.

When using **Session Pooling** mode, this alert metric can be relaxed appropriately.

```yaml
# pgbouncer client queue exists
- alert: PgbouncerClientQueue
  expr: pgbouncer:db:waiting_clients > 1
  for: 1m
  labels: { level: 0, severity: CRIT, category: pgsql }
  annotations:
    summary: "CRIT PgbouncerClientQueue: {{ $labels.ins }}@{{ $labels.instance }} [{{ $labels.datname }}]"
    description: |
      pgbouncer:db:waiting_clients[ins={{ $labels.ins }}, instance={{ $labels.instance }}, datname={{ $labels.datname }}] = {{ $value | printf "%.0f" }} > 1

```

The number of back-end connections is a vital warning metric. If the back-end connections consistently reach the maximum number of connections, it often means an avalanche as well. The number of queued connections in the connection pool also reflects this situation but does not cover the case where the application is directly connected to the database.

Currently, Pigsty uses **connection utilization** as an alerting metric, i.e., the percentage of available database connections that have been used, with a P1 alert departing if it exceeds 70% for 3 minutes.


```yaml
# database connection usage > 70%
- alert: PostgresConnUsageHigh
  expr: pg:db:conn_usage > 0.70
  for: 3m
  labels: { level: 1, severity: WARN, category: pgsql }
  annotations:
    summary: "WARN PostgresConnUsageHigh: {{ $labels.ins }}@{{ $labels.instance }} [{{ $labels.datname }}]"
    description: |
      pg:db:conn_usage[ins={{ $labels.ins }}, instance={{ $labels.instance }}, datname={{ $labels.datname }}] = {{ $value | printf "%.3f" }} > 0.70

```



**Idle in Transaction**

The number of connections with `Idle in Transaction` status in the database, more than 2 for 3 minutes, is the departure of the P1 alert.

```yaml
# database connection usage > 70%
- alert: PostgresIdleInXact
  expr: pg:db:ixact_backends > 1
  for: 3m
  labels: { level: 2, severity: INFO, category: pgsql }
  annotations:
    summary: "Info PostgresIdleInXact: {{ $labels.ins }}@{{ $labels.instance }} [{{ $labels.datname }}]"
    description: |
      pg:db:ixact_backends[ins={{ $labels.ins }}, instance={{ $labels.instance }}, datname={{ $labels.datname }}] = {{ $value | printf "%.0f" }} > 1

```



**Resource Alert**

Age (XID) usage exceeds 80% departure P0 alert, which means the system is about to run out of transaction number resources and enter the XID Wraparound state.


```yaml
# database age saturation > 80%
- alert: PostgresXidWarpAround
  expr: pg:db:age > 0.80
  for: 1m
  labels: { level: 0, severity: CRIT, category: pgsql }
  annotations:
    summary: "CRIT PostgresXidWarpAround: {{ $labels.ins }}@{{ $labels.instance }} [{{ $labels.datname }}]"
    description: |
      pg:db:age[ins={{ $labels.ins }}, instance={{ $labels.instance }}, datname={{ $labels.datname }}] = {{ $value | printf "%.0f" }} > 80%

```

