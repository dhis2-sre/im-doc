# Instance Manager API User Guide

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Audience](#audience)
- [Requirements](#requirements)
- [Getting started](#getting-started)
  - [Docs](#docs)
  - [User scripts](#user-scripts)
  - [Environment](#environment)
- [Services](#services)
  - [User](#user)
    - [Signup](#signup)
    - [Signin](#signin)
    - [Me](#me)
    - [Groups](#groups)
      - [Create group](#create-group)
      - [Cluster configuration](#cluster-configuration)
      - [Add user to group](#add-user-to-group)
  - [Instance Manager](#instance-manager)
    - [Auto login](#auto-login)
    - [Hello, World!](#hello-world)
    - [List instances](#list-instances)
    - [Deploying instances](#deploying-instances)
      - [Deploy a "monolith" instance (both DHIS2 core and a database)](#deploy-a-monolith-instance-both-dhis2-core-and-a-database)
      - [Deploy database](#deploy-database)
      - [Deploy pgAdmin](#deploy-pgadmin)
      - [Deploy DHIS2 core](#deploy-dhis2-core)
    - [Stream logs](#stream-logs)
    - [Destroy instance](#destroy-instance)
  - [Database Manager](#database-manager)
    - [Upload database](#upload-database)
    - [List databases](#list-databases)
    - [Download database](#download-database)
    - [Update database (rename)](#update-database-rename)
    - [Find database URL](#find-database-url)
    - [Find database](#find-database)
    - [Lock database](#lock-database)
    - [Unlock database](#unlock-database)
    - [Save as](#save-as)
  - [Inspector](#inspector)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Audience

This tutorial is intended for users of the Instance Manager service API.

We'll be targeting the dev environment on the test cluster found
at [https://api.im.dev.test.c.dhis2.org](https://api.im.dev.test.c.dhis2.org).

Alternatively, a small project which can assists with the creation of a locally running cluster can be
found [here](https://github.com/dhis2-sre/im-cluster).

# Requirements

The following applications are needed by the scripts

* [httpie](https://github.com/httpie/httpie)
* [jq](https://github.com/stedolan/jq)
* [direnv](https://direnv.net/) (optional, but very nice to have!)

# Getting started

Run the below command to confirm the gateway is running.

```sh
http https://api.im.dev.test.c.dhis2.org/gateway/health
```

The health status of the individual services can be found via the below links

* [User](https://api.im.dev.test.c.dhis2.org/users/health)
* [Instance](https://api.im.dev.test.c.dhis2.org/instances/health)
* [Database](https://api.im.dev.test.c.dhis2.org/databases/health)

## Docs

Each service serves its own documentation which can be found via the following links

* [User](https://api.im.dev.test.c.dhis2.org/users/docs)
* [Instance](https://api.im.dev.test.c.dhis2.org/instances/docs)
* [Database](https://api.im.dev.test.c.dhis2.org/databases/docs)

## User scripts

Each service strives to implement set of scripts such that each endpoint comes with an example of interaction.

* [User](https://github.com/dhis2-sre/im-user/tree/master/scripts)
* [Instance](https://github.com/dhis2-sre/im-manager/tree/master/scripts)
* [Database](https://github.com/dhis2-sre/im-database-manager/tree/master/scripts)

## Environment

Each service operates independently and serves its own set of end points. However, the services deployed in the "dev"
environment are all accessed through a [gateway](https://github.com/dhis2-sre/go-api-gateway).

Several services, including the gateway itself, depend on the user services so let's focus on that.

```sh
git clone git@github.com:dhis2-sre/im-user.git

cd scripts/
```

For the sake of simplicity the user scripts relies on a few environment variables. So in order to interact with the
application the environment needs to be configured.

An example of such configuration can be found in `.env.example`. It's recommended that you make a copy and populate it
with your own credentials.

```sh
cp .env.example .env
```

In order to automatically export the variables, the author of this application recommends [direnv](https://direnv.net/).

If we're targeting the "dev" environment, the variable `INSTANCE_HOST`, should be defined as below

```sh
INSTANCE_HOST=https://api.im.dev.test.c.dhis2.org
```

Alternatively, a locally running cluster could be targeted as below

```sh
INSTANCE_HOST=:8080
```

> **_NOTE:_** Since the service we're targeting is running behind the gateway its health check endpoint is no longer
> served via `/health` but rather from `/users/health`.

Considering the above, the health script needs to be updated to use the correct path.

```sh
#!/usr/bin/env bash

set -euo pipefail

$HTTP "$INSTANCE_HOST/users/health"
```

The same goes for the health scripts found for the other services.

The environment is configured correctly if the `health.sh` script returns 200 and "status: up"

```sh
./health.sh
```

# Services

In the following we'll go over each service and show examples of interacting with the various endpoints.

## User

The user service is in charge of users, groups and tokens.

### Signup

A user can be created using the `signUp.sh` script.

The script will automatically use the credentials defined in `.env`.

```sh
./signUp.sh
```

### Signin

After successfully signing up, the newly created user, can be used to sign in and retrieve an access token.

```sh
./signIn.sh
```

The above script echos the command used to export the access token as the variable ACCESS_TOKEN.

So the below command can be used as a shortcut to signin and export the access token.

```sh
export ACCESS_TOKEN && eval $(./signIn.sh) && echo $ACCESS_TOKEN
```

Assuming the signin was successful the access token will be printed on the terminal.

### Me

The details of the current user can be retrieved by running the `me.sh` script.

```sh
./me.sh
```

### Groups

For the user to actually be able to do anything it needs to be part of a group. Only administrative users can create and
add other users to groups.

The credentials for the initially created administrator can be found in `helm/data/secrets/*/values.yaml`.

Logging in as administrator can be done using the `signInAdmin.sh` script.

Alternatively, an existing administrator can add your user to the relevant groups.

#### Create group

A group can be created using the `createGroup.sh` script.

Run the below command to create a group called "test-group" with hostname "im.c.127.0.0.1.nip.io".

```sh
./createGroup.sh test-group im.c.127.0.0.1.nip.io
```

#### Cluster configuration

A cluster configuration can be added to a group using the `addClusterConfigToGroup.sh` script.

Run the below command to add a cluster configuration to the group called "test-group".

```sh
./createGroup.sh test-group ~/.kube/openstack.yaml
```

#### Add user to group

A user can be added to a given group using the `addUserToGroup.sh` script.

Run the below command to add the user with id "123" to the group called "test-group".

```sh
./addUserToGroup.sh test-group 123
```

Assuming the above returns 201 you can use the "me.sh" script to retrieve your updated user details. But before doing so
you need to sign in with non-administrative user.

```sh
export ACCESS_TOKEN && eval $(./signIn.sh) && echo $ACCESS_TOKEN
```

And then you can get your details.

```sh
./me.sh
```

Which should show the groups of which the user is a member.

## Instance Manager

The instance manager is in charge of creating, deploying, destroying and streaming of logs

```sh
git clone git@github.com:dhis2-sre/im-manager.git

cd scripts/
```

We can assert the service is running by running the below command

```sh
http https://api.im.dev.test.c.dhis2.org/instances/health
```

The service is running correctly if the above returns 200 and "status: up".

### Auto login

Configure a `.env` file matching the credentials used for signing up on the user service.

Most of the user scripts rely on the `auth.sh` script which will automatically request a new token if current token
isn't found or has expired.

### Hello, World!

The intention of the `hello.sh` script is to illustrate a "complete" workflow covering creating, deploying, streaming
of logs and finally destroying an instance.

Run the below command and click `ctrl` + `c` to destroy the instance

```sh
./hello.sh whoami test
```

### List instances

A list of instances can be retrieved using the `list.sh` script.

```sh
./list.sh
```

The instance objects and friends contains several properties which might not be relevant if you're just trying to get a
list of instance names and their groups. In which case the below command might prove handy

```sh
./list.sh | jq -r '.[] | .Name, .Instances[]?.Name'
```

### Deploying instances

#### Deploy a "monolith" instance (both DHIS2 core and a database)

DHIS2 core along with a database can be deployed using the `deploy-dhis2.sh` script.

```sh
./deploy-dhis2.sh test-group test-instance
```

#### Deploy database

A database instance can be deployed using the `deploy-dhis2-db.sh` script.

```sh
./deploy-dhis2-db.sh test-group test-db-instance
```

#### Deploy pgAdmin

A pgAdmin instance can be deployed using the `deploy-pgadmin.sh` script.

The pgAdmin instance (test-pgadmin-instance) will connect to the database (test-db-instance) and retrieve the
credentials needed to connect.

```sh
./deploy-pgadmin.sh test-group test-db-instance test-pgadmin-instance
```

#### Deploy DHIS2 core

A DHIS2 core instance can be deployed using the `deploy-dhis2-core.sh` script.

The DHIS2 core instance (test-dhis2-core-instance) will connect to the database (test-db-instance) and retrieve the
credentials needed to connect.

```sh
./deploy-dhis2-core.sh test-group test-db-instance test-dhis2-core-instance
```

### Stream logs

Logs can be streamed by using the `logs.sh` script.

Run the below command to stream logs from the instance with the name "test-instance" belonging to the group called "
test-group"

```sh
./logs.sh test-group test-instance
```

### Destroy instance

An instance can be destroyed using the `destroy.sh` script.

Run the below command to destroy the instance with the name "test-instance" belonging to the group called "test-group"

```sh
./destroy.sh test-group test-instance
```

Note that the `destroy.sh` script can be used to destroy multiple instances.

```sh
./destroy.sh test-group test-instance1 test-instance2 ...
```

## Database Manager

The Database Manager is in charge of creating, deleting, uploading, download, locking, unlocking and everything else
related to databases.

### Upload database

Databases can be uploaded by using the `upload.sh` script.

Run the below command to upload a database belonging to the group named whoami.

```sh
./upload.sh whoami ~/Downloads/dhis2-db-sierra-leone.sql.gz
```

### List databases

Databases can be listed by using the `list.sh` script.

Run the below command to list all databases.

```sh
./list.sh
```

### Download database

Databases can be downloaded using the `download.sh` script.

Run the below command to download the database with id 1

```sh
./download.sh 1 > ~/Downloads/dhis2-db-sierra-leone.sql.gz
```

### Update database (rename)

Databases can be renamed using the `update.sh` script.

Run the below command to rename the database with id "1" to "Sierra Leone Renamed"

```sh
./update.sh 1 "Sierra Leone Renamed"
```

### Find database URL

???

### Find database

A specific database can be retrieved by using the `findById.sh` script.

Run the below command to retrieve the database with id "1"

```sh
./findById.sh 1
```

### Lock database

A database can be locked by using the `lock.sh` script.

Databases can be locked by an instance. The locking instance id is stored as part of the database in the "InstanceID"
column.

Once an instance has acquired a lock only that instance can save the database.

Other users can still launch instances seeded with a locked database but won't be able to save to the original database.
However, saving to a new destination using "save as" is still possible

Run the below command to lock the database with id "1" to the instance with id "2"

```sh
./lock.sh 1 2
```

### Unlock database

A database can be unlocked by using the `unlock.sh` script.

Run the below command to unlock the database with id "1"

```sh
./unlock.sh 1
```

### Save as

A database can be saved, under a new name, by using the `unlock.sh` script.

Run the below command to save the database associated with the instance named `db-1` in the group `whoami`
as `new-database.pgc`

```sh
./saveAs.sh whoami db-1 new-database.pgc custom
```

## Inspector

...
