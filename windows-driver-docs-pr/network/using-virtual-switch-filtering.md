---
title: Using Virtual Switch Filtering
description: Using Virtual Switch Filtering
ms.assetid: 09325037-F9A4-45C8-97E0-8FCA7D42A120
ms.date: 04/20/2017
ms.localizationpriority: medium
---

# Using Virtual Switch Filtering

## Overview of Virtual Switch Filtering

Virtual Switch Filtering is supported in Windows 8 and later versions of Windows.

This WFP feature allows filtering on fields of the MAC header, IP header, and upper protocol ports as well as virtual switch specific fields such as virtual port (VPort) and virtual machine identifier (VM ID). These layers are invoked on a per-packet basis for all packets traversing the virtual switch. These layers are accessed from a virtual switch extension filter—a type of NDIS lightweight filter (LWF) driver.

A callout driver calls the [**FwpsvSwitchEventsSubscribe0**](https://msdn.microsoft.com/library/windows/hardware/hh439687) function to register callback entry points for virtual switch layer events.

The entry points for the callback notification functions are specified in an [**FWPS\_VSWITCH\_EVENT\_DISPATCH\_TABLE0**](https://msdn.microsoft.com/library/windows/hardware/hh451263) structure. The callback functions that are available include:

* [*FWPS\_VSWITCH\_FILTER\_ENGINE\_REORDER\_CALLBACK0*](https://msdn.microsoft.com/library/windows/hardware/hh451267)
* [*FWPS\_VSWITCH\_INTERFACE\_EVENT\_CALLBACK0*](https://msdn.microsoft.com/library/windows/hardware/hh451269)
* [*FWPS\_VSWITCH\_LIFETIME\_EVENT\_CALLBACK0*](https://msdn.microsoft.com/library/windows/hardware/hh451271)
* [*FWPS\_VSWITCH\_POLICY\_EVENT\_CALLBACK0*](https://msdn.microsoft.com/library/windows/hardware/hh451272)
* [*FWPS\_VSWITCH\_PORT\_EVENT\_CALLBACK0*](https://msdn.microsoft.com/library/windows/hardware/hh451276)
* [*FWPS\_VSWITCH\_RUNTIME\_STATE\_RESTORE\_CALLBACK0*](https://msdn.microsoft.com/library/windows/hardware/hh451281)
* [*FWPS\_VSWITCH\_RUNTIME\_STATE\_SAVE\_CALLBACK0*](https://msdn.microsoft.com/library/windows/hardware/hh451286)

The [**FWPS\_VSWITCH\_EVENT\_TYPE**](https://msdn.microsoft.com/library/windows/hardware/hh451265) enumeration defines the values for the *eventType* parameter of the virtual switch notification functions.

The callout driver must eventually call [**FwpsvSwitchEventsUnsubscribe0**](https://msdn.microsoft.com/library/windows/hardware/hh439691) to free the system resources.

If a callout driver returns STATUS\_PENDING from a WFP notification function, WFP will return STATUS\_PENDING to the OID request handler. The callout driver must call the [**FwpsvSwitchNotifyComplete0**](https://msdn.microsoft.com/library/windows/hardware/hh439695) function to complete the pending operation. After the **FwpsvSwitchNotifyComplete0** call, WFP calls the [**NdisFOidRequestComplete**](https://msdn.microsoft.com/library/windows/hardware/ff561833) function to complete the OID for the virtual switch.

Callbacks should not add or delete WFP filters synchronously in the context of the notification functions. In addition, if the notification function allows the callback to return STATUS\_PENDING, and the callout returns STATUS\_PENDING, the callout should not add or delete WFP filters before completing the notification.

## WFP Virtual Switch Filter Layer and Fields

[Run-time Filtering Layer Identifiers](https://msdn.microsoft.com/library/windows/hardware/ff570731) for virtual switch filtering include:

* FWPS\_LAYER\_INGRESS\_VSWITCH\_ETHERNET
* FWPS\_LAYER\_EGRESS\_VSWITCH\_ETHERNET
* FWPS\_LAYER\_INGRESS\_VSWITCH\_TRANSPORT\_V4
* FWPS\_LAYER\_INGRESS\_VSWITCH\_TRANSPORT\_V6
* FWPS\_LAYER\_EGRESS\_VSWITCH\_TRANSPORT\_V4
* FWPS\_LAYER\_EGRESS\_VSWITCH\_TRANSPORT\_V6

[Data Field Identifiers](https://msdn.microsoft.com/library/windows/hardware/ff546312) for virtual switch filtering include:

* [**FWPS\_FIELDS\_EGRESS\_VSWITCH\_ETHERNET**](https://msdn.microsoft.com/library/windows/hardware/hh439709)
* [**FWPS\_FIELDS\_EGRESS\_VSWITCH\_TRANSPORT\_V4**](https://msdn.microsoft.com/library/windows/hardware/hh439715)
* [**FWPS\_FIELDS\_EGRESS\_VSWITCH\_TRANSPORT\_V6**](https://msdn.microsoft.com/library/windows/hardware/hh439721)
* [**FWPS\_FIELDS\_INGRESS\_VSWITCH\_ETHERNET**](https://msdn.microsoft.com/library/windows/hardware/hh439733)
* [**FWPS\_FIELDS\_INGRESS\_VSWITCH\_TRANSPORT\_V4**](https://msdn.microsoft.com/library/windows/hardware/hh439738)
* [**FWPS\_FIELDS\_INGRESS\_VSWITCH\_TRANSPORT\_V6**](https://msdn.microsoft.com/library/windows/hardware/hh439745)

## Guidance For WFP Virtual Switch Callout Writers

### Port 0 Traffic

For WFP virtual switch callouts, traffic from port 0 (the default port ID) is trusted and should not be filtered. This is because, generally, traffic over port 0 originates from other extensions in the driver stack and is thus treated by the data path as privileged and trusted. Virtual switch extensions will sparingly use port 0 for situations such as originating a control packet, which should not be filtered and rejected by any underlying extensions. For more information about Hyper-V extensible switch source port mofification, see [Modifying a Packet's Extensible Switch Source Port Data](modifying-a-packet-s-extensible-switch-source-port-data.md).

### Callout Matching Rules

When defining a matching rule for filtering, virtual switch callouts should not use the MAC address as a basis for comparison. MAC addresses can change at runtime, and some ports may generate traffic from multiple MAC addresses. Instead, callouts should use a more durable matching rule such as NIC ID, which will not change.

### I/O Virtualization (IOV) and WFP Coexistence

WFP cannot be enabled on an IOV switch and is blocked by the OS if an attempt is made to enable it.

### Enabling or Disabling WFP

Installers for WFP virtual switch callouts should not modify the WFP extension enabled state; that is, they should not enable or disable WFP itself.



