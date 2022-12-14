### Install

Firstly, launch npm install process in order to download and install all dependencies.

```
npm install
```

### Configure

#### Auto Configuration (using of Mapping Tool)

Then, configure the properties file in order to set parameters about North side (Context Broker), agent server and South
side (OPC UA endpoint).

```
#SOUTHBOUND CONFIGURATION (OPC UA)
#Namespace to ignore
namespace-ignore=2,7
################ OPTIONAL FILTERING ################
nodes-filtering-in=ns\=1;s\=Oxigen,ns\=1;s\=Speed
nodes-filtering-out=ns\=1;s\=Temperature
####################################################
#OPC UA Endpoint
endpoint=opc.tcp://localhost:4334/UA/CarServer

#NORTHBOUND CONFIGURATION (ORION CB)
context-broker-host=192.168.56.101
context-broker-port=1026
fiware-service=opcua_car
fiware-service-path=/demo

#AGENT SERVER CONFIGURATION
server-base-root=/
server-port=4041
device-registry-type=memory
provider-url=http://192.168.56.1:4041
device-registration-duration=P1M
log-level=INFO
#MongoDB Agent Config
mongodb-host=192.168.56.101
mongodb-port=27017
mongodb-db=iotagent
mongodb-retries=5
mongodb-retry-time=5

#DATATYPE MAPPING OPCUA --> NGSI
OPC-datatype-Number=Number
OPC-datatype-Decimal128=Number
OPC-datatype-Double=Number
OPC-datatype-Float=Number
OPC-datatype-Integer=Integer
OPC-datatype-UInteger=Integer
OPC-datatype-String=Text
OPC-datatype-ByteString=Text
#END DATATYPE MAPPING OPCUA --> NGSI
```

Using of Auto Configuration create a mapping for all OPC UA objects (except those with namespace to ignore matching):
all OPC UA variables will be configured as active attributes whereas all OPC UA methods will be configured as commands.
It is possible modify configuration output (config.json file in same path) manually in order to drop some
attributes/command or add lazy attributes.

#### Manual Configuration (editing config.json file)

```
{
    "logLevel": "INFO",
    "contextBroker": {
        "host": "192.168.56.101",
        "port": 1026
    },
    "server": {
        "port": 4041,
        "baseRoot": "/"
    },
    "deviceRegistry": {
        "type": "memory"
    },
    "mongodb": {
        "host": "192.168.56.101",
        "port": "27017",
        "db": "iotagent",
        "retries": 5,
        "retryTime": 5
    },
    "types": {
        "Car": {
            "service": "opcua_car",
            "subservice": "/demo",
            "active": [
                {
                    "name": "Engine_Temperature",
                    "type": "Number"
                },
                {
                    "name": "Engine_Oxigen",
                    "type": "Number"
                }
            ],
            "lazy": [
            	{
                    "name": "Speed",
                    "type": "Number"
                }
            ],
            "commands": [
                {
                    "name": "Stop",
                    "type": "command"
                },
                {
                    "name": "Accelerate",
                    "type": "command"
                }
            ]
        }
    },
    "browseServerOptions": null,
    "service": "opcua_car",
    "subservice": "/demo",
    "providerUrl": "http://192.168.56.1:4041",
    "deviceRegistrationDuration": "P1M",
    "defaultType": null,
    "contexts": [
        {
            "id": "Car_1",
            "type": "Car",
            "service": "opcua_car",
            "subservice": "/demo",
            "mappings": [
                {
                    "ocb_id": "Engine_Temperature",
                    "opcua_id": "ns=1;s=Temperature",
                    "object_id": null,
                    "inputArguments": []
                },
                {
                    "ocb_id": "Engine_Oxigen",
                    "opcua_id": "ns=1;s=Oxigen",
                    "object_id": null,
                    "inputArguments": []
                }
            ]
        }
    ],
    "contextSubscriptions": [
        {
            "id": "Car",
            "type": "Car",
            "mappings": [
                {
                    "ocb_id": "Stop",
                    "opcua_id": "ns=1;s=Stop",
                    "object_id": "ns=1;i=1000",
                    "inputArguments": []
                },
                {
                    "ocb_id": "Accelerate",
                    "opcua_id": "ns=1;s=Accelerate",
                    "object_id": "ns=1;i=1000",
                    "inputArguments": [
                        {
                            "dataType": 7,
                            "type": "Intensity"
                        }
                    ]
                },
                {
                    "ocb_id": "Speed",
                    "opcua_id": "ns=1;s=Speed",
                    "object_id": null,
                    "inputArguments": []
                }
            ]
        }
    ]
}
```

### Run

Finally, run the agent.

```
node index.js
```
