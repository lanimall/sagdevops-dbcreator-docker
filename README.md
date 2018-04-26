# DB Creator for SoftwareAG products

A project to automatically add the right DB schemas / objects etc... for any products

## Requirements

### Docker Available
Before you start ensure you have installed [Docker](https://www.docker.com/products/overview)
including docker-compose tool.


### Clone or Download this project

For git clone:
```
git clone --recurse https://github.com/lanimall/sagdevops-dbcreator-docker.git
```

If you download instead of "git clone", it will not download the "antcc" submodule. 
So in this case, make sure to download that submodule separately and add it to the antcc subfolder.

### Get Command Central image from Docker Store
Also, the project relies on the Command Central image that can be found on Docker Store.
Refer to https://store.docker.com/images/softwareag-commandcentral

Once you login to docker, you should pull the Command Central image locally.

If not logged in yet:

```bash
docker login
```

Then, pull the image as follow:

```bash
docker pull store/softwareag/commandcentral:10.1.0.1-server
```

Image should not be in your local reporisotry. Verify:

```bash
docker images
```

### Create the base Oracle image for SoftwareAG products

Simply build the base image "softwareag/base-oracle-xe-11g:latest" by running:

```bash
docker-compose -f docker-compose-build.yml build
Building db
Step 1/2 : FROM wnameless/oracle-xe-11g
 ---> f794779ccdb9
Step 2/2 : ADD init.sql /docker-entrypoint-initdb.d/
 ---> Using cache
 ---> f1a2053eed31
Successfully built f1a2053eed31
Successfully tagged softwareag/base-oracle-xe-11g:latest
```

## Creating the Database for a SoftwareAG product

### Creation

To create a database with all the DB objects required by the SoftwareAG products, 
simply use the rigtht docker-compose file in following commands:

For BPMS (IS/BPMS/MWS):

```bash
docker-compose run --rm setup_bpms_db
```

For IS:

```bash
docker-compose run --rm setup_is_db
```

For MWS:

```bash
docker-compose run --rm setup_mws_db
```

### Verifications (using BPMS configuration)

The verification is the same for all 3 commands run above... for example purpose, we'll use the BPMS config.

Once you've run 1 of the docker-compose command above, it will take a few minutes, and all you should see running are the
docker oracle instances (since the instance named "setup_*" will exit after completion).

Verify that indeed you see only the running DB instance (and not the setup_* instance...which would mean the setup is still going on)

```bash
docker ps

CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS                                                      NAMES
ceedc56c1354        softwareag/oracle-xe-11g   "/bin/sh -c '/usr/sb…"   About an hour ago   Up 8 minutes        8080/tcp, 0.0.0.0:49160->22/tcp, 0.0.0.0:49161->1521/tcp   wmdbcreator_wMBPMS_db_1
```

Now, let's check that indeed the required objects were created in this DB instance.

Login to the running DB instance and use sqlplus as usual:

```bash
docker exec -it ceedc56c1354 /bin/bash
root@ceedc56c1354:/# sqlplus /nolog

SQL*Plus: Release 11.2.0.2.0 Production on Thu Apr 5 15:41:05 2018

Copyright (c) 1982, 2011, Oracle.  All rights reserved.

SQL> connect webmbpms/strong123!
Connected.
SQL> select tablespace_name, table_name from user_tables;
TABLESPACE_NAME 	       TABLE_NAME
------------------------------ ------------------------------
WEBMDATA		       COMPONENT_EVENT
WEBMDATA		       AGW_EVENT_TXN
WEBMDATA		       AGW_EVENT_METRICS
WEBMDATA		       AGW_EVENT_LC
WEBMDATA		       AGW_EVENT_ERR
WEBMDATA		       AGW_EVENT_PV
WEBMDATA		       AGW_EVENT_MON
WEBMDATA		       AGW_EVENT_EG
etc...
etc...
etc...
```

## Storing as a Docker Image

At this point, we can easily create instances with the right schema (as seen above), but not yet save it all in a complete Dockerfile.

As such, not ideal for now, but we can commit the DBs after executing the docker-compose.yml file.

NOTE: Update the commands below with your docker instance IDs.

```bash
docker commit 32e26cd5cade registry.docker.tests:5000/softwareag_dbs/is-oracle:10.1

docker commit 11cf6b503bb3 registry.docker.tests:5000/softwareag_dbs/mws-oracle:10.1

docker commit 0d71bc1218c2 registry.docker.tests:5000/softwareag_dbs/bpms-oracle:10.1
```

Then, you can push each image to an internal registry for re-use accross your projects!

```bash
docker push registry.docker.tests:5000/softwareag_dbs/is-oracle:10.1

docker push registry.docker.tests:5000/softwareag_dbs/mws-oracle:10.1

docker push registry.docker.tests:5000/softwareag_dbs/bpms-oracle:10.1
```

Then, you could simply use a simpler docker file (like the one shown in [docker-compose-db-simple.yml](./docker-compose-db-simple.yml),
and create bare SAG DBs without the setup phases by executing:

For BPMS (IS/BPMS/MWS):

```bash
docker-compose -f docker-compose-db-simple.yml up -d bpms_db
```

For IS:

```bash
docker-compose -f docker-compose-db-simple.yml up -d is_db
```

For MWS:

```bash
docker-compose -f docker-compose-db-simple.yml up -d mws_db
```