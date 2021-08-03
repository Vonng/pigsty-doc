# Development Log

#### 2021-07-22

* refactor `load_config.sh` with python to `load_conf.py`
* add `inventory_cmdb` and `inventory_conf` to switch between static config and dynamic inventory
* add new dashboards: pgsql-activity which focus on cluster level activities
* add new dashboards: pgcat-bloat which focus on table & index bloat
* fix minor dashboard bugs
* fix pg_exporter config `pg_repl` collector version overlap for PG12
* update covid dashboards & isd dashboards
* prepare for 1.0.0


#### 2021-07-21

* add ipython jupyterlab to meta packages
* add pip support for meta python env
* add grafana plugins echarts & csv/json data source
* prepare for v1.0.0-beta2


#### 2021-07-20

* bug fix: register role does not run on all meta nodes 
* add check for createpg createdb createuser scripts

#### 2021-07-19

* Use docsify as documentation solution


#### 2021-07-15

* Calibration of dashboard data links
* remove `grafana` & `prometheus` database definition in config file
  instead, change grafana primary database to postgres will be a tutorial for get start with pigsty
* build & fix dashboard data links
* add `catlog` `pglog` alias for meta node environment (get and pour pgsql log)
* change default timezone from Asia/ShangHai to Asia/Hong_kong
* add `pg_shared_libraries` to customize extensions
* install citus, timescaledb by default
* prepare for v1.0.0-beta1
 

#### 2021-07-14

* add pgsql-xacts dashboard 
* add pgsql-persist dashboard
* add pgsql-tables dashboard
* add pglog-analysis dashboard
* add pglog-session dashboard
* add softlinks in files dir to dashboards



#### 2021-07-13

* Release pigsty v1.0.0-alpha2
* Fix systemd-devel deps failure on VPC.
* remove core apps
* refactor cmdb application, use pg_datbases.meta.baseline to provisioning cmdb schema
* add `load_config` to parse and activate config from config file
* Integrate pglog schema into `pigsty` schema
* Update files/conf

#### 2021-07-12

* alert panel now links to alertmanager
* add click-able data-link to most graph
* release pg_exporter v0.4.0 (remove beta)
* adjust home & overview & cluster & instance layout

```bash
v1.0.0-alpha2 Release

* New Dashboard:  `pgcat-query`, get detailed information (including raw sql string) directly from corresponding postgres datasource
* New Dashboard:  `pgsql-queries`, runs on instance level, focusing on instance pgbouncer queries and rt, table qps, query qps, etc...
* Update Dashboard: `pgsql-alert`, add Active Alert Table (you can silence alert from table panel)
* Update Dashboards:  adjust layout and content for `pgsql` dashboards.
* Dashboard Templates: Infra links are automaticlly adjusted with `nginx_upstream`
* Dashboard management script: `grafana.py`: now CI is much more easier for dashboards.
* Clickable Panel: Add data links to graphic elements.
* fix loki extraction bugs, add `unzip` to basic util set.
* fix util scripts and basic document (quick-start, download, roadmap, contribution)
* Software upgrade: grafana 8.0.5, vip-manager 1.0, prometheus 2.28.3 , consul v1.10.0, haproxy 2.2.12, pg_exporter 0.4.0
* Switch to pg_exporter v0.4.0 and new metrics set (tested in production for almost 2month, now is out of beta)
* Add time-sync option to vagrantfile
* Add new role `register` for pgsql & infra interaction
* Add new role `environ` for meta node env setup.
* Alerting rules overhaul (two implementation: Prometheus version & Grafan version)
* Remove `svc`, `role`, `ip` labels for all metrics
* ....
```


#### 2021-07-09

* now comes to the juice part, monitoring dashboard designing
* add links between pgcat & pgsql, e.g table level dashboard
* add a pgcat-query dashboard which aims at pg_stat_statements view
* add alertmanager links on alert timeline panel
* add links to graph, so user can click graphic element and jump to corresponding dashboard
* finish pgsql-queries, and back port to pgsql query



#### 2021-07-08

* use [acpgh].pigsty as placeholder, passing `nginx_upstream` via environ, replace http host when provisioning dashboards
* add pgsql-queries dashboard which runs on instance level, focusing on instance pgbouncer queries and rt, table qps, query qps, etc...


#### 2021-07-07

* Add baidu netdisk download for mainland China
  https://pan.baidu.com/s/1DZIa9X2jAxx69Zj-aRHoaw 8su9
* Grafana static provision have some down-sides: root privileges / can't update home dashboard. I wonder if we could switch to API provisioning instead.
* Use pure python for grafana provisioning `grafana.py`


#### 2021-07-06

* Use v1.0.0-alpha1 instead. Since the change are significant, it is not appropriate to use v0.10. 
* Remove the crud haproxy index pages, using grafana table & data links instead 
* At last register by instance may be the easiest way to implement and manage
* Add new role `loki`
* Add new role `promtail`
* Register datasource when create new database with `pgsql-createdb.yml`


#### 2021-07-05

* Extract a new role named `register` to handler all interaction between pgsql & infra.
* Extract a new role named `envrion` to setup meta node environment including: ssh, metadb, env vars, etc... 
* Dashboard tags now have hierarchy:  `Pigsty` is the top tier, Application name `PGSQL` `PGLOG` is second tier 
  * `Overview`, `Cluster`, `Instance`,`Database` are filter with `Pigsty` and `<Level>` tags. which means the nav-link can cross multiple applications

#### 2021-07-04

* [Milestone](milestone.md) chart of Pigsty


#### 2021-06-30

* Rough implementation on v0.10.0-alpha1
* Setup environment for admin user (pgpass, pg_service, env vars,)
* Application install script will have environ
* Fix nofile limit on postgres|pgbouncer|patroni
* [Milestone](./milestone.md) planning.


#### 2021-06-29

* Remake release system
* Have a draft on application installation standard
* Use 'v' prefixed fully qualified version string
* remove polysh from default pkg (unstable when downloading) 
* remove grafana plugins, since lot's of them were covered in grafana 8.0


#### 2021-06-28

* Remake alerting rules 


#### 2021-06-25

* Remake infra-rules and pgsql-rules

#### 2021-06-23

* Remake PGSQL node


#### 2021-06-10

It's time to have an overhaul on monitoring system, which includes:
* Upgrade `pg_exporter` to 0.4.0 , re-write metric definition and add support for PostgreSQL 14  
* Use static file service discovery by default to reduce dependency for monitoring system
* Use static label set (job,cls,ins), remove (svc,role,ip) from labels, Which makes identity immutable
* Redesign entire monitoring system to use new label system and embrace Grafana 8.0
* Using grafana 8.0 new features

#### 2021-06-01

Well it's good to write some dev logs.