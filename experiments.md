# Experiments Conducted
## Configuring a Vulnerable Environment
To make it possible to perform attacks on the setup homelab, vulnerabilities needed to be established. 

![Version-1-Vulnerabilities](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-vulnerabilities.png)

The vulnerabilities setup (shown in the image above) are:
- Insecure RDP (On DC)
- No Encryption (On DC)
- Insecure SSH Passwords (On Security Server)
- Phished Credentials (On Linux Workstation)
- Insecure WinRM Service (On Windows Workstation)

### Setting up SSH
---
Running ```sudo apt install openssh-server -y``` installed the ssh server service, which I then started with ```sudo systemctl start ssh; sudo systemctl enable ssh```. I then changed UFW rules to allow SSH connections.

![Version-1-SSH-Setup](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-ssh-started.png)

Proof of this can be seen above.

![Version-1-SSH-Config](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-edit-ssh-config.png)

I then edited these two options in the SSH config file and restarted the service.

I then performed the same steps on the Linux Client, apart from allowing root login in the SSH config file.

### Setting up Email
I installed postfix on the linux client and edited the configuration to link it to the homelab network, create a directory for mail to be stored in, and setup an email address mapping to the janed account. 