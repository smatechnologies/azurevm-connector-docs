---
sidebar_label: 'Solution Manager Sub-type'
title: Solution Manager sub-type operation
description: "How to define and run Azure VM jobs using the Solution Manager ACS sub-type, including task types, field definitions, return codes, and logging."
tags:
  - Procedural
  - Automation Engineer
  - Connectors
---

# Solution Manager sub-type operation

**Theme:** Configure | **Audience:** Automation Engineer

## What is it?

The Solution Manager sub-type provides an ACS-based Azure VM job type within OpCon's Solution Manager interface. It enables you to define, manage, and monitor Azure VM jobs entirely through Solution Manager without using Enterprise Manager or the command line.

- Use this sub-type when your OpCon environment runs version 25.0.3 or higher and you manage jobs through Solution Manager.
- Use this sub-type when you want connector configuration (`Connector.config`) centralized within OpCon rather than managed as a file on the agent server.

:::note
All interactions with the Solution Manager sub-type must be completed using Solution Manager.
:::

## Prerequisites

Before defining Azure VM jobs in Solution Manager, confirm the following:

- The Azure VM Connector is installed on the OpCon server or Relay server. See [Installation](./installation.md).
- The ACSAZUREVM script type, script runner, Connector.config script, and data script are created. See the Solution Manager sub-type installation section in [Installation](./installation.md).
- The AzureVM agent definition is created and saved in Solution Manager.

## Define an Azure VM job

To define an Azure VM job in Solution Manager, complete the following steps:

1. In Solution Manager, go to the job definition.
2. From the **Job Type** list, select **AzureVM**.
3. From the **Task Type** list, select the task to perform.
4. Enter the required field values for the selected task.
5. Select **Save**.

The following task types are available:

- **Create Virtual Machine**
- **Deallocate Virtual Machine**
- **Delete Virtual Machine**
- **List Virtual Machines**
- **PowerOff Virtual Machine**
- **Restart Virtual Machine**
- **Start Virtual Machine**

## Task type field reference

### Create virtual machine

Creates a new virtual machine in the Azure environment from a defined image.

| Field | Required | Description |
|---|---|---|
| **Resource Group** | Yes | The resource group where the virtual machine will be created |
| **Machine Name** | Yes | The name of the virtual machine to create |
| **Private IP Address Property Name** | No | The OpCon global property name to store the VM's private IP address after creation |
| **Public IP Address Property Name** | No | The OpCon global property name to store the VM's public IP address after creation |
| **Region** | Yes | The Azure region where the VM will be created. Select from the list |
| **Custom Image** | No | Select this option when using a custom image. Disk Size is not required for custom images |
| **Image** | Yes | The image to use when creating the VM. Select from the list |
| **Image Size** | Yes | The size of the VM. For custom images, the size must match the size used when the image was created |
| **Network** | Yes | The virtual network for the VM. Enter `new` to create a new network |
| **Address Space** | Yes | The subnet address space. Enter `10.0.0.0/24` for a new network, or enter the existing subnet name |
| **Admin User** | Yes | The administrator user name for the VM, following the naming requirements for the selected VM type |
| **User Password** | Yes | The password for the administrator user |

:::caution Not every combination of image, operating system and network is supported
Creating a virtual machine behaves differently depending on the three choices you make. The following combinations work as described:

| Image | Network | Windows | Linux |
|---|---|---|---|
| Standard | New (`new`) | Supported | **Not supported** — nothing is created and the job fails |
| Standard | Existing | Creates the machine, then **reports failure** | Supported |
| Custom | New (`new`) | Supported | Supported |
| Custom | Existing | Creates the machine, then **reports failure** | Supported |

To create a Linux machine on a new network, use a custom image. To create a Windows machine on an existing network, expect the job to fail and check Azure before rerunning it.
:::

:::caution A reported failure does not always mean nothing was created
When the virtual machine is a **Windows** machine being added to an **existing** network, the connector creates the machine but then reports the job as failed and returns `1`. The machine exists in Azure, and neither IP address property is set.

Check the resource group in Azure before rerunning the job. If the machine is there, delete it first, or the rerun creates a second one.
:::

### Deallocate virtual machine

Stops the virtual machine and deallocates all its resources. The VM is removed from the allocated resource pool.

| Field | Required | Description |
|---|---|---|
| **Resource Group** | Yes | The resource group where the virtual machine is allocated |
| **Machine Name** | Yes | The name of the virtual machine to deallocate |

### Delete virtual machine

Removes a virtual machine from the resource group.

| Field | Required | Description |
|---|---|---|
| **Resource Group** | Yes | The resource group where the virtual machine is allocated |
| **Machine Name** | Yes | The name of the virtual machine to delete |

### List virtual machines

Returns status information for all virtual machines in the resource group.

| Field | Required | Description |
|---|---|---|
| **Resource Group** | Yes | The resource group to list virtual machines for |

Sample output:

