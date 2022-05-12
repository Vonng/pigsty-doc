# Development Log

#### 2022-05-05

* New CMDB Design
* Now cmdb works for redis & gpsql

#### 2022-05-04

* Dashboard Overhaul: Infra-Overview
* Add node_crontab_overwrite implementation
* Test citus/dcs3/mxdb conf
* node_local_repo_url -> node_repo_local_urls


#### 2022-05-02

* add crontab implementation


#### 2022-05-01

* Refactor redis.yml & redis-remove.yml
* Remove role redis, redis-remove, redis-exporter
* Fix dashboard redis aliveness issue
* Upgrade to redis v6.2.7


#### 2022-04-30

* Register Infra as common consul service
* Make consul service discovery great again
* implement `node_data_dir` which create /data during node init if not exists
* add pg_probackup to default package list
* integration test on pigsty-citus


#### 2022-04-29

New Vars:

node_data_dir
node_crontab

Bulk Rename:

node_ntp_config -> node_ntp_enabled
node_admin_setup -> node_admin_enabled
node_admin_pks -> node_admin_pk_list
node_dns_hosts -> node_etc_hosts_default
node_dns_hosts_extra -> node_etc_hosts
node_dns_server -> node_dns_method
node_local_repo_url -> node_repo_local_urls
node_packages -> node_packages_default
node_extra_packages -> node_packages
node_packages_meta -> node_packages_meta
node_meta_pip_install -> node_packages_meta_pip
node_sysctl_params -> node_tune_params
dcs_name -> consul_name
dcs_exists_action -> consul_clean
dcs_disable_purge -> consul_safeguard
app_list -> nginx_indexes
grafana_plugin -> grafana_plugin_method
grafana_cache -> grafana_plugin_cache
grafana_plugins -> grafana_plugin_list
grafana_git_plugin_git -> grafana_plugin_git
haproxy_admin_auth_enabled -> haproxy_auth_enabled
pg_exists_action -> pg_clean
pg_disable_purge -> pg_safeguard
pg_shared_libraries -> pg_libs



#### 2022-04-28

* Upgrade Loki to v2.5.0 with new rpm packager `nfpm`
* Upgrade pg_exporter to v0.5.0 with new rpm packager `nfpm`
* Upgrade grafana version to 8.4.6
* Upgrade Consul to 1.13
* Upgrade Pgbouncer to 1.17
* Upgrade vip-manager to 1.0.2
* Upgrade Grafana to 8.5.0
* Remove pgweb by default (since docker will cover it)
* Remove role jupyter, move it's logic into `infra-jupyter.yml`
* Refactor docker role, it can be enabled per nodes.


#### 2022-04-26

Add new options:

```bash
nameserver_enabled: false  # setup dnsmasq
prometheus_enabled: true   # setup prometheus
grafana_enabled: true      # setup grafana
docker_enable: true        # setup docker
loki_enabled: true         # setup loki
```


#### 2022-04-24

English Document Released!



#### 2022-03-30

* Pigsty v1.4.0 Released!


#### 2022-03-24

* Get the latest source with `bash -c "$(curl -fsSL http://download.pigsty.cc/get)"`
* add download script to get pigsty/pkg/app/matrix packages from github
* if Github is not viable, use CDN instead.


#### 2022-03-20

* Now playbooks are divided into 4 major groups: `infra`, `nodes`, `pgsql`, `redis`
* add promtail to `infra.yml`, `nodes.yml`, etc...


#### 2022-03-19

* build haproxy , loki, promtail rpm for pigsty
* enable loki by default
* enable promtail by default
* promtail now collect nodes


#### 2022-03-08

* Greenplum/MatrixDB Support finished!
  * Simplify `pigsty-matrixdb.yml` playbook
  * Remove role `gp_prepare` & `gp_provision`
  * Remove playbook `gpsql-post.yml`
  * Update default pigsty-mxdb.yml example
* Fix all example configs
* Add `-d postgres` when execute `pgsql-createuser.yml`
* Software update:
  * pg_exporter v0.4.1 : new timeout parameter, bug fix, pgbouncer v1.16 support, etc...
  * greenplum 6.19.1 -> 6.19.3
  * grafana 8.4.2 -> 8.4.3


#### 2022-03-05

