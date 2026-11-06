# Splunk-window-SIEM-lab
Hands-on Windows SIEM lab using Splunk Enterprise, Universal Forwarder, Windows Server 2022 and Sysmon for security monitoring, detection and investigation.

# Table of Contents

Overview

Project Objectives

Tools & Technology

Architecture

# Overview
This project demonstrates the implementation of a lightweight __Security Information and Event Management (SIEM)__ environment using Splunk.

The lab consists of a __Windows 10 physical host__ running __Splunk Enterprise__ and a __Windows Server 2022 in a virtual machine__ configured with __Splunk Universal Forwarder and Sysmon.__

Windows Event Logs and Sysmon telemetry are forwarded to Splunk Enterprise, where they are indexed, searched, analyzed, and displayed through a custom security monitoring dashboard.

The project also involved investigating suspicious Windows authentication activity to determine whether the observed events represented a genuine security incident.

The primary focus was gaining practical experience in __SIEM implementation, Windows log collection, SPL queries, security monitoring, and evidence-based investigation.__

# Project Objectives
- Collect Windows Security, System, and Application logs. <br>
- Collect Sysmon telemetry. <br>
- Forward endpoint logs to Splunk Enterprise. <br>
- Detect failed and successful authentication events. <br>
- Monitor Windows logon activity. <br>
- Investigate suspicious authentication patterns. <br>
- Analyze process creation and network activity. <br>
- Build a security monitoring dashboard. <br>


# Tools & Technology
- __Splunk Enterprise__ <br>
- __Splunk Universal Forwarder__ <br>
- __Windows Server 2022__ <br>
- __Windows 10 Laptop__ <br>
- __Sysmon__ <br>
- __Splunk Search Processing Language (SPL)__ <br>
- __VirtualBox__ <br>
- __Windows Event Logs__ <br>


# Architecture

The lab uses a host-only network to connect Windows Server 2022 to Splunk Enterprise running on the Windows 10 host.

```text
       WINDOWS 10 HOST
    ┌─────────────────────────┐
    │                         │
    │    Splunk Enterprise    │
    │                         │
    │    Receiving Port: 9997 │
    └────────────┬────────────┘
                 │
                 │ TCP 9997
                 │
          HOST-ONLY NETWORK
                 │
                 ▼
    ┌─────────────────────────┐
    │  WINDOWS SERVER 2022 │  │
    │     RUNNING IN A
          VIRTUALBOX          │
    │  ┌───────────────────┐  │
    │  │ Splunk Universal  │  │
    │  │ Forwarder         │  │
    │  └───────────────────┘  │
    │                         │
    │  ┌───────────────────┐  │
    │  │      Sysmon       │  │
    │  └───────────────────┘  │
    │                         │
    │  Windows Event Logs     │
    └─────────────────────────┘
```

# Project implementation
The implementation was completed in stages, beginning with the lab environment and ending with security monitoring and dashboard development.

## Step 1: Prepare the Lab Environment
The first stage involved preparing the machines required for the SIEM environment. <br>

The following systems were used:<br>
- __Physical Windows 10 laptop.__ <br>
- __Windows Server 2022 virtual machine.__ <br>
- __Oracle VirtualBox.__ <br>

Due to the laptop's hardware limitations, the SIEM platform was installed directly on the physical Windows 10 machine rather than running additional resource-intensive virtual machines.<br>

The Windows Server 2022 machine was retained as the monitored endpoint.<br>

__Expected outcome:__ Both systems are available and can communicate through the selected network configuration.


## Step 2: Install Splunk Enterprise

Splunk Enterprise was installed on the physical Windows 10 host.

__Expected outcome:__  Splunk Enterprise is running and its web interface is accessible.

## Step 3: Configure Network Communication

A host-only network was configured in VirtualBox to allow communication between Windows Server 2022 and the Windows 10 host.

Connectivity was tested from the server using __ping__.  The connection was successful.

__Expected outcome:__ The Windows Server VM can reach the Splunk Enterprise host.

## Step 4: Configure Splunk Receiving

Splunk Enterprise was configured to receive forwarded data through TCP port __9997__.

The receiving port was enabled in Splunk Enterprise.

A Windows Defender Firewall inbound rule was also created on the Windows 10 host

The connection was tested from Windows Server 2022 using: <br>
```Test-NetConnection 192.168.56.1 -Port 9997 ```

The test returned a successful TCP connection.

__Expected outcome:__ The server can communicate with the Splunk receiving port.

## Step 5: Install Splunk Universal Forwarder

Splunk Universal Forwarder was installed on the Windows Server 2022 virtual machine.

The forwarder acts as the log collection agent responsible for sending endpoint data to Splunk Enterprise.

The forwarding configuration was checked 

__Expected outcome:__ The Universal Forwarder successfully connects to Splunk Enterprise.

## Step 6: Configure Windows Event Log Collection

The Universal Forwarder was configured to collect the following Windows Event Logs:

- __Security__ <br>
- __System__ <br>
- __Application__ <br>

The input configuration was maintained in the forwarder's __inputs.conf file.__

__Expected outcome:__ Windows Security, System, and Application events are configured for collection.

## Step 7: Configure Sysmon Collection

Sysmon was used to provide additional endpoint visibility beyond standard Windows Event Logs.

The Universal Forwarder was configured to collect the Sysmon Operational channel.

Sysmon telemetry includes process creation, network connections, file activity, registry changes, and DNS queries.

__Expected outcome:__ Sysmon events are collected and forwarded to Splunk Enterprise.

## Step 8: Verify Log Forwarding and Ingestion


After configuring the Windows event inputs, searches were performed in Splunk to verify that data was arriving.

__Windows Events__
<img width="1920" height="1029" alt="Windows_Events" src="https://github.com/user-attachments/assets/36bb967d-0b0e-4035-b53a-f152b42d542c" />
<br><br>
__Sysmon Events__
<img width="1920" height="1026" alt="Sysmon_logs" src="https://github.com/user-attachments/assets/8b902e4a-faa6-4e0f-972b-00709e5fe4fa" />

__Expected outcome:__ Events are visible in Splunk Search.


## Step 9: Analyze Windows Security Events

Once data ingestion was confirmed, SPL searches were developed to monitor authentication and endpoint activity.

Failed Authentication
index=windows_event_logs sourcetype="WinEventLog:Security" EventCode=4625 earliest=-24h
| stats count by host

index=windows_event_logs sourcetype="WinEventLog:Security" EventCode=4625 earliest=-24h
| stats count by host
































