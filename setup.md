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

![Version1 IP Config DC](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-config-ip-dc.png)

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

## Windows Enterprise Client
Next I setup a Windows Enterprise client using the same setup process as the Windows Server machine. 

The same as with the Windows Server setup process, I partitioned the disks using the default values, and chose to use the partition with the largest size.

Once the installation process was completed, I went into the network settings and set the machine's IP to 10.0.0.100, and I set the DNS server to 10.0.0.5 (the domain controller's IP).

![Version1 IP Config Win Client](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-config-ip-winclient.png)

### Joining Active Directory
Then I went into control panel and changed the workgroup name of the computer. I changed the computer name to project-x-win-client to adhere with the naming convention provided by the tutorial, and I changed the domain to corp.project-x-dc.com (the domain I set up for the DC). Once I did this, it prompted me to log in as a user, so I logged in as a user I previously made on the DC. Then I received a confirmation message showing I had connected to the DC. 

![Version1 Connect to DC Win Client](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-winclient-connect-to-dc.png)

Closing out of control panel prompted me to restart the computer for changes to take effect. Once it restarted, I logged in as johnd on the active directory login I made under the CORP domain.

![Version1 Login Win Client](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-winclient-login.png)

## Ubuntu Desktop Client
Next I setup an Ubuntu Desktop client using the same setup process as previous virtual machines, though the setup wizard differed as it is Ubuntu.

Once the installation process was complete, I went into network settings and set the machine's IP to 10.0.0.101 and the DNS server to 10.0.0.5 (the domain controller's IP).

![Version1 IP Config Linux Client](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-linuxclient-config-ip.png)


### Joining Active Directory
I then began installing Samba Winbind, a third party program that allows Ubuntu to connect to Windows Active Directory. I ran the command ```sudo apt -y install winbind libpam-winbind libnss-winbind krb5-config samba-dsdb-modules samba-vfs-modules``` which began installing the library and its dependencies, which prompted me to enter the domain "CORP.PROJECT-X-DC.COM".

I then edited the smb.conf file according to the tutorial:
```     
    [global]
       kerberos method = secrets and keytab
       realm = CORP.PROJECT-X-DC.COM
       workgroup = CORP
       security = ads
       template shell = /bin/bash
       winbind enum groups = Yes
       winbind enum users = Yes
       winbind separator = +
       idmap config * : rangesize = 1000000
       idmap config * : range = 1000000-19999999
       idmap config * : backend = autorid
```
And I edited the nsswitch.conf file to have

```
    passwd:         files systemd winbind
    group:          files systemd winbind
```

I went into the PAM configuration settings using ```sudo pam-update``` and cycled to the Create home directory on login option and set it to yes.

Then I added ```nameserver 10.0.0.5``` (the DNS server) into the resolv.conf file.

I joined the AD domain using the administrator account I set up on the DC.

![Version1 AD Join Linux Client](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-linuxclient-joinad.png)

I then logged into the AD using sudo login and entering the login credentials of the account I setup on the DC and successfully logged in.

![Version1 AD Login Linux Client](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-linuxclient-aduserlogin.png)

## Corporate Server
To save on setup time, I first cloned the Ubuntu Desktop Client machine.

As was done previously, I changed the IP address of the machine, but I left the subnet mask, default gateway and DNS address the same as these are static.

I changed the hostname of the machine using ```sudo hostnamectl set-hostname corp-svr```

I then created a new user using ```sudo adduser project-x-admin``` and added them to the sudoers group using ```sudo usermod -aG sudo project-x-admin```

Switching users shows the new user and hostname have been successfully been created.

![Version1 CorpSvr Login Created](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-corpsvr-login-created.png)

### Joining Active Directory
Most of the setup was completed when setting up the Ubuntu Desktop Client machine, so it is now as simple as joining the active directory as an admin and logging into to create a home directory.

![Version1 CorpSvr AD Connect](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-corpsvr-adconnect.png)

### Installing Docker
The commands used for installing Docker Engine are found on the page https://docs.docker.com/engine/install/ubuntu/

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

The command below is to run a docker container image to test that the installation was successful.
```
sudo docker run hello-world
```

### Setting up MailHog
I created a mailhog directory and then created a docker-compose.yml file and entered the following:

```
version: "3"
services:
  mailhog:
    image: mailhog/mailhog
    container_name: mailhog
    ports:
      - "1025:1025"
      - "8025:8025"
```

This script pulls and creates the mailhog container and maps the port 1025 (default SMTP port) to port 8025 (where the MailHog service will be hosted).

### Setting up Email Poller
I created a script email_poller.sh in the home directory of the Linux client which contained the following code:

