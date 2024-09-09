# Azure Sentinel Map with Live Attacks
This is a project to set up a honeypot in order to collect attack log data, query it in a SIEM and display it on a map.\
![image](https://github.com/Tala1122/AzureSentinelAttackMap/blob/main/img1.png)\
Steps:
<li>1. Create a Microsoft Azure Account</li>
<li>2. Create a Virtual Machine</li>
<li>3. Create a Log Analytics Workspace</li>
<li>4. Configure Microsoft Sentinel</li>
<li>5. Disable the Firewall in the Virtual Machine</li>
<li>6. Scripting the Security Log Exporter</li>
<li>7. Create Custom Logs</li>
<li>8. Extract Fields from Custom Logs</li>
<li>9. Map Data in Microsoft Sentinel</li>

**Create a Microsoft Azure Account**\
You can create a Free Microsoft Azure Account where you get $200 for 30 days.\
**Create a Virtual Machine**\
You can then create a virtual machine and a new resource group called "honeypotlab". You should also create a user and password for the VM and allow RDP (port 3389) as a public inbound port. In network settings, delete the existing rule and create a new inbound rule with:
<li>Destination port ranges: *</li>
<li>Protocol: Any</li>
<li>Action: Allow</li>
<li>Priority: 100</li>

**Create a Log Analytics Workspace**
You can then create a logs analytics workspace and put it in the same resource group as the VM. Then look up "Microsoft Defender in the Cloud". And adjust the following settings:
<li>Cloud Security Posture Management: ON</li>
<li>Servers: ON</li>
<li>SQL servers on machines: OFF</li>

**Configure Microsoft Sentinel**
You can then create Microsoft Sentinel and select the Log Analytics workspace name we created: honeypot-log.

**Disable the Firewall in the VM**
You can log into the VM using the established username and password over RDP. Then you can try to ping the VM from your local machine, you will see that pings time out. To avoid that, look up the Firewall in the VM and turn the Firewall State to OFF for Domain Profile, Private Profile and Public Profile. Then you can see that pings will go through.

**Scripting the Security Log Exporter**
In the VM open Powershell ISE and copy this Powershell [script](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1). Make an account with ipgeolocation.io and get an API key that you can put into the script. After that you can run the script and start to see attempted logins that are coming in. 

**Create Custom Logs**
Open file named "failed_rdp" on the VM, and paste its contents on your own device. In Azure, go to the log analytics workspace and create custom logs. Add the file that you just created and then pick the collection path: "C:\ProgramData\failed_rdp.log". After that run a query to see the available data (FAILED_RDP_WITH_GEO_CL).

**Extract Fields from Custom Logs**
Right click on any of the log results then select Extract fields from 'FAILED_RDP_WITH_GEO_CL'. Highlight the value you want and then name the field and select the appropriate data type. Do this for the latitude, longitude, destination-host, username, source-host, state, country and timestamp.

**Map Data in Microsoft Sentinel**
Add a workbook in Microsoft Sentinel, add the following query and then run it. 
```KQL
FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
| where destinationhost_CF != "samplehost"
| where sourcehost_CF != ""
```
You should then see results, you can then click the Visualization dropdown menu and set up the following Map settings:
Layout Settings
- Location info using > Latitude/Longitude
- Latitude > latitude_CF
- Longitude > longitude_CF
- Size by > event_count
Color Settings
- Coloring Type: Heatmap 
- Color by > event_count
- Aggregation for color > Sum of values
- Color palette > Green to Red
Metric Settings
- Metric Label > label_CF
- Metric Value > event_count

Save as "Failed RDP World Map" in the same region and under the resource group: honeypotlab.

![map](https://github.com/Tala1122/AzureSentinelAttackMap/blob/main/img2.png)
