# Notice


[IBM Decision Optimization on Cloud](http://www.ibm.com/software/analytics/docloud/) (or DOcplexcloud) is a service that lets you solve CPLEX, CP Optimizer, and OPL problems on the Cloud. You can access the interactive service called DropSolve or you can use use the API to integrate the service into your application. Here is a quick [introduction](http://developer.ibm.com/docloud/documentation/decision-optimization-on-cloud/). This module provides a wrapper over the REST API using Promises.

Example
-------

In this following example, we submit an OPL project made of several files. The `execute` function takes an object to configure how the job will
be created and monitored. This object provides the client with the list of attachments to create (`attachments` property) and where to get their streams. It also 
indicates if the live log must be streamed (`logstream` property) and to which stream. Additional parameters can be declared as well (`parameters` property).
The `execute` function creates the job, uploads the attachments, and monitors the execution asynchronously. It fires events when the job is created, processed, interrupted, failed, or if an error occurs. 

```
var docplexcloud = require('docplexcloud-nodejs-api');
var fs = require('fs');
var client = docplexcloud({
url : process.env.URL,
clientId : process.env.KEY
})

client.execute({
logstream : process.stdout,
parameters : { "oaas.TIME_LIMIT" : 3*60*1000},
attachments : [
{name : '.oplproject', 
stream : fs.createReadStream('test/warehouse-location/.oplproject')},
{name : 'warehouse_cloud.dat', 
stream : fs.createReadStream('test/warehouse-location/warehouse_cloud.dat')},
{name : 'warehouse_cloud.mod', 
stream : fs.createReadStream('test/warehouse-location/warehouse_cloud.mod')},
{name : 'warehouse_data.mod', 
stream : fs.createReadStream('test/warehouse-location/warehouse_data.mod')},
]})
.on('created', function(jobid){console.log(jobid+" created")})
.on('processed', function(jobid){
console.log(jobid+" processed");
client.downloadAttachment(jobid,'solution.json',fs.createWriteStream('test/warehouse-location/solution.json'))
.then(function () {return client.downloadLog(jobid,fs.createWriteStream('test/warehouse-location/solution.log'))})
})
.on('interrupted', function(jobid){console.log("job was interrupted")})
.on('failed', function(jobid){console.log("job failed")})
.on('error', function(error){console.log(error)})

```

Basic API
---------

The basic API is a simple wrapper of the DOcplexcloud REST API returning Promises. It can be used to perform simple actions and chain them using promises. To know more about the different actions, parameters, and returned information, you can refer to the REST API [documentation](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/).

```
client.listJobs()
```
Returns the list of jobs.  
**See:** [GET /jobs](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/getJobs)

```
client.deleteJobs()
```
Deletes all the jobs.  
**See:** [DELETE /jobs](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/deleteJobs)

```
client.createJob(data)
```
Creates a new job.  
**Parameter:** `data` the creation parameters.  
**See:** [POST /jobs](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/createJob)

```
client.getJob(jobid)
```
Returns a job.  
**Parameter:** `jobid` the job id.  
**See:** [GET /jobs/{id}](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/getJob)

```
client.deleteJob(jobid)
```
Deletes a job.  
**Parameter:** `jobid` the job id.  
**See:** [DELETE /jobs/{id}](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/deleteJob)

```
client.executeJob(jobid)
```
Executes a job.  
**Parameter:** `jobid` the job id.  
**See:** [POST /jobs/{id}/execute](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/startJob)

```
client.getJobExecutionStatus(jobid)
```
Returns the job execution status.  
**Parameter:** `jobid` the job id.  
**See:** [GET /jobs/{id}/execute](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/getJobStatus)

```
client.abortJob(jobid, kill)
```
Aborts a job. 
**Parameter:** `jobid` the job id.  
**Parameter:** `kill` sets the abort mode to kill.  
**See:** [DELETE /jobs/{id}/execute](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/abortJob)

```
client.uploadAttachment(jobid, attid, stream)
```
Upload an attachment; the attachment will be compressed automatically.  
**Parameter:** `jobid` the job id.  
**Parameter:** `attid` the attachment name.  
**Parameter:** `stream` the stream to read from.  
**See:** [PUT /jobs/{id}/attachments/{attid}/blob](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/uploadJobAttachment)

```
client.downloadAttachment(jobid, attid, stream)
```
Download an attachment.  
**Parameter:** `jobid` the job id.  
**Parameter:** `attid` the attachment name.  
**Parameter:** `stream` the stream to write to.  
**See:** [GET /jobs/{id}/attachments/{attid}/blob](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/downloadJobAttachment)

``` 
client.getLogItems(jobid,start,continuous) 
```
Returns the log items.  
**Parameter:** `jobid` the job id.  
**Parameter:** `start` the starting index.  
**Parameter:** `continuous` continuous mode indicator.  
**See:** [GET /jobs/{id}/log/items](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/getJobLogItems)

```
client.downloadLog(jobid, stream)
```
Download the log.  
**Parameter:** `jobid` the job id.  
**Parameter:** `stream` the stream to write to.  
**Parameter:** [GET /jobs/{id}/log/blob](https://api-swagger-oaas.docloud.ibmcloud.com/api_swagger/#!/jobs/downloadLog)

Event API
---------

The event API lets you submit and monitor your jobs in a very simple way.  

```
client.execute(data)
```
Submits and monitors a job execution.  
**Parameter:** `data` the data containing attachments, parameters.  
**Returns:** the event emitter to attach event callbacks.  

```
client.submit(data)
```
Submits but does not monitor the job.  
**Parameter:** `data` the data containing attachments, parameters.  
**Returns:** the event emitter to attach event callbacks.  

```
client.create(data)
```
Creates the job, but does not submit or monitor it.  
**Parameter:** `data` the data containing attachments, parameters.  
**Returns:** the event emitter to attach event callbacks (error and created only).  


Status
------
Under development, module API can change without notice.
