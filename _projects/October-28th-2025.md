---
title: "Getting Started with Atomic Red Team Testing on Linux"
author: Mers Waf
tags:
  - dfir
  - labs
  - linux
---

### **Introduction**

Atomic Red Team is a library of simple tests that every security team can use to test their controls. These tests are mapped to the MITRE ATT&CK® framework, making it easy to validate your detection capabilities against known adversary techniques. In this guide, we'll walk through installing and running Atomic Red Team tests on Ubuntu Linux.

## Prerequisites

First, ensure your system is up to date and install the necessary packages:

bash

```
# Update the list of packages
sudo apt-get update

# Install pre-requisite packages
sudo apt-get install -y wget apt-transport-https software-properties-common

# Get the version of Ubuntu
source /etc/os-release

# Download the Microsoft repository keys
wget -q https://packages.microsoft.com/config/ubuntu/$VERSION_ID/packages-microsoft-prod.deb

# Register the Microsoft repository keys
sudo dpkg -i packages-microsoft-prod.deb

# Clean up the repository keys file
rm packages-microsoft-prod.deb

# Update package list with Microsoft repositories
sudo apt-get update
```

## Installing PowerShell

Since Atomic Red Team uses PowerShell, we need to install it on our Linux system:

bash

```
# Install PowerShell
sudo apt-get install -y powershell

# Start PowerShell session
pwsh
```

*Source: [Microsoft PowerShell Installation Guide](https://learn.microsoft.com/en-us/powershell/scripting/install/install-ubuntu?view=powershell-7.5)*

## Installing Atomic Red Team

Within your PowerShell session, install the necessary modules:

powershell

```
# Install required PowerShell modules
Install-Module -Name invoke-atomicredteam,powershell-yaml -Scope CurrentUser

# Download and install Atomic Red Team
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
Install-AtomicRedTeam -getAtomics -Force

# Alternative manual installation method
git clone https://github.com/redcanaryco/invoke-atomicredteam.git /usr/local/share/powershell/Modules/Invoke-AtomicRedTeam
```

## Your First Atomic Test

Let's start with a simple, safe test to verify our installation. We'll use **T1082 - System Information Discovery**, which gathers basic system information without making any changes:

bash

```
pwsh -exec bypass -Command "Import-Module '/usr/local/share/powershell/Modules/Invoke-AtomicRedTeam/Invoke-AtomicRedTeam.psd1' -Force; Invoke-AtomicTest T1082 -TestNumbers 3"
```

This test runs commands like `uname -a` to display system information - perfect for validating your setup works correctly.
