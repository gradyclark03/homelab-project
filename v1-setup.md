# Version1 Setup
Setting up the homelab will involve the creation of 6 virtual machines which will interact with each other as shown in the architecture diagram below.

![Version1 Architecture](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-architecture.png)

The first step was to download all of the ISO files required for the virtual machines which can be seen below.

![Version1 ISO Files](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-ISOs.png)

I then configured a NAT network in VirtualBox and enabled DHCP.

## Directory Services Server
### Creating VM and Initializing Domain Controller
I began by configuring the Directory Services Server.

I created a new Windows Server 2022 machine on VirtualBox and gave it the following settings:

![Version1 DC HW Settings](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/dc-hardware-config.png)

Following this, I booted up the VM and went through the Windows Setup wizard.

According to the tutorial, it is suggested to create a new partition and use the default size (51198MB in my case). This created a 100MB partition which split off from the existing 50GB partition.

Once the user was setup and I was in Windows Server, I set the IP to 10.0.0.5 through control panel:

![Version1 IP Config](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-config-ip.png)

I then added Active Directory Domain Services, DHCP and DNS Servers to the Windows Server:

![Version1 Add AD](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-add-ad.png)

I received a prompt to setup the domain controller and created the root domain name as can be seen below:

![Version1 DC Creation](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-dc-creation.png)

The machine restarted and applied changes and once I was brought to the login screen, the CORP prefix appeared, indicating that Active Directory was working and the Domain Controller was setup:

![Version1 DC Creation](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-ad-works.png)

### Initializing DNS and DHCP
I navigated to the DNS Manager from the Server Manager and created a forwarder for my server to google's DNS server (8.8.8.8) which gives the server access to the internet:

![Version1 DNS Init](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-dns-add-google.png)

I navigated to the DHCP manager from the Server Manager and created a new IPv4 scope:

![Version1 DHCP Setup 1](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-dhcp-config1.png)

I set the number of IPs to be leased to be 100, so from 10.0.0.100 to 10.0.0.200, and set the subnet mask to 24:

![Version1 DHCP Setup 2](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-dhcp-config2.png)

I then committed and activated the DHCP service.

Once these were setup, I navigated to Active Directory Users and Computrs from the Tools dropdown in the top right of the Server Manager and created two users for the workstation machines.
