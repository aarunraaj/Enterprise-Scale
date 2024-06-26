## Introduction

### Deprecation
The Log Analytics agent, also known as the Microsoft Monitoring Agent (MMA), is on a deprecation path and won't be supported after August 31, 2024. Any new data centers brought online after January 1 2024 will not support the Log Analytics agent. If you use the Log Analytics agent to ingest data to Azure Monitor, migrate to the new Azure Monitor agent prior to that date.

Portal Accelerator has been updated and new ALZ deployments will use AMA exclusively. We are working on bringing this update to Bicep and Terraform in the short-term. Additional Brownfield guidance for adopting AMA in existing environments will also be made available. 

### Timing
The migration from MMA to AMA has been a mayor project across multiple teams within Microsoft. ALZ held off on implementing AMA up to this point to ensure that a good feature set was available across all the different solutions. While there still are a few gaps, which are detailed below, we feel that the current AMA configuration is ready to be implemented in ALZ.

## Strategy
1. Include AMA for Greenfield customers using the portal deployment.
2. Brownfield adoption guidance. This will include:
    - Implementation guidance
    - Breaking changes
    - Cleanup guidance
    - Quick reference to public documentations for migration guidance for individual solutions
3. Include AMA for Greenfield and Brownfield customers using either a Bicep or Terraform deployment.

## Table: AMA parity status


| Service                                                                   | What it does                         | Status | Parity |
| ------------------------------------------------------------------------- | ------------------------------------ |--------|---------------|
| Agent health | Monitors agent heartbeat  | Deprecating. You can query the heartbeat. AMBA already has an Alert Rule for this.  | N/A|
| Sentinel | Security information and event management | Public Preview - Migrated to AMA | Windows Firewall Logs (Private preview), Application and service logs|
| Change Tracking | This feature tracks changes in virtual machines hosted in Azure, on-premises, other clouds | GA - Migrated to AMA | Parity|
| Azure Monitor --> VM Insights | Monitoring VMs | GA - Migrated to AMA | Parity |
| Update Management | Manages VM patches and updates | GA - Migrated to Azure Update Management (AUM) that does not require an agent  | |
| SQL Vulnerability Assessment Solution | Helps discover, track, and remediate potential database vulnerabilities | GA - Migrated to AMA and is now part of Microsoft Defender for SQL | Parity|
| SQL Advanced Thread Protection Solution | Detects anomalous activities indicating unusual and potentially harmful attempts to access or exploit databases | GA - Migrated to AMA and is now part of Microsoft Defender for SQL | Parity|
| SQL Assessment Solution | Identifies possible performance issues and evaluates that your SQL Server is configured to follow best practices. | GA - Now part of SQL best practices assessment. | Current ALZ Status 'Removed' due to LAW deployment constraint with ALZ design principles (requires LAW per subscription), ALZ team will work with relevant product team to address|
| MDfC for Servers | Provide server protections through Microsoft Defender for Endpoint or extended protection with just-in-time network access, file integrity monitoring, vulnerability assessment, and more. | GA (See parity column for detail) - Migrated to MDC (Agentless) | Features in development: FIM, Endpoint protection discovery recommendations, OS Misconfigurations (ASB recommendations). Features on backlog: Adaptive Application controls |
| MDfC for SQL Server Virtual Machines | Protect your entire database estate with attack detection and threat response for the most popular database types in Azure to protect the database engines and data types, according to their attack surface and security risks. | GA - Migrated to MDC (Agentless) | |


## Summary of changes to ALZ Code and Policies

### Removed ARM resources.
- Agent Health: Deprecated.
- Change Tracking (Automation account)
- Update Management (Automation account)
- VM Insights (Legacy solution/ MMA)
- SQL Assessment (Legacy solution)
- Sql Vulnerability Assessment (Legacy solution)
- Sql Advanced Threat Protection (Legacy solution)

### Removed Azure Policy Assignments
- PolicySetDefinition: Enable Azure Monitor for Virtual Machine Scale Sets / Legacy - Enable Azure Monitor for Virtual Machine Scale Sets
- PolicySetDefinition: Enable Azure Monitor for VMs / Legacy - Enable Azure Monitor for VMs

## New ARM Resources
- Resource group for each subscription containing User Assigned Managed Identity.
    - Default name: rg-ama-prod-001
- User Assigned Managed Identity
    - Name: id-ama-prod-<region>-001
- Data collection rules
    - dcr-changetracking-prod-<region>-001
    - dcr-defendersql-prod-<region>-001
    - dcr-vminsights-prod-<region>-001

## New Custom Policy Definitions
- Configure periodic checking for missing system updates on azure virtual machines and Arc-enabled virtual machines.
    - Windows: /providers/Microsoft.Authorization/policyDefinitions/59efceea-0c96-497e-a4a1-4eb2290dac15
    - Linux: /providers/Microsoft.Authorization/policyDefinitions/59efceea-0c96-497e-a4a1-4eb2290dac15
    - Windows: /providers/Microsoft.Authorization/policyDefinitions/bfea026e-043f-4ff4-9d1b-bf301ca7ff46
    - Linux: /providers/Microsoft.Authorization/policyDefinitions/bfea026e-043f-4ff4-9d1b-bf301ca7ff46
- Configure SQL VMs and Arc-enabled SQL Servers to install Microsoft Defender for SQL and AMA with a user-defined LA workspace
    - Configure Arc-enabled SQL Servers with Data Collection Rule Association to Microsoft Defender for SQL user-defined DCR
    - Configure Arc-enabled SQL Servers to automatically install Microsoft Defender for SQL and DCR with a user-defined LA workspace
    - Configure SQL Virtual Machines to automatically install Azure Monitor Agent
    - Configure SQL Virtual Machines to automatically install Microsoft Defender for SQL and DCR with a user-defined LA workspace
    - Configure SQL Virtual Machines to automatically install Microsoft Defender for SQL
- Deploy User Assigned Managed Identity for VM Insights

## New Policy Assignments
- Enable Azure Monitor for VMSS with Azure Monitoring Agent(AMA) 
- Enable Azure Monitor for VMs with Azure Monitoring Agent(AMA) 
- Enable Azure Monitor for Hybrid VMs with AMA 
- Configure periodic checking for missing system updates on azure virtual machines and Arc-enabled virtual machines.
- Deploy User Assigned Managed Identity for VM Insights
- Configure SQL VMs and Arc-enabled SQL Servers to install Microsoft Defender for SQL and AMA with a user-defined LA workspace
- Enable Change Tracking and Inventory for Arc-enabled virtual machines
- Enable Change Tracking and Inventory for virtual machines
- Enable ChangeTracking and Inventory for virtual machine scale sets
