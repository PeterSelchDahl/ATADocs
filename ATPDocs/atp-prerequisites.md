---
# required metadata

title: Azure Advanced Threat Protection prerequisites
description: Describes the requirements for a successful deployment of Azure ATP in your environment
keywords:
author: shsagir
ms.author: shsagir
manager: shsagir
ms.date: 03/15/2020
ms.topic: conceptual
ms.collection: M365-security-compliance
ms.service: azure-advanced-threat-protection
ms.assetid: 62c99622-2fe9-4035-9839-38fec0a353da

# optional metadata

#ROBOTS:
#audience:
#ms.devlang:
ms.reviewer: itargoet
ms.suite: ems
#ms.tgt_pltfrm:
#ms.custom:

---

# Azure ATP prerequisites

This article describes the requirements for a successful deployment of Azure ATP in your environment.

>[!NOTE]
> For information on how to plan resources and capacity, see [Azure ATP capacity planning](atp-capacity-planning.md).

Azure ATP is composed of the Azure ATP cloud service, which consists of the Azure ATP portal and the Azure ATP sensor. For more information about each Azure ATP component, see [Azure ATP architecture](atp-architecture.md).

Azure ATP protects your on-premises Active Directory users and/or users synced to your Azure Active Directory. To protect an environment made up of only AAD users, see [AAD Identity Protection](https://docs.microsoft.com/azure/active-directory/identity-protection/overview).

To create your Azure ATP instance, you'll need an AAD tenant with at least one global/security administrator. Each Azure ATP instance supports a multiple Active Directory forest boundary and Forest Functional Level (FFL) of Windows 2003 and above.

This prerequisite guide is divided into the following sections to ensure you have everything you need to successfully deploy Azure ATP.

[Before you start](#before-you-start): Lists information to gather and accounts and network entities you'll need to have before starting to install.

[Azure ATP portal](#azure-atp-portal-requirements): Describes Azure ATP portal browser requirements.

[Azure ATP sensor](#azure-atp-sensor-requirements): Lists Azure ATP sensor hardware, and software requirements.

[Azure ATP standalone sensor](#azure-atp-standalone-sensor-requirements): The Azure ATP Standalone Sensor is installed on a dedicated server and requires port mirroring to be configured on the domain controller to receive network traffic.

> [!NOTE]
> Azure ATP standalone sensors do not support the collection of Event Tracing for Windows (ETW) log entries that provide the data for multiple detections. For full coverage of your environment, we recommend deploying the Azure ATP sensor.

## Before you start

This section lists information you should gather as well as accounts and network entity information you should have before starting Azure ATP installation.

- Acquire a license for Enterprise Mobility + Security 5 (EMS E5) directly via the [Microsoft 365 portal](https://www.microsoft.com/cloud-platform/enterprise-mobility-security-pricing) or use the Cloud Solution Partner (CSP) licensing model. Standalone Azure ATP licenses are also available.

- Verify the domain controller(s) you intend to install Azure ATP sensors on have internet connectivity to the Azure ATP Cloud Service. The Azure ATP sensor supports the use of a proxy. For more information on proxy configuration, see [Configuring a proxy for Azure ATP](configure-proxy.md).

- At least one of the following directory services accounts with read access to all objects in the monitored domains:
  - A **standard** AD user account and password. Required for sensors running Windows Server 2008 R2 SP1.
  - A **group Managed Service Account** (gMSA). Requires Windows Server 2012 or above.  
  All sensors must have permissions to retrieve the gMSA account's password.  
  To learn about gMSA accounts, see [Getting Started with Group Managed Service Accounts](/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_CreateGMSA).

    The following table shows which AD user accounts can be used with which server versions:

    |Account type|Windows Server 2008 R2 SP1|Windows Server 2012 or above|
    |---|---|---|
    |**Standard** AD user account|Yes|Yes|
    |**gMSA** account|No|Yes|

    > [!NOTE]
    >
    > - For sensor machines running Windows Server 2012 and above, we recommend using a **gMSA** account for its improved security and automatic password management.
    > - If you have multiple sensors, some running Windows Server 2008 and others running Windows Server 2012 or above, in addition to the recommendation to use a **gMSA** account, you must also use at least one **standard** AD user account.
    > - If you have set custom ACLs on various Organizational Units (OU) in your domain, make sure that the selected user has read permissions to those OUs.

- If you run Wireshark on Azure ATP standalone sensor, restart the Azure Advanced Threat Protection sensor service after you've stopped the Wireshark capture. If you don't restart the sensor service, the sensor stops capturing traffic.

- If you attempt to install the Azure ATP sensor on a machine configured with a NIC Teaming adapter, you'll receive an installation error. If you want to install the Azure ATP sensor on a machine configured with NIC teaming, see [Azure ATP sensor NIC teaming issue](troubleshooting-atp-known-issues.md#nic-teaming).

- **Deleted Objects** container Recommendation: User should have read-only permissions on the Deleted Objects container. Read-only permissions on this container allows Azure ATP to detect user deletions from your Active Directory. For information about configuring read-only permissions on the Deleted Objects container, see the **Changing permissions on a deleted object container** section of the [View or Set Permissions on a Directory Object](https://technet.microsoft.com/library/cc816824%28v=ws.10%29.aspx) article.

- Optional **Honeytoken**: A user account of a user who has no network activities. This account is configured as an Azure ATP Honeytoken user. For more information about using Honeytokens, see [Configure exclusions and Honeytoken user](install-atp-step7.md).

- Optional: When deploying the standalone sensor, it is necessary to forward Windows events 4726, 4728, 4729, 4730, 4732, 4733, 4743, 4753, 4756, 4757, 4758, 4763, 4776, 7045, and 8004 to Azure ATP to further enhance Azure ATP authentication based detections, additions to sensitive groups and suspicious service creation detections.  Azure ATP sensor receives these events automatically. In Azure ATP standalone sensor, these events can be received from your SIEM or by setting Windows Event Forwarding from your domain controller. Events collected provide Azure ATP with additional information that is not available via the domain controller network traffic.

## Azure ATP portal requirements

Access to the Azure ATP portal is via a browser, supporting the following browsers and settings:

- A browser that supports TLS 1.2, such as:
  - Microsoft Edge
  - Internet Explorer version 11 and above
  - Google Chrome 30.0 and above
- Minimum screen width resolution of 1700 pixels
- Firewall/proxy open - To communicate with the Azure ATP cloud service, *.atp.azure.com port 443 must be open in your firewall/proxy.

    > [!NOTE]
    > You can also use our Azure service tag (**AzureAdvancedThreatProtection**) to enable access to Azure ATP. For more information about service tags, see [Virtual network service tags](https://docs.microsoft.com/azure/virtual-network/service-tags-overview) or [download the service tags](https://www.microsoft.com/download/details.aspx?id=56519) file.

 ![Azure ATP architecture diagram](media/azure-atp-architecture.png)

> [!NOTE]
> By default, Azure ATP supports up to 200 sensors. If you want to install more sensors, contact Azure ATP support.

## Azure ATP Network Name Resolution (NNR) requirements

Network Name Resolution (NNR) is a main component of Azure ATP functionality. For the Azure ATP service to work properly, at least one of the following NNR methods must be accessible for Azure ATP sensors:

1. **NTLM over RPC** (TCP Port 135)
2. **NetBIOS** (UDP port 137)
3. **RDP** (TCP port 3389) - only the first packet of Client hello
4. **Queries of the DNS server using reverse DNS lookup of the IP address** (UDP 53)

For methods 1, 2 and 3 to work, the relevant ports must be opened inbound from the Azure ATP sensors to devices on the network. To learn more about Azure ATP and NNR, see [Azure ATP NNR policy](atp-nnr-policy.md).

## Azure ATP sensor requirements

This section lists the requirements for the Azure ATP sensor.

### General

> [!NOTE]
> Make sure [KB4487044](https://support.microsoft.com/help/4487044/windows-10-update-kb4487044) is installed when using Server 2019. Azure ATP Sensors already installed on 2019 servers without this update will be automatically stopped.

The Azure ATP sensor supports installation on a domain controller running Windows Server 2008 R2 SP1 (not including Server Core), Windows Server 2012, Windows Server 2012 R2, Windows Server 2016 (including Windows Server Core but not Windows Nano Server), Windows Server 2019 (including Windows Core but not Windows Nano Server).

The domain controller can be a read-only domain controller (RODC).

For your domain controllers to communicate with the cloud service, you must open port 443 in your firewalls and proxies to *.atp.azure.com.

During installation, if .Net Framework 4.7 or later is not installed, the .Net Framework 4.7 is installed and might require a reboot of the domain controller, if a restart is already pending.

> [!NOTE]
> A minimum of 5 GB of disk space is required and 10 GB is recommended. This includes space needed for the Azure ATP binaries, Azure ATP logs, and performance logs.

### Server specifications

The Azure ATP sensor requires a minimum of 2 cores and 6 GB of RAM installed on the domain controller.
For optimal performance, set the **Power Option** of the machine running the Azure ATP sensor to **High Performance**.

Azure ATP sensors can be deployed on domain controllers of various loads and sizes, depending on the amount of network traffic to and from the domain controllers, and the amount of resources installed.

For Windows Operating systems 2008R2 and 2012, Azure ATP Sensor is not supported in a [Multi Processor Group](https://docs.microsoft.com/windows/win32/procthread/processor-groups) mode. For more information about multi-processor group mode, see [troubleshooting](troubleshooting-atp-known-issues.md#multi-processor-group-mode).

>[!NOTE]
> When running as a virtual machine, dynamic memory or any other memory ballooning feature is not supported.

For more information about the Azure ATP sensor hardware requirements, see [Azure ATP capacity planning](atp-capacity-planning.md).

### Time synchronization

The servers and domain controllers onto which the sensor is installed must have time synchronized to within five minutes of each other.

### Network adapters

The Azure ATP sensor monitors the local traffic on all of the domain controller's network adapters.  
After deployment, use the Azure ATP portal to modify which network adapters are monitored.

The sensor is not supported on domain controllers running Windows 2008 R2 with Broadcom Network Adapter Teaming enabled.

### Ports

The following table lists the minimum ports that the Azure ATP sensor requires:

|Protocol|Transport|Port|To/From|Direction|
|------------|-------------|--------|-----------|-------------|
|**Internet ports**|||||
|SSL (*.atp.azure.com)|TCP|443|Azure ATP cloud service|Outbound|
|SSL(localhost)|TCP|444|localhost|Both|
|**Internal ports**|||||
|DNS|TCP and UDP|53|DNS Servers|Outbound|
|Netlogon (SMB, CIFS, SAM-R)|TCP/UDP|445|All devices on network|Outbound|
|Syslog (optional)|TCP/UDP|514, depending on configuration|SIEM Server|Inbound|
|RADIUS|UDP|1813|RADIUS|Inbound|

### Windows Event logs

Azure ATP detection relies on the following specific Windows Event logs that the sensor parses from your domain controllers: 4726, 4728, 4729, 4730, 4732, 4733, 4743, 4753, 4756, 4757, 4758, 4763, 4776, 7045, and 8004. For the correct events to be audited and included in the Windows Event log, your domain controllers require accurate Advanced Audit Policy settings. For more information about setting the correct policies, see, [Advanced audit policy check](atp-advanced-audit-policy.md). To [make sure Windows Event 8004 is audited](configure-windows-event-collection.md#ntlm-authentication-using-windows-event-8004) as needed by the service, review your [NTLM audit settings](https://blogs.technet.microsoft.com/askds/2009/10/08/ntlm-blocking-and-you-application-analysis-and-auditing-methodologies-in-windows-7/).

> [!NOTE]
>
> - Using the Directory service user account, the sensor queries endpoints in your organization for local admins using SAM-R (network logon) in order to build the [lateral movement path graph](use-case-lateral-movement-path.md). For more information, see [Configure SAM-R required permissions](install-atp-step8-samr.md).

## Azure ATP standalone sensor requirements

This section lists the requirements for the Azure ATP standalone sensor.

> [!NOTE]
> Azure ATP standalone sensors do not support the collection of Event Tracing for Windows (ETW) log entries that provide the data for multiple detections. For full coverage of your environment, we recommend deploying the Azure ATP sensor.

### General

The Azure ATP standalone sensor supports installation on a server running Windows Server 2012 R2 or Windows Server 2016 (Include server core).
The Azure ATP standalone sensor can be installed on a server that is a member of a domain or workgroup.
The Azure ATP standalone sensor can be used to monitor Domain Controllers with Domain Functional Level of Windows 2003 and above.

For your standalone sensor to communicate with the cloud service, port 443 in your firewalls and proxies to *.atp.azure.com must be open.

For information on using virtual machines with the Azure ATP standalone sensor, see [Configure port mirroring](configure-port-mirroring.md).

> [!NOTE]
> A minimum of 5 GB of disk space is required and 10 GB is recommended. This includes space needed for the Azure ATP binaries, Azure ATP logs, and performance logs.

### Server specifications

For optimal performance, set the **Power Option** of the machine running the Azure ATP standalone sensor to **High Performance**.<br>
Azure ATP standalone sensors can support monitoring multiple domain controllers, depending on the amount of network traffic to and from the domain controllers.

>[!NOTE]
> When running as a virtual machine, dynamic memory or any other memory ballooning feature is not supported.

For more information about the Azure ATP standalone sensor hardware requirements, see [Azure ATP capacity planning](atp-capacity-planning.md).

### Time synchronization

The servers and domain controllers onto which the sensor is installed must have time synchronized to within five minutes of each other.

### Network adapters

The Azure ATP standalone sensor requires at least one Management adapter and at least one Capture adapter:

- **Management adapter** - used for communications on your corporate network. The sensor will use this adapter to query the DC it's protecting and performing resolution to machine accounts. <br>This adapter should be configured with the following settings:

    - Static IP address including default gateway

    - Preferred and alternate DNS servers

    - The **DNS suffix for this connection** should be the DNS name of the domain for each domain being monitored.

        ![Configure DNS suffix in advanced TCP/IP settings](media/ATP-DNS-Suffix.png)

        > [!NOTE]
        > If the Azure ATP standalone sensor is a member of the domain, this may be configured automatically.

- **Capture adapter** - used to capture traffic to and from the domain controllers.

    > [!IMPORTANT]
    >
    > - Configure port mirroring for the capture adapter as the destination of the domain controller network traffic. For more information, see [Configure port mirroring](configure-port-mirroring.md). Typically, you need to work with the networking or virtualization team to configure port mirroring.
    > - Configure a static non-routable IP address (with /32 mask) for your environment with no default sensor gateway and no DNS server addresses. For example, 10.10.0.10/32. This ensures that the capture network adapter can capture the maximum amount of traffic and that the management network adapter is used to send and receive the required network traffic.

### Ports

The following table lists the minimum ports that the Azure ATP standalone sensor requires configured on the management adapter:

|Protocol|Transport|Port|From|To|Direction|
|------------|-------------|--------|-----------|-------------|
|**Internet ports**|||||
|SSL (*.atp.azure.com)|TCP|443|Azure ATP Sensor|Azure ATP cloud service|Outbound|
|**Internal ports**|||||
|LDAP|TCP and UDP|389|Domain controllers|Azure ATP Sensor|Outbound|
|Secure LDAP (LDAPS)|TCP|636|Domain controllers|Azure ATP Sensor|Outbound|
|LDAP to Global Catalog|TCP|3268|Domain controllers|Azure ATP Sensor|Outbound|
|LDAPS to Global Catalog|TCP|3269|Domain controllers|Azure ATP Sensor|Outbound|
|Kerberos|TCP and UDP|88|Domain controllers|Azure ATP Sensor|Outbound|
|Netlogon (SMB, CIFS, SAM-R)|TCP and UDP|445|All devices on network|Outbound|
|Windows Time|UDP|123|Domain controllers|Azure ATP Sensor|Outbound|
|DNS|TCP and UDP|53|DNS Servers|Azure ATP Sensor|Outbound|
|Syslog (optional)|TCP/UDP|514, depending on configuration|Azure ATP Sensor|SIEM Server|Inbound|
|RADIUS|UDP|1813|RADIUS|Azure ATP Sensor|Inbound|

> [!NOTE]
>
> - Using the Directory service user account, the sensor queries endpoints in your organization for local admins using SAM-R (network logon) in order to build the [lateral movement path graph](use-case-lateral-movement-path.md). For more information, see [Configure SAM-R required permissions](install-atp-step8-samr.md).

## See Also

- [Azure ATP sizing tool](https://aka.ms/aatpsizingtool)
- [Azure ATP architecture](atp-architecture.md)
- [Install Azure ATP](install-atp-step1.md)
- [Network Name Resolution (NNR)](atp-nnr-policy.md)
- [Check out the Azure ATP forum!](https://aka.ms/azureatpcommunity)
