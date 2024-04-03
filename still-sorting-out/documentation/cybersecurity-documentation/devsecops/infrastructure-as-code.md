---
description: https://tryhackme.com/room/introtoiac
---

# 🪩 Infrastructure as code

Instead of provisioning and managing infrastructure manually, as discussed above, infrastructure as code allows the automation of these tasks through code. This code can be used to define a "desired state" for the infrastructure, acting as a blueprint for IaC tools to provision the infrastructure.

<figure><img src="../../../../.gitbook/assets/image (450).png" alt=""><figcaption></figcaption></figure>

Having infrastructure deployed from code also means it can be a collaborative process and a shared responsibility. This means knowledge of infrastructure and its configuration isn't siloed to a single team member, avoiding knowledge dependencies whilst supporting continuous integration (CI).

IaC is:

**Scalable,**

**Versionable** and

&#x20;**Repeatable**&#x20;

### IaC in Practice&#x20;

\
**Infrastructure Automation:** Since IaC allows the automation of the provisioning and configuration of an infrastructure, reprovisioning and re-configuring an infrastructure in a backup data centre is streamlined and less time-consuming.

**Data Backup and Recovery:** Data loss or corruption is one of the most significant risks associated with disaster recovery and can be very costly to the affected company. Infrastructure as code can be used to automate the backup and recovery of data. Backup procedures can be defined in code, which will be triggered in the event of a disaster to ensure data is restored.

**Failover Automation:** During a disaster, critical services can become unavailable as they will point to the broken infrastructure. IaC can be used to automate the failover process so critical services use the backup infrastructure, which reduces downtime.

**Configuration Consistency:** IaC keeps infrastructure consistent and well-documented; this consistency is vital during the disaster recovery process and helps avoid misconfigurations (often caused by simple human error or lack of documentation on the current configuration) during the transition to a backup environment.

**Scaling Resources:** The recovery process can be resource intensive, and IaC can be used to scale resources to handle the increased workloads (and scale back down afterwards).&#x20;

### Tools

#### Declarative vs. Imperative

**Declarative**

This involves declaring an explicit desired state for your infrastructure, min/max resources, x components, etc.; the IaC tool will perform actions based on what is defined.

**Imperative**

This approach involves defining specific commands to be run to achieve the desired state; these commands need to be executed in a particular order. Imperative IaC is usually not idempotent in that if you run these tools multiple times, the same result won’t be replicated, or the tool may run into some configuration issues. This is because these commands will be run regardless of the state of your infrastructure, so running these commands on an already provisioned infrastructure may cause additional unwanted infrastructure to be added or may break the configuration of some existing components (depending on the set-up of the infrastructure and the commands being run).

### Agent-based vs. Agentless

**Agent-based**

An “agent” will need to be installed on the server that is to be managed. It runs in the background and acts as a communication channel between the IaC tool and the resources that need managing.\
An agent-based IaC tool can perform tasks even when the system has limited connectivity or is offline, making it a robust choice for automation; however, you need to take care with maintenance when using these tools, monitoring the agent software closely so it can be restarted in the event of a crash.

**Agentless**

Agentless IaC tools, as you may have guessed, do not require agents to be installed on target systems. Instead, these tools leverage existing communication protocols like SSH, WinRM or Cloud APIs to interact with and provision resources on the target system. Agentless IaC tools benefit from being easier to use; their simplicity during set-up can mean faster deployment time, and their agentless nature can also mean the tool can be deployed across multiple environments without custom agent changes needing to be made based on each environment’s needs, making it a flexible choice. Also, compared to agent-based IaC tools, there is less maintenance and no risks surrounding the securing of an agent.

### Immutable vs. Mutable&#x20;

**Mutable**

Mutable infrastructure means you can make changes to that infrastructure in place. So, in our current example, we can upgrade the application on the server it’s currently running on. The advantage here is that no additional resources need to be provisioned for this update, with the resources already being used just needing to be changed.

**Immutable**\
Immutable infrastructure means you cannot make changes to it. Once an infrastructure has been provisioned, that’s how it will be until it’s destroyed. Now, let’s apply that to our example. When this new version 2 of the web application is deployed, an entirely new infrastructure will be deployed with this new version.

***

**On-prem IaC:** The main advantage of on-premises infrastructure as code is that it allows complete control. This complete control is sometimes required due to regulations/security and compliance requirements. This is common in bigger banking corporations or businesses that deal with customer-sensitive or government data. These regulations can call for data sovereignty (meaning the data has to be stored and processed within the country). Whilst cloud providers offer specialised government cloud environments for some of these cases (e.g. us gov cloud in AWS), these offerings are only available in certain regions; for regions like Africa, there is no dedicated government cloud, meaning on-premises infrastructure as code would need to be used to adhere to strict security/compliance requirements.&#x20;

**Cloud**-based **IaC:** It has many advantages for fast-paced and growing businesses. The scalability allows infrastructure to be rapidly scaled up or down to meet fluctuating demand without the constraints of physical on-premises infrastructure. Cloud-based infrastructure also allows infrastructure to be deployed globally across different regions, reducing user latency, which can be essential for businesses that need to provide a service to customers worldwide with little to no latency, like online gaming. With cloud providers billing on a pay-as-you-go basis, this can mean that companies can pay only for the infrastructure that they need/use, meaning investment doesn’t need to be made into physical hardware and the maintenance of that hardware.
