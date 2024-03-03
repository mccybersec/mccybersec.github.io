---
title: "DevSecOps with Microsoft Defender for DevOps" 
categories:
  - Microsoft Defender for Cloud
---

#### DevSecOps defined
_DevSecOps, which stands for development, security, and operations, is a framework that integrates security into all phases of the software development lifecycle. Organizations adopt this approach to reduce the risk of releasing code with security vulnerabilities. Through collaboration, automation, and clear processes, teams share responsibility for security, rather than leaving it to the end when issues can be much more difficult and costly to address. DevSecOps is a critical component of a multicloud security strategy_ <br>
(from Microsoft Security [blog](https://www.microsoft.com/en-us/security/business/security-101/what-is-devsecops))

Two tools that Microsoft offers and that help us in this context are:
- Microsoft Defender for Cloud
- GitHub Advanced Security

### GitHub Advanced Security
GitHub Advanced Security consists of CodeQL, Code Scanning, Secret Scanning, Security Overview and Dependency Review - the idea is to continuously analyze the code in your Github repositories in order to:
- **perform code scanning** <br>
It is an analysis of static code to identify vulnerabilities related to HOW the code was written. A simple example is a SQL injection: in the code you get the user's input and use it to fill a SQL query without sanitizing the input first.

- **use Dependabot** <br>
It analyzes the dependency graph of your code base in order to identify the use of deprecated libraries or those with well-known vulnerabilities that could be exploited by attackers as entry points to your environment.

- **identify Secrets** <br>
you scan the code looking for strings that match secrets regex - such as Storage keys. Some are built-in, but you can create your custom secret finding.

### Defender for Cloud
Defender for Cloud is composed by two great features: Workload Protection and Cloud Security Posture Management (CSPM). <br>
In the DevSecOps scenario, Defender for Cloud is used for CSPM by integrating findings identified by Github Advanced Security into the organization's cloud ecosystem. Enrichment that can be had with Defender for DevOps are:
- Scurity recommendations to fix code vulnerabilities
- Security recommendations to discover exposed secrets
- Security recommendations to fix open source vulnerabilities
- Security recommendations to fix infrastructure as code misconfigurations
- Security recommendations to fix DevOps environment misconfigurations
- Attack path analysis (a feature that correlates the configuration of you DevOps environment with the other Cloud resources protected by Defender for Cloud to identify possible attack paths that could potentially be used by an attacker)


Below are some features that I consider to be of added value when using Defender for DevOps.

- **single pane of glass of all your repositories, regardless of where they reside (Github, Azure DeOp, GitLab)**
![Defender for Cloud](/assets/images/single-view.png)

- **contextualization of any surface attacks involving repositories**
![Attack Path Analysis](/assets/images/attackpathanalysis.png)

- **Using the Cloud Security Explorer to query your resources visually**<br>
The following example returns all cloud resources provisioned by IaC templates with high severity misconfigurations
![Cloud Security Explorer](/assets/images/cloud-security-explorer.png)


- **Response workflow automation triggers using Logic App in response to alerts and recommendations related to your DevOps environment raised by Defender for Cloud**


For more information don't hesitate to contact me!<br>
Thank you for taking time to read.

Stay tuned!<br>
_Mario_