# Version1 Architecture
Following Grant Collin's course on Project Security on building a Cybersecurity Homelab, the homelab will follow the structure pictured below.

![Version1 Architecture](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-architecture.png)

The 6 machines involved in the homelab are:
- Directory Services Server
- Corporate Server (Server on Left of Diagram)
- 2 Enterprise Workstations
- Security Workstation
- Security Server

### Directory Services Server
This server will be responsible for running the Active Directory (AD) for the workstations in the network, which is known as a Domain Controller. It will also run a DNS and DHCP server for the network. It will run on **Windows Server 2025**.

### Corporate Server
This server will be responsible for running services within the simulated business of the network, such as SMTP or FTP. It will run on **Ubuntu Desktop 22.04**.

### 2 Enterprise Workstations
One machine will be a **Windows 11** workstation and the other will be an **Ubuntu Desktop 22.04** workstation. They will act as employees of the business in the network and will interact with the services provided by the Corporate Server. They will be monitored by the Security Server.

### Security Workstation
This workstation will run on **Security Onion 2.4** and will provide a variety of tools for network security monitoring, log management and intrusion detection. These tools are Zeek, Suricata, and Elastic Stack. This machine will monitor the Enterprise Workstations and interact with the Security Server.

### Security Server
This server will run on **Ubuntu Server 22.04** and will interact with the Security Workstation to process and respond to events occurring on the network to ensure the security of connected clients.

## Hardware
For this homelab, I am using a spare computer which contains an Intel I7-7700K which is a Quad core processor, as well as 24GB of 2133MHz RAM. It also contains a 1TB HDD. The following minimum specs for each VM are provided by Grant Collins so I will be using them throughout the project:

![Version1 Minimum VM Specs](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/min-specs.png)