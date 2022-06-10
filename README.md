## cromwell-server

A paired server + mysql database for running cromwell jobs

This server should hopefully be persistently running at `mammoth.wusm.wustl.edu:8000`

Largely pulled from https://github.com/broadinstitute/cromwell/tree/develop/scripts/docker-compose-mysql with some minor changes to update cromwell version and port mapping

#### Installation

[Docker](https://hub.docker.com/) and [docker-compose](https://docs.docker.com/compose/install/)

#### Starting the server and database

There should be a (hopefully) persistent server running on mammoth.wusm.wustl.edu. If that server goes down for some reason, to start/restart it run the following from `/lake1/cromwell-server`

```bash
docker-compose up --force-recreate
```

This should start two containers, the cromwell server and mysql database.

###### Ports

The cromwell server will be accessable on port 8000, the mysql server will be on port 3307. If different ports are desired, these can be changed from within the `docker-compose.yml` config file.

###### Cromwell version

The default cromwell version is `78-38cd360`. If a different version of cromwell is desired, you can select a version tag from the [cromwell docker repo](https://hub.docker.com/r/broadinstitute/cromwell/tags) and insert that tag in the FROM section of the Dockerfile at `compose/cromwell/Dockerfile`.

#### Connecting to the server

The cromwell server will be running on port 8000 by default. So to connect the simplest approach is to map that port with ssh to your local machine and connect by typing `localhost:8000` in the browser.

From this page you can see a description of all endpoints available.

If the server is running on one of the dinglab cluster nodes you could also do something like `mammoth.wusm.wustl.edu:8000` in your browser, no port mapping required.


#### Compute1 example job config

An [example job config](https://github.com/estorrs/wombat/blob/master/wombat/templates/cromwell-config-db.compute1.template.dat) that can be submitted from compute1. 

## Common Issues

#### Resetting the changelock

Periodically, when submitting a cromwell job you will get an error like the following

```bash
liquibase.exception.LockException: Could not acquire change log lock.  Currently locked by 172.17.0.1 (172.17.0.1) since 6/9/22, 9:35 PM
```

To fix this error, you will need to run the follow the basic commands [here](https://stackoverflow.com/questions/62455159/liquibase-lock-could-not-acquire-change-log-lock-in-postgresql-docker-image).

First, log in to the mysql database.

To do so, first find the docker image running the database

```bash
docker ps
```

One of the containers should have `estorrs/cromwelldb-mysql:5.7` as an image. This is the container runnin the mysql database. Take note of the CONTAINER ID. We will now exec into this container.

```bash
docker exec -it <CONTAINER_ID> /bin/bash
```

You should now be inside the docker image. To log in to the mysql database run the following. When prompted for the password enter `cromwell`

```bash
mysql -h localhost -u root -p cromwell_db
```

You should now be at the mysql command prompt. To see what is causing the lock, enter

```bash
 select * from DATABASECHANGELOGLOCK;
 ```

To clear the lock, enter 

```bash
update DATABASECHANGELOGLOCK set LOCKED=false, LOCKGRANTED=null, LOCKEDBY=null where ID=1;
```

The lock should now be removed.
