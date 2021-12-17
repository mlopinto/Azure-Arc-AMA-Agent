# Azure-Arc-AMA-Agent / DCR Rule Association
Are you struggling using Azure Policy to auto detect and deploy the **Azure Monitor Agent** **(AMA)** to Azure Arc enabled servers? Well, this article is for you. This will provide step-by-step instructions on how to test out this use case in your environment prior to deployment to your production environment.

If your organization has a manual process to apply logging rules and the agent AMA manually, this presents risk especially unnecessary time spent by an internal resource. Responsible parties can miss out on critical log sources if the VM is online and processing company data without this telemetry. Plus, the overhead maintenance to ensure these are running can be a burden and present operational risks that can be mitigated with automation. Allow the Azure policy to continously evaluate your VMs and then act. For example, if the rule evaluates that the VM is out of compliance, a remediation task can be automatically generated to auto deploy the agent and associated rules.

For the AMA to provide you actionable data in Azure Monitor or Sentinel, it requires a **Data Collection Rule (DCR)** that must be created in the same resource group as the Azure Arc servers. This DCR defines what VM performance counters, Windows event logs or Linux syslog data gathered from the VM that should be sent to Azure and where it should be stored. For details on DCR rules, please visit here and here. This method allows for granularity and allows you to select what logs your organization truly needs rather than shipping subsets of the data through the old classification (minimal, common, all) used with the Log Analytics agent.

These instructions will show you how to create an Azure Policy/Initiative that will continuously evaluate if your new Azure Arc VMs have the agent installed and the associated DCR. This will only showcase the Windows use case, but the JSON can be replicated, and resource values can be changed to Linux values to obtain similar telemetry on Linux machines.

## Prerequisites
Resource Policy Contributor, Owner or Contributor RBAC access. Be sure to only apply roles that provide you the least amount of privilege required for your role. Owner and Contributor provide heightened access and should be seldomly applied unless the user is an administrator of Azure or certain resource groups.

## Instructions
Make sure that the Azure Arc agent is running on the machine prior to assigning the policy. In order to enable the agent you can follow these instructions here.

In Azure Policy, create a new Definition that will review the Azure Arc enabled server validate whether the agent is present and if it is not it will remediate automatically. Copy and paste the Azure-Arc-AMA-Deployment.json file into your new policy. There is no parameters that need to be elected for this policy. 

Additionally, create a separate policy that will recognize if the associated data collection rules are applied to the Azure Arc enabled server. Copy and paste this .json file to your new policy definition. Under parameters you will need to declare the DCR Resource ID parameter. This allows the rule to associate the right DCR rule to this server. You can find this string by going to the DRC rule in the Azure Monitor blade, clicking on JSON view, copying the Resource ID and pasting it in the DCR Resource ID field back on your policy screen.

Once the two rules are created, you can collapse those into an initiative and assign the initiative to the Resource Group where the Azure Arc enabled servers lives. Or you can assign them individually to the resource group.

To validate, allow the policies to run over night or you can manually trigger a review by opening up the CLI and running "az policy state trigger-scan".

## Validation
The easiest way I have found to test this policy is to install the ARC agent on an on-prem or other cloud test or dev server using the instructions in the Azure Arc console. Another alternative would be to stand up a low scale VM on your workstation through Hyper-V. You can follow this video as a guide here. This will require you to ensure it has internet access to the following sites
