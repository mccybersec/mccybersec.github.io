---
title: "Capabilities with Microsoft Sentinel" 
categories:
  - Microsoft Sentinel
---

Sentinel is the Microsoft SIEM (Security Information Event Management) and SOAR (Security Orchestration Automation and Response) solution. SIEM concerns everything related to the collection of data from the various sources of the organization, its storage, retention and access governance. SOAR concerns everything related to the reporting, mitigation, containment and eradication of a (possible) threat.


![Sentinel SIEM and SOAR](/assets/images/siem-soar.png)

The ring that connects these two aspects are the __analytics rules__ - defined rules that match the collected data and produce Incidents in Sentinel. Sentinel offers different types of analytics rules: scheduled, near real time, fusion engine, Machine Learning based, anomaly detection, etc - these will be covered in a future blog post.

SOAR capability is characterized by the execution of a set of operations in sequence, often refered as _playbook_. In the Azure ecosystem, playbooks are built with Logic Apps - an Azure resource defined with a no/low code approach. A Logic App is a static definition of steps to perform, in which it is possible to insert conditional if/else logic and cyclic iterations. Each Logic App has an entry point - _trigger_ - which initites the creation of a new instance and its execution: the trigger can be HTTP, HTTP Endpoint, Recurrence, and Request.

![Logic App](/assets/images/logic-app.png)

The desired workflow is to execute a specific playbook if a specific incident is created in Sentinel. In this scenario it is necessary to introduce a further concept: Automation Rules. An automation rule aims to automatically respond to an incident and perform management operations on it, such as changing its status and risk, assigning an owner, classifying it or running a workbook.

The following diagram summarizes the concepts described above
1. le varie sorgenti inviano dati a Sentinel, che sono memorizzati
2. in Sentinel sono definite delle analytics rule che analizzano continuamente i dat
3. se una analytic è matchata, essa può sollevare un incident
4. a un incident è associata una o più automation rules che possono effettuare le seguenti azioni
    - change status
    - change severity
    - assign owner
    - add tags
    - add task
    - run playbook
5. logic app workflow is executed

![Logic App](/assets/images/end-to-end.png)


It `s important to note that:

1. An automation rule can be invoked when:
    - an incident is created, with matching conditions based on the source that created the incident, the analytics rules and metadata associated with the incident itself (severity, status, tactics etc) or the entity involved (account name/domain, file name/hash, process id /key, etc)
    - an incident is updated, with matching conditions based on when an incident is created + information on who updated the incident (an automation, an application, a playbook, a manual user action, etc) + information on what they look like change the properties
    - an alert is created

2. An event (incident/alert creation or incident update) can invoke the execution of multiple automation rules. You can assign a priority value to each automations rule to ensure that they are processed in the desired order. Take note that later automation rules will evaluate the conditions of the incident according to its state after being acted on by previous automation rules.

3. An automation rule can contain several actions of the same type - such as invoking two or more different playbooks.

[Here](https://github.com/Azure/Azure-Sentinel/tree/master/Playbooks) the unified Microsoft Sentinel and Microsoft 365 Defender Github repository that contains several great playbook examples.

For more information don't hesitate to contact me!<br>
Thank you for taking time to read.

Stay tuned!<br>
_Mario_