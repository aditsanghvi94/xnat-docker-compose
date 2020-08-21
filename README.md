# Dockerized XNAT
Use this repository to quickly deploy an [XNAT](https://xnat.org/) instance on [docker](https://www.docker.com/).

## Introduction

This repository contains files to bootstrap XNAT deployment. The build creates three containers:

- **[Tomcat](http://tomcat.apache.org/) + XNAT**: The XNAT web application
- [**Postgres**](https://www.postgresql.org/): The XNAT database
- [**nginx**](https://www.nginx.com/): Web proxy sitting in front of XNAT

## Prerequisites

* [docker](https://www.docker.com/)
* [docker-compose](http://docs.docker.com/compose) (Which is installed along with docker if you download it from their site)

## Usage


1. Clone the [xnat-docker-compose](https://github.com/aditsanghvi94/xnat-docker-compose) repository.)

```
$ git clone https://github.com/aditsanghvi94/xnat-docker-compose
$ cd xnat-docker-compose
```

2. Configurations: The default configuration is sufficient to run the deployment. The following files can be modified if you want to change the default configuration

    - **docker-compose.yml**: How the different containers are deployed. There is a section of build arguments (under `services → xnat-web → build → args`) to control some aspects of the build.
        * If you want to download a different version of XNAT, you can change the `XNAT_VER` variable to some other release.
        * The `TOMCAT_XNAT_FOLDER` build argument is set to `ROOT` by default; this means the XNAT will be available at `http://localhost`. If, instead, you wish it to be at `http://localhost/xnat` or, more generally, at `http://localhost/{something}`, you can set `TOMCAT_XNAT_FOLDER` to the value `something`.
        * If you need to control some arguments that get sent to tomcat on startup, you can modify the `CATALINA_OPTS` environment variable (under `services → xnat-web → environment`).
    - **xnat/Dockerfile**: Builds the xnat-web image from a tomcat docker image.

3. Start the system

```
$ docker-compose up 
```

Note that at this point, if you go to `localhost` you won't see a working web application. It takes upwards of a minute
to initialize the database, and you can follow progress by reading the docker compose log of the server:

```
docker-compose logs -f --tail=20 xnat-web
Attaching to xnatdockercompose_xnat-web_1
xnat-web_1    | INFO: Starting Servlet Engine: Apache Tomcat/7.0.82
xnat-web_1    | Oct 24, 2017 3:17:02 PM org.apache.catalina.startup.HostConfig deployWAR
xnat-web_1    | INFO: Deploying web application archive /opt/tomcat/webapps/xnat.war
xnat-web_1    | Oct 24, 2017 3:17:14 PM org.apache.catalina.startup.TldConfig execute
xnat-web_1    | INFO: At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
xnat-web_1    | SOURCE: /opt/tomcat/webapps/xnat/
xnat-web_1    | ===========================
xnat-web_1    | New Database -- BEGINNING Initialization
xnat-web_1    | ===========================
xnat-web_1    | ===========================
xnat-web_1    | Database initialization complete.
xnat-web_1    | ===========================
xnat-web_1    | Oct 24, 2017 3:18:27 PM org.apache.catalina.startup.HostConfig deployWAR
xnat-web_1    | INFO: Deployment of web application archive /opt/tomcat/webapps/xnat.war has finished in 84,717 ms
xnat-web_1    | Oct 24, 2017 3:18:27 PM org.apache.coyote.AbstractProtocol start
xnat-web_1    | INFO: Starting ProtocolHandler ["http-bio-8080"]
xnat-web_1    | Oct 24, 2017 3:18:27 PM org.apache.coyote.AbstractProtocol start
xnat-web_1    | INFO: Starting ProtocolHandler ["ajp-bio-8009"]
xnat-web_1    | Oct 24, 2017 3:18:27 PM org.apache.catalina.startup.Catalina start
xnat-web_1    | INFO: Server startup in 84925 ms
```

Your XNAT will soon be available at https://localhost/app/template/Login.vm.

## Mounted Data

When you bring up XNAT with `docker-compose up`, several directories are created (if they don't exist already) to store the persistent data.

* **postgres-data** - Contains the XNAT database
* **xnat/plugins** - Initially contains nothing. However, you can customize your XNAT with plugins by placing jars into this directory and restarting XNAT.
* **xnat-data/archive** - Contains the XNAT archive
* **xnat-data/build** - Contains the XNAT build space. This is useful when running the container service plugin.
* **xnat-data/home/logs** - Contains the XNAT logs.


## Troubleshooting

### Get a shell in a running container
Say you want to examine some files in the running `xnat-web` container. You can `exec` a command in that container to open a shell.

```
$ docker-compose exec xnat-web bash
```

* The `docker-compose exec` part of the command is what tells docker-compose that you want to execute a command inside a container.
* The `xnat-web` part says you want to execute the command in whatever container is running for your xnat-web service. If, instead, you want to open a shell on the database container, you would use `xnat-db` instead.
* The `bash` part is the command that will be executed in the container. It could really be anything, but in this case we want to open a shell. Running `bash` will do just that. You will get a command prompt, and any further commands you issue will be run inside this container.

### Read Tomcat logs

List available logs

```
$ docker-compose exec xnat-web ls /usr/local/tomcat/logs

catalina.2018-10-03.log      localhost_access_log.2018-10-03.txt
host-manager.2018-10-03.log  manager.2018-10-03.log
localhost.2018-10-03.log
```

View a particular log

```
$ docker-compose exec xnat-web cat /usr/local/tomcat/logs/catalina.2018-10-03.log
```

### Controlling Instances

#### Stop Instances
Bring all the instances down by running

```
$ docker-compose down
```

If you want to bring everything down *and* remove all the images that were built, you can run

```
$ docker-compose down --rmi all
```

#### Bring up instances
This will bring all instances up again. The `-d` means "detached" so you won't see any output to the terminal.

```
$ docker-compose up -d
```

(If you like seeing the terminal output, you can leave off the `-d` option. The various containers will print output to the terminal as they come up. If you close this connection with `Ctrl+C`, the containers will be stopped or killed.)

#### Restart
If an instance is having problems, you can restart it.
```
$ docker-compose restart xnat-web
```

#### Rebuild after making changes
If you have changed a `Dockerfile`, you will need to rebuild an image before the changes are picked up.

```
$ docker-compose build xnat-web
```

It is possible that you will need to use the `--no-cache` argument, if you have only changed local files and not the `Dockerfile` itself.
