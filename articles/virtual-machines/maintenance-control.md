---
title: Overview of Maintenance control for Azure virtual machines using the Azure portal 
description: Learn how to control when maintenance is applied to your Azure VMs using Maintenance Control.
author: cynthn
ms.service: virtual-machines
ms.subservice: maintenance
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 10/06/2021
ms.author: cynthn
#pmcontact: shants
---

# Managing platform updates with Maintenance Control 

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Manage platform updates, that don't require a reboot, using maintenance control. Azure frequently updates its infrastructure to improve reliability, performance, security or launch new features. Most updates are transparent to users. Some sensitive workloads, like gaming, media streaming, and financial transactions, can't tolerate even few seconds of a VM freezing or disconnecting for maintenance. Maintenance control gives you the option to wait on platform updates and apply them within a 35-day rolling window. 

Maintenance control lets you decide when to apply updates to your isolated VMs and Azure dedicated hosts.

With maintenance control, you can:
- Batch updates into one update package.
- Wait up to 35 days to apply updates. 
- Automate platform updates by configuring a maintenance schedule.
- Maintenance configurations work across subscriptions and resource groups. 

## Limitations

- VMs must be on a [dedicated host](./dedicated-hosts.md), or be created using an [isolated VM size](isolation.md).
- Maintenance window duration can vary month over month and sometimes it can take upto 2 hours to apply the pending updates once it is initiated by the user.  
- After 35 days, an update will automatically be applied.
- Rack level maintenance cannot be controlled through maintenance control.
- User must have **Resource Contributor** access.

## Management options

You can create and manage maintenance configurations using any of the following options:

- [Azure CLI](maintenance-control-cli.md)
- [Azure PowerShell](maintenance-control-powershell.md)
- [Azure portal](maintenance-control-portal.md)

For an Azure Functions sample, see [Scheduling Maintenance Updates with Maintenance Control and Azure Functions](https://github.com/Azure/azure-docs-powershell-samples/tree/master/maintenance-auto-scheduler).

## Next steps

To learn more, see [Maintenance and updates](maintenance-and-updates.md).