* Overhaul dashboards
  * Nodes Cluster / Nodes Instance / Nodes Overview
  * Add ds datasource variable to all dashboards (so you can change prometheus datasource by select hidden variables)
  * Reforge PGSQL Cluster Dashboards
  * Add new dashboards: PGSQL Databases which focus on database dimension among cluster
  * Adjust links for pgsql-node to nodes-instance
  * Add database & service nav for pgsql-overview

#### 2022-02-22

* software upgrade
  * postgres minor version update: 14.1 -> 14.2
  * prometheus version 2.33
  * timescaledb 2.6.0
  * postgis 3.2
  * grafana 8.4.2

* add ins , cls label to node metrics


#### 2022-02-14

* PGSQL Overview Dashboard Rework
* PGSQL Cluster Dashboard Reforge

#### 2022-02-13

* Node Overview Dashboard
* Node Instance Dashboard

#### 2022-02-02

* Add matrixdb support

#### 2022-01-28

* Dashboard adjustment for new label structure

#### 2022-01-25

* Redesign ip based node metrics
* refactor on pg_exporter node_exporter deployment

#### 2022-01-24

* Software upgrade
  * HAProxy 2.2 -> 2.5
  * Greenplum: 6.18 -> 6.19
  * Pev2 : 0.23 -> 0.24
  * pgweb : 0.11.9 -> 0.11.10
  * loki, promtail, logcli, loki-canary: 2.4.1 -> 2.4.2

#### 2022-01-22

* Monitoring v9 Launch
  * split nodes monitoring from infra & pgsql
  * re-forge on labels, keys & etc..
* Reforge prometheus rules: infra, nodes, pgsql, redis, ...


#### 2022-01-21

* split role monitor into pg_exporter & node_exporter
* add new parameter: pg_exporter_params
* upgrade default haproxy version from 2.2 to 2.5
* bug fix: pip3 install jupyter failed
  now a pip3 upgrade is performed before download & install

#### 2021-12-29

* disable repo on other meta nodes in infra-demo.yml
  Now bootstrap on multiple meta nodes are much easier
* enhancement: remove Require=consul from patroni systemd service
  Which makes dcs migration much easier
* fix pg_exporter.yml pg_index column sequence
  Which is a workaround for a known bug of PostgreSQL

#### 2021-12-17

* Add terraform support for Aliyun

#### 2021-12-09

* Redis Dashboards Enhancement
* PGCAT Dashboards Enhancement
* Separate loki & pgweb from standard infra.yml playbook

#### 2021-12-04

* v1.3.1 Release
* Bug fix
  * configure check_bin software version
  * add citus to pigsty-pub4 pg-meta cluster by default
  * add auth parts to environ patronictl.yml
* add option citus.node_conninfo: 'sslmode=prefer' for all conf template
* add example configuration files: citus, dcs3, pub4
* pg alias for patronictl on meta nodes
* add patroni to prometheus targets (since 2.1.1)

#### 2021-12-03

* add ftp to package list
* add one-pass init playbook (for 3-dcs x 3-node deploy)
* add reloadha & reloadhba shell shortcuts

#### 2021-11-29

* Add redis support for pigsty
* pgcat overhaul
* new key metrics panel for pgsql cluster & pgsql instance
  ....


#### 2021-09-23

* [ENHANCEMENT] home page overhaul
* [ENHANCEMENT] add jupyter lab integration
* [ENHANCEMENT] add pgweb console integration
* [ENHANCEMENT] update default pkg.tgz software version:


#### 2021-09-18

* add pev2 support
* add pgbadger support


#### 2021-09-17

* [ENHANCEMENT] add `pg_dummy_filesize` to create fs space placeholder

#### 2021-09-14

* release v1.0.1
* huge amount of documentation updates
* fix some minor bugs


2021-09-14

* Documentation Update
  * Chinese document now viable
  * Machine-Translated English document now viable
* Bug Fix: `pgsql-remove` does not remove primary instance.
* Bug Fix: replace pg_instance with pg_cluster + pg_seq
  * Start-At-Task may fail due to pg_instance undefined
* Bug Fix: remove citus from default shared preload library
  * citus will force max_prepared_transaction to non-zero value
* Bug Fix: ssh sudo checking in `configure`:
  * now `ssh -t sudo -n ls` is used for privilege checking
* Typo Fix: `pg-backup` script typo
* Alert Adjust: Remove ntp sanity check alert (dupe with ClockSkew)
* Exporter Adjust: remove collector.systemd to reduce overhead



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