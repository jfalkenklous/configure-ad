<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2025
- Windows 10

<h2>High-Level Deployment and Configuration Steps</h2>

- Create Resource Group and Virtual Network in Azure
- Create VM using Windows Server (DC-1)
- Create VM Using Windows 10 (Client)
- Ensure both VM's are connected to the same VNet and configure IP of DC-1 to be static

<h2>Deployment and Configuration Steps</h2>

<p>
<img width="1910" height="905" alt="image" src="https://github.com/user-attachments/assets/cc012f6e-3ae2-4406-b34a-244dfdaf99ec" />
</p>
<p>
We will deploy two virtual machines on the same virtual network (VNet): a Windows Server 2025 VM named DC-1 and a Windows 10 VM named Client. After provisioning, we’ll configure DC-1 with a static IP address instead of a dynamic one. This allows Client to join the domain reliably and use DC-1 as its DNS server.
</p>
<br />

<p>
<img width="1047" height="776" alt="image" src="https://github.com/user-attachments/assets/c2ee86c5-c37b-46fa-a8e0-36313150e011" />
</p>
<p>
Next, we’ll connect to the domain controller via RDP and disable the firewall for the Domain, Private, and Public profiles. To access the firewall settings, right-click the Start menu, choose Run, and enter wf.msc. In the Windows Defender Firewall console, confirm that the firewall is turned off for all profiles.
</p>
<br />

<p>
<img width="1422" height="649" alt="image" src="https://github.com/user-attachments/assets/9eba97e3-5160-44f5-80bb-e513515e17bf" />
</p>
<p>
Next, we’ll update the DNS settings on Client, replacing its current DNS server with the static private IP address assigned to DC-1. This is done through the networking settings in the Azure portal. Once updated, restart both virtual machines to ensure the new DNS configuration is applied.
</p>
<br />

<p>
<img width="658" height="668" alt="image" src="https://github.com/user-attachments/assets/723a3af7-83c3-4bc9-9882-12c1d08ac8ee" />
</p>
<p>
Next, we’ll RDP into the Client virtual machine and use PowerShell to ping DC-1 by its IP address. This should succeed now that the firewall on DC-1 is disabled and can respond to ICMP requests. We’ll then run ipconfig /all on Client to verify that DC-1 is correctly set as the VM’s DNS server.
</p>
<br />

<p>
<img width="1147" height="772" alt="image" src="https://github.com/user-attachments/assets/32ed0d74-3908-4a0e-a3d3-3a24dcafe9c9" />
</p>
<p>
Next, we’ll use Server Manager on the domain controller to install the Active Directory Domain Services (AD DS) role.
</p>
<br />

<p>
<img width="1263" height="745" alt="image" src="https://github.com/user-attachments/assets/b68d25c5-1c8e-4906-8fbb-b810037eff94" />
</p>
<p>
We’ll then promote DC-1 to a domain controller and create a new forest using the domain name mydomain.com.
</p>
<br />

<p>
<img width="785" height="603" alt="image" src="https://github.com/user-attachments/assets/4e196953-df42-49e6-a141-a945e7310d45" />
</p>
<p>
Next, we’ll open Active Directory Users and Computers and create two organizational units (_EMPLOYEES and _ADMINS). To do this, right-click mydomain.com, select New, and choose Organizational Unit.
</p>
<br />

<p>
<img width="869" height="688" alt="image" src="https://github.com/user-attachments/assets/5ae76c12-ea35-4768-838c-c075694b9753" />
</p>
<p>
In the _ADMINS OU, we’ll create a new user named Jane Doe with the username jane_admin.
</p>
<br />

<p>
<img width="1374" height="860" alt="image" src="https://github.com/user-attachments/assets/150d3e31-d4ac-47e0-805e-ca20930ecaf8" />
</p>
<p>
Next, we’ll grant Jane Doe Domain Admin privileges. To do this, right-click the account, open Properties, go to the Member Of tab, and select Add. In the “Enter object names” field, type Domain Admins and press Enter to finalize the change.
</p>
<br />

<p>
<img width="396" height="484" alt="image" src="https://github.com/user-attachments/assets/0477859a-53a4-4efb-8d68-e07ad1dd909b" />
</p>
<p>
Now, sign out of DC-1 and reconnect via RDP using the credentials mydomain.com\jane_admin along with the assigned password. This account will be used for all future logins to DC-1.
</p>
<br />

<p>
<img width="1852" height="864" alt="image" src="https://github.com/user-attachments/assets/efc58335-f717-42e4-8bfb-93d54bc7c895" />
</p>
<p>
Next, we’ll join Client to the domain. RDP into the VM, right-click the Windows logo, select System, and click Rename this PC (Advanced). Then, click Change, choose Domain, and enter the domain name mydomain.com. The VM will restart to apply the changes.
</p>
<br />

<p>
<img width="1223" height="784" alt="image" src="https://github.com/user-attachments/assets/478dc414-284f-4c79-9ed6-8b17b34740f1" />
</p>
<p>
Return to DC-1 and open Active Directory Users and Computers. Create a new organizational unit named _CLIENTS under mydomain.com.
</p>
<br />

<p>
<img width="1262" height="941" alt="image" src="https://github.com/user-attachments/assets/ab4235f8-fe6c-41b3-bff1-14a4db26ac15" />
</p>
<p>
Next, log into Client as jane_admin. Right-click the Windows logo, select System, and then choose Remote Desktop. Click Select users that can remotely access this PC and add Domain Users to grant them RDP access to the VM.
</p>
<br />

<p>
<img width="1023" height="952" alt="image" src="https://github.com/user-attachments/assets/bc7ac74b-039e-470b-a09c-38f59830732d" />
</p>
<p>
Log into DC-1 as jane_admin and open PowerShell ISE with administrative privileges. Create a new file, paste the script from this link (https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1) into it, and run the script. Watch as the user accounts are created automatically.
</p>
<br />

<p>
<img width="1077" height="964" alt="image" src="https://github.com/user-attachments/assets/ed42ec01-1c55-4f0f-ae1b-d77d15bcf391" />
</p>
<p>
After executing the script, open Active Directory Users and Computers (ADUC) to verify that the user accounts have been created within the _EMPLOYEES organizational unit.
</p>
<br />

<p>
<img width="1088" height="773" alt="image" src="https://github.com/user-attachments/assets/90d11f16-37d8-4905-a609-9b4e125a705a" />
</p>
<p>
Finally, log into the Client VM using one of the user accounts generated by the PowerShell script, entering the username and the default password Password1. Once logged in, open PowerShell to confirm that you are signed in as one of the script-created users.
</p>
<br />
