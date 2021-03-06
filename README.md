Tutorial quality responsibility: [@noor-latif ](https://github.com/noor-latif)
# Node-CLI
Using Node.js for automatic device provisioning towards Azure   
[What is Azure IoT Hub Device Provisioning Service?](https://docs.microsoft.com/en-us/azure/iot-dps/about-iot-dps)

- [Node-CLI](#node-cli)
  * [Setup](#setup)
    + [Requirements](#requirements)
  * [1. Initial Azure Infrastructure](#1-initial-azure-infrastructure)
    + [a. Create IoT Hub](#a-create-iot-hub)
    + [b. Create Device Provisioning Service](#b-create-device-provisioning-service)
  * [2. Create enrollment sample in node.js](#2-create-enrollment-sample-in-nodejs)
    + [Requirements:](#requirements-)
  * [3. CLI Usage](#3-cli-usage)

## Setup
For the device enrollment script to work, we first need to setup our Azure infrastructure.
We can create this with Azure CLI for Windows. [Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?SilentAuth=1&activetab=pivot:overviewtab) lets use connect to Azure Cloud Shell to perform the commands:

### Requirements
- Azure CLI for Windows ([Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?SilentAuth=1&activetab=pivot:overviewtab))
- Change elastic-resource to your own resource group name in commands below.
- Change devhub-iot to your own resource group name in commands below.
  (the easiest way is to copy this guide and do a search & replace for your own names)
- If you get an error, it most likely means that the name you chose has already been picked.

## 1. Initial Azure Infrastructure
### a. Create IoT Hub
**-- install az iot extension in Azure CLI --**
```
az extension add --upgrade --name azure-iot
```
**-- create resource group --**
```
az group create --location westeurope --name elastic-resource
```
**-- set resource group as default (optional) --**
```
az configure --defaults group=elastic-resource
```
**-- create iot hub --**
```
az iot hub create --location westeurope --resource-group elastic-resource --name devhub-iot --sku F1 --partition-count 2
```
### b. Create Device Provisioning Service
-- Create the service --  
```
az iot dps create --name auto --resource-group elastic-resource --location westeurope
```
-- Get the IoT Hub connection string -- 
```
$env:hubConnectionString=$(az iot hub connection-string show --hub-name devhub-iot --key-type primary --resource-group elastic-resource -o tsv)
```
-- Display the connection string (it's now saved as a system variable) -- 
```
echo $env:hubConnectionString
```
-- Link the new DPS to our IoT Hub with the newly acquired connection string --
```
az iot dps linked-hub create --dps-name auto --resource-group elastic-resource --connection-string $env:hubConnectionString --location westeurope
```
-- Verify that link was successful (look for "iotHubs" in the list to see linked iot hub under "name" -- 
```
az iot dps show --name auto
```
-- Get the DPS connection string and save it to system variable -- 
```
$env:dpsConnectionString=$(az iot dps connection-string show --dps-name auto -o tsv)
```
-- Display the DPS connection string (it's now saved as a system variable) -- 
```
echo $env:dpsConnectionString
```
## 2. Create enrollment sample in node.js
### Requirements:
Your system variables should have connection strings saved to them. 
Try this with "echo" command, it should display connection strings.
* npm install azure-iot-provisioning-service
* echo $env:dpsConnectionString
* echo $env:hubConnectionString
* endorsement key: 
AToAAQALAAMAsgAgg3GXZ0SEs/gakMyNRqXXJP1S124GUgtk8qHaGzMUaaoABgCAAEMAEAgAAAAAAAEAxsj2gUScTk1UjuioeTlfGYZrrimExB+bScH75adUMRIi2UOMxG1kw4y+9RW/IVoMl4e620VxZad0ARX2gUqVjYO7KPVt3dyKhZS3dkcvfBisBhP1XH9B33VqHG9SHnbnQXdBUaCgKAfxome8UmBKfe+naTsE5fkvjb/do3/dD6l4sGBwFCnKRdln4XpM03zLpoHFao8zOwt8l/uP3qUIxmCYv9A7m69Ms+5/pCkTu/rK4mRDsfhZ0QLfbzVI6zQFOKF/rwsfBtFeWlWtcuJMKlXdD8TXWElTzgh7JS4qhFzreL0c1mI0GCj+Aws0usZh7dLIVPnlgZcBhgy1SSDQMQ==
* (for production you need to create your own endorsement key)

## 3. CLI Usage
The Node CLI takes 2 parameters on launch.
```
node-dps.js <dps connection string> <endorsement key>
```
Example usage in powershell:
```
node .\node-dps.js $env:dpsConnectionString AToAAQALAAMAsgAgg3GXZ0SEs/gakMyNRqXXJP1S124GUgtk8qHaGzMUaaoABgCAAEMAEAgAAAAAAAEAxsj2gUScTk1UjuioeTlfGYZrrimExB+bScH75adUMRIi2UOMxG1kw4y+9RW/IVoMl4e620VxZad0ARX2gUqVjYO7KPVt3dyKhZS3dkcvfBisBhP1XH9B33VqHG9SHnbnQXdBUaCgKAfxome8UmBKfe+naTsE5fkvjb/do3/dD6l4sGBwFCnKRdln4XpM03zLpoHFao8zOwt8l/uP3qUIxmCYv9A7m69Ms+5/pCkTu/rK4mRDsfhZ0QLfbzVI6zQFOKF/rwsfBtFeWlWtcuJMKlXdD8TXWElTzgh7JS4qhFzreL0c1mI0GCj+Aws0usZh7dLIVPnlgZcBhgy1SSDQMQ==
```
