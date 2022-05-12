# Docker Applications

Pigsty v1.5.0-beta comes with Docker and Docker Compose deployment support, where Docker Daemon will be enabled by default on the meta node.

You can use Docker to deploy and launch software applications quickly. You can directly access the PostgreSQL/Redis database deployed on the host in the container using the connection string.

* [PgAdmin4](#PgAdmin4): A GUI tool for managing PostgreSQL instances.
* [PGWeb](#PGWEB): A tool automatically generates back-end API services based on PG database schema.
* [PostgREST](#PostgREST): A tool to automatically generate backend API services based on PG database schema.
* [ByteBase](#ByteBase): A GUI tool for making PostgreSQL schema changes.
* [Jupyter Lab](#Jupyter): A battery-included Python lab environment for data analysis and processing.

You can also use Docker to execute some battery-included command tools.

* [SchemaSPY](#SchemaSPY): Generates detailed visual reports of database schemas.
* [Pgbadger](#discourse): Generate database log report.

You can also use Docker to pull up some battery-included open-source SaaS services.

* [Gitlab](#Gitlab): open-source code hosting platform.
* Habour: open-source mirror repo
* Jira: open-source project management platform.
* Confluence: open-source knowledge hosting platform.
* Odoo: open-source ERP
* [Mastodon](#Mastodon): PG-based social network
* [Discourse](#Discourse): open-source forum based on PG and Redis


--------------------



## Add Upstream to Nginx

Most of the software described in this article provides a web interface to the public. While it can be accessed directly via IP: Port, we recommend using a domain name and unifying access from the Nginx proxy. Use the following configuration and commands to register a new service with Nginx.

```bash
# Add new Nginx services definition
nginx_upstreams:
  - { name: kong         , domain: api.pigsty , endpoint: "127.0.0.1:8880"   } #== v optional ==#
  - { name: pgadmin      , domain: adm.pigsty , endpoint: "127.0.0.1:8885"   }
  - { name: pgweb        , domain: cli.pigsty , endpoint: "127.0.0.1:8886"   }
  - { name: bytebase     , domain: ddl.pigsty , endpoint: "127.0.0.1:8887"   }
  - { name: jupyter      , domain: lab.pigsty , endpoint: "127.0.0.1:8888"   }
  
./infra.yml -t nginx_config,nginx_restart    # refresh nginx conf, and restart to apply
```


--------------------


## PgAdmin4

[PGAdmin4](https://www.pgadmin.org/) is the popular PG control tool; use the following command to pull up the PgAdmin4 service on the meta node, default to host `8885` port, username `admin@pigsty.cc`, password: `pigsty`.

```bash
cd ~/pigsty/app/pgadmin ; make up
# docker-compose up -d , which is
docker run --init --name pgadmin --restart always --detach --publish 8885:80 \
	-e PGADMIN_DEFAULT_EMAIL=admin@pigsty.cc -e PGADMIN_DEFAULT_PASSWORD=pigsty \
	dpage/pgadmin4
```

Copy the server access information to the /tmp/servers.json file and re-import it.

```bash
make conf       # provision pgadmin with pigsty pg servers list 
make dump       # dump servers.json from pgadmin container
```




## PGWeb

[PGWeb](https://github.com/sosedoff/pgweb) is a browser-based PG client tool. Use the following command to pull up the PGWEB service on the meta node, defaulting to the host `8081` port.

```bash
cd ~/pigsty/app/pgweb ; docker-compose up -d
docker run --init --name pgweb --restart always --detach --publish 8886:8081 sosedoff/pgweb
```

Users need to fill in the database connection string, for example, the default CMDB: 

`postgres://dbuser_dba:DBUser.DBA@10.10.10.10:5432/meta?sslmode=disable`.



## PostgREST

[PostgREST](https://postgrest.org/en/stable/index.html) is a binary component that automatically generates a REST API based on the PostgreSQL database schema.

The following command will pull up postgrest using docker (local port 8082, using the default admin user, exposing the Pigsty CMDB schema).

```bash
cd ~/pigsty/app/postgrest ; docker-compose up -d
docker run --init --name postgrest --restart always --detach --publish 8884:8081 postgrest/postgrest
```

http://home.pigsty.cc:8883/ shows the available API that exposed by PostgREST

Visiting http://10.10.10.10:8884 will show all the definitions of the auto-generated APIs, which can be automatically generated in the [Swagger Editor](https://editor.swagger.io).

`curl http://10.10.10.10:8884/pg_cluster` will anonymously access the data table `pigsty.pg_cluster`.

If you want to add, delete, check and design more fine-grained privilege control, please refer to [Tutorial 1 - The Golden Key](https://postgrest.org/en/stable/tutorials/tut1.html) to generate a signed JWT.



## ByteBase

[ByteBase](https://bytebase.com/) is a tool for making database schema changes. The following command will start a ByteBase on meta node port 8887.

```bash
cd app/bytebase; docker-compose up -d
docker run --init --name bytebase \
        --restart always --detach \
        --publish 8887:8887 \
        --volume /data/bytebase/data:/var/opt/bytebase \
        bytebase/bytebase:1.0.4 \
        --data /var/opt/bytebase \
        --host http://ddl.pigsty \
        --port 8887 \
        --pg postgres://dbuser_bytebase:DBUser.Bytebase@10.10.10.10:5432/bytebase
```

Visit http://10.10.10.10:8887/ to use ByteBase. To start schema changes, you need to create the project, environment, instance, and database.




## Jupyter

[Jupyter Lab](https://github.com/jupyter/docker-stacks) is a data analysis environment. The following command will start a Jupyter Server on port 8084.

```bash
docker run -it --restart always --detach --name jupyter -p 8083:8888 -v "${PWD}":/tmp/notebook jupyter/scipy-notebook
docker logs jupyter # Print logs and get Token of login
```

Visit http://10.10.10.10:8888/ to use JupyterLab, (you need to fill in the auto-generated Token). Note that Pigsty also has JupyterLab installed on the host.




--------------------


## SchemaSPY

Generate a database schema report using CMDB as an example. The following `docker`, using.

```bash
docker run -v /www/schema/pg-meta/meta/pigsty:/output andrewjones/schemaspy-postgres:latest \
    -host 10.10.10.10 -port 5432 -u dbuser_dba -p DBUser.DBA -db meta -s pigsty
```

Then visit http://pigsty/schema/pg-meta/meta/pigsty to access the Schema report.




--------------------

## Gitlab

Please refer to the [Gitlab Docker Deploy Doc](https://docs.gitlab.com/ee/install/docker.html) to complete the Docker deployment.

```bash
export GITLAB_HOME=/data/gitlab

sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 23:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --shm-size 256m \
  gitlab/gitlab-ee:latest
  
sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```



## Discourse

Build open source forum Discourse. You need to adjust the config `app.yml`, focusing on the SMTP part of the config.

<details><summary>Sample Discourse config</summary>


```yaml
templates:
  - "templates/web.china.template.yml"
  - "templates/postgres.template.yml"
  - "templates/redis.template.yml"
  - "templates/web.template.yml"
  - "templates/web.ratelimited.template.yml"
## Uncomment these two lines if you wish to add Lets Encrypt (https)
# - "templates/web.ssl.template.yml"
# - "templates/web.letsencrypt.ssl.template.yml"
expose:
  - "80:80"   # http
  - "443:443" # https
params:
  db_default_text_search_config: "pg_catalog.english"
  db_shared_buffers: "768MB"
env:
  LC_ALL: en_US.UTF-8
  LANG: en_US.UTF-8
  LANGUAGE: en_US.UTF-8
  EMBER_CLI_PROD_ASSETS: 1
  UNICORN_WORKERS: 4
  DISCOURSE_HOSTNAME: forum.pigsty
  DISCOURSE_DEVELOPER_EMAILS: 'fengruohang@outlook.com,rh@vonng.com'
  DISCOURSE_SMTP_ENABLE_START_TLS: false
  DISCOURSE_SMTP_AUTHENTICATION: login
  DISCOURSE_SMTP_OPENSSL_VERIFY_MODE: none
  DISCOURSE_SMTP_ADDRESS: smtpdm.server.address
  DISCOURSE_SMTP_PORT: 80
  DISCOURSE_SMTP_USER_NAME: no_reply@mail.pigsty.cc
  DISCOURSE_SMTP_PASSWORD: "<password>"
  DISCOURSE_SMTP_DOMAIN: mail.pigsty.cc
volumes:
  - volume:
      host: /var/discourse/shared/standalone
      guest: /shared
  - volume:
      host: /var/discourse/shared/standalone/log/var-log
      guest: /var/log

hooks:
  after_code:
    - exec:
        cd: $home/plugins
        cmd:
          - git clone https://github.com/discourse/docker_manager.git
run:
  - exec: echo "Beginning of custom commands"
  # - exec: rails r "SiteSetting.notification_email='no_reply@mail.pigsty.cc'"
  - exec: echo "End of custom commands"
```

</details>

Then, just execute the following command and pull up Discourse.

```bash
./launcher rebuild app
```




## Mastodon

TBD
