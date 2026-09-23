---
sidebar_label: 'Installation'
title: Azure VM Connector installation
description: "Step-by-step instructions for installing and configuring the Azure VM Connector, including Enterprise Manager and Solution Manager sub-type setup."
tags:
  - Procedural
  - System Administrator
  - Connectors
---

# Azure VM Connector installation

**Theme:** Configure | **Audience:** System Administrator

## What is it?

The Azure VM Connector installation prepares the connector and OpCon environment to run Azure VM automation jobs. Installation involves placing the connector files on a Windows agent, encoding Azure account credentials, and configuring either the Enterprise Manager or Solution Manager sub-type.

- Use this procedure when setting up the connector for the first time in your OpCon environment.
- Use this procedure when adding a new Windows agent or OpCon instance that needs Azure VM automation capability.

## Supported software versions

The following software versions are required to use the Azure VM Connector.

| Component | Minimum Version |
|---|---|
| OpCon (Solution Manager sub-type) | 25.0.3 or higher |
| OpCon (Enterprise Manager sub-type) | 19.0 or higher |
| OpCon REST API | Required |
| Java | Embedded OpenJDK 11 (bundled with connector) |

## Installation

To install the Azure VM Connector, complete the following steps:

1. Install the Azure VM Connector files.
2. Configure the `Connector.config` file.
3. Complete the sub-type-specific setup:
   - For Enterprise Manager: install the job sub-type plug-in and create global properties.
   - For Solution Manager: create AzureVM scripts and an AzureVM agent definition.

### Install the connector files

The Azure VM Connector can be installed on any Windows Server that has an OpCon Windows agent installed.

:::note
When using the Solution Manager sub-type, the connector must be installed on either the same Windows server as the OpCon installation or the same Windows server as the Relay installation.
:::

To install the connector files, complete the following steps:

