# FIWARE IoT Agent for the OPC-UA

[![FIWARE IoT Agents](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/iot-agents.svg)](https://www.fiware.org/developers/catalogue/)
[![](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/iot-agents.svg)](https://stackoverflow.com/questions/tagged/fiware+iot)

An Internet of Things Agent accepting data from OPC UA devices. This IoT Agent is designed to be a bridge between the
OPC Unified Architecture protocol and the
[NGSI](https://swagger.lab.fiware.org/?url=https://raw.githubusercontent.com/Fiware/specifications/master/OpenAPI/ngsiv2/ngsiv2-openapi.json)
interface of a context broker.

The intended level of complexity to support these operations should consider a limited human intervention (mainly during
the setup of a new OPC UA endpoint), through the mean of a parametrization task (either manual or semi-automatic, using
a text-based parametrization or a simple UI to support the configuration) so that no software coding is required to
adapt the agent to different OPC UA devices.

It is based on the [IoT Agent Node.js Library](https://github.com/telefonicaid/iotagent-node-lib). Further general
information about the FIWARE IoT Agents framework, its architecture and the common interaction model can be found in the
library's GitHub repository.

This project is part of [FIWARE](https://www.fiware.org/). For more information check the FIWARE Catalogue entry for the
[IoT Agents](https://github.com/Fiware/catalogue/tree/master/iot-agents).

## How to use this image

The IoT Agent must be instantiated and connected to an instance of the
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/), a sample `docker-compose` file can be found
below.

If the `IOTA_REGISTRY_TYPE=mongodb`, a [MongoDB](https://www.mongodb.com/) database instance is also required - the
example below assumes that you have a `/data` directory in your hosting system in order to hold database files - please
amend the attached volume to suit your own configuration.

```yml
version: "3.1"

services:
    iotcarsrv:
        hostname: iotcarsrv
        image: iotagent4fiware/opcuacarsrv
        volumes:
            - ./CARCONF/car_config.json:/opt/opc-ua-car-server/Car/Car1/config/car_config.json
        networks:
            - hostnet
        ports:
            - "5001:5001"

    iotage:
        hostname: iotage
        image: iotagent4fiware/iotagent-opcua
        networks:
            - hostnet
            - iotnet
        ports:
            - "4001:4001"
            - "4081:8080"
        depends_on:
            - iotcarsrv
            - iotmongo
            - orion
        volumes:
            - ./AGECONF:/opt/iotagent-opcua/conf
            - ./certificates:/opt/iotagent-opcua/certificates
        command: /usr/bin/tail -f /var/log/lastlog

    iotmongo:
        hostname: iotmongo
        image: mongo:3.4
        networks:
            - iotnet
        volumes:
            - iotmongo_data:/data/db
            - iotmongo_conf:/data/configdb

    ################ OCB ################

    orion:
        hostname: orion
        image: fiware/orion:latest
        networks:
            - hostnet
            - ocbnet
        ports:
            - "1026:1026"
        depends_on:
            - orion_mongo
        command: -dbhost orion_mongo

    orion_mongo:
        hostname: orion_mongo
        image: mongo:3.6
        networks:
            ocbnet:
                aliases:
                    - mongo
        ports:
            - "27017:27017"
        command: --bind_ip_all --smallfiles
        volumes:
            - mongodb:/data

volumes:
    iotmongo_data: ~
    iotmongo_conf: ~
    mongodb: ~

networks:
    hostnet: ~
    iotnet: ~
    ocbnet: ~
```

## Configuration with environment variables

Many settings can be configured using Docker environment variables. A typical IoT Agent Docker container is driven by
environment variables such as those shown below:

-   `IOTA_CB_HOST` - Hostname of the context broker to update context
-   `IOTA_CB_PORT` - Port that context broker listens on to update context
-   `IOTA_NORTH_PORT` - Port used for configuring the IoT Agent and receiving context updates from the context broker
-   `IOTA_REGISTRY_TYPE` - Whether to hold IoT device info in memory or in a database
-   `IOTA_MONGO_HOST` - The hostname of MongoDB - used for holding device and service information
-   `IOTA_MONGO_PORT` - The port that MongoDB is listening on
-   `IOTA_MONGO_DB` - The name of the database used in MongoDB
-   `IOTA_HTTP_PORT` - The port where the IoT Agent listens for IoT device traffic over HTTP
-   `IOTA_PROVIDER_URL` - URL passed to the Context Broker when commands are registered, used as a forwarding URL
    location when the Context Broker issues a command to a device

### Further Information

The full set of overrides for the general parameters applicable to all IoT Agents are described in the Configuration
section of the IoT Agent Library
[Installation Guide](https://iotagent-node-lib.readthedocs.io/en/latest/installationguide/index.html#configuration).

Further settings for IoT Agent for OPC-UA itself - can be found in the IoT Agent for OPC-UA
[Installation Guide](https://iotagent-opcua.readthedocs.io/en/latest/installationguide/index.html#configuration).

## How to build an image

The [Dockerfile](https://github.com/Engineering-Research-and-Development/iotagent-opcua/blob/master/docker/Dockerfile)
associated with this image can be used to build an image in several ways:

-   By default, the `Dockerfile` retrieves the **latest** version of the codebase direct from GitHub (the `build-arg` is
    optional):

```console
docker build -t iot-agent . --no-cache --build-arg DOWNLOAD=latest
```

-   You can alter this to obtain the last **stable** release run this `Dockerfile` with the build argument
    `DOWNLOAD=stable`

```console
docker build -t iot-agent . --no-cache --build-arg DOWNLOAD=stable
```

-   You can also download a specific release by running this `Dockerfile` with the build argument `DOWNLOAD=<version>`

```console
docker build -t iot-agent . --no-cache --build-arg DOWNLOAD=1.7.0
```

## Building from your own fork

To download code from your own fork of the GitHub repository add the `GITHUB_ACCOUNT`, `GITHUB_REPOSITORY` and
`SOURCE_BRANCH` arguments (default `master`) to the `docker build` command.

```console
docker build -t iot-agent . \
    --build-arg GITHUB_ACCOUNT=<your account> \
    --build-arg GITHUB_REPOSITORY=<your repo> \
    --build-arg SOURCE_BRANCH=<your branch>
```

## Building from your own source files

Alternatively, if you want to build directly from your own sources, please copy the existing `Dockerfile` into file the
root of the repository and amend it to copy over your local source using :

```Dockerfile
COPY . /opt/iotagent-opcua/
```

Full instructions can be found within the `Dockerfile` itself.

### Using PM2

The IoT Agent within the Docker image can be run encapsulated within the [pm2](http://pm2.keymetrics.io/) Process
Manager by adding the `PM2_ENABLED` environment variable.

```console
docker run --name iotagent -e PM2_ENABLED=true -d iotagent4fiware/iotagent-opcua
```

Use of pm2 is **disabled** by default. It is unnecessary and counterproductive to add an additional process manager if
your dockerized environment is already configured to restart Node.js processes whenever they exit (e.g. when using
[Kubernetes](https://kubernetes.io/))

### Docker Secrets

As an alternative to passing sensitive information via environment variables, `_FILE` may be appended to some sensitive
environment variables, causing the initialization script to load the values for those variables from files present in
the container. In particular, this can be used to load passwords from Docker secrets stored in
`/run/secrets/<secret_name>` files. For example:

```console
docker run --name iotagent -e IOTA_AUTH_PASSWORD_FILE=/run/secrets/password -d iotagent4fiware/iotagent-opcua
```

Currently, this `_FILE` suffix is supported for:

-   `IOTA_AUTH_USER`
-   `IOTA_AUTH_PASSWORD`
-   `IOTA_AUTH_CLIENT_ID`
-   `IOTA_AUTH_CLIENT_SECRET`
-   `IOTA_MONGO_USER`
-   `IOTA_MONGO_PASSWORD`


## Best Practices

### Increase ULIMIT in Docker Production Deployments

Default settings for ulimit on a Linux system assume that several users would share the system. These settings limit the
number of resources used by each user. The default settings are generally very low for high performance servers and
should be increased. By default, we recommend, that the IoTAgent - UL server in high performance scenarios, the
following changes to ulimits:

```console
ulimit -n 65535        # nofile: The maximum number of open file descriptors (most systems do not allow this
                                 value to be set)
ulimit -c unlimited    # core: The maximum size of core files created
ulimit -l unlimited    # memlock: The maximum size that may be locked into memory
```

If you are just doing light testing and development, you can omit these settings, and everything will still work.

To set the ulimits in your container, you will need to run IoTAgent - UL Docker containers with the following additional
--ulimit flags:

```console
docker run --ulimit nofile=65535:65535 --ulimit core=100000000:100000000 --ulimit memlock=100000000:100000000 \
--name iotagent -d iotagent4fiware/iotagent-opcua
```

Since “unlimited” is not supported as a value, it sets the core and memlock values to 100 GB. If your system has more
than 100 GB RAM, you will want to increase this value to match the available RAM on the system.

> Note: The --ulimit flags only work on Docker 1.6 or later. Nevertheless, you have to "request" more resources (i.e.
> multiple cores), which might be more difficult for orchestrates ([Docker Engine](https://docs.docker.com/engine) or
> [Kubernetes](https://kubernetes.io)) to schedule than a few different containers requesting one core (or less...) each
> (which it can, in turn, schedule on multiple nodes, and not necessarily look for one node with enough available
> cores).

If you want to get more details about the configuration of the system and node.js for high performance scenarios, please
refer to the [Installation Guide](https://fiware-iotagent-ul.rtfd.io/en/latest/installationguide/index.html).

### Set-up appropriate Database Indexes

If using Mongo-DB as a data persistence mechanism (i.e. if `IOTA_REGISTRY_TYPE=mongodb`) the device and service group
details are retrieved from a database. The default name of the IoT Agent database is `iotagentopcua`. Database access
can be optimized by creating appropriate indices.

For example:

```console
docker exec  <mongo-db-container-name> mongo --eval '
    conn = new Mongo();
    db = conn.getDB("iotagentul");
    db.createCollection("devices");
    db.devices.createIndex({"_id.service": 1, "_id.id": 1, "_id.type": 1});
    db.devices.createIndex({"_id.type": 1});
    db.devices.createIndex({"_id.id": 1});
    db.createCollection("groups");
    db.groups.createIndex({"_id.resource": 1, "_id.apikey": 1, "_id.service": 1});
    db.groups.createIndex({"_id.type": 1});' > /dev/null
```

The name of the database can be altered using the `IOTA_MONGO_DB` environment variable. Alter the `conn.getDB()`
statement above if an alternative database is being used.
