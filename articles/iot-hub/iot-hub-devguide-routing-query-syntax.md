---
title: Query on Azure IoT Hub message routing | Microsoft Docs
description: Learn about the IoT Hub message routing query language that you can use to apply rich queries to messages to receive the data that matters to you. 
author: ash2017
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/13/2018
ms.author: asrastog
ms.custom: ['Role: Cloud Development', 'Role: Data Analytics']
---

# IoT Hub message routing query syntax

Message routing enables users to route different data types namely, device telemetry messages, device lifecycle events, and device twin change events to various endpoints. You can also apply rich queries to this data before routing it to receive the data that matters to you. This article describes the IoT Hub message routing query language, and provides some common query patterns.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Message routing allows you to query on the message properties and message body as well as device twin tags and device twin properties. If the message body is not JSON, message routing can still route the message, but queries cannot be applied to the message body.  Queries are described as Boolean expressions where a Boolean true makes the query succeed which routes all the incoming data, and Boolean false fails the query and no data is routed. If the expression evaluates to null or undefined, it is treated as false and an error will be generated in IoT Hub [routes resource logs](monitor-iot-hub-reference.md#routes) logs in case of a failure. The query syntax must be correct for the route to be saved and evaluated.  

## Message routing query based on message properties 

The IoT Hub defines a [common format](iot-hub-devguide-messages-construct.md) for all device-to-cloud messaging for interoperability across protocols. IoT Hub message assumes the following JSON representation of the message. System properties are added for all users and identify content of the message. Users can selectively add application properties to the message. We recommend using unique property names as IoT Hub device-to-cloud messaging is not case-sensitive. For example, if you have multiple properties with the same name, IoT Hub will only send one of the properties.  

```json
{ 
  "message": { 
    "systemProperties": { 
      "contentType": "application/json", 
      "contentEncoding": "UTF-8", 
      "iothub-message-source": "deviceMessages", 
      "iothub-enqueuedtime": "2017-05-08T18:55:31.8514657Z" 
    }, 
    "appProperties": { 
      "processingPath": "{cold | warm | hot}", 
      "verbose": "{true, false}", 
      "severity": 1-5, 
      "testDevice": "{true | false}" 
    }, 
    "body": "{\"Weather\":{\"Temperature\":50}}" 
  } 
} 
```

### System properties

System properties help identify contents and source of the messages. 

| Property | Type | Description |
| -------- | ---- | ----------- |
| contentType | string | The user specifies the content type of the message. To allow query on the message body, this value should be set application/JSON. |
| contentEncoding | string | The user specifies the encoding type of the message. Allowed values are UTF-8, UTF-16, UTF-32 if the contentType is set to application/JSON. |
| iothub-connection-device-id | string | This value is set by IoT Hub and identifies the ID of the device. To query, use `$connectionDeviceId`. |
| iothub-connection-module-id | string | This value is set by IoT Hub and identifies the ID of the edge module. To query, use `$connectionModuleId`. |
| iothub-enqueuedtime | string | This value is set by IoT Hub and represents the actual time of enqueuing the message in UTC. To query, use `enqueuedTime`. |
| dt-dataschema | string |  This value is set by IoT hub on device-to-cloud messages. It contains the device model ID set in the device connection. To query, use `$dt-dataschema`. |
| dt-subject | string | The name of the component that is sending the device-to-cloud messages. To query, use `$dt-subject`. |

As described in the [IoT Hub Messages](iot-hub-devguide-messages-construct.md), there are additional system properties in a message. In addition to above properties in the previous table, you can also query **connectionDeviceId**, **connectionModuleId**.

### Application properties

Application properties are user-defined strings that can be added to the message. These fields are optional.  

### Query expressions

A query on message system properties needs to be prefixed with the `$` symbol. Queries on application properties are accessed with their name and should not be prefixed with the `$`symbol. If an application property name begins with `$`, then IoT Hub will search for it in the system properties, and it is not found, then it will look in the application properties. For example: 

To query on system property contentEncoding 

```sql
$contentEncoding = 'UTF-8'
```

To query on application property processingPath:

```sql
processingPath = 'hot'
```

To combine these queries, you can use Boolean expressions and functions:

```sql
$contentEncoding = 'UTF-8' AND processingPath = 'hot'
```

A full list of supported operators and functions is shown in [Expression and conditions](iot-hub-devguide-query-language.md#expressions-and-conditions).

## Message routing query based on message body

To enable querying on message body, the message should be in a JSON encoded in either UTF-8, UTF-16 or UTF-32. The `contentType` must be set to `application/JSON` and `contentEncoding` to one of the supported UTF encodings in the system property. If these properties are not specified, IoT Hub will not evaluate the query expression on the message body. 

The following example shows how to create a message with a properly formed and encoded JSON body: 

```javascript
var messageBody = JSON.stringify(Object.assign({}, {
    "Weather": {
        "Temperature": 50,
        "Time": "2017-03-09T00:00:00.000Z",
        "PrevTemperatures": [
            20,
            30,
            40
        ],
        "IsEnabled": true,
        "Location": {
            "Street": "One Microsoft Way",
            "City": "Redmond",
            "State": "WA"
        },
        "HistoricalData": [
            {
                "Month": "Feb",
                "Temperature": 40
            },
            {
                "Month": "Jan",
                "Temperature": 30
            }
        ]
    }
}));

// Encode message body using UTF-8  
var messageBytes = Buffer.from(messageBody, "utf8");

var message = new Message(messageBytes);

// Set message body type and content encoding 
message.contentEncoding = "utf-8";
message.contentType = "application/json";

// Add other custom application properties   
message.properties.add("Status", "Active");

deviceClient.sendEvent(message, (err, res) => {
    if (err) console.log('error: ' + err.toString());
    if (res) console.log('status: ' + res.constructor.name);
});
```

> [!NOTE] 
> This shows how to handle the encoding of the body in javascript. If you want to see a sample in C#, download the [Azure IoT C# Samples](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/main.zip). Unzip the master.zip file. The Visual Studio solution *SimulatedDevice*'s Program.cs file shows how to encode and submit messages to an IoT Hub. This is the same sample used for testing the message routing, as explained in the [Message Routing tutorial](tutorial-routing.md). At the bottom of Program.cs, it also has a method to read in one of the encoded files, decode it, and write it back out as ASCII so you can read it. 

### Query expressions

A query on a message body needs to be prefixed with `$body`. You can use a body reference, body array reference, or multiple body references in the query expression. Your query expression can also combine a body reference with message system properties, and message application properties reference. For example, the following are all valid query expressions:

```sql
$body.Weather.HistoricalData[0].Month = 'Feb' 
```

```sql
$body.Weather.Temperature = 50 AND $body.Weather.IsEnabled 
```

```sql
length($body.Weather.Location.State) = 2 
```

```sql
$body.Weather.Temperature = 50 AND processingPath = 'hot'
```

> [!NOTE]
> To filter a twin notification payload based on what changed, run your query on the message body:
>
> ```sql
> $body.properties.desired.telemetryConfig.sendFrequency
> ```

> [!NOTE]
> You can run queries and functions only on properties in the body reference. You can't run queries or functions on the entire body reference. For example, the following query is *not* supported and will return `undefined`:
>
> ```sql
> $body[0] = 'Feb'
> ```

## Message routing query based on device twin 

Message routing enables you to query on [Device Twin](iot-hub-devguide-device-twins.md) tags and properties, which are JSON objects. Querying on module twin is also supported. A sample of Device Twin tags and properties is shown below.

```JSON
{
    "tags": { 
        "deploymentLocation": { 
            "building": "43", 
            "floor": "1" 
        } 
    }, 
    "properties": { 
        "desired": { 
            "telemetryConfig": { 
                "sendFrequency": "5m" 
            }, 
            "$metadata" : {...}, 
            "$version": 1 
        }, 
        "reported": { 
            "telemetryConfig": { 
                "sendFrequency": "5m", 
                "status": "success" 
            },
            "batteryLevel": 55, 
            "$metadata" : {...}, 
            "$version": 4 
        } 
    } 
} 
```

> [!NOTE]
> Modules do not inherit twin tags from their corresponding devices. Twin queries for messages originating from device modules (for example from IoT Edge modules) query against the module twin and not the corresponding device twin.

### Query expressions

A query on a device or module twin needs to be prefixed with `$twin`. Your query expression can also combine a twin tag or property reference with a body reference, a message system properties reference, and/or a message application properties reference. We recommend using unique names in tags and properties as the query is not case-sensitive. This applies to both device twins and module twins. Also refrain from using `twin`, `$twin`, `body`, or `$body`, as a property names. For example, the following are all valid query expressions: 

```sql
$twin.properties.desired.telemetryConfig.sendFrequency = '5m'
```

```sql
$body.Weather.Temperature = 50 AND $twin.properties.desired.telemetryConfig.sendFrequency = '5m'
```

```sql
$twin.tags.deploymentLocation.floor = 1 
```

Routing query on body or device twin with a period in the payload or property name is not supported.

## Next steps

* Learn about [message routing](iot-hub-devguide-messages-d2c.md).
* Try the [message routing tutorial](tutorial-routing.md).
