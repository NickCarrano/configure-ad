<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setup Resources in Azure
- Ensure Connectivity between the client and Domain Controller
- Install Active Directory
- Create an Admin and Normal User Account in AD
- Join Client-1 to your domain (mydomain.com)
- Setup Remote Desktop for non-administrative users on Client-1


<h2>Deployment and Configuration Steps</h2>

<p>
<H3>Setup Resources in Azure</H3>

![image](https://github.com/user-attachments/assets/311e4c45-e7f7-49dc-bfa5-29319938c82f)

</p>
<p>

- Create a 'Windows Server 2022' virtual machine
  - This will be our Domain Controller
  - Name it DC-1
- Set DC-1's NIC private IP address to be static
- Create a 'Windows 10' virtual machine
  - This will be our Client
  - Name it Client-1
    - Be sure to use the same Resource Group and Vnet that were created when we created DC-1
- Ensure both virtual machines are in the same Vnet (This can be done with Network Watcher)

</p>
<br />

<p>
<h3>Ensure Connectivity between the client and Domain Controller</h3>

![Screenshot_203](https://github.com/user-attachments/assets/ca644e9f-f40e-4165-ab4c-b1c30ca22717)


</p>
<p>

- Login to Client-1 with Remote Desktop
    - Ping DC-1's private IP adress with 'ping -t (ip address)'
      - The ping should fail
- Login to DC-1 and enable ICMPv4 on the local windows firewall
  - Open 'Windows Defender Firewall with Advanced Security'
    - Go to 'Inbound Rules' on the left side
    - Sort the list by 'Protocol'
    - Enable both of the 'Core Network Diagnostics - ICMP Echo Request' rules that are listed as ICMPv4
- Return to Client-1 and see the ping has succeeded

</p>
<br />

<p>
<h3>Install Active Directory</h3>

![Screenshot_204](https://github.com/user-attachments/assets/3ba4ff87-b8a1-4a51-be55-c05bd38d8ad3)

</p>
<p>

- Login to DC-1 and install Active Directory Domain Services
  - In the Server Manager Dashboard click 'Add roles and features'
  - Continue through the wizard and when reaching the 'Server Roles' section, check the box next to 'Active Directory Domain Services'
  - Continue through the wizard and click 'Install' when prompted
- Premote DC-1 to a Domain controller
  - In the top right of the Server Manager Dashboard there should be a yellow notification icon
    - Click this and then click 'Premote this server to a domain controller'
  - In the wizard, check 'Add a new forest and choose your root domain name
    - We will be using mydomain.com but you can use any name, just remember what it is as well as the password
  - Continue through the wizard and once installation completes, restart DC-1 and log back in as (DomainName)\\(Username)
    - For us this will be mydomain.com\labuser
</p>
<br />
<p>
<h3>Create an Admin and Normal User Account in Active Directory</h3>

![Screenshot_205](https://github.com/user-attachments/assets/47fd20d2-c7f4-4b7d-b8da-fda15a507e08)


</p>
<p>

- Create an Admin user
  - In the uper right corner of the Server Manager Dashboard open the 'Tools' pulldown
  - Open 'Active Directory Users and Computers'
  - Open the 'mydomain.com' section
  - Create an Organizational Unit (OU) called '_EMPLOYEES'
  - Create another OU called '_ADMINS'
  - In the 'Users' section, create a new user named 'Jane Doe' with the username of 'jane_admin'
    - Use a memorable password or the same password you have been using up to this point
- Add jane_admin to the 'Domain Admins' security group
  - Right-click on Jane Doe and select 'Add to a Group'
  - In the field at the bottom type 'domain admins' and click 'Check names'
    - If done correctly the name should now be underlined
  - Click 'OK'
- Create a Normal User
  - In the 'Users' section, create a new user named 'John Doe' with the username of 'john_doe'
    - Use a memorable password or the same password you have been using up to this point
- Log out or close the Remote Desktop Connection to DC-1 and log back in as 'mydomain.com\jane_admin'
  - This will be our Admin account from now on
</p>
<br />

<p>
<h3>Join Client-1 to your Domain (mydomain.com)</h3>

![Screenshot_206](https://github.com/user-attachments/assets/5cc5f22c-972e-4ce7-9ca6-1ec5b34a8242)


</p>
<p>

- Set Client-1's DNS settings to DC-1's private IP address
  - In the Azure portal, navigate to Virtual Machines -> Client-1 -> Network Settings
  - Click on the 'Network Interface' section towards the top
  - Click the 'DNS Servers' section on the left side
  - Change DNS Servers from 'Inherit from Virtual Network' to 'Custom'
  - In the field, enter DC-1's Private IP adress and click 'Save'
  - Once the change is made, restart Client-1
- Join Client-1 to the Domain
  - Login to Client-1 through Remote Desktop Connection
    - Use the original local Admin to login (labuser)
  - Right-click on the start menu and go to 'System'
  - On the right side, select 'Rename this PC (Advanced)'
  - Click the button labeled 'Change'
  - Change the 'Member of' section from 'Workgroup' to 'Domain'
  - Enter the name of your domain (mydomain.com) and click 'OK'
  - Enter the full login info for the 'jane_admin' account and click 'OK'
    - The system will welcome you to the domain and ask you to restart your computer
- Verify Client-1 is visible from DC-1
  - Login to DC-1 and return to 'Active Directory Users and Computers'
  - Go to the 'Computers' section and make sure Clinet-1 is there

</p>
<br />

<p>
<h3>Setup Remote Desktop for Non-Administrative Users on Client-1</h3>

![Screenshot_207](https://github.com/user-attachments/assets/c6c79a66-5364-4612-80e9-99f71cdc5dd6)


</p>
<p>

- Log into Client-1 as 'mydomain.com\jane_admin' and open system properties
  - Click 'Remote Desktop' on the right
- Under 'User Accounts' click 'Select users that can remotely access this PC'
  - Click 'Add'
- Type 'domain users' in the field and click 'Check Names'
  - Click 'OK'
- You can now log into Client-1 as a non-administrative user
  - Test this by logging out of Client-1 and logging back in as 'mydomain.com\john_doe'
</p>
<br />
