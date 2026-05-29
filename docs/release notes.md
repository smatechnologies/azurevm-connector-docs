---
sidebar_label: 'Release Notes'
title: Azure VM Connector release notes
description: "Version history and change details for the Azure VM Connector, including new features, improvements, and bug fixes."
tags:
  - Reference
  - System Administrator
  - Connectors
---

# Azure VM Connector release notes

## 26

### 26.0.0

*05/2026*

This release addresses security vulnerabilities in third-party libraries.

#### Bug fixes

**Replaced the abandoned org.ini4j library to remediate [CVE-2022-41404](https://nvd.nist.gov/vuln/detail/CVE-2022-41404).** The org.ini4j project is no longer maintained. The dependency was replaced with a custom internal implementation.

**Upgraded the Jackson libraries from 2.10.0 to 2.18.2.** The previous version contained multiple known vulnerabilities. This update brings the dependency to a current, supported release.
