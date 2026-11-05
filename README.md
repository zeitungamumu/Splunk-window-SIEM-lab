# Splunk-window-SIEM-lab
Hands-on Windows SIEM lab using Splunk Enterprise, Universal Forwarder, Windows Server 2022 and Sysmon for security monitoring, detection and investigation.

# Table of Contents

Overview

Project Objectives

Tools & Technology

Architecture

# Overview
This project demonstrates the implementation of a lightweight __Security Information and Event Management (SIEM)__ environment using Splunk.

The lab collects Windows security and system telemetry from a __Windows Server 2022__ and forwards the data to __Splunk Enterprise running on a Windows 10 host.__

__Sysmon__ is also used to provide additional endpoint telemetry, including process creation, network connections, file activity, and other system events.

# Project Objectives
- Collect Windows Security, System, and Application logs.

- Collect Sysmon telemetry.

- Forward endpoint logs to Splunk Enterprise.

- Detect failed and successful authentication events.

- Monitor Windows logon activity.

- Investigate suspicious authentication patterns.

- Analyze process creation and network activity.

- Build a security monitoring dashboard.


# Tools & Technology
__Splunk Enterprise__

__Splunk Universal Forwarder__

__Windows Server 2022__

__Windows 10 Laptop__

__Sysmon__

__Splunk Search Processing Language (SPL)__

__VirtualBox__

__Windows Event Logs__