```
#!/bin/bash

MAILHOG_IP="10.0.0.8"  
TO_EMAIL="janed"
POLL_INTERVAL=30  # seconds

echo "📡 Janed's Mail Watcher started... polling every $POLL_INTERVAL seconds"
echo "🔎 Watching for new mail sent to: $TO_EMAIL@"

# Keep track of seen message IDs
SEEN_IDS_FILE="/tmp/mailhog_seen_ids_janed.txt"
touch "$SEEN_IDS_FILE"

while true; do
  # Fetch current message list
  curl -s http://$MAILHOG_IP:8025/api/v2/messages | jq -c '.items[]' | while read -r msg; do
    TO=$(echo "$msg" | jq -r '.To[].Mailbox')
    ID=$(echo "$msg" | jq -r '.ID')

    if [[ "$TO" == "$TO_EMAIL" && ! $(grep -Fx "$ID" "$SEEN_IDS_FILE") ]]; then
      SUBJECT=$(echo "$msg" | jq -r '.Content.Headers.Subject[0]')
      BODY=$(echo "$msg" | jq -r '.Content.Body')

      echo -e "\n📬 New Email Received!"
      echo "Subject: $SUBJECT"
      echo "From: $(echo "$msg" | jq -r '.Content.Headers.From[0]')"
      echo "Date: $(echo "$msg" | jq -r '.Created')"
      echo -e "Message:\n$BODY"
      echo "-----------------------------------"

      echo "$ID" >> "$SEEN_IDS_FILE"
    fi
  done

  sleep "$POLL_INTERVAL"
done
```

After creating it, I ran the script in the background using ```./sudo email_poller.sh &```

I was able to confirm MailHog and the email poller were working together by using a python script to send an email to MailHog, which then appeared as a notification in the client's terminal as can be seen below:

![Version1 Client Email Test](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-client-emailtest.png)

## Security Workstation
I created the Security Onion virtual machine and ran through the setup wizard, creating the account I will use on the machine. Once the setup wizard had completed, I set the root password using ```sudo passwd root``` and typing my desired password into the prompts, and I then took a snapshot of the machine.

No further configuration was required for the time being.

## Security Server
Like with the Corporate Server, I cloned the Ubuntu Desktop client to reduce setup time.

I then performed the same steps I performed for the Corporate Server to setup a new user and change the IP address.

### Joining Active Directory
Joining the Active Directory was identical to the steps in the Corporate Server, but a sec-box account had to be created to log in to the Active Directory.

I went into the Active Directory Users and Computers menu and created SecUser:

![Version1 AD Creating SecUser](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-adcreating-secuser.png)

I then went into SecUser and created a Domain Group called **Domain Admins** and assigned SecUser to that group:

![Version1 AD Creating Domain Group](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-creating-addomaingroup.png)

I then logged into the new SecUser account on the AD successfully as it created a home directory on login:

![Version1 SecUser Login](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-secuser-login.png)

### Installing Wazuh (SIEM)
I first installed curl on the machine and then ran the following command to download and install Wazuh ```curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i```.

Once the installation process finished, I went to https://localhost to access the Wazuh dashboard. I logged in using the credentials generated during the install process.

![Version1 Wazuh Dashboard](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-wazuh-dash.png)

### Deploying a Windows Agent
To receive data from clients, agents needs to be deployed on the machines. To do this I went to Agents Management -> Summary in the menu on the Wazuh dashboard. I then entered the server's IP address and the name of the agent which generated the following code: ```Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.0-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='10.0.0.10' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='homelab-win-client'```. Running that code followed by ```NET START Wazuh``` on the Windows Client activated the Windows Agent.

![Version1 Wazuh WinAgent](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-wazuh-winagent.png)

I also repeated the same steps on the domain controller.

### Deploying a Linux Agent
To set up the Linux agent, I went to Agents Management -> Summary in the menu on the Wazuh dashboard. Like with the Windows client, I entered the server's IP address and the name of the agent which generated the following code: ```wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.0-1_amd64.deb && sudo WAZUH_MANAGER='10.0.0.10' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='homelab-linux-client' dpkg -i ./wazuh-agent_4.14.0-1_amd64.deb```. Running that code followed by ```sudo systemctl daemon-reload sudo systemctl enable wazuh-agent sudo systemctl start wazuh-agent``` on the Linux Client activated the Linux Agent.

![Version1 Wazuh LinuxAgent](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-wazuh-linuxagent.png)

### Managing Groups on Wazuh
Groups are useful to separate agents to create different configuration information for the different operating systems and have different logs and data collected depending on the operating system.

To do this I went to Agents Management -> Groups in the menu on the Wazuh dashboard. I then clicked Add New Group and made a 'Windows' group and a 'Linux' group. To then add the clients to the groups, I went to Agents Management -> Summary and clicked on the 3 dots on the right of each agent and clicked 'Edit Group'. I then assigned them to their appropriate groups depending on their OS.

![Version1 Wazuh Groups](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-wazuh-groups.png)

To edit the configuration information for each group I went to Agents Management -> Groups, and clicked on a group, starting with Windows. I then went to Files and clicked on the edit icon for the 'agent.conf' file. I edited it as follows:

![Version1 Wazuh Winconf](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-wazuh-winconf.png)

This sets up Wazuh to monitor the Windows Security and Application Event logs.

I performed the same steps to edit the 'agent.conf' file for the Linux group, with different contents as seen below:

![Version1 Wazuh Linuxconf](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-wazuh-linuxconf.png)

This sets up Wazuh to monitor the default Linux logs.

# Next Steps
Following the initial configuration and setup, I began to setup vulnerabilities and initiate attacks. This is detailed [here](https://github.com/gradyclark03/homelab-project/blob/main/experiments.md).