# Release Notes

## General

The AzureVM Connector supports two sub-type options:

### Enterprise Manager
Enterprise Manager provides a sub-type plugin module that is copied into the Enterprise Manager plugins directory. The plugin provides a simple mechanism to create the commandline options required when executing the connector.

The sub-type exists as a Windows job sub-type **Azure VM**. 

### Solution Manager
OpCon version 25.0.3 or greater includes the ACS framework. The ACS framework provides the sub-type mechanism supporting the AzureVM connector. It provides a mechanism to centralize the configuration file (Connector.config) within the OpCon environment and provides a wrapper to the AzureVM connector. 

THe supplied integration available from the FTP site in section integrations can be downloaded and the items extracted and copied into the **\\plugins\\ACSAZUREVM** directory. 

The implementation exists as an ACS Agent **AzureVM** and associated job with several task types.

### ACS VMWare Connector

2026 May

### What's new

:eight_spoked_asterisk: **CON-1503**: Implemented the monitoring for task completion to use the TaskStatus method instead of waiting for task completion in the TaskStart method.