```
------------------------------------------------------------------------------------------------------------------ 
List of virtual machines in resource Group  MY_RESOURCEGROUP
------------------------------------------------------------------------------------------------------------------ 
Virtual Machine Name     Region              Current State                 IP Private     IP Public      OS Type   
------------------------------------------------------------------------------------------------------------------ 
LINUX001                 West Europe         PowerState/running            10.0.0.11      203.0.113.11   LINUX     
LINUX002                 West Europe         PowerState/running            10.0.0.12      203.0.113.12   LINUX     
LINUX003                 West Europe         PowerState/running            10.0.0.13      203.0.113.13   LINUX     
WINSRV01                 West Europe         PowerState/running            10.0.0.14      203.0.113.14   WINDOWS   
LINUX004                 West Europe         PowerState/stopped            10.0.0.15      N/A            LINUX     
------------------------------------------------------------------------------------------------------------------ 
```

### PowerOff virtual machine

Shuts down a virtual machine. The VM is stopped but resources remain allocated.

| Field | Required | Description |
|---|---|---|
| **Resource Group** | Yes | The resource group where the virtual machine is allocated |
| **Machine Name** | Yes | The name of the virtual machine to power off |

### Restart virtual machine

Restarts a virtual machine. Optionally stores the IP addresses after restart.

| Field | Required | Description |
|---|---|---|
| **Resource Group** | Yes | The resource group where the virtual machine is allocated |
| **Machine Name** | Yes | The name of the virtual machine to restart |
| **Private IP Address Property Name** | No | The OpCon global property name to store the VM's private IP address after restart |
| **Public IP Address Property Name** | No | The OpCon global property name to store the VM's public IP address after restart |

### Start virtual machine

Starts a stopped virtual machine. Optionally stores the IP addresses after startup.

| Field | Required | Description |
|---|---|---|
| **Resource Group** | Yes | The resource group where the virtual machine is allocated |
| **Machine Name** | Yes | The name of the virtual machine to start |
| **Private IP Address Property Name** | No | The OpCon global property name to store the VM's private IP address after startup |
| **Public IP Address Property Name** | No | The OpCon global property name to store the VM's public IP address after startup |

## Return codes

| Code | Meaning |
|---|---|
| `0` | Success — the job completed processing |
| `1` | Failure — an exception occurred during job processing |
| `99` | A credential value in `Connector.config` is missing |
| `401` | A credential value in `Connector.config` is present but empty |

To detect a failure, set the **Failure Criteria** for the job to **NE** (Not Equal) to `0`, which catches all three failure codes.

:::note `99` and `401` point at the configuration file, not at Azure
Both codes mean the connector could not read an encoded value from the `Connector.config` script. Despite its resemblance to an HTTP status, `401` is not a rejection by Azure or by the OpCon API.
:::

## Logging

All output produced by an AzureVM job is available in the job output log. Retrieve the log using the OpCon JORS (Job Output Retrieval System) capability in Solution Manager.

## FAQs

**What is the difference between Deallocate and PowerOff?**
PowerOff stops the VM while Azure continues to reserve the allocated resources, which means compute charges may still apply. Deallocate stops the VM and releases its allocated resources, which stops compute charges while keeping the VM definition intact.

**Can I store the VM's IP address in an OpCon global property after startup?**
Yes. In the **Private IP Address Property Name** and **Public IP Address Property Name** fields, enter the name of the OpCon global property (not the value) where the IP address should be stored. The connector writes the address to the property after the task completes.

**Where does the connector find the Azure credentials for Solution Manager jobs?**
The credentials are read from the `Connector.config` script stored in Solution Manager. The AzureVM agent definition links the job to the correct config script.

**Can I add custom images or regions to the lists?**
Yes. Edit the data script associated with the AzureVM agent. Add entries using the `IMAGE image-name`, `SIZE size-name`, or `REGION region-name` format. The new values appear in the lists when defining a job. Image names must follow the `publisher_offer_sku` form described in [Installation](./installation.md).

**How do I retrieve job output after a job runs?**
Use Solution Manager's JORS capability to view job output. Select the completed job and use the **View Output** option to access the connector log for that run.

## Glossary

**ACS framework** — The Agent Configuration Service framework in OpCon 25.0.3 and later that centralizes connector configuration within OpCon and provides a wrapper for ACS-compatible connectors such as the Azure VM Connector.

**AzureVM agent** — The OpCon agent definition of type **AzureVM** that links Solution Manager job definitions to the connector installation, config script, and data script.

**Deallocate** — An Azure operation that stops a VM and releases its compute resources, distinguishing it from a simple power-off where resources remain reserved.

**Resource group** — An Azure container that holds related resources such as virtual machines. All connector tasks target a single resource group per job.

**JORS** — Job Output Retrieval System. The OpCon feature that stores and provides access to job output logs. Used to review connector output and diagnose issues.

**Data script** — The Solution Manager script associated with an AzureVM agent that defines the available values (types, regions, images, sizes) used when defining Azure VM jobs.
