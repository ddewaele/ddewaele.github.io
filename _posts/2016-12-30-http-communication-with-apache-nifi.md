---
layout: post
title: HTTP communication with Apache NiFi
date: 2016-12-29 15:28:14 +0200
excerpt_separator: <!--more-->
---

# Introduction 

In this post we'll be looking at several ways to use NiFi to interact with HTTP resources, both from a client and from a server perspective.

Nifi comes with a set of core processors allowing you to interact with filesystems, MQTT brokers, Hadoop filesystems, Kafka, ....... It also comes bundled with a set of HTTP processors that you can use to either expose or consume HTTP based resources.

We'll be looking at the following processors that ship with Nifi:

- [GetHTTP](#GetHTTP)
- [PostHTTP](#PostHTTP)
- [ListenHTTP](#ListenHTTP)
- [InvokeHTTP](#InvokeHTTP)
- [HandlerHttpRequest](#HandlerHttpRequest)

For each processor we're going to take a closer look at

- Flow definition (how a typical NiFi flow would look like with this processor)
- Processor configuration (how the processor can be configured)
- Scheduling (how the processor is scheduled)
- Data flow (how the data flows within the flow)
- Thoughts and use-cases

<!--more-->

<a name="GetHTTP"></a>

# GetHTTP

## Flow definition

Where as the ListenHTTP acts as an HTTP server, exposing an HTTP resource for the outside world to consume, The [GetHTTP processor](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi.processors.standard.GetHTTP/) is a true client. Its job is to connect to an HTTP endpoint, and process the response as a FlowFile so that it can be further pushed upstream.

In this example we're going to be using the GetHTTP processor to access the Nifi REST API. 

![]({{ site.url }}/assets/images/nifi/http/getHttp-flow.png)

## Processor configuration

The GetHTTP processor needs to be configured with the target host / path that we want to connect to.

![]({{ site.url }}/assets/images/nifi/http/getHttp-props.png)

Although it is nice that the GetTCP processor supports the NiFi Expression Language so that we can make the URL dynamic, the fact that the processor doesn't accept incoming flowfiles does limit its use-cases.

Suppose we have a dynamic list of REST endpoints that we want to query then we cannot use this processor. For that we would need to look at the InvokeHTTP processor.

## Processor scheduling

Before we didn't need to schedule the processor, as it was constantly listening for HTTP requests. As we now have a processor from a client perspective, we don't want it constantly firing HTTP requests. As this processor cannot be triggered by incoming flowfiles, we need to setup the scheduling ourselves. This processor is typically scheduled using either a timer or cron based schedule.

![]({{ site.url }}/assets/images/nifi/http/getHttp-scheduling.png)

## Data flow

If we look at the logfile you should see this every 30 seconds :

```
2016-12-28 16:58:33,561 INFO [Timer-Driven Process Thread-4] o.a.n.processors.standard.LogAttribute LogAttribute[id=01591074-6575-1391-9453-fc3988d88965] logging for flow file StandardFlowFileRecord[uuid=71a6ceed-312d-4d81-bb2d-f222f88f7b30,claim=StandardContentClaim [resourceClaim=StandardResourceClaim[id=1482936126260-2, container=default, section=2], offset=211997, length=1594],offset=0,name=system-diagnostics,size=1594]
--------------------------------------------------
Standard FlowFile Attributes
Key: 'entryDate'
        Value: 'Wed Dec 28 16:58:33 UTC 2016'
Key: 'lineageStartDate'
        Value: 'Wed Dec 28 16:58:33 UTC 2016'
Key: 'fileSize'
        Value: '1594'
FlowFile Attribute Map Content
Key: 'filename'
        Value: 'system-diagnostics'
Key: 'gethttp.remote.source'
        Value: 'localhost'
Key: 'mime.type'
        Value: 'application/json'
Key: 'path'
        Value: './'
Key: 'uuid'
        Value: '71a6ceed-312d-4d81-bb2d-f222f88f7b30'
--------------------------------------------------
{"systemDiagnostics":{"aggregateSnapshot":{"totalNonHeap":"182.16 MB","totalNonHeapBytes":191004672,"usedNonHeap":"172.2 MB","usedNonHeapBytes":180562144,"freeNonHeap":"9.96 MB","freeNonHeapBytes":10442528,"maxNonHeap":"-1 bytes","maxNonHeapBytes":-1,"totalHeap":"512 MB","totalHeapBytes":536870912,"usedHeap":"255.32 MB","usedHeapBytes":267722144,"freeHeap":"256.68 MB","freeHeapBytes":269148768,"maxHeap":"512 MB","maxHeapBytes":536870912,"heapUtilization":"50.0%","availableProcessors":8,"processorLoadAverage":4.35888671875,"totalThreads":104,"daemonThreads":41,"flowFileRepositoryStorageUsage":{"freeSpace":"1.68 GB","totalSpace":"464.77 GB","usedSpace":"463.09 GB","freeSpaceBytes":1808060416,"totalSpaceBytes":499046809600,"usedSpaceBytes":497238749184,"utilization":"100.0%"},"contentRepositoryStorageUsage":[{"identifier":"default","freeSpace":"1.68 GB","totalSpace":"464.77 GB","usedSpace":"463.09 GB","freeSpaceBytes":1808060416,"totalSpaceBytes":499046809600,"usedSpaceBytes":497238749184,"utilization":"100.0%"}],"garbageCollection":[{"name":"G1 Young Generation","collectionCount":18470,"collectionTime":"00:01:11.568","collectionMillis":71568},{"name":"G1 Old Generation","collectionCount":1,"collectionTime":"00:00:00.648","collectionMillis":648}],"statsLastRefreshed":"16:58:33 UTC","versionInfo":{"javaVendor":"Oracle Corporation","javaVersion":"1.8.0_40","osName":"Mac OS X","osVersion":"10.10.5","osArchitecture":"x86_64","buildTag":"nifi-1.1.0-RC2","buildRevision":"f61e42c","buildBranch":"NIFI-3100-rc2","buildTimestamp":"11/26/2016 04:39:37 UTC","niFiVersion":"1.1.0"}}}}
```

The NiFi provenance viewer will also show you a nice formattedview of the payload

![]({{ site.url }}/assets/images/nifi/http/json-payload.png)

## Thoughts and use-cases

- Very simple processor with only 1 outgoing relationship (success)
- No error transitions possible
- Does not allow for incoming connections
- Good for simple polling scenarios, where NiFi needs to periodically fetch data from a remote resource.


<a name="PostHTTP"></a>

## PostHTTP

### Flow definition

[PostHTTP](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi.processors.standard.PostHTTP/) is similar to [GetHTTP processor](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi.processors.standard.GetHTTP/), acting as an HTTP client only this time to execute HTTP POST requests. Its job is to connect to an HTTP endpoint, execute the POST and process the response as a FlowFile so that it can be further pushed upstream.

In this example we're going to be using the PostHTTP processor to access a simple REST API REST API. 

![]({{ site.url }}/assets/images/nifi/http/postHttp-flow.png)

### Processor configuration

The PostHTTP processor needs to be configured with the target host / path that we want to connect to.

![]({{ site.url }}/assets/images/nifi/http/postHttp-props.png)

In contrast to the GetTCP processor, the PostHTTP processor does allow incoming flowfiles so that they can be used as the payload of the POST request.

### Processor scheduling

Before we didn't need to schedule the processor, as it was constantly listening for HTTP requests. As we now have a processor from a client perspective, we don't want it constantly firing HTTP requests. As this processor cannot be triggered by incoming flowfiles, we need to setup the scheduling ourselves. This processor is typically scheduled using either a timer or cron based schedule.

![]({{ site.url }}/assets/images/nifi/http/getHttp-scheduling.png)

### Data flow

If we look at the logfile you should see this every 30 seconds :

```
2016-12-30 20:57:47,662 INFO [Timer-Driven Process Thread-3] o.a.n.processors.standard.LogAttribute LogAttribute[id=13911575-1180-107b-6ee4-0040928adf51] logging for flow file StandardFlowFileRecord[uuid=407681f9-fd1a-4c14-8bea-c2dd0230540b,claim=StandardContentClaim [resourceClaim=StandardResourceClaim[id=1483127389761-379, container=default, section=379], offset=885598, length=122],offset=0,name=2420839022986047,size=122]
--------------------------------------------------
Standard FlowFile Attributes
Key: 'entryDate'
        Value: 'Fri Dec 30 20:57:47 UTC 2016'
Key: 'lineageStartDate'
        Value: 'Fri Dec 30 20:57:47 UTC 2016'
Key: 'fileSize'
        Value: '122'
FlowFile Attribute Map Content
Key: 'filename'
        Value: '2420839022986047'
Key: 'path'
        Value: './'
Key: 'uuid'
        Value: '407681f9-fd1a-4c14-8bea-c2dd0230540b'
--------------------------------------------------
{
   "firstName":"first_d6e20941-b061-4b5e-a7dc-a7aca5201652",
   "lastName":"last_b3b8a5ce-7b3b-4044-a3f3-80de807b6bb4"
}
```

The NiFi provenance viewer will also show you a nice formattedview of the payload

![]({{ site.url }}/assets/images/nifi/http/json-payload.png)

### Thoughts and use-cases

- Very simple processor with only 1 outgoing relationship (success)
- No error transitions possible
- Does not allow for incoming connections
- Good for simple polling scenarios, where NiFi needs to periodically fetch data from a remote resource.

<a name="ListenHTTP"></a>

## ListenHTTP

### Flow definition

The [ListenHTTP processor](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi.processors.standard.ListenHTTP/) starts an HTTP endpoint / web server. As such you can think of it as a server component. 

Nifi runs by default on port 8080, and can be accessed using [http://localhost:8080/nifi](http://localhost:8080/nifi). When you start a ListenHTTP processor you start an additional webserver for the sole purpose of handling requests from external clients and converting those requests into Flowfiles that you use within your flow.

(External) clients can connect to the HTTP endpoint and the ListenHTTP processor will take care of creating a FlowFile for that particular client request.

We'll start with a very simple flow that will hookup our ListenHTTP processor to a LogAttribute, so we can see in the nifi logfile how requests are processed.

![]({{ site.url }}/assets/images/nifi/http/listenHttp-flow.png)

The request sent to the processor can contain a body. Use-case where you wouldn't need a body is if you were us the processor to trigger a certain action (set something in motion inside your dataflow).

This processor is limited in functionality.

- it will always return an HTTP 200 code to the client.
- it only supports the HTTP POST method
- it does not support incoming flowfiles. It can only be triggered from outside the flow.

It does have some throttling capabilities and allows for source ip whitelisting.

### Processor configuration

The ListenHTTP processor requires little configuration. We need to provide the HTTP listening port and the basepath (contextpath) that clients will need to use to access it.

![]({{ site.url }}/assets/images/nifi/http/listenHttp-props.png)

With this configuration, Nifi will start a webserver on port 7001, a

Unfortunately it does not support the Nifi expression language. So port and basePath need to be "hardcoded" in the processor.

### Processor scheduling

As far a scheduling is concerned we can keep the defaults. The processor will be constantly triggered to see if there were any incoming requests needing to be processed.

![]({{ site.url }}/assets/images/nifi/http/listenHttp-scheduling.png)

### Data flow

With that flow and configuration in place, clients can start sending requests to the endpoint. We'll start by sending an HTTP POST request without any data using curl. You should receive an HTTP 200 response without a body.

```
$ curl -v -X POST http://localhost:7001/contentListener
```

In the nifi log file (${NIFI_HOME}/logs/nifi-app.log) you should see this:

```
2016-12-28 16:47:07,358 INFO [Timer-Driven Process Thread-3] o.a.n.processors.standard.LogAttribute LogAttribute[id=0159106b-6575-1391-6e08-a158b7c2c21a] logging for flow file StandardFlowFileRecord[uuid=332290fb-f7e7-4fbe-a91b-efbb2935232d,claim=StandardContentClaim [resourceClaim=StandardResourceClaim[id=1482936126260-2, container=default, section=2], offset=494, length=0],offset=0,name=2233016934597090,size=0]
--------------------------------------------------
Standard FlowFile Attributes
Key: 'entryDate'
        Value: 'Wed Dec 28 16:47:07 UTC 2016'
Key: 'lineageStartDate'
        Value: 'Wed Dec 28 16:47:07 UTC 2016'
Key: 'fileSize'
        Value: '0'
FlowFile Attribute Map Content
Key: 'filename'
        Value: '2233016934597090'
Key: 'path'
        Value: './'
Key: 'restlistener.remote.source.host'
        Value: '127.0.0.1'
Key: 'restlistener.remote.user.dn'
        Value: 'none'
Key: 'uuid'
        Value: '332290fb-f7e7-4fbe-a91b-efbb2935232d'
--------------------------------------------------
```
Notice how NiFi captured some attributes like the source, the path used, .... A couple of things to note :

- As we didn't send a payload with our request, a FlowFile is generated without any content (only attributes).
- As we haven't configured the processor to pass through HTTP headers, these are lost in the dataflow (User-Agent, any custom HTTP headers)

We can also send data and custom headers with our request:

```
curl -v -X POST -H "customerHttpHeader: someHeaderValue" -d "someDataPosted" http://localhost:7001/contentListener
```

Make sure that you configure the processor to passthrough the HTTP headers using a regular expression if you want to see them in your dataflow.

Here's the same logging, this time with a body and passing through the HTTP headers

```
2016-12-29 00:21:27,741 INFO [Timer-Driven Process Thread-9] o.a.n.processors.standard.LogAttribute LogAttribute[id=107b117e-1391-1575-8dc2-f2701b0b0c56] logging for flow file StandardFlowFileRecord[uuid=ee941d2a-94d4-4a7f-9bd4-03a7edef21a2,claim=StandardContentClaim [r
esourceClaim=StandardResourceClaim[id=1482970834456-1, container=default, section=1], offset=14, length=14],offset=0,name=2260274668959019,size=14]
--------------------------------------------------
Standard FlowFile Attributes
Key: 'entryDate'
        Value: 'Thu Dec 29 00:21:27 UTC 2016'
Key: 'lineageStartDate'
        Value: 'Thu Dec 29 00:21:27 UTC 2016'
Key: 'fileSize'
        Value: '14'
FlowFile Attribute Map Content
Key: 'Accept'
        Value: '*/*'
Key: 'Content-Length'
        Value: '14'
Key: 'Content-Type'
        Value: 'application/x-www-form-urlencoded'
Key: 'Host'
        Value: 'localhost:7001'
Key: 'User-Agent'
        Value: 'curl/7.43.0'
Key: 'customerHttpHeader'
        Value: 'someHeaderValue'
Key: 'filename'
        Value: '2260274668959019'
Key: 'path'
        Value: './'
Key: 'restlistener.remote.source.host'
        Value: '127.0.0.1'
Key: 'restlistener.remote.user.dn'
        Value: 'none'
Key: 'uuid'
        Value: 'ee941d2a-94d4-4a7f-9bd4-03a7edef21a2'
--------------------------------------------------
someDataPosted
```


When we open up the data provenance for this processor, we can see the 2 requests that we performed (one without a payload, and one with a payload)
![]({{ site.url }}/assets/images/nifi/http/http-requests-data-provenance.png)

The payload that we sent is put into a flowFile and can be used upstream for further processing.
![]({{ site.url }}/assets/images/nifi/http/data-provenance-msg-payload.png)

### Thoughts and use-cases

- Very simple processor with only 1 outgoing relationship (success)
- Only supports the HTTP POST verb
- No error transitions possible
- Does not allow for incoming connections
- Good for setting up a very simple HTTP inbound endpoint in our dataflow for external clients to call. (to trigger some part of the flow)


<a name="InvokeHTTP"></a>

# InvokeHTTP

## Flow definition

The [InvokeHTTP processor](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi.processors.standard.InvokeHTTP/) is an HTTP client processor that can be configured in a dynamic way. It has a number of advantages as opposed to the GetTCP client processor we discussed.

- Both the HTTP endpoint URL and the HTTP method can be defined using the NiFi expression language.
- The InvokeHTTP processor has a lot more routing options.
- The processor supports incoming flowFiles from other processors

Our invokeHttp flow will look like this.

![]({{ site.url }}/assets/images/nifi/http/invokeHttp-flow.png)

## Processor configuration

The processor supports EL for its HTTP url / method. It also offers some control over how the HTTP response should be handled. You can opt to generate a new FlowFile and route it to one of the success / failures routes, or opt to add the response as an attribute to the original FlowFile (the original flowfile will always be routed via the "original" route)

![]({{ site.url }}/assets/images/nifi/http/invokeHttp-props.png)

## Data flow

In our flow, a GenerateFlowFile processor is scheduled to run every x seconds. It will updates some attributes that will be used to determine the URL that the invokeHttp processor will need to call.

```
2016-12-28 20:20:06,351 INFO [Timer-Driven Process Thread-5] o.a.n.processors.standard.LogAttribute LogAttribute[id=0159107b-6575-1391-9336-ae0e4f959a03] logging for flow file StandardFlowFileRecord[uuid=88dc9486-30b4-434e-b95a-3e6b66771791,claim=StandardContentClaim [resourceClaim=StandardResourceClaim[id=1482882111108-1, container=default, section=1], offset=932636, length=85],offset=0,name=2245794674306254,size=85]
--------------------------------------------------
Standard FlowFile Attributes
Key: 'entryDate'
        Value: 'Wed Dec 28 20:20:06 UTC 2016'
Key: 'lineageStartDate'
        Value: 'Wed Dec 28 20:20:06 UTC 2016'
Key: 'fileSize'
        Value: '85'
FlowFile Attribute Map Content
Key: 'Content-Type'
        Value: 'application/json;charset=UTF-8'
Key: 'Date'
        Value: 'Wed, 28 Dec 2016 20:20:06 GMT'
Key: 'OkHttp-Received-Millis'
        Value: '1482956406344'
Key: 'OkHttp-Sent-Millis'
        Value: '1482956406341'
Key: 'Transfer-Encoding'
        Value: 'chunked'
Key: 'X-Application-Context'
        Value: 'application:7002'
Key: 'filename'
        Value: '2245794674306254'
Key: 'invokehttp.request.url'
        Value: 'http://localhost:7002/users'
Key: 'invokehttp.status.code'
        Value: '200'
Key: 'invokehttp.status.message'
        Value: ''
Key: 'invokehttp.tx.id'
        Value: '758888f0-4241-4786-ac8f-31b4f3331bbc'
Key: 'mime.type'
        Value: 'application/json;charset=UTF-8'
Key: 'path'
        Value: './'
Key: 'uuid'
        Value: '88dc9486-30b4-434e-b95a-3e6b66771791'
--------------------------------------------------
[{"firstName":"first1","lastName":"last1"},{"firstName":"first2","lastName":"last2"}]
```

### Thoughts and use-cases

- Combines the client and the server part.
- Lots of error transitions possible, including retries.
- Allows for incoming flowFiles from other processors
- Fill NiFi EL support.
- Good for setting up an HTTP inbound endpoint in our dataflow for other processes to call

<a name="HandlerHttpRequest"></a>

# HandlerHttpRequest / HandlerHttpResponse

## Flow definition

So far we've always looked at one piece of the puzzle. Either a processor was acting as a server (ListenHttp), or a processor was acting as a client (GetHTTP / InvokeHTTP). 

The HandlerHttpRequest / HandlerHttpResponse processor pair allows you to do both. In other words, you can create a complete webservice where both request and response handling can be implemented using a Processor. You get to decide how / where the request is being handled, and what the response will look like.

A typicall flow will look like this, where you have a HandlerHttpRequest processor on one side handling the request, some processing in the middle, and a HandlerHttpResponse on the other side to output the response. 

![]({{ site.url }}/assets/images/nifi/http/httpHandlers-flow.png)

For those of you who are paying attention, you might have noticed some magic here. How does the HandlerHttpRequest processor know what response to return to the client. There is no connecton between the  HandlerHttpRequest and 	HandlerHttpResponse processor. 

.... or is there ?

### Controller Services

ControllerServices are a mechanism in NiFi for creating services that are shared among all Processors, ReportingTasks, and other ControllerServices. Unfortunately they are way beyond the scope of this article, but think of them as a construct that will allow the HandlerHttpRequest and HandlerHttpResponse to correlate their flowfiles, and ensure that a particular request gets the corresponding response.

![]({{ site.url }}/assets/images/nifi/http/controller-services.png)

The StandardHttpContextMap is one of those controller services that are used by the HandlerHttpRequest / HandlerHttpResponse processor pair, giving them the ability to store and retrieve HTTP requests and responses external to a Processor, so that multiple Processors can interact with the same HTTP request.


### Processor configuration

The HandlerHttpRequest Processor allows us to configure all aspects of the HTTP endpoint. As such it is similar to ListenHTTP, but with the following differences

- It supports incoming flowfiles. 
- It supports all HTTP verbs
- It supports the HTTP controller service (to correlate request / responses)

Think of it as ListenHTTP on steroids.

![]({{ site.url }}/assets/images/nifi/http/handleHttpRequest-props.png)

The HandlerHttpRespone Processor, when using the same controller service

![]({{ site.url }}/assets/images/nifi/http/handleHttpRequest-props.png)

### Processor scheduling

As far a scheduling is concerned we can keep the defaults. The processor will be constantly triggered to see if there were any incoming requests needing to be processed.

### Thoughts and use-cases

- Combines the client and the server part.
- Lots of error transitions possible, including retries.
- Allows for incoming flowFiles from other processors
- Fill NiFi EL support.
- Good for setting up an HTTP inbound endpoint in our dataflow for other processes to call


## Conclusions

In this post we've covered the following processors, each serving a specific use-case to implement HTTP based communication in NiFi.

- [ListenHTTP](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi.processors.standard.ListenHTTP/)
- [GetHTTP](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi.processors.standard.GetHTTP/)
- [InvokeHTTP]((https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi.processors.standard.InvokeHTTP/))
- [HandlerHttpRequest](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi.processors.standard.HandleHttpRequest/) / [HandlerHttpResponse](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi.processors.standard.HandleHttpResponse/)


The ListenHTTP and GetHTTP are very simple to use and setup, but are also rather static and thus limiting if you have a use-case that is more dynamic in nature.

With the InvokeHttp processors things are getting interesting as it supports the NiFi expression language and can handle incoming FlowFiles. This way you can think of use-cases where you first select a list of servers that you want to target (by reading a file, or by calling another REST endpoint), and then send out a flowfile for each indivual server to the InvokeHTTP processor.

If you want to implement our own webservice in Nifi, with its own response handling, then the HandlerHttpRequest / HandlerHttpResponse are all you need.


|                Processor               | Description                                 | Pro                                       | Con                                                 | Use-cases                                                            |
|:--------------------------------------:|---------------------------------------------|-------------------------------------------|-----------------------------------------------------|----------------------------------------------------------------------|
|               ListenHTTP               | Exposes an HTTP endpoint                    | Simple to setup                           | Too simple / No EL support / No incoming flowfiles  | Provide a simple HTTP webhook into your flow                         |
|                 GetHTTP                | Consume an HTTP (GET) endpoint                  | Simple to setup                           | Too simple /  No incoming flowfiles                 | Provide a simple HTTP client to poll an external http based resource |
|                 PostHTTP                | Consume an HTTP (POST) endpoint            | Simple to setup                           | Too simple /  No incoming flowfiles                 | Provide a simple HTTP client to poll an external http based resource |

|               InvokeHTTP               | Consume an HTTP endpoint                    | Supports EL / Incoming flowfiles          |                                                     | A much more dynamic GetHTTP flow                                     |
| HandlerHttpRequest HandlerHttpResponse | Expose an HTTP endpoint and send a response | Allows you to implement a real webservice | 2 Seperate processors to setup + controller service | Implement a webservice in your flow                                  |
