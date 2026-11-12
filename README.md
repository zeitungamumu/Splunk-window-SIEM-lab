# Splunk-window-SIEM-lab
Hands-on Windows SIEM lab using Splunk Enterprise, Universal Forwarder, Windows Server 2022 and Sysmon for security monitoring, detection and investigation.

# Table of Contents

- [Overview](#overview)

- [Project Objectives](#project-objectives)

- [Tools & Technology](#tools--technology)

- [Architecture](#architecture)

- [Project implementation](#project-implementation)

- [Detection and Investigation](#detection-and-investigation)

- [Results and Key Findings](#results-and-key-findings)

- [Challenges Encountered](#challenges-encountered)

- [Lessons Learned](#lessons-learned)

- [Limitations and Future Improvements](#limitations-and-future-improvements)

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

### Failed Authentication

```spl
index=windows_event_logs sourcetype="WinEventLog:Security" EventCode=4625 earliest=-24h
| stats count by host
```
<img width="1920" height="919" alt="Failed authentication" src="https://github.com/user-attachments/assets/bf171875-0d0a-4ee7-b572-0b37bc106ff5" />


## Successful Authentication
```spl
index=windows_event_logs sourcetype="WinEventLog:Security" EventCode=4624 earliest=-24h
| stats count by host
```
<img width="1920" height="924" alt="Successful Authentication" src="https://github.com/user-attachments/assets/7168e521-a575-4491-bf2d-b6846cf9d0b8" />


## Top Accounts Targeted by Failed Logons
```spl
index=windows_event_logs sourcetype="WinEventLog:Security" EventCode=4625 earliest=-24h
| rex field=Message "(?s)Account For Which Logon Failed:.*?Account Name:\s+(?<target_account>[^\r\n]+)"
| eval account=coalesce(target_account,TargetUserName,Account_Name)
| where isnotnull(account) AND account!="-"
| stats count AS failed_attempts by account
| sort -failed_attempts
| head 10
```
<img width="1915" height="925" alt="Top Accounts Targeted by Failed Logons" src="https://github.com/user-attachments/assets/ac8a94d9-5dad-4bed-a98a-2510f16c73c8" />

## Process Creation Monitoring
```spl
index=sysmon_logs EventCode=1 earliest=-24h
| where NOT like(Image,"%\\SplunkUniversalForwarder\\%")
| stats count AS process_creations by Image
| sort -process_creations
| head 10
```
<img width="1920" height="918" alt="Process Creation Monitoring" src="https://github.com/user-attachments/assets/dccf7eff-9830-4c5f-8234-7b7c7852f8ba" />

## Network Connection Monitoring
```spl
index=sysmon_logs EventCode=3 earliest=-24h
| table _time host User Image SourceIp SourcePort DestinationIp DestinationPort Protocol Initiated
| sort -_time
| head 10
```
<img width="1908" height="919" alt="Network Connection Monitoring" src="https://github.com/user-attachments/assets/f391072d-acfc-4a1d-adea-d9879c090489" />

__Expected outcome:__ Security activity can be searched, filtered, and analyzed using SPL.




## Step 10: Build the Splunk Dashboard

The final implementation stage involved creating a Classic Simple XML dashboard titled:

__Windows Security Monitoring Dashboard__

__Description:__

Windows Server security monitoring using Windows Event Logs and Sysmon.

The dashboard was designed to provide an overview of authentication activity and endpoint behavior.

__The following panels were created:__

__1. Total Security Events__

__2. Failed Logons__

__3. Successful Logons__

__4. Sysmon Events__

__5. Failed vs Successful Logons__

__6. Failed Logons Over Time__

__7. Top Accounts Targeted by Failed Logons__

__8. Windows Logon Types__

__9. Top Process Creation Activity__

__10. Recent Sysmon Network Connections__

__11. Recent Failed Authentication Events__

The dashboard uses a combination of single-value panels, column charts, line charts, bar charts, pie charts, and statistics tables.

The searches use a rolling 24-hour time range.

__Expected outcome:__ A centralized dashboard displaying Windows security monitoring information.

__Evidences:__

![splunk_dashboard.xml](splunk_dashboard.xml)

![splunk_dashboard.pdf](splunk_dashboard.pdf)



 # Detection and Investigation

A practical investigation was conducted using Windows Security events collected from the Windows Server 2022 machine.

__Investigation Scenario__

Repeated failed authentication attempts against the Administrator account were identified.

__The objective was to determine:__

- Whether the failed logons were suspicious.

- Whether authentication eventually succeeded.

- Which account was targeted.

- Where the authentication attempts originated.

- Whether the activity was associated with remote access.

- Whether the available evidence supported an actual security incident.

## Observed Evidence
4 IT Department, 5 Sales Department and 8 Administrator
I investigated the Administrator.
Eight failed Administrator authentication attempts were identified.

|Attribute |Observation|
| :--- | :---: |
|Event ID  |   4625    |
|Account   |  Administrator|
|Number of failures|8|
|Logon Type|2 – Interactive|
|Source Address|127.0.0.1|
|Status| 0xC000006D|
|Substatus|0xC000006A|
|Process|svchost.exe|
|Process ID| 432 |

The substatus 0xC000006A indicated an incorrect password.

A successful Administrator logon (Event ID 4624) was subsequently observed.

An Event ID 4648 also showed explicit credential usage involving the Administrator account.

## Additional Investigation

__Further analysis included:__

- Reviewing Sysmon process creation events.

- Examining the services associated with process ID 432.

- Reviewing RemoteConnectionManager and LocalSessionManager logs.

- Checking for Windows Remote Interactive logons (Logon Type 10).

- Examining related authentication events.

- No corresponding Logon Type 10 events were found in the investigated time window.

The process ID was associated with normal Windows services, but the available evidence did not establish the exact initiating component or person responsible for the authentication sequence.

## Investigation Conclusion

__Classification: Inconclusive__

The investigation confirmed that eight failed local interactive Administrator authentication attempts were followed by a successful logon.

However, the available telemetry was insufficient to establish that the activity was malicious or externally initiated.

The investigation therefore did not classify the activity as a confirmed security incident.


# Results and Key Findings

The project successfully demonstrated:

- Deployment of Splunk Enterprise.<br>
- Configuration of a Windows Server endpoint for log collection.<br>
- Successful communication over the host-only network.<br>
- Successful forwarding through TCP port 9997.<br>
- Collection of Windows Security, System, and Application logs.<br>
- Collection of Sysmon telemetry.<br>
- Searching and analyzing events using SPL.<br>
- Monitoring of Windows authentication activity.<br>
- Investigation of suspicious authentication events.<br>
- Development of an 11-panel security monitoring dashboard.<br>

The investigation also demonstrated that suspicious activity must be evaluated using multiple sources of evidence before making a security incident determination.


# Challenges Encountered

Several technical challenges were encountered during implementation.

## Hardware Limitations

The physical laptop had limited RAM and available disk space.

__Resolution:__ Splunk Enterprise was installed directly on the Windows 10 host, while Windows Server 2022 remained the monitored virtual machine.

## Network Connectivity

The Universal Forwarder initially required network and firewall configuration before it could communicate with Splunk Enterprise.

__Resolution:__ A host-only network was configured, and TCP connectivity was verified on port 9997.

## Forwarder Configuration

Windows Event Log inputs and Sysmon collection required configuration and validation.

__Resolution:__ The inputs.conf configuration was reviewed and validated.

## Data Analysis

Some authentication events required additional investigation before their meaning could be established.

__Resolution:__ Related Windows Security events, Sysmon activity, process information, and Remote Desktop logs were examined.

## Dashboard Layout

Arranging multiple panels into a readable dashboard required adjustments to the Classic Simple XML layout.

__Resolution:__ Panels were organized into rows and appropriate visualizations were selected for the different searches.


# Lessons Learned

This project provided practical experience in:

- __SIEM architecture and implementation.<br>__
- __Windows event log collection.<br>__
- __Splunk Enterprise configuration.<br>__
- __Splunk Universal Forwarder deployment.<br>__
- __Windows endpoint monitoring.<br>__
- __Sysmon integration.<br>__
- __SPL search development.<br>__
- __Authentication analysis.<br>__
- __Process and network monitoring.<br>__
- __Dashboard development.<br>__
- __Evidence-based security investigations.<br>__

__An important lesson was that a suspicious event is not automatically proof of compromise.__

__Effective security monitoring requires understanding the context of events, correlating related telemetry, and distinguishing confirmed findings from assumptions.__


# Limitations and Future Improvements

## Current Limitations

- The lab currently monitors one Windows Server endpoint.

- The environment is designed for learning rather than production deployment.

- Detection coverage is limited to the configured Windows and Sysmon telemetry.

- The investigation did not establish a confirmed malicious incident.

- Automated incident response has not been implemented.

## Future Improvements

- Potential improvements include:

- Add more monitored Windows endpoints.

- Integrate Active Directory telemetry.

- Develop additional detection rules.

- Implement automated alerts.

- Improve event correlation.

- Develop MITRE ATT&CK-aligned detections.

- Add File Integrity Monitoring.

- Integrate SOAR capabilities.

- Develop automated incident response workflows.

__Note:__ SOAR is a planned future enhancement and is not part of the current implementation.























