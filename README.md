<h1>Azure Sentinel Live Attack Demonstration Home Lab</h1>

<h2>Description</h2>
This is a walkthrough of how I used Microsoft Azure and created a virtual machine in the cloud running Windows 10. I exposed a VM to the internet and used Azure Log Analytics Workspace, Microsoft Defender for Cloud, and Azure Sentinel to collect and aggregate the attack data and display it on a map in Azure Sentinel. This project will showcase the use of a few different tools and resources. I will be using PowerShell to scan EventViewer in the exposed VM, specifically eventID 4625 which is failed logon attempts, and send that data to a logfile. The PowerShell Script also sends the IP address from any failed logons to IPgeolocation.io via an API, so later that information can be used Azure Sentinel to map where the logon attempts originated from. This project was done to gain experience with SIEMs, cloud concepts and resources, API's, and Microsoft Azure.I learned how to provision and configure resources in the cloud, how to read SIEM logs and much more. This was a fun project and I hope anyone reading this appreciates the work that went into this project.
<br />

<h2>Utilities Used</h2>

- <b>Azure Sentinel</b> 
- <b>Log Analytics Workbooks</b>
- <b>Microsoft Defender for Cloud</b>
- <b>Vitural Machines</b>
- <b>Remote Desktop</b>
- <b>PowerShell</b>
- <b>API's</b>
- <b>EventViewer</b>
- <b>Firewalls</b>

<h2>Environments Used </h2>

- <b>Microsoft Azure</b>
- <b>Windows 10</b> (21H2)

<h2 align="center">Program walk-through</h2>

<p align="center">
<b>The first thing I am going to do is create a Microsoft Azure account, this will be the cloud environment I'll use to provision my resources. I will take advantage of the $200 credit I'll receive to do this project. The resources I'm using are not very resource heavy, so my credit can be used towards future projects</b> <br/>
</p>

