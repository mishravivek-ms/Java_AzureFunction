
Azure Event Grid trigger for Azure Functions
============================================


In this article
---------------

Use the function trigger to respond to an event sent by an [Event Grid source](../event-grid/overview). You must have an event subscription to the source to receive events. To learn how to create an event subscription, see [Create a subscription](event-grid-how-tos#create-a-subscription). For information on binding setup and configuration, see the [overview](functions-bindings-event-grid).

Note

Event Grid triggers aren't natively supported in an internal load balancer App Service Environment (ASE). The trigger uses an HTTP request that can't reach the function app without a gateway into the virtual network.

Important

This article uses tabs to support multiple versions of the Node.js programming model. The v4 model is generally available and is designed to have a more flexible and intuitive experience for JavaScript and TypeScript developers. For more details about how the v4 model works, refer to the [Azure Functions Node.js developer guide](functions-reference-node). To learn more about the differences between v3 and v4, refer to the [migration guide](functions-node-upgrade-v4).

Azure Functions supports two programming models for Python. The way that you define your bindings depends on your chosen programming model.

*   [v2](#tabpanel_1_python-v2)
*   [v1](#tabpanel_1_python-v1)

The Python v2 programming model lets you define bindings using decorators directly in your Python function code. For more information, see the [Python developer guide](functions-reference-python?pivots=python-mode-decorators#programming-model).

The Python v1 programming model requires you to define bindings in a separate _function.json_ file in the function folder. For more information, see the [Python developer guide](functions-reference-python?pivots=python-mode-configuration#programming-model).

This article supports both programming models.

Example
-------

For an HTTP trigger example, see [Receive events to an HTTP endpoint](../event-grid/receive-events).

The type of the input parameter used with an Event Grid trigger depends on these three factors:

*   Functions runtime version
*   Binding extension version
*   Modality of the C# function.

A C# function can be created by using one of the following C# modes:

*   [Isolated worker model](dotnet-isolated-process-guide): Compiled C# function that runs in a worker process that's isolated from the runtime. Isolated worker process is required to support C# functions running on LTS and non-LTS versions .NET and the .NET Framework. Extensions for isolated worker process functions use `Microsoft.Azure.Functions.Worker.Extensions.*` namespaces.
*   [In-process model](functions-dotnet-class-library): Compiled C# function that runs in the same process as the Functions runtime. In a variation of this model, Functions can be run using [C# scripting](functions-reference-csharp), which is supported primarily for C# portal editing. Extensions for in-process functions use `Microsoft.Azure.WebJobs.Extensions.*` namespaces.

Important

[Support will end for the in-process model on November 10, 2026](https://aka.ms/azure-functions-retirements/in-process-model). We highly recommend that you [migrate your apps to the isolated worker model](migrate-dotnet-to-isolated-model) for full support.

*   [Isolated worker model](#tabpanel_2_isolated-process)
*   [In-process model](#tabpanel_2_in-process)

When running your C# function in an isolated worker process, you need to define a custom type for event properties. The following example defines a `MyEventType` class.

        public class MyEventType
        {
            public string Id { get; set; }
    
            public string Topic { get; set; }
    
            public string Subject { get; set; }
    
            public string EventType { get; set; }
    
            public DateTime EventTime { get; set; }
    
            public IDictionary<string, object> Data { get; set; }
        }
    }


The following example shows how the custom type is used in both the trigger and an Event Grid output binding:

    public static class EventGridFunction
    {
        [Function(nameof(EventGridFunction))]
        [EventGridOutput(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]
        public static MyEventType Run([EventGridTrigger] MyEventType input, FunctionContext context)
        {
            var logger = context.GetLogger(nameof(EventGridFunction));
    
            logger.LogInformation(input.Data.ToString());
    
            var outputEvent = new MyEventType()
            {
                Id = "unique-id",
                Subject = "abc-subject",
                Data = new Dictionary<string, object>
                {
                    { "myKey", "myValue" }
                }
            };
    
            return outputEvent;
        }
    }


The following example shows a Functions version 4.x function that uses a `CloudEvent` binding parameter:

    using Azure.Messaging;
    using Microsoft.Azure.WebJobs;
    using Microsoft.Azure.WebJobs.Extensions.EventGrid;
    using Microsoft.Extensions.Logging;
    
    namespace Company.Function
    {
        public static class CloudEventTriggerFunction
        {
            [FunctionName("CloudEventTriggerFunction")]
            public static void Run(
                ILogger logger,
                [EventGridTrigger] CloudEvent e)
            {
                logger.LogInformation("Event received {type} {subject}", e.Type, e.Subject);
            }
        }
    }


The following example shows a Functions version 4.x function that uses an `EventGridEvent` binding parameter:

    using Microsoft.Azure.WebJobs;
    using Azure.Messaging.EventGrid;
    using Microsoft.Azure.WebJobs.Extensions.EventGrid;
    using Microsoft.Extensions.Logging;
    
    namespace Company.Function
    {
        public static class EventGridTriggerDemo
        {
            [FunctionName("EventGridTriggerDemo")]
            public static void Run([EventGridTrigger] EventGridEvent eventGridEvent, ILogger log)
            {
                log.LogInformation(eventGridEvent.Data.ToString());
            }
        }
    }


The following example shows a function that uses a `JObject` binding parameter:

    using Microsoft.Azure.WebJobs;
    using Microsoft.Azure.WebJobs.Extensions.EventGrid;
    using Newtonsoft.Json;
    using Newtonsoft.Json.Linq;
    using Microsoft.Extensions.Logging;
    
    namespace Company.Function
    {
        public static class EventGridTriggerCSharp
        {
            [FunctionName("EventGridTriggerCSharp")]
            public static void Run([EventGridTrigger] JObject eventGridEvent, ILogger log)
            {
                log.LogInformation(eventGridEvent.ToString(Formatting.Indented));
            }
        }
    }


This section contains the following examples:

*   [Event Grid trigger, String parameter](#event-grid-trigger-string-parameter)
*   [Event Grid trigger, POJO parameter](#event-grid-trigger-pojo-parameter)

The following examples show trigger binding in [Java](functions-reference-java) that use the binding and generate an event, first receiving the event as `String` and second as a POJO.

### Event Grid trigger, String parameter

      @FunctionName("eventGridMonitorString")
      public void logEvent(
        @EventGridTrigger(
          name = "event"
        )
        String content,
        final ExecutionContext context) {
          context.getLogger().info("Event content: " + content);
      }


### Event Grid trigger, POJO parameter

This example uses the following POJO, representing the top-level properties of an Event Grid event:

    import java.util.Date;
    import java.util.Map;
    
    public class EventSchema {
    
      public String topic;
      public String subject;
      public String eventType;
      public Date eventTime;
      public String id;
      public String dataVersion;
      public String metadataVersion;
      public Map<String, Object> data;
    
    }


Upon arrival, the event's JSON payload is de-serialized into the `EventSchema` POJO for use by the function. This process allows the function to access the event's properties in an object-oriented way.

      @FunctionName("eventGridMonitor")
      public void logEvent(
        @EventGridTrigger(
          name = "event"
        )
        EventSchema event,
        final ExecutionContext context) {
          context.getLogger().info("Event content: ");
          context.getLogger().info("Subject: " + event.subject);
          context.getLogger().info("Time: " + event.eventTime); // automatically converted to Date by the runtime
          context.getLogger().info("Id: " + event.id);
          context.getLogger().info("Data: " + event.data);
      }


In the [Java functions runtime library](/en-us/java/api/overview/azure/functions/runtime), use the `EventGridTrigger` annotation on parameters whose value would come from Event Grid. Parameters with these annotations cause the function to run when an event arrives. This annotation can be used with native Java types, POJOs, or nullable values using `Optional<T>`.

*   [Model v4](#tabpanel_3_nodejs-v4)
*   [Model v3](#tabpanel_3_nodejs-v3)

The following example shows an event grid trigger [TypeScript function](functions-reference-node?tabs=typescript).

    import { app, EventGridEvent, InvocationContext } from '@azure/functions';
    
    export async function eventGridTrigger1(event: EventGridEvent, context: InvocationContext): Promise<void> {
        context.log('Event grid function processed event:', event);
    }
    
    app.eventGrid('eventGridTrigger1', {
        handler: eventGridTrigger1,
    });


TypeScript samples are not documented for model v3.

*   [Model v4](#tabpanel_4_nodejs-v4)
*   [Model v3](#tabpanel_4_nodejs-v3)

The following example shows an event grid trigger [JavaScript function](functions-reference-node).

    const { app } = require('@azure/functions');
    
    app.eventGrid('eventGridTrigger1', {
        handler: (event, context) => {
            context.log('Event grid function processed event:', event);
        },
    });


The following example shows a trigger binding in a _function.json_ file and a [JavaScript function](functions-reference-node) that uses the binding.

Here's the binding data in the _function.json_ file:

    {
      "bindings": [
        {
          "type": "eventGridTrigger",
          "name": "eventGridEvent",
          "direction": "in"
        }
      ],
      "disabled": false
    }


Here's the JavaScript code:

    module.exports = async function (context, eventGridEvent) {
        context.log("JavaScript Event Grid function processed a request.");
        context.log("Subject: " + eventGridEvent.subject);
        context.log("Time: " + eventGridEvent.eventTime);
        context.log("Data: " + JSON.stringify(eventGridEvent.data));
    };


The following example shows how to configure an Event Grid trigger binding in the _function.json_ file.

    {
      "bindings": [
        {
          "type": "eventGridTrigger",
          "name": "eventGridEvent",
          "direction": "in"
        }
      ]
    }


The Event Grid event is made available to the function via a parameter named `eventGridEvent`, as shown in the following PowerShell example.

    param($eventGridEvent, $TriggerMetadata)
    
    # Make sure to pass hashtables to Out-String so they're logged correctly
    $eventGridEvent | Out-String | Write-Host


The following example shows an Event Grid trigger binding and a Python function that uses the binding. The example depends on whether you use the [v1 or v2 Python programming model](functions-reference-python).

*   [v2](#tabpanel_5_python-v2)
*   [v1](#tabpanel_5_python-v1)

    import logging
    import json
    import azure.functions as func

    app = func.FunctionApp()

    @app.function_name(name="eventGridTrigger")
    @app.event_grid_trigger(arg_name="event")
    def eventGridTest(event: func.EventGridEvent):
    result = json.dumps({
    'id': event.id,
    'data': event.get_json(),
    'topic': event.topic,
    'subject': event.subject,
    'event_type': event.event_type,
    })

        logging.info('Python EventGrid trigger processed an event: %s', result)


Here's the binding data in the _function.json_ file:

    {
      "bindings": [
        {
          "type": "eventGridTrigger",
          "name": "event",
          "direction": "in"
        }
      ],
      "disabled": false,
      "scriptFile": "__init__.py"
    }


Here's the Python code:

    import json
    import logging
    
    import azure.functions as func
    
    def main(event: func.EventGridEvent):
    
        result = json.dumps({
            'id': event.id,
            'data': event.get_json(),
            'topic': event.topic,
            'subject': event.subject,
            'event_type': event.event_type,
        })
    
        logging.info('Python EventGrid trigger processed an event: %s', result)


Attributes
----------

Both [in-process](functions-dotnet-class-library) and [isolated worker process](dotnet-isolated-process-guide) C# libraries use the [EventGridTrigger](https://github.com/Azure/azure-functions-eventgrid-extension/blob/master/src/EventGridExtension/TriggerBinding/EventGridTriggerAttribute.cs) attribute. C# script instead uses a function.json configuration file as described in the [C# scripting guide](functions-reference-csharp#event-grid-trigger).

*   [Isolated worker model](#tabpanel_6_isolated-process)
*   [In-process model](#tabpanel_6_in-process)

Here's an `EventGridTrigger` attribute in a method signature:

    [Function(nameof(EventGridFunction))]
    [EventGridOutput(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]
    public static MyEventType Run([EventGridTrigger] MyEventType input, FunctionContext context)
    {


Here's an `EventGridTrigger` attribute in a method signature:

    [FunctionName("EventGridTest")]
    public static void EventGridTest([EventGridTrigger] JObject eventGridEvent, ILogger log)
    {


Annotations
-----------

The [EventGridTrigger](/en-us/java/api/com.microsoft.azure.functions.annotation.eventgridtrigger) annotation allows you to declaratively configure an Event Grid binding by providing configuration values. See the [example](#example) and [configuration](#configuration) sections for more detail.

Configuration
-------------

*   [Model v4](#tabpanel_7_nodejs-v4)
*   [Model v3](#tabpanel_7_nodejs-v3)

The `options` object passed to the `app.eventGrid()` method currently doesn't support any properties for model v4.

The following table explains the binding configuration properties that you set in the _function.json_ file.

Property

Description

**type**

Required - must be set to `eventGridTrigger`.

**direction**

Required - must be set to `in`.

**name**

Required - the variable name used in function code for the parameter that receives the event data.

Configuration
-------------

The following table explains the binding configuration properties that you set in the _function.json_ file. There are no constructor parameters or properties to set in the `EventGridTrigger` attribute.

function.json property

Description

**type**

Required - must be set to `eventGridTrigger`.

**direction**

Required - must be set to `in`.

**name**

Required - the variable name used in function code for the parameter that receives the event data.

See the [Example section](#example) for complete examples.

Usage
-----

The Event Grid trigger uses a webhook HTTP request, which can be configured using the same [_host.json_ settings as the HTTP Trigger](functions-bindings-http-webhook#hostjson-settings).

The parameter type supported by the Event Grid trigger depends on the Functions runtime version, the extension package version, and the C# modality used.

*   [Extension v3.x](#tabpanel_8_extensionv3_in-process)
*   [Extension v2.x](#tabpanel_8_extensionv2_in-process)
*   [Functions 1.x](#tabpanel_8_functionsv1_in-process)
*   [Extension v3.x](#tabpanel_8_extensionv3_isolated-process)
*   [Extension v2.x](#tabpanel_8_extensionv2_isolated-process)
*   [Functions 1.x](#tabpanel_8_functionsv1_isolated-process)

In-process C# class library functions supports the following types:

*   [Azure.Messaging.CloudEvent](/en-us/dotnet/api/azure.messaging.cloudevent)
*   [Azure.Messaging.EventGrid](/en-us/dotnet/api/azure.messaging.eventgrid.eventgridevent)
*   [Newtonsoft.Json.Linq.JObject](https://www.newtonsoft.com/json/help/html/t_newtonsoft_json_linq_jobject.htm)
*   [System.String](/en-us/dotnet/api/system.string)

In-process C# class library functions supports the following types:

*   [Microsoft.Azure.EventGrid.Models.EventGridEvent](/en-us/dotnet/api/microsoft.azure.eventgrid.models.eventgridevent)
*   [Newtonsoft.Json.Linq.JObject](https://www.newtonsoft.com/json/help/html/t_newtonsoft_json_linq_jobject.htm)
*   [System.String](/en-us/dotnet/api/system.string)

In-process C# class library functions supports the following types:

*   [Newtonsoft.Json.Linq.JObject](https://www.newtonsoft.com/json/help/html/t_newtonsoft_json_linq_jobject.htm)
*   [System.String](/en-us/dotnet/api/system.string)

When you want the function to process a single event, the Event Grid trigger can bind to the following types:

Type

Description

JSON serializable types

Functions tries to deserialize the JSON data of the event into a plain-old CLR object (POCO) type.

`string`

The event as a string.

[BinaryData](/en-us/dotnet/api/system.binarydata)1

The bytes of the event message.

[CloudEvent](/en-us/dotnet/api/azure.messaging.cloudevent)1

The event object. Use when Event Grid is configured to deliver using the CloudEvents schema.

[EventGridEvent](/en-us/dotnet/api/azure.messaging.eventgrid.eventgridevent)1

The event object. Use when Event Grid is configured to deliver using the Event Grid schema.

When you want the function to process a batch of events, the Event Grid trigger can bind to the following types:

Type

Description

`CloudEvent[]`1,  
`EventGridEvent[]`1,  
`string[]`,  
`BinaryData[]`1

An array of events from the batch. Each entry represents one event.

1 To use these types, you need to reference [Microsoft.Azure.Functions.Worker.Extensions.EventGrid 3.3.0 or later](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker.Extensions.EventGrid/3.3.0) and the [common dependencies for SDK type bindings](dotnet-isolated-process-guide#sdk-types).

Requires you to define a custom type, or use a string. See the [Example section](#example) for examples of using a custom parameter type.

Functions version 1.x doesn't support the isolated worker process.

The Event Grid event instance is available via the parameter associated to the `EventGridTrigger` attribute, typed as an `EventSchema`.

The Event Grid instance is available via the parameter configured in the _function.json_ file's `name` property.

The Event Grid instance is available via the parameter configured in the _function.json_ file's `name` property, typed as `func.EventGridEvent`.

Event schema
------------

Data for an Event Grid event is received as a JSON object in the body of an HTTP request. The JSON looks similar to the following example:

    [{
      "topic": "/subscriptions/{subscriptionid}/resourceGroups/eg0122/providers/Microsoft.Storage/storageAccounts/egblobstore",
      "subject": "/blobServices/default/containers/{containername}/blobs/blobname.jpg",
      "eventType": "Microsoft.Storage.BlobCreated",
      "eventTime": "2018-01-23T17:02:19.6069787Z",
      "id": "{guid}",
      "data": {
        "api": "PutBlockList",
        "clientRequestId": "{guid}",
        "requestId": "{guid}",
        "eTag": "0x8D562831044DDD0",
        "contentType": "application/octet-stream",
        "contentLength": 2248,
        "blobType": "BlockBlob",
        "url": "https://egblobstore.blob.core.windows.net/{containername}/blobname.jpg",
        "sequencer": "000000000000272D000000000003D60F",
        "storageDiagnostics": {
          "batchId": "{guid}"
        }
      },
      "dataVersion": "",
      "metadataVersion": "1"
    }]


The example shown is an array of one element. Event Grid always sends an array and may send more than one event in the array. The runtime invokes your function once for each array element.

The top-level properties in the event JSON data are the same among all event types, while the contents of the `data` property are specific to each event type. The example shown is for a blob storage event.

For explanations of the common and event-specific properties, see [Event properties](../event-grid/event-schema#event-properties) in the Event Grid documentation.

Next steps
----------

*   If you have questions, submit an issue to the team [here](https://github.com/Azure/azure-functions-eventgrid-extension/issues)
*   [Dispatch an Event Grid event](functions-bindings-event-grid-output)

Feedback
--------

Coming soon: Throughout 2024 we will be phasing out GitHub Issues as the feedback mechanism for content and replacing it with a new feedback system. For more information see: [https://aka.ms/ContentUserFeedback](https://aka.ms/ContentUserFeedback).

Submit and view feedback for

[This product](https://feedback.azure.com/d365community/forum/9df02822-f224-ec11-b6e6-000d3a4f0da0) This page

[View all page feedback](https://github.com/MicrosoftDocs/azure-docs/issues)

* * *

Additional resources
--------------------

[](#)

[California Consumer Privacy Act (CCPA) Opt-Out Icon Your Privacy Choices](https://aka.ms/yourcaliforniaprivacychoices)

Theme

*   Light
*   Dark
*   High contrast

*   [Previous Versions](/en-us/previous-versions/)
*   [Blog](https://techcommunity.microsoft.com/t5/microsoft-learn-blog/bg-p/MicrosoftLearnBlog)
*   [Contribute](/en-us/contribute/)
*   [Privacy](https://go.microsoft.com/fwlink/?LinkId=521839)
*   [Terms of Use](/en-us/legal/termsofuse)
*   [Trademarks](https://www.microsoft.com/legal/intellectualproperty/Trademarks/)
*   © Microsoft 2024

Additional resources
--------------------

### In this article

[](#)

[California Consumer Privacy Act (CCPA) Opt-Out Icon Your Privacy Choices](https://aka.ms/yourcaliforniaprivacychoices)

Theme

*   Light
*   Dark
*   High contrast

*   [Previous Versions](/en-us/previous-versions/)
*   [Blog](https://techcommunity.microsoft.com/t5/microsoft-learn-blog/bg-p/MicrosoftLearnBlog)
*   [Contribute](/en-us/contribute/)
*   [Privacy](https://go.microsoft.com/fwlink/?LinkId=521839)
*   [Terms of Use](/en-us/legal/termsofuse)
*   [Trademarks](https://www.microsoft.com/legal/intellectualproperty/Trademarks/)
*   © Microsoft 2024