1. Download the `AzureVM_Windows.zip` file from the desired [release available here](https://github.com/SMATechnologies/azure-vm-java/releases).
2. Extract the zip file to the directory where you want to install the connector.
3. Confirm that the following items are present in the root installation directory after extraction:
   - `AzureVM.exe` — the connector executable
   - `Encrypt.exe` — the encoding utility
   - `Connector.config` — the connector configuration file
   - `java/` — directory containing the bundled OpenJDK 11
   - `emplugins/` — directory containing the Enterprise Manager plug-in

### Encode credentials using the Encrypt utility

`Encrypt.exe` encodes a value so that it is not readable at a glance in `Connector.config`. Use it on every credential value before placing it in the file.

:::caution Encoding is not encryption
The value is base64-encoded and then hex-encoded. There is no key, so anyone who can read `Connector.config` can recover the original credential. Treat the file as a secret in its own right: restrict its file permissions to the account the OpCon agent runs as, keep it out of source control, and protect it wherever it is backed up.
:::

To encode a value, complete the following steps:

1. Open a command prompt on the Windows server where the connector is installed.
2. Run the following command, replacing `value` with the text to encode:

```
Encrypt.exe -v "value"
```

3. Copy the encoded output. You will use it in the `Connector.config` file.

### Configure the Connector.config file

The `Connector.config` file stores Azure account credentials and the OpCon API connection details. All credential values must be encoded with `Encrypt.exe` before being added to the file.

The Azure account information required consists of the subscription ID, the tenant ID, the client ID, and the secret key. Retrieve these values from your Azure environment.

To configure the OpCon API connection automatically, run the `--setup` switch:

```
AzureVM.exe --setup -username name -password pwd -address server:port -tls
```

The `--setup` switch requires the following arguments:

| Argument | Description |
|---|---|
| `-username` | An OpCon user name that is a member of the `ocadm` role |
| `-password` | The password of the OpCon user |
| `-address` | The address and port number of the OpCon REST API server |
| `-tls` | Include this flag if the OpCon REST API requires a TLS connection |

:::note The OpCon REST API certificate is not validated
The connector does not check the OpCon REST API server certificate, so a TLS connection here encrypts the traffic but does not confirm which server answered. Treat the connection between the connector and the OpCon REST API as one that needs a trusted network.
:::

The following shows an example `Connector.config` file after setup. The keyword spellings below are the ones the connector reads — note that the OpCon API keywords have no underscore between `OPCON` and `API`:

```
[CONNECTOR]
NAME=Azure VM Connector
DEBUG=OFF

[MSAZURE]
TENANT = (encoded value)
SUBSCRIPTION = (encoded value)
CLIENT = (encoded value)
KEY = (encoded value)

[OPCON API]
OPCONAPI_ADDRESS = address:port
OPCONAPI_USING_TLS = True
OPCONAPI_TOKEN = (encoded value)
```

The configuration keywords are defined as follows:

| Keyword | Section | Type | Description |
|---|---|---|---|
| `NAME` | `[CONNECTOR]` | Text | A display name for this connector instance. Used in logging |
| `DEBUG` | `[CONNECTOR]` | Text | `ON` adds OpCon REST API request and response logging. Any other value, such as `OFF`, leaves it off. **The keyword must be present** |
| `TENANT` | `[MSAZURE]` | Text | The encoded tenant ID |
| `SUBSCRIPTION` | `[MSAZURE]` | Text | The encoded subscription ID |
| `CLIENT` | `[MSAZURE]` | Text | The encoded client ID |
| `KEY` | `[MSAZURE]` | Text | The encoded secret key |
| `OPCONAPI_ADDRESS` | `[OPCON API]` | Text | The OpCon REST API address. Set by the `--setup` switch |
| `OPCONAPI_USING_TLS` | `[OPCON API]` | Text | Whether the OpCon REST API uses TLS. Set by the `--setup` switch |
| `OPCONAPI_TOKEN` | `[OPCON API]` | Text | The encoded application token. Set by the `--setup` switch |

:::caution Do not remove the `DEBUG` keyword
The connector reads `DEBUG` without a default and fails to start if the keyword is missing. To turn the extra logging off, set it to `OFF` rather than deleting the line.
:::

:::note `DEBUG` does not control the connector's own log file
The connector writes its own log regardless of this setting. Refer to [Connector log file](#connector-log-file).
:::

### Connector log file

The connector writes a log file of its own, separately from the job output that OpCon captures. It is enabled by default and needs no configuration.

| Property | Value |
|---|---|
| Location | `<installation_dir>\log\storage.log` |
| Rolls at | 100 MB |
| History kept | Previous files are removed when the connector starts |

:::caution The log file contains your credentials in readable form
Each run records the Azure tenant ID, subscription ID, client ID and secret key, and the OpCon API token, **after decoding them**. The values that are encoded in `Connector.config` appear in this file as plain text.

Restrict access to the `log` directory to the account the OpCon agent runs as, and remove or redact the file before sharing it with anyone, including when attaching diagnostics to a support case.
:::

### Enterprise Manager sub-type installation

To install the Enterprise Manager sub-type, complete the following steps:

1. Copy the Enterprise Manager plug-in from the `<installation_dir>\emplugins` directory to the `dropins` directory of your Enterprise Manager installation. Create the `dropins` directory if it does not exist.
2. Restart Enterprise Manager. The **Azure VM** Windows job sub-type will be visible.

:::note
If the sub-type does not appear, restart Enterprise Manager using **Run as Administrator**. After this step, Enterprise Manager can be used normally.
:::

#### Create the AzureVmPath global property

Create a global property named `AzureVmPath` that contains the full path of the connector installation directory.

#### Create special global properties

The Azure VM Connector uses three global properties to populate the lists offered when creating virtual machine job definitions.

| Property | Description |
|---|---|
| `AZURE_WINDOWS_SERVERS` | Comma-separated list of Windows VM image names. Double quotes surrounding each value must be retained. |
| `AZURE_LINUX_SERVERS` | Comma-separated list of Linux VM image names. Double quotes surrounding each value must be retained. |
| `AZURE_SERVER_SIZES` | Comma-separated list of VM size names. Double quotes surrounding each value must be retained. |

Each image name must be written as `publisher_offer_sku`, using the Azure publisher, offer and SKU names joined by underscores. The connector splits the name on the underscores to look the image up, so a name in any other form is not found.

:::tip Example
`Canonical_UbuntuServer_18.04-LTS` selects the publisher `Canonical`, the offer `UbuntuServer` and the SKU `18.04-LTS`.
:::

:::note An image version cannot be pinned
The connector uses the most recently published image under the SKU you name. If you need a specific version, use a custom image instead.
:::

Sample values for each property are provided in the lists below.

```
List of Windows Virtual Machines 

"MicrosoftSQLServer_SQL2012SP3-WS2012R2_Enterprise","MicrosoftSQLServer_SQL2012SP4-WS2012R2_Enterprise","MicrosoftSQLServer_SQL2012SP4-WS2012R2_Express","MicrosoftSQLServer_SQL2012SP4-WS2012R2_Standard","MicrosoftSQLServer_SQL2014SP2-WS2012R2_Enterprise","MicrosoftSQLServer_SQL2014SP2-WS2012R2_Express",
"MicrosoftSQLServer_SQL2014SP2-WS2012R2_Standard","MicrosoftSQLServer_sql2014sp3-ws2012r2_enterprise","MicrosoftSQLServer_sql2014sp3-ws2012r2_express","MicrosoftSQLServer_sql2014sp3-ws2012r2_standard","MicrosoftSQLServer_SQL2016-WS2012R2_Enterprise","MicrosoftSQLServer_SQL2016SP1-WS2016_Enterprise","MicrosoftSQLServer_SQL2016SP1-WS2016_Express","MicrosoftSQLServer_SQL2016SP1-WS2016_Standard","MicrosoftSQLServer_SQL2016SP2-WS2016_Enterprise","MicrosoftSQLServer_SQL2016SP2-WS2016_Express","MicrosoftSQLServer_SQL2016SP2-WS2016_Standard","MicrosoftSQLServer_SQL2017-WS2016_Enterprise","MicrosoftWindowsServer_WindowsServer_2012-Datacenter","MicrosoftWindowsServer_WindowsServer_2012-R2-Datacenter","MicrosoftWindowsServer_WindowsServer_2016-Datacenter","MicrosoftWindowsServer_WindowsServer_2019-Datacenter"

List of Linux Virtual Machines 

"Canonical_UbuntuServer_12.04.3-LTS","Canonical_UbuntuServer_12.04.4-LTS","Canonical_UbuntuServer_12.04.5-LTS","Canonical_UbuntuServer_14.04.0-LTS","Canonical_UbuntuServer_14.04.1-LTS","Canonical_UbuntuServer_14.04.2-LTS","Canonical_UbuntuServer_14.04.3-LTS","Canonical_UbuntuServer_14.04.4-LTS","Canonical_UbuntuServer_14.04.5-LTS","Canonical_UbuntuServer_16.04-LTS","Canonical_UbuntuServer_16.04.0","Canonical_UbuntuServer_18.04","Canonical_UbuntuServer_18.10","RedHat_RHEL_6.7","RedHat_RHEL_6.8","RedHat_RHEL_6.9","RedHat_RHEL_6.10","RedHat_RHEL_7.2","RedHat_RHEL_7.3","RedHat_RHEL_7.4",
"RedHat_RHEL_7.5","SUSE_SLES_11-SP4","SUSE_SLES_12-SP4","SUSE_SLES_15","MicrosoftSQLServer_SQL2017-RHEL7_Enterprise","MicrosoftSQLServer_SQL2017-RHEL7_Express","MicrosoftSQLServer_SQL2017-RHEL7_Standard","MicrosoftSQLServer_SQL2017-SLES12SP2_Enterprise","MicrosoftSQLServer_SQL2017-SLES12SP2_Express","MicrosoftSQLServer_SQL2017-Ubuntu1604_Enterprise","MicrosoftSQLServer_SQL2017-Ubuntu1604_Express"

List of Virtual Machine Sizes

"Standard_A0","Standard_A1","Standard_A2","Standard_A3","Standard_A5","Standard_A4","Standard_A6","Standard_A7","Basic_A0","Basic_A1","Basic_A2","Basic_A3","Basic_A4","Standard_D1_v2","Standard_D2_v2","Standard_D3_v2","Standard_D4_v2","Standard_D5_v2","Standard_D11_v2","Standard_D12_v2","Standard_D13_v2","Standard_D14_v2","Standard_D15_v2","Standard_D2_v2_Promo","Standard_D3_v2_Promo","Standard_D4_v2_Promo","Standard_D5_v2_Promo","Standard_D11_v2_Promo","Standard_D12_v2_Promo","Standard_D13_v2_Promo","Standard_D14_v2_Promo","Standard_F1","Standard_F2","Standard_F4","Standard_F8","Standard_F16","Standard_A1_v2","Standard_A2m_v2","Standard_A2_v2","Standard_A4m_v2","Standard_A4_v2","Standard_A8m_v2","Standard_A8_v2","Standard_DS1","Standard_DS2","Standard_DS3","Standard_DS4","Standard_DS11","Standard_DS12","Standard_DS13","Standard_DS14","Standard_D2_v3","Standard_D4_v3","Standard_D8_v3","Standard_D16_v3","Standard_D32_v3","Standard_D1","Standard_D2","Standard_D3","Standard_D4","Standard_D11","Standard_D12","Standard_D13","Standard_D14","Standard_B1ms","Standard_B1s","Standard_B2ms","Standard_B2s","Standard_B4ms","Standard_B8ms","Standard_DS1_v2","Standard_DS2_v2","Standard_DS3_v2","Standard_DS4_v2","Standard_DS5_v2","Standard_DS11-1_v2","Standard_DS11_v2","Standard_DS12-1_v2","Standard_DS12-2_v2","Standard_DS12_v2","Standard_DS13-2_v2","Standard_DS13-4_v2","Standard_DS13_v2","Standard_DS14-4_v2","Standard_DS14-x8_v2","Standard_DS14_v2","Standard_DS2_v2_Promo","Standard_DS3_v2_Promo","Standard_DS4_v2_Promo","Standard_DS5_v2_Promo","Standard_DS11_v2_Promo","Standard_DS12_v2_Promo","Standard_DS13_v2_Promo","Standard_DS14_v2_Promo","Standard_F1s","Standard_F2s","Standard_F4s","Standard_F8s","Standard_F16s","Standard_D64_v3","Standard_D2s_v3","Standard_D4s_v3","Standard_D8s_v3","Standard_D16s_v3","Standard_D32s_v3","Standard_D64s_v3","Standard_E2_v3","Standard_E4_v3","Standard_E8_v3","Standard_E16_v3","Standard_E20_v3","Standard_E32_v3","Standard_E64i_v3","Standard_E64_v3","Standard_E2s_v3","Standard_E4-2s_v3","Standard_E4s_v3","Standard_E8-2s_v3","Standard_E8-4s_v3","Standard_E8s_v3","Standard_E16-4s_v3","Standard_E16-8s_v3","Standard_E16s_v3","Standard_E20s_v3","Standard_E32-8s_v3","Standard_E32-16s_v3","Standard_E32s_v3","Standard_E64-16s_v3","Standard_E64-32s_v3","Standard_E64is_v3","Standard_E64s_v3","Standard_DS15_v2","Standard_F2s_v2","Standard_F4s_v2","Standard_F8s_v2","Standard_F16s_v2","Standard_F32s_v2","Standard_F64s_v2",
"Standard_F72s_v2","Standard_G1","Standard_G2","Standard_G3","Standard_G4","Standard_G5","Standard_GS1","Standard_GS2","Standard_GS3","Standard_GS4","Standard_GS4-4","Standard_GS4-8","Standard_GS5","Standard_GS5-8","Standard_GS5-16","Standard_L4s","Standard_L8s","Standard_L16s","Standard_L32s"
```

### Solution Manager sub-type installation

:::note
All interactions with the Solution Manager sub-type must be completed using Solution Manager.
:::

To install the Solution Manager sub-type, complete the following steps:

1. Download the `ACSAZUREVM` zip file from the FTP site under **OpCon Releases\\Integrations\\AZUREVM**.
2. Extract the `ACSAZUREVM` directory and copy it into the `\SAM\plugins` directory for OpCon and Relay installations.
3. Restart the required services:
   - For OpCon installations: stop and restart the **SMA OpCon RestAPI** and **SMA OpCon Service Manager** services.
   - For Relay installations: stop and restart the **Relay Service**.

#### Create the scripts

When using the Solution Manager sub-type, two scripts must be created: one containing the `Connector.config` information and one containing the list information.

To create the required script type and runner, complete the following steps:

1. In Solution Manager, select **Library**.
2. Select **Scripts**.
3. Select **Script Types** from the upper right.
   1. Select **+Add**.
   2. In the **Name** field, enter `ACSAZUREVM`.
   3. In the **File Extension** field, enter `txt`.
   4. In the **Description** field, enter `Used for ACSAZUREVM Integration`.
   5. Select **Save**.
4. Select **Script Runners** from the upper right.
   1. Select **+Add**.
   2. In the **Name** field, enter `ACSAZUREVM`.
   3. In the **OS** field, select **AzureVM**.
   4. In the **Type** field, select **ACSAZUREVM**.
   5. In the **Command** field, enter `cmd.exe /c`.
   6. Select **Save**.

To create the Connector.config script, complete the following steps:

1. In Solution Manager, select **Library** > **Scripts**.
2. Select **Scripts** from the upper right, then select **+Add**.
3. In the **Name** field, enter a name. Use the proposed agent name with `_config` appended.
4. In the **Type** field, select **ACSAZUREVM**.
5. Assign the required roles.
6. In the **Script** field, paste the contents of your `Connector.config` file.
7. Select **Save**.

To create the list information script, complete the following steps:

1. In Solution Manager, select **Library** > **Scripts**.
2. Select **Scripts** from the upper right, then select **+Add**.
3. In the **Name** field, enter a name. Use the proposed agent name with `_data` appended.
4. In the **Type** field, select **ACSAZUREVM**.
5. Assign the required roles.
6. In the **Script** field, paste the data script content provided below.
7. Select **Save**.

The data script content defines the available types, regions, images, and sizes. Additional values can be added using the same `KEY value` format.

```
TYPE Windows
TYPE Linux
REGION Australia Central
REGION Australia Central 2
REGION Australia East
REGION Australia Southeast
REGION Brasil South
REGION Canada Central
REGION Canada East
REGION Central India
REGION Central US
REGION China East
REGION China East 2
REGION China North
REGION China North 2
REGION East Asia
REGION East US
REGION East US 2
REGION France Central
REGION France South
REGION Germany Central
REGION Germany North
REGION Japan East
REGION Japan West
REGION Korea Central
REGION Korea South
REGION North Central US
REGION North Europe
REGION South Central US
REGION South India
REGION SouthEast Asia
REGION UK South
REGION UK West
REGION West Central US
REGION West Europe
REGION West India
REGION West US 2
REGION West US
IMAGE MicrosoftSQLServer_SQL2012SP3-WS2012R2_Enterprise
IMAGE MicrosoftSQLServer_SQL2012SP4-WS2012R2_Enterprise
IMAGE MicrosoftSQLServer_SQL2012SP4-WS2012R2_Express
IMAGE MicrosoftSQLServer_SQL2012SP4-WS2012R2_Standard
IMAGE MicrosoftSQLServer_SQL2014SP2-WS2012R2_Enterprise
IMAGE MicrosoftSQLServer_SQL2014SP2-WS2012R2_Express
IMAGE MicrosoftSQLServer_SQL2014SP2-WS2012R2_Standard
IMAGE MicrosoftSQLServer_sql2014sp3-ws2012r2_enterprise
IMAGE MicrosoftSQLServer_sql2014sp3-ws2012r2_express
IMAGE MicrosoftSQLServer_sql2014sp3-ws2012r2_standard
IMAGE MicrosoftSQLServer_SQL2016-WS2012R2_Enterprise
IMAGE MicrosoftSQLServer_SQL2016SP1-WS2016_Enterprise
IMAGE MicrosoftSQLServer_SQL2016SP1-WS2016_Express
IMAGE MicrosoftSQLServer_SQL2016SP1-WS2016_Standard
IMAGE MicrosoftSQLServer_SQL2016SP2-WS2016_Enterprise
IMAGE MicrosoftSQLServer_SQL2016SP2-WS2016_Express
IMAGE MicrosoftSQLServer_SQL2016SP2-WS2016_Standard
IMAGE MicrosoftSQLServer_SQL2017-WS2016_Enterprise
IMAGE MicrosoftWindowsServer_WindowsServer_2012-Datacenter
IMAGE MicrosoftWindowsServer_WindowsServer_2012-R2-Datacenter
IMAGE MicrosoftWindowsServer_WindowsServer_2016-Datacenter
IMAGE MicrosoftWindowsServer_WindowsServer_2019-Datacenter
IMAGE Canonical_UbuntuServer_12.04.3-LTS
IMAGE Canonical_UbuntuServer_12.04.4-LTS
IMAGE Canonical_UbuntuServer_12.04.5-LTS
IMAGE Canonical_UbuntuServer_14.04.0-LTS
IMAGE Canonical_UbuntuServer_14.04.1-LTS
IMAGE Canonical_UbuntuServer_14.04.2-LTS
IMAGE Canonical_UbuntuServer_14.04.3-LTS
IMAGE Canonical_UbuntuServer_14.04.4-LTS
IMAGE Canonical_UbuntuServer_14.04.5-LTS
IMAGE Canonical_UbuntuServer_16.04-LTS
IMAGE Canonical_UbuntuServer_16.04.0
IMAGE Canonical_UbuntuServer_18.04
IMAGE Canonical_UbuntuServer_18.10
IMAGE RedHat_RHEL_6.7
IMAGE RedHat_RHEL_6.8
IMAGE RedHat_RHEL_6.9
IMAGE RedHat_RHEL_6.10
IMAGE RedHat_RHEL_7.2
IMAGE RedHat_RHEL_7.3
IMAGE RedHat_RHEL_7.4
IMAGE RedHat_RHEL_7.5
IMAGE SUSE_SLES_11-SP4
IMAGE SUSE_SLES_12-SP4
IMAGE SUSE_SLES_15
IMAGE MicrosoftSQLServer_SQL2017-RHEL7_Enterprise
IMAGE MicrosoftSQLServer_SQL2017-RHEL7_Express
IMAGE MicrosoftSQLServer_SQL2017-RHEL7_Standard
IMAGE MicrosoftSQLServer_SQL2017-SLES12SP2_Enterprise
IMAGE MicrosoftSQLServer_SQL2017-SLES12SP2_Express
IMAGE MicrosoftSQLServer_SQL2017-Ubuntu1604_Enterprise
IMAGE MicrosoftSQLServer_SQL2017-Ubuntu1604_Express
SIZE Standard_A0
SIZE Standard_A1
SIZE Standard_A2
SIZE Standard_A3
SIZE Standard_A5
SIZE Standard_A4
SIZE Standard_A6
SIZE Standard_A7
SIZE Basic_A0
SIZE Basic_A1
SIZE Basic_A2
SIZE Basic_A3
SIZE Basic_A4
SIZE Standard_D1_v2
SIZE Standard_D2_v2
SIZE Standard_D3_v2
SIZE Standard_D4_v2
SIZE Standard_D5_v2
SIZE Standard_D11_v2
SIZE Standard_D12_v2
SIZE Standard_D13_v2
SIZE Standard_D14_v2
SIZE Standard_D15_v2
SIZE Standard_D2_v2_Promo
SIZE Standard_D3_v2_Promo
SIZE Standard_D4_v2_Promo
SIZE Standard_D5_v2_Promo
SIZE Standard_D11_v2_Promo
SIZE Standard_D12_v2_Promo
SIZE Standard_D13_v2_Promo
SIZE Standard_D14_v2_Promo
SIZE Standard_F1
SIZE Standard_F2
SIZE Standard_F4
SIZE Standard_F8
SIZE Standard_F16
SIZE Standard_A1_v2
SIZE Standard_A2m_v2
SIZE Standard_A2_v2
SIZE Standard_A4m_v2
SIZE Standard_A4_v2
SIZE Standard_A8m_v2
SIZE Standard_A8_v2
SIZE Standard_DS1
SIZE Standard_DS2
SIZE Standard_DS3
SIZE Standard_DS4
SIZE Standard_DS11
SIZE Standard_DS12
SIZE Standard_DS13
SIZE Standard_DS14
SIZE Standard_D2_v3
SIZE Standard_D4_v3
SIZE Standard_D8_v3
SIZE Standard_D16_v3
SIZE Standard_D32_v3
SIZE Standard_D1
SIZE Standard_D2
SIZE Standard_D3
SIZE Standard_D4
SIZE Standard_D11
SIZE Standard_D12
SIZE Standard_D13
SIZE Standard_D14
SIZE Standard_B1ms
SIZE Standard_B1s
SIZE Standard_B2ms
SIZE Standard_B2s
SIZE Standard_B4ms
SIZE Standard_B8ms
SIZE Standard_DS1_v2
SIZE Standard_DS2_v2
SIZE Standard_DS3_v2
SIZE Standard_DS4_v2
SIZE Standard_DS5_v2
SIZE Standard_DS11-1_v2
SIZE Standard_DS11_v2
SIZE Standard_DS12-1_v2
SIZE Standard_DS12-2_v2
SIZE Standard_DS12_v2
SIZE Standard_DS13-2_v2
SIZE Standard_DS13-4_v2
SIZE Standard_DS13_v2
SIZE Standard_DS14-4_v2
SIZE Standard_DS14-x8_v2
SIZE Standard_DS14_v2
SIZE Standard_DS2_v2_Promo
SIZE Standard_DS3_v2_Promo
SIZE Standard_DS4_v2_Promo
SIZE Standard_DS5_v2_Promo
SIZE Standard_DS11_v2_Promo
SIZE Standard_DS12_v2_Promo
SIZE Standard_DS13_v2_Promo
SIZE Standard_DS14_v2_Promo
SIZE Standard_F1s
SIZE Standard_F2s
SIZE Standard_F4s
SIZE Standard_F8s
SIZE Standard_F16s
SIZE Standard_D64_v3
SIZE Standard_D2s_v3
SIZE Standard_D4s_v3
SIZE Standard_D8s_v3
SIZE Standard_D16s_v3
SIZE Standard_D32s_v3
SIZE Standard_D64s_v3
SIZE Standard_E2_v3
SIZE Standard_E4_v3
SIZE Standard_E8_v3
SIZE Standard_E16_v3
SIZE Standard_E20_v3
SIZE Standard_E32_v3
SIZE Standard_E64i_v3
SIZE Standard_E64_v3
SIZE Standard_E2s_v3
SIZE Standard_E4-2s_v3
SIZE Standard_E4s_v3
SIZE Standard_E8-2s_v3
SIZE Standard_E8-4s_v3
SIZE Standard_E8s_v3
SIZE Standard_E16-4s_v3
SIZE Standard_E16-8s_v3
SIZE Standard_E16s_v3
SIZE Standard_E20s_v3
SIZE Standard_E32-8s_v3
SIZE Standard_E32-16s_v3
SIZE Standard_E32s_v3
SIZE Standard_E64-16s_v3
SIZE Standard_E64-32s_v3
SIZE Standard_E64is_v3
SIZE Standard_E64s_v3
SIZE Standard_DS15_v2
SIZE Standard_F2s_v2
SIZE Standard_F4s_v2
SIZE Standard_F8s_v2
SIZE Standard_F16s_v2
SIZE Standard_F32s_v2
SIZE Standard_F64s_v2
SIZE Standard_F72s_v2
SIZE Standard_G1
SIZE Standard_G2
SIZE Standard_G3
SIZE Standard_G4
SIZE Standard_G5
SIZE Standard_GS1
SIZE Standard_GS2
SIZE Standard_GS3
SIZE Standard_GS4
SIZE Standard_GS4-4
SIZE Standard_GS4-8
SIZE Standard_GS5
SIZE Standard_GS5-8
SIZE Standard_GS5-16
SIZE Standard_L4s
SIZE Standard_L8s
SIZE Standard_L16s
SIZE Standard_L32s
```

#### Create the AzureVM agent definition

To create the AzureVM agent in Solution Manager, complete the following steps:

1. In Solution Manager, select **Library** > **Agents**.
2. Select **+Add**.
3. In the **Name** field, enter the name of the agent.
4. From the **Type** list, select **AzureVM**.
5. In the **AzureVM Settings** section, enter the required information.
6. In the **Client Information** section:
   - In the **Directory** field, enter the installation directory of the AzureVM Connector.
   - In the **Name** field, enter `AzureVM.exe`.
   - In the **Config File Name** field, enter `Connector.config`.
7. In the **Config Script** section:
   - From the **Script Runner** list, select **ACSAZUREVM**.
   - From the **Script** list, select the config script you created.
8. In the **Drop-down Script** section:
   - From the **Script Runner** list, select **ACSAZUREVM**.
   - From the **Script** list, select the data script you created.
9. Select **Save**.

## FAQs

**Do I need to encode all values in Connector.config?**
Yes. The tenant ID, subscription ID, client ID, secret key, and OpCon API token must all be encoded using `Encrypt.exe` before being placed in `Connector.config`. The `--setup` switch encodes and inserts the OpCon API token automatically. Encoding is not encryption and does not protect the file — refer to [Encode credentials using the Encrypt utility](#encode-credentials-using-the-encrypt-utility).

**What happens if I install the Enterprise Manager plug-in and the Azure VM sub-type does not appear?**
Restart Enterprise Manager using **Run as Administrator**. If the sub-type still does not appear, confirm that the plug-in `.jar` file was copied to the correct `dropins` directory and that Enterprise Manager was fully restarted.

**Can I run multiple Azure VM Connector instances on the same server?**
Yes. Each connector instance requires its own installation directory and its own `Connector.config` file configured with the appropriate Azure account credentials.

**Is the connector compatible with all Azure regions?**
The connector supports all Azure regions listed in the data script. Additional regions can be added to the data script using the `REGION region-name` format.

## Glossary

**ACS framework** — The Agent Configuration Service framework in OpCon 25.0.3 and later that centralizes connector configuration within OpCon and provides a wrapper for ACS-compatible connectors.

**Connector.config** — The configuration file used by the Azure VM Connector to store encoded Azure account credentials and OpCon API connection settings.

**Encrypt.exe** — The encoding utility bundled with the Azure VM Connector. It accepts a plaintext value and returns a base64-then-hex encoded string for use in `Connector.config`. The encoding has no key and is reversible, so it conceals a credential from casual reading but does not protect it.

**ACSAZUREVM** — The script type and runner name used to integrate the Azure VM Connector with the Solution Manager ACS framework.

**dropins** — The Enterprise Manager plug-in discovery directory. Plug-in files placed here are loaded automatically when Enterprise Manager starts.
