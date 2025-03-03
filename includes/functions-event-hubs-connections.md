---
author: mattchenderson
ms.service: azure-functions
ms.topic: include
ms.date: 10/08/2021
ms.author: mahender
---

## Connections

The `connection` property is a reference to environment configuration which specifies how the app should connect to Event Hubs. It may specify:

- The name of an application setting containing a [connection string](#connection-string)
- The name of a shared prefix for multiple application settings, together defining an [identity-based connection](#identity-based-connections).

If the configured value is both an exact match for a single setting and a prefix match for other settings, the exact match is used.

### Connection string

Obtain this connection string by clicking the **Connection Information** button for the [namespace](../articles/event-hubs/event-hubs-create.md#create-an-event-hubs-namespace), not the event hub itself. The connection string must be for an Event Hubs namespace, not the event hub itself.

When used for triggers, the connection string must have at least "read" permissions to activate the function. When used for output bindings, the connection string must have "send" permissions to send messages to the event stream.

This connection string should be stored in an application setting with a name matching the value specified by the `connection` property of the binding configuration.

### Identity-based connections

If you are using [version 5.x or higher of the extension](../articles/azure-functions/functions-bindings-event-hubs.md#event-hubs-extension-5x-and-higher), instead of using a connection string with a secret, you can have the app use an [Azure Active Directory identity](../articles/active-directory/fundamentals/active-directory-whatis.md). To do this, you would define settings under a common prefix which maps to the `connection` property in the trigger and binding configuration.

In this mode, the extension requires the following properties:

| Property                  | Environment variable template                       | Description                                | Example value                                        |
|---------------------------|-----------------------------------------------------|--------------------------------------------|------------------------------------------------|
| Fully Qualified Namespace | `<CONNECTION_NAME_PREFIX>__fullyQualifiedNamespace` | The fully qualified Event Hubs namespace. | <event_hubs_namespace>.servicebus.windows.net |

Additional properties may be set to customize the connection. See [Common properties for identity-based connections](../articles/azure-functions/functions-reference.md#common-properties-for-identity-based-connections).

[!INCLUDE [functions-identity-based-connections-configuration](./functions-identity-based-connections-configuration.md)]

[!INCLUDE [functions-event-hubs-permissions](./functions-event-hubs-permissions.md)]
