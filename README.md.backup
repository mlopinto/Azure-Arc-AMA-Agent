# Azure-Arc-AMA-Agent / DCR Rule Association
Are you struggling using Azure Policy to auto detect and deploy the **Azure Monitor Agent** **(AMA)** to Azure Arc enabled servers? Well, this page is for you. This will provide step-by-step instructions on how to test out this use case in your environment prior to deployment to your production environment.

**_WHY_**: If your organization has a manual process to configure the AMA agent manually and enable logging rules independently, this presents risk especially unnecessary time by an operational resource and window of time where you may miss out on critical log sources if the VM is online and processing company data without this telemetry. Plus, the overhead maintenance to ensure these are running can be a burden and present operational risks that can be mitigated with automation. Allow the Azure policy to continously evaluate your VMs and then act. For example, if the rule evaluates that the VM is out of compliance, a remediation task can be automatically generated to auto deploy the agent and associated rules.

For the AMA to provide you actionable data in Azure Monitor or Sentinel, it requires a **Data Collection Rule (DCR)** that must be created in the same resource group as the Azure Arc servers. This DCR defines what VM performance counters, Windows event logs or Linux syslog data gathered from the VM that should be sent to Azure and where it should be stored. For details on DCR rules, please visit here <https://docs.microsoft.com/en-us/azure/azure-monitor/agents/data-collection-rule-overview?WT.mc_id=modinfra-17603-pierrer&WT.mc_id=modinfra-21191-pierrer>. This method allows for granularity and lets you determine what logs your organization truly needs. This replaces the old feature where you had to ship the data based on a subset of categories (minimal, common, all), used with the Log Analytics agent.

These instructions will show you how to create an Azure Policy/Initiative that will continuously evaluate if your new Azure Arc VMs have the agent installed and the associated DCR. This will only showcase the Windows use case, but the JSON can be replicated, and resource values can be changed to Linux values to obtain similar telemetry on Linux machines. 

## Prerequisites
**Role Based Access**: Resource Policy Contributor, Owner or Contributor (_Be sure to only apply roles that provide you the least amount of privilege required for your role. Owner and Contributor provide heightened access and should be seldomly applied unless the user is an administrator of Azure or certain resource groups._) The most important role is the Resource Policy Contributor so that these policies can be constructed.

**Resource Properties**: At the subscription level, in the Resource Properities tab make sure "Microsoft.HybridCompute, Microsoft.GuestConfiguration" are registered. You will experience an error in the deployment of your ARC agent if this is not enabled.

**VM Access**: The easiest way I have found to test this policy following the instructions in the Azure Arc console is to install the ARC agent on an on-prem or test or dev server, residing in another cloud service. Another alternative would be to stand up a low scale VM on your workstation through Hyper-V. You can follow this video as a guide here <https://www.youtube.com/watch?v=KvYMjdwlC6E>. The Hyper-V scenario will require you to ensure the resource can communicate to the following site <https://docs.microsoft.com/en-us/azure/azure-arc/servers/agent-overview#networking-configuration> and TLS 1.2 is enabled; I found this to be a good site to auto-enable TLS, run the script via PS here <https://stackoverflow.com/questions/55914397/enable-tls-and-disable-ssl-via-powershell-script>. 

## Instructions
Make sure that the Azure Arc agent is running on the machine prior to assigning the policy. In order to enable the agent you can follow these instructions here<https://docs.microsoft.com/en-us/azure/azure-arc/servers/agent-overview>.

In Azure Policy, create a new Definition that will review the Azure Arc enabled server validate whether the agent is present and if it is not it will remediate automatically. Copy and paste the Azure-Arc-AMA-Deployment.json file into your new policy. There is no parameters that need to be elected for this policy but the remediation task should be created so it automatically applies the agent. If you forget this step, the policy will still evaluate the VM but not apply the AMA agent requiring manual intervention. 

Additionally, create a separate policy that will recognize if the associated data collection rules are applied to the Azure Arc enabled server. Copy and paste this AMA-DCR-Deployment.json file to your new policy definition. Under parameters you will need to declare the DCR Resource ID parameter. This allows the rule to associate the right DCR rule to this server. You can find this string by going to the DRC rule in the Azure Monitor blade, clicking on JSON view, copying the Resource ID and pasting it in the DCR Resource ID field back on your policy screen.

Once the two rules are created, you can collapse those into an initiative and assign the initiative to the Resource Group where the Azure Arc enabled servers lives. Or you can assign them individually to the resource group.

To validate, allow the policies to run over night or you can manually trigger a review by opening up the Azure CLI and running "az policy state trigger-scan".

## Validation
The AMA-Agent policy will evaluate the applicable resources contained in the resource group and then remediate any non-compliant resources. Once you see the resources registering as compliant, navigate to the Azure Arc server and view the extension tab. You should see the AzureMonitorWindowsAgent and ensure it has succeeded. 

Likewise, if the AMA-DCR rule is showing as compliant, query the Azure Monitor logs to verify the logs are collecting as expected. I usually run the following which will provide me validation the DCR rules are populating as ancitipated:

          HeartBeat
          | where Computer contains "xxxxx"
          
## Additions
If you would like to use the policy to enforce more granular control you can add in different parameters into the policy rule.  An example of this would be to include:
          
          {
             "field": "name",
             "like": "WIN*"
          },
          
In this instance, you can declare that the policy enforces when the name of the Azure Arc enabled server starts with "WIN". This way you can leverage more granular rules for your mission critical applications and more generic logging for your less ciritical applications allowing you to customize and optimize cost.
 
