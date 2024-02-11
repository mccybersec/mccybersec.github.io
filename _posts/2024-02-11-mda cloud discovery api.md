---
title: "Defender for Cloud Apps - Cloud Discovery API" 
categories:
  - Defender for Cloud Apps
---

Microsoft Defender for Cloud Apps (MDA) is a CASB - Cloud Access Security Broker. The product analyzes user behavior towards cloud applications in order to identify anomalous/risky behaviour. <br>
There are 4 fundamental pillars of MDA.

1. **Cloud Discovery** <br>
Having a vision of all the cloud applications used by users (shadow IT) is an important element of the security of your environment: understanding which applications, making a security assessment (if compliant with regulatory standards, if they offer security features), understanding which users/ip/devices are used and how many bytes are exchanged with them. To obtain this information you can integrate proxies, firewalls with MDA and obtain network logs.

2. **Threat Protection & Quick mitigation & SaaS Security Posture Management (SSPM)** <br>
To date it is possible to connect various applications to MDA to have greater visibility of the actions carried out by users: Office365, Azure, Github, Salesforce are just some examples. Through the app connectors it is possible to have more in-depth visibility of the sign-in, copy, share, password reset operations carried out by users and the configurations they have - in order to provide recommendations to improve security. The applications also offer a set of APIs to mitigate the risk such as password reset, suspend user.

3. **Data Protection** <br>
In addition to obtaining the activity logs with the app connector, you also obtain the log files: from the single MDA portal it is possible to view the set of files stored on the cloud applications and related metadata (creation date, sharing level, collaborators, content and sensitivity labels). MDA in this case helps protect data at rest by reducing the exposure of sensitive data saved on cloud applications, data in motion by applying sensitivity labels before transfer from/to cloud applications and data in use by blocking operations or requiring explicit authentication checks.

4. **App Governance** <br>
OAuth applications can be very risky for your environment: these applications can have risky permissions and can be attack vectors. The goal is to reduce the attack surface in cloud apps based environments. MDA analyzes static information (which permissions the applications have, identifying the highly privileged ones), dynamic information (how the permissions are used, identifying the over privileged ones) and behavioral information (which applications are accessing sensitive content in an anomalous way).


Let's focus on Cloud Discovery. <br>
As mentioned above it is necessary to obtain network logs. Below is an example of network log from squid proxy.

```
10.0.6.193 - Christine@fabrikam.com [21/Apr/2023:00:00:00 -1000] "GET http://clearbooks.co.uk HTTP/1.1" 200 796 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64; rv:35.0) Gecko/20100101 Firefox/35.0" TCP_MISS:HIER_DIRECT
``` 

Network logs can be obtained in several ways. <br>
- **Microsoft Defender for Endpoint (MDE) sensor** <br>
From the MDE settings it is possible to define that the logs captured by the sensor are sent to MDA for cloud discovery.
- **Secure Web Gateway** <br>
With simple configurations it is possible to send data captured by Zscaler, iboss, Corrata, Menlo Security to MDA.
- **Proxy/firewall/Security Information Event Management tools (SIEM)** <br>
To date, MDA is able to parse 35 different log formats from the main vendors (Cisco ASA, Juniper, Palo Alto Firewall, Squid). You can also create your own log format. <br>
Logs can be uploaded into MDA in 3 ways
     - Snapshot reports: manually downloaded from the source and statically loaded into MDA.
     - Log collector: a process that receives logs via syslog/FTP from various sources and forwards them to MDA
     - Cloud Discovery API: topic of this post

MDA proposes three APIs to execute sequentially for Cloud Discovery.
1. **Initiate file upload** (__GET /api/v1/discovery/upload_url/__) <br>
This API represents the manifestation of wanting to upload network logs. <br>
Request: filename to upload and source type (equivalent to 'I want to upload the mylogs.csv file which contains logs retrieved from Squid proxy)
Response: url where to upload
2. **Perform file upload** (__PUT https://<initiate_file_upload_response_url>__) <br>
This API performs the file transfer
3. **Finalize file upload** (__POST /api/v1/discovery/done_upload/__) <br>
This API represents instructions about how to treat the newly obtained file: whether it should be a snapshot report or whether it should be associated with some source in MDA.

To upload data to MDA you must authenticate and have the correct write permissions. It is possible perform it in 3 different ways
- **Legacy Method** <br>
you generate a token from the security.microsoft.com portal and insert it into the request header.
- **Application Context** <br>
you create an app registrations in Microsoft Entra ID and assign write permissions to the application.
- **User Context** <br>
you create an app registrations in Microsoft Entra ID but the permissions permissions are assigned to the signed-in user.


Below is a simple example of a powershell script that uploads the zscaler-cef.txt file saved locally and associates it with the cloud-discovery-MDA-API source (already created) in MDA. It is performed using legacy authentication.

```powershell
<#
Perform operations to retrieve logs from the source.
Output: zcaler-cef.txt file
#>

#Initiate file upload API
$headers = @{
  Authorization = 'Token <YOUR-TOKEN>'
}
$Uri = "https://<YOUR-TENANT-INFO>.portal.cloudappsecurity.com/api/v1/discovery/upload_url/?filename=zscalecef.txt&source=ZSCALER_CEF"
$response = Invoke-WebRequest -Uri $Uri -Method Get -Headers $headers -UseBasicParsing

#Perform file upload
$Uri = $response.Content | ConvertFrom-Json
$Uri = $Uri.url
$headers = @{
  "x-ms-blob-type" = 'BlockBlob'
}
Invoke-WebRequest -Uri $Uri -Method Put -Headers $headers -InFile "zscaler-cef.txt" -UseBasicParsing

#Finalize file upload
$headers = @{
  Authorization = 'Token <YOUR-TOKEN>'
}
$body = @{
    "uploadUrl" = $Uri
    "inputStreamName" = "cloud-discovery-MDA-API"
}
$Uri = "https://<YOUR-TENANT-INFO>.portal.cloudappsecurity.com/api/v1/discovery/done_upload/"
Invoke-WebRequest -Uri $Uri -Method Post -Headers $headers -Body $body -UseBasicParsing
```

![Cloud Discovery](/assets/images/mda-clouddiscovery.png)


For more information don't hesitate to contact me!<br>
Thank you for taking time to read.

Stay tuned!<br>
_Mario_