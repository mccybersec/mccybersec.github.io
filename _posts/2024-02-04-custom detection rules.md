---
title: "Custom Detection Rules in Defender XDR" 
categories:
  - Defender XDR
---

Microsoft Defender XDR (formally known as Defender 365) is the set of products that Microsoft offers for Extended Detection and Response capability. Into this catalog we find:
- **Defender for Endpoint**<br>
By installing a sensor (the MDE sensor) on the endpoints it is possible to retrieve the list of installed apps - with the related settings and versions - the running processes - any parents and child processes that are associeted to them. This allows you to have a great view of your Device Inventory.

- **Defender for Office 365**<br>
It is a cloud-based email filtering service designed to protect your organization against advanced threats related to email and collaboration tools. Several capabilities like Email Protection (phishing defense, suspicious emails and links, safe attachments and links), investigation and remediation.

- **Defender for Identity**<br>
Microsoft Defender for Identity is a cloud-based security solution that helps you to Prevent breaches, Detect threats, Investigate suspicious activities and Respond to attacks that involve your identities. It is a sensor that is installed on Active Directory, Active Directory Federation Services (AD FS), or Active Directory Certification Services (AD CS) servers.

- **Defender for Cloud Apps**<br>
It is a CASB - Cloud Access Security Broker - that has the goal to protect our your user are interacting with Cloud Applications. It is useful for discovering the Shadow IT, Data Loss Prevention (at rest, in transit and in use), threat protection and SaaS security posture management (SSPM). 

The 4 products mentioned above are intrinsically connected in a single platform - _security.microsoft.com_.
Closer to Defender XDR there are Defender for Cloud (_i_) - whose objective is to protect your infrastructure and carry out Cloud Posture Management with different plans (Defender for Servers/Containers/Databases/Storage/API/DevOps/App Services) - Defender for IoT (_ii_) and Entra ID Identity Protection (_iii_).

Let's focus on Defender XDR.
The four products collect logs that are sent to the Defender engine - which raises alerts and incidents based on Microsoft's proprietary internal playbooks. This is one of the big differences with Security Information and Event Management (SIEM) Sentinel in which - in addition to autonomous rules that use Machine Learning - it is left to the SOC team to define under which conditions to raise an incident or not.

The raw data collected is however available to the user in tabular format, who can query them using Advanced Hunting with KQL - Kusto Query Language. So... why not use a KQL query to raise an alert? The Custom Detection Rules come into play in this context! <br>
A Custom Detection rule is characterized by
- a KQL query
- a frequency, i.e. how often to execute the KQL query (every hour, every 3/12/24 hours)
- the impacted entities, a mapping between a value identified by the KQL query and an entity which can be Device/Mailbox/User
- the actions to be performed such as Isolate device/Run antivirus scan/Delete email/Force password reset.
- metadata relating to the title of the raised alert, its the severity, the mapping to a MITER&Attack technique.

Let's see a practical example.<br>
Microsoft Defender for Endpoint has a great feature: when it notices that a device is internet exposed, it automatically inserts an _Internet facing_ tag associated to it. We want to create a Detection Rule that raises an alert listing all devices that are Internet faced. Your devices are evaluated every day.

```KQL
DeviceInfo
| where Timestamp > ago(30d)
| where IsInternetFacing
| extend InternetFacingInfo  = AdditionalFields
| extend InternetFacingReason = extractjson("$.InternetFacingReason", InternetFacingInfo, typeof(string)), 
   InternetFacingLocalPort = extractjson("$.InternetFacingLocalPort", InternetFacingInfo, typeof(int)),
   InternetFacingScannedPublicPort = extractjson("$.InternetFacingPublicScannedPort", InternetFacingInfo, typeof(int)),
   InternetFacingScannedPublicIp = extractjson("$.InternetFacingPublicScannedIp", InternetFacingInfo, typeof(string)), 
   InternetFacingLocalIp = extractjson("$.InternetFacingLocalIp", InternetFacingInfo, typeof(string)),   
   InternetFacingTransportProtocol=extractjson("$.InternetFacingTransportProtocol", InternetFacingInfo, typeof(string)), 
   InternetFacingLastSeen = extractjson("$.InternetFacingLastSeen", InternetFacingInfo, typeof(datetime))
| summarize arg_max(Timestamp, *) by DeviceId
```
The query above returns all the devices that are internet exposed, and for each of them the most recent info log.


![KQL results](/assets/images/snapshotresults.png)

During the creation of the Custom Detection Rule it is possible to map between entities and columns retrieved from the KQL query. In the example below, KQL has independently understood that it can uniquely identify a Device using value of _DeviceName_ or _DeviceId_ coloumn.

![impacted entities](/assets/images/impactedentities.png)

The actions that the custom Detection Rules currently allow are the following. These are very useful for a timely SOAR perspective - Security Operation, Automation and Response.

![impacted entities](/assets/images/actions.png)

In summary, 4 main benefits from Custom Detection Rules.

1. **Proactive Monitoring** - no need of manually hunting query execution 
2. **Customizability** - based on KQL
3. **Automated Alerts and Responsed** - alerts are generated and actions are triggered against involved entities
4. **Integration** - KQL among endpoint, identity, email and cloud apps data 

For more information don't hesitate to contact me!<br>
Thank you for taking time to read.

Stay tuned!<br>
_Mario_