![Create_Azure_Subscription](https://user-images.githubusercontent.com/108043108/225348757-c41744df-2be1-4ffc-87a6-aa258a4102ef.JPG)
![Set_Up_Completed](https://user-images.githubusercontent.com/108043108/225348950-5d9c01fa-a707-4813-a6f3-012c703c41df.JPG)

<br />
<br />
<p align="center">
<b>The next thing I'll do is start the process of creating my virtual machines. Provisioning a VM can be a long process so while I move on to the next step, my VM can be provisioned in the background.</b> <br/>
</p>

![Create_VM](https://user-images.githubusercontent.com/108043108/225350437-f6fc6e29-6821-48d5-a379-2099477589ae.JPG)

<br />
<br />
<p align="center">
<b></b>At this point in the VM creation process I need to make sure that I create a new Resource Group that all of my future resources will be under. A resource group in Azure is a logical grouping of tools, services, configurations and more that exist under one banner so they can be created and deleted at the same time (they share the same lifespan). If I have resources outside of a particular resource group, if I delete that resource group the ones outside of it will still exist. It makes it easier to manage your resources if they're all in the same place. I decided to name this resource group "HoneyPot_Lab" and I name the virtual machine "HoneyPot-VM".
<br/>
</p>

![New_Resource_Group](https://user-images.githubusercontent.com/108043108/225352474-3b252a72-20a6-4a56-be2a-6405d4f79cc4.JPG)
![Create_VM_2](https://user-images.githubusercontent.com/108043108/225353275-82a1ee46-586a-4ce1-9697-440c57528b3c.JPG)

<br />
<br />
<p align="center">
<b>I scroll down on that same page and I now have to choose the size of the VM I'm going to provision. In the photo I initially chose Standard_B1s, as circled in green. Later on I decided to upgrade it to Standard_B2s which gave 2 CPU cores instead of 1 and more RAM. The first one I chose was just too slow. PowerShell kept crashing and the VM was lagging a lot. After choosing the size of the VM I create the Admin account. I chose a unique admin name and a 30 character password made up of special characters, numbers, and a mix of lowercase and uppercase letters. Since I knew people would be trying to log into the exposed VM, most likely through brute forcing and dictionary attacks a strong password was a necessity. The public inbound port rules, essentially which ports will be open to be able to connect to the VM. At this step you can set mutltiple authentication methods like SSH but I chose to only allow RDP.</b> <br/>
</p>

![Create_VM_3](https://user-images.githubusercontent.com/108043108/225365594-cf2fe158-d887-4bf9-aca6-d423019404a6.jpg)

<br />
<br />
<p align="center">
<b>The next step is to create a new Network Security Group (NSG). An NSG is basically a Firewall that can create and enforce rules on inbound and outbound traffic to Azure resources. For this project we don't want any rules on traffic. We want to allow anyone and everyone to be able to communicate with the honeypot VM. There is a default inbound rule, so we'll delete that one and create a new inbound rule that will allow EVERYTHING into the VM. On the Destination Port Ranges box I wrote an asterisk (*) for Anything. This will allow for any port ranges. We'll allow any protocol. For Priority, I set it at 100 because the lower the number the higher the priority. So if there is a rogue rule somewhere, this rule will have priority over it. With these rules set it will allow any and all traffic into our VM. I would NEVER EVER do this in a real production environment.</b> <br/>
</p>

![Networking_NSG_1](https://user-images.githubusercontent.com/108043108/225353983-66d6e530-783f-4db8-9720-23e575719b5f.JPG)
![Network_NSG_2](https://user-images.githubusercontent.com/108043108/225367854-f6c685bb-6e96-4981-8f20-2f3a300f1987.JPG)
![Network_NSG_3](https://user-images.githubusercontent.com/108043108/225367866-761bcc17-991d-4af1-8798-db7bca2acff4.JPG)
![Network_NSG_4JPG](https://user-images.githubusercontent.com/108043108/225367879-4aff3f39-e232-4404-aa7d-30bac028fc60.JPG)

<br />
<br />
<p align="center">
<b>I review, create, and deploy the VM as the last step. </b> <br/>
</p>

![Create_VM_4](https://user-images.githubusercontent.com/108043108/225372110-b0dcf0b3-b991-41d5-8fbd-3817cdc56b8e.JPG)
![Deploying_VM](https://user-images.githubusercontent.com/108043108/225372118-70617d94-0808-4fcd-9a27-0413066d471d.JPG)

<br />
<br />
<p align="center">
<b>While my VM is deploying, I can get started on setting up Log Analytics Workspace. These can all be found in the Azure home dashboard, or you can search for them in the search bar. When I create a Log Analytics Workspace, I make sure to put it in my HoneyPot_Lab resource group so it can be deleted when I delete that resource group. I name the instance LAW-HoneyPot.</b> <br/>
</p>

![Log_Analytics_Workspace](https://user-images.githubusercontent.com/108043108/225373030-1633fcf3-49cf-4a6f-afa7-33337a60c57c.JPG)
![Log_Analytics_Workspace_2](https://user-images.githubusercontent.com/108043108/225373041-89acebf7-1328-4b8a-96ee-6c1a946c7df0.JPG)

<br />
<br />
<p align="center">
<b>Now that the VM's firewall is disabled, I try to Ping it again from my native PC and this time it is successful.</b> <br/>
</p>

![Ping_works](https://user-images.githubusercontent.com/108043108/177887412-f8078e7b-13a3-4480-b000-5ae84108cfab.JPG)

<br />
<br />
<p align="center">
<b>By this time Nessus Essentials successfully downloads. The first thing I want to do is create a new scan, then select Basic Network Scan.</b> <br/>
</p>

![Create_New_Scan](https://user-images.githubusercontent.com/108043108/177887661-793476e8-7e68-4c21-8983-8f955d7cff2e.JPG)

![basic_network_scan](https://user-images.githubusercontent.com/108043108/177887672-2d955508-edf1-4735-b78a-2836a81d2c9f.JPG)


<br />
<br />
<p align="center">
<b>The newly created scan asks me to name the scan and select a target to scan. I configure it to scan the VM's IP address which is 192.168.50.185</b> <br/>
</p>

![Scan_VM_IP](https://user-images.githubusercontent.com/108043108/177887914-87a2da10-be51-481c-a672-ec1104e3df7a.JPG)

<br />
<br />
<p align="center">
<b>I launch the newly created scan and it immediately goes to work scanning for any known vulnerabilities. When the grey checkmark appears, the scan is complete.</b> <br/>
</p>

![Launch_newly_created_scan](https://user-images.githubusercontent.com/108043108/177888041-95c53001-b8d2-4355-9311-3ee2637dff94.JPG)

![Scan_Running](https://user-images.githubusercontent.com/108043108/177888049-661dddfc-ebe0-42cf-ad87-14bc5af53fe5.gif)

![Scan_Completed](https://user-images.githubusercontent.com/108043108/177888068-25d4a86a-c150-4e77-a993-9df4178c5ba1.JPG)

<br />
<br />
<p align="center">
<b>Lets look at the results! It is showing 33 results, 32 of which are info and 1 low. If this was an actual production environment these most likely would be left alone. The Info results are probably because some things don't have proper credentials and are not essentially vulnerabilities</b> <br/>
</p>

![Results_Of_Scan](https://user-images.githubusercontent.com/108043108/177888196-3141c52f-df79-4c58-9fee-5daa68d78c5b.gif)

<br />
<br />
<p align="center">
<b>Looking at one of the INFO results you can see that the Target Credential Status By Authentication Procotol was triggered because we did not actually provide any credentials for this scan.</b> <br/>
</p>

![No_credentials_Given](https://user-images.githubusercontent.com/108043108/177888648-5679c8c1-e8ce-47e3-a424-067375964054.JPG)

<br />
<br />
<p align="center">
<b>Next thing I do is configure the VM to be able to accept authenticated scans and provide credentials to Nessus. I will then rescan the VM and compare the results. I go to services.MSC to start this process and enable Remote Registry. This will allow Nessus to connect to the VM's registry and properly scan for vulnerabilities such as insecure connections or deprecated cipher suites. I'm following these steps from Nessus and what they recommend to actually do credentialed scans. There might be a better way to do this.</b> <br/>
</p>

![Services_MSC](https://user-images.githubusercontent.com/108043108/177888931-9dbd1224-5155-4129-ba09-4f495976a0e2.JPG)

![Enabling_Remote_Registry](https://user-images.githubusercontent.com/108043108/177889042-0a885e18-bf64-4390-8f43-454bdaf79004.gif)

<br />
<br />
<p align="center">
<b>From there I go to User Account Controls and disable it. I have to do this because this VM is not on a domain so I kind of have to do hacker stuff to get it to work properly. I would never do this in an actual organization or production environment.</b> <br/>
</p>

![user_account_Settings](https://user-images.githubusercontent.com/108043108/177889466-907890fc-e67e-490b-8a17-b335263d0359.JPG)

![User_Account_settings_configuration](https://user-images.githubusercontent.com/108043108/177889473-ba988135-ce62-4717-bf04-1e426ec8c4b6.gif)

<br />
<br />
<p align="center">
<b>Then I'm going to open the registry and add a key that is suppose to allow Nessus to connect in by further disabling user account controls.</b> <br/>
</p>

![Registry_editor](https://user-images.githubusercontent.com/108043108/177889663-24088363-2291-4af0-8ea9-6f6486fc82ad.JPG)

<br />
<br />
<p align="center">
<b>Now I navigate the Registry to the file that Nessus instructs us to (highlighted Yellow in the search bar) and I have to add a DWORD value and name it LocalAccountTokenFilterPolicy and give it a value of 1. </b> <br/>
</p>

![Creating_a_new_DWORD](https://user-images.githubusercontent.com/108043108/177889894-f94747fa-fa05-401f-82d4-1c9ec9faa57f.JPG)

![DWORD_name](https://user-images.githubusercontent.com/108043108/177889904-aa294428-0864-4018-8cb5-b61da5e65478.JPG)

![Edit_DWORD](https://user-images.githubusercontent.com/108043108/177889915-70e9814c-5167-460c-be85-10abf056fd8f.JPG)

<br />
<br />
<p align="center">
<b>After doing that I have to restart the VM so the changes can take effect.</b> <br/>
</p>

![Restart_The_VM](https://user-images.githubusercontent.com/108043108/177889955-2511a00a-9b59-4b4d-a9be-c2abda9869a6.JPG)

<br />
<br />
<p align="center">
<b>With the registry configured, it is now time to go back into Nessus and configure the scan I created. I have to add the Credentials to the scan so it can work properly. The credentials I'm talking about is the username and password of the VM. This will allow Nessus to use those credentials in places where it is required in the VM registry.</b> <br/>
</p>

![Configure_Nessus](https://user-images.githubusercontent.com/108043108/177890138-e6420231-4ce8-4a2d-99fb-e1e992da6ebb.JPG)

![Adding_Credentials](https://user-images.githubusercontent.com/108043108/177890151-27496a20-72b8-4763-8c82-368a06a15d3f.gif)

<br />
<br />
<p align="center">
<b>After the scan is properly configured with the right credentials, I run it again.</b> <br/>
</p>

![Run_The_Scan_Again](https://user-images.githubusercontent.com/108043108/177890377-b412d84a-d8e2-40da-9995-dddbd3c4d388.JPG)

<br />
<br />
<p align="center">
<b>This new scan has given us a lot more vulnerabilities than the first one because it is able to scan deeper into the VM due to having credentials. The top picture is the new credentialed scan and the bottom picture is from the first non-credentialed scan. Most of the vulnerabilities found is probably because the version of Windows 10 this VM is running is not up to date.</b> <br/>
</p>

![new_scan_properly_credentialed](https://user-images.githubusercontent.com/108043108/177890582-7f4d0eec-3b5f-4a5f-b708-47b3ccfb5d83.JPG)

![Old_scan_not_credentialed](https://user-images.githubusercontent.com/108043108/177890589-e4e882ab-6733-4930-ad4c-1115a2c620b3.JPG)

<br />
<br />
<p align="center">
<b>I want to see how powerful this Nessus Scanner is so I'm going to download a very old version of Firefox which probably has many vulnerabilities and see if Nessus can discover them (I'm sure it will.)</b><br/>
</p>

![downloading_an_old_version_of_firefox](https://user-images.githubusercontent.com/108043108/177890787-5cd80be3-99a1-40a4-bddc-feb06ea9cda7.JPG)

<br />
<br />
<p align="center">
<b>After a deprecated version of Firefox is downloaded, I run another scan. We can see many new alerts and vulnerabilities just from Firefox! 68 Critical!</b> <br/>
</p>

![Old_Firefox_Scan](https://user-images.githubusercontent.com/108043108/177890872-ed890db9-a15d-4de9-b3b3-6723dd161f22.JPG)

<br />
<br />
<p align="center">
<b>A comparison between scans to show the progression of alerts and vulnerabilities.</b>  <br/>
</p>

![Vulnerability_Comparison](https://user-images.githubusercontent.com/108043108/177890918-c2fc7078-34b9-4120-a430-f096169ae177.jpg)

<br />
<br />
<p align="center">
<b>Showing what some of the alerts and vulnerabilites look like. We can see most of the Critical alerts are just from Firefox. A few ways we can remediate some of the vulnerabilities is by either Updated Firefox, which will probably remediate a lot of them, or we can simply delete Firefox.</b>  <br/>
</p>

![Showing_Vulnerabilities](https://user-images.githubusercontent.com/108043108/177891162-cf86e335-0539-4f9f-abb2-7150b482e689.gif)

<br />
<br />
<p align="center">
<b>To start the process of remediating vulnerabilities, I elect to just delete Firefox. That will instantly fix a lot of these issues.</b>  <br/>
</p>

![Fixing_Vulnerabilities_Uninstall_Firefox](https://user-images.githubusercontent.com/108043108/177891363-17bdc0ea-c872-434f-95a6-4bf6e4694ddf.JPG)

<br />
<br />
<p align="center">
<b>To remediate the Windows vulnerabilities, I choose to update Windows. This version is old so it takes a few restarts to get it up to date.</b>  <br/>
</p>

![Remediating_Vulnerabilities_Updating_WIndows_1](https://user-images.githubusercontent.com/108043108/177891464-75d2b603-cadf-4923-bc1f-61409b11169d.gif)

![Remediating_Vulnerabilities_Updating_Windows](https://user-images.githubusercontent.com/108043108/177891436-a7b26e21-8376-4650-aaee-aaf4bc2e451e.JPG)

<br />
<br />
<p align="center">
<b>After a few restarts, Windows is finally up to date. I run one more Nessus scan to find if the steps I took to remediate some of the alerts worked.</b>  <br/>
</p>

![VM_Up_To_date](https://user-images.githubusercontent.com/108043108/177891501-767d6764-8e4a-4bd0-85cc-6b70e48c62aa.JPG)

<br />
<br />
<p align="center">
<b>Here is a final comparison between the four scans I took while doing this lab. The last picture is the after remediation scan. There we can see a lot of the vulnerabilities that were being alerted are gone! Still a 1 critical but I'll remediate that another time!</b>  <br/>
</p>

![After_Vulnerability_Remediation](https://user-images.githubusercontent.com/108043108/177891719-3066c9bb-38dd-4b10-b8bb-6f9dd8a45a09.JPG)

<br />
<br />


<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
