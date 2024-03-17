---
title: "How to continuously import Threat Intelligence Indicators in Microsoft Sentinel" 
categories:
  - Microsoft Sentinel
---

Microsoft Sentinel - our SIEM and SOAR Solution - has several methods to import your own threat intelligence data (BYOTI) or simply integrate the Microsoft Defender Threat Intelligence. <br>
Everything is performed using the [Threat Intelligence Solution](https://azuremarketplace.microsoft.com/en/marketplace/apps/azuresentinel.azure-sentinel-solution-threatintelligence-taxii?tab=Overview) in the Sentinel Content Hub. This solution contains several resources:
- 47 Analytics rule
- 4 Data connectors
- 5 Hunting queries
- 1 Workbook

Let's discuss the Data connectors:
- [Threat Intelligence Platforms - BEING DEPRECATED]()
- [Microsoft Defender Threat Intelligence (Preview)](#mdti)
- [Threat intelligence - TAXII](#taxii)
- [Threat Intelligence Upload Indicators API (Preview)](#tiapi)

## <a name="mdti">Microsoft Defender Threat Intelligence (Preview)</a>
This connector is used to import Indicators of Compromise (IOCs) from Microsoft Defender Threat Intelligence (MDTI) into Sentinel. <br>
To date, MDTI is offered in two plans: free version and premium version. Premium is not required for this connector.

| free version | premium version |
| ------------ | --------------- |
| Public indicators of compromise (IOCs) | Public indicators of compromise (IOCs) |
| Open-source intelligence (OSINT) | Open-source intelligence (OSINT) |
| Common vulnerabilities and exposures (CVEs) | Common vulnerabilities and exposures (CVEs) |
| Articles and analysis from Microsoft Threat Intelligence | Articles and analysis from Microsoft Threat Intelligence |
| Defender Threat Intelligence datasets | Defender Threat Intelligence datasets |
| Intelligence Profiles | Intelligence Profiles |
| _none_ |  Microsoft IOCs |
| _none_ | Microsoft-enriched OSINT |
| _none_ | URL and file intelligence |

Activating the connector is very simple: simply enable the connector.

![MDT](/assets/images/mdti.png)

## <a name="taxii">Threat intelligence - TAXII</a>
This connector is used to retrieve STIX (Structured Threat Information Expression) data from TAXII (Trusted Automated Exchange of Intelligence Information) servers. The difference between TAXII and STIX is simple: the first represents a serialization language, the second is an application layer protocol (https://oasis-open.github.io/cti-documentation/).
The connector works with a pull approach - where we can decide:
- which indicators: All available, at most one month old, at most one week old, at most one day old
- how often to fetch: ounces per day, ounces per minute, ounces per minute

To do this you need to provide the API root URL, Collection ID and any username and password. <br>
Want to test a free feed? use _https://otx.alienvault.com/taxii/root_ as API root URL and <i>user_AlienVault_</i> as collection ID. (Thanks to [Melina Ryan](https://au.linkedin.com/in/melinaryan) for the tip!)


![TAXII](/assets/images/taxii.png)


## <a name="tiapi">Threat Intelligence Upload Indicators API (Preview)</a>
This connector uses a push approach: calling the Microsoft Sentinel data plane API directly from another application. <br>
The endpoint to use has the following format
```https://sentinelus.azure-api.net/workspaces/[WorkspaceID]/threatintelligenceindicators:upload?api-version=2022-07-01 ```
you need to have a Microsoft Entra access token, which can be recovered by following the [official documentation](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/app-aad-token#get-an-azure-ad-access-token).<br>
> NOTE: also in this case the data is passed in STIX format

Below is an example powershell script that sends two one IOC.<br>
_If you want to test it quickly, you can create a logic App with a managed identity with Sentinel Contributor permissions._

```powershell
# Define the API endpoint
$apiEndpoint = "https://sentinelus.azure-api.net/workspaces/[WorkspaceID]/threatintelligenceindicators:upload?api-version=2022-07-01"

# Define the headers
$headers = @{
    "Authorization" = "Bearer [Your_AAD_Access_Token]"
    "Content-Type" = "application/json"
}

# Define the JSON object for the request body
$requestBody = @{
    "sourcesystem" = "test"
    "indicators" = @(
        @{
            "type" = "indicator"
            "spec_version" = "2.1"
            "id" = "indicator--10000003-71a2-445c-ab86-927291df48f8"
            "name" = "Test Indicator 1"
            "created" = "2010-02-26T18:29:07.778Z"
            "modified" = "2011-02-26T18:29:07.778Z"
            "pattern" = "[ipv4-addr:value = '172.29.6.7']"
            "pattern_type" = "stix"
            "valid_from" = "2015-02-26T18:29:07.778Z"
        }
    )
} | ConvertTo-Json

# Perform the HTTP POST request
$response = Invoke-RestMethod -Uri $apiEndpoint -Method Post -Headers $headers -Body $requestBody

# Display the response (optional)
$response
```

![LOGIC APP](/assets/images/logicapp.png)

--- 

IoCs are stored in the ThreatIntelligenceIndicator table in Sentinel – this means they can be used in investigation, analysis, threat hunting and analytics rules!

![KQL Threat Intelligence](/assets/images/kql.png)




For more information don't hesitate to contact me!<br>
Thank you for taking time to read.

Stay tuned!<br>
_Mario_