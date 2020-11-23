# Node-RED MS SQL to IoT Edge

This repo contains a sample project that highlights using [Node-RED](https://nodered.org/) to query a Microsoft SQL database (in this example [Azure SQL Edge](https://azure.microsoft.com/en-us/services/sql-edge/)) and send the data up to [IoT Hub](https://azure.microsoft.com/en-us/services/iot-hub/) via [IoT Edge](https://docs.microsoft.com/en-us/azure/iot-edge/about-iot-edge?view=iotedge-2018-06). This sample exclusively uses publicly avaialble containers as modules so no Docker build steps are necessary.

## Modules

The deployment manifest template in this repository (`deployment.template.json`) defines two modules for this project:

#### Azure SQL Edge

The [Azure SQL Edge](https://azure.microsoft.com/en-us/services/sql-edge/) module included in this project is an example of *one* possible Microsoft SQL database that could be queried.  In production, the Azure SQL Edge module can be deleted from the deployment manifest template and another source could be defined in Node-RED such as a standalone SQL Server Instance or perhaps a SQL Server instance backing an [Operational Historian](https://en.wikipedia.org/wiki/Operational_historian) in an industrial scenario.

#### Node-RED

[Node-RED](https://nodered.org/) is an open source workflow programming tool which can connect, transform, and send data to and from a wide variety of different platforms.  In this project we define two flows, one which connects repeatedly to the Azure SQL Edge module, and sets up a test database and table if needed, then populates them with random fake sensor data.  The other flow repeatedly queries this data and sends it to a corresponding IoT Edge module which routes data back to IoT Hub via .  This second flow represents what we would see in a real-world implementation.

There are two NPM packages leveraged that install plugins for Node-RED.  The first is [node-red-contrib-mssql-plus](https://flows.nodered.org/node/node-red-contrib-mssql-plus) which we can use to connect and query Microsoft SQL Server versiosn 2000-2019, as well as Azure Databases including Azure SQL Edge. The second is [node-red-contrib-azure-iot-edge-module](https://flows.nodered.org/node/node-red-contrib-azure-iot-edge-module) which allowes Node-RED to communciate securely with IoT Edge, and the second is 

## Deployment

#### IoT Edge
If you aren't familiar with the steps needed to configure IoT Edge, please see [this document](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge?view=iotedge-2018-06&tabs=linux).  Once installed, to deploy this sample, simply follow the instructions given to [Deploy to IoT Edge with Visual Studio Code](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-deploy-modules-vscode?view=iotedge-2018-06).  You can also use this project as a reference and [deploy modules using the Azure Portal](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-deploy-modules-portal?view=iotedge-2018-06).  If you're using [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/) you can also deploy continuously to IoT Edge with the [Azure DevOps IoT Edge Task](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-continuous-integration-continuous-deployment?view=iotedge-2018-06).

Once the modules are deployed and running on IoT Edge, the Node-RED UI should be available over http on port 1880.

#### Node-RED
As the deployment template defines a [Docker bind mount](https://docs.docker.com/storage/bind-mounts/) of `/iot-edge-data/node-red` for the node-RED container, we need to ensure the `data/node-red` folder from this project is moved to that location and is writable by whatever user IoT Edge is running as (often `iotedge`).  This ensure that Node-RED can read the appropraite package list and flows.

After first setup, you may notice the two required plugins are missing from Node-RED.  The publicly distributed Node-RED container does not actually run `npm install` on startup, so we need to run the following commands one time:

```
docker exec -it NodeRED /bin/bash;
cd /data;
npm install;
```

Once the installation has completed we can exit the docker container and restart the Node-RED module with:

```
iotedge restart NodeRED
```

*Note: In a productionized version of this flow, you'd probably want to build these packages into your own container build and release process.*

#### IoT Explorer

After installation is complete, the Edge modules should boot up, begin querying the data store, and sending telemetry up to IoT Hub.  You can easily view this telemetry using the [Azure IoT Explorer](https://docs.microsoft.com/en-us/azure/iot-pnp/howto-use-iot-explorer).  The data being set up should look the sample below, with constantly changing values:

```json
{
  "body": [
    {
      "tag": "ITEM_COUNT_GOOD",
      "value": 97
    },
    {
      "tag": "ITEM_COUNT_BAD",
      "value": 4
    },
    {
      "tag": "YIELD",
      "value": 32
    }
  ],
  "enqueuedTime": "2020-11-22T23:49:59.421Z"
}
```
