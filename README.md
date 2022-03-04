# SIEM_SOC
# Supporting SOC infrastructure - Combining offensive security, defensive security, and network analysis

[comment]: # (this is the syntax for a comment)

This project is organized into the following sections and subsections:
- **Network Topology:** Live network environment
    - Host machine
    - Attacking machine
    - Vulnerable Web Server
    - Wordpress Target
    - ELK Server
- **Offensive Security:** Security assessment of vulnerable VM and verification of working Kibana rules.
    - Critical Vulnerabilities
    - Exploits Used
    - Avoiding Detection
    - Maintaining Access   
- **Defensive Security:** Creating and implementing alerts and thresholds
    - Alerts Implemented
    - Hardening
    - Implementing Policies
- **Network Analysis:** Network forensics and analysis
    - Traffic Profile
    - Normal Activity
    - Malicious activity
___

In this activity SIEM engineers conduct offensive security, defensive security, and network analysis acivities to provide the SOC team with a comprehensive security report. Approaching potential cybersecurity threats from these three perspectives is likely to provide a more complete threat analysis resulting in more effective mitigation strategies. 

## Network Topology

In this environment the ELK server is monitoring the vulnerable and taregeted machines as they are being attacked by the Kali machine. 
![image](https://github.com/duffian/SIEM_SOC/blob/c5adcec83f0fa95bcf9e85064ce7755635b05f36/images/proj3_topology.png)

## Offensive Security

The objective of the offensive SIEM engineering team is to identify critical vulnerabilities in the system and exploit them. Documention of these vulnerabilities is reported to the SOC team. 

**Critical Vulnerabilities**

Our assessment uncovered the following critical vulnerabilities

    - Poorly secured ssh port
    - SQL enumeration
    - Weak user password
    - No file security

![image](https://github.com/duffian/SIEM_SOC/blob/b9bad2b5e48bc897300f255d3772730ce87673ee/images/adn5.png)

**Exploitation - Poorly Secured SSH Port**

    - What tool or technique did you use to exploit the vulnerability?

Nmap port scanning.

`nmap -O 192.168.1.0/24`

![image](https://github.com/duffian/SIEM_SOC/blob/ba1a85cafab83e375e0afd9a6100861d9ea0c7aa/images/linux_nmapcommand.png)
![image](https://github.com/duffian/SIEM_SOC/blob/a10b171bf63d285966e70f6b8a8195b9f5c65c7a/images/nmapscanreport.png)
![image](https://github.com/duffian/SIEM_SOC/blob/a10b171bf63d285966e70f6b8a8195b9f5c65c7a/images/7_ssh.png)

    - What did the exploit achieve?

Unauthorized access to the Target 1 machine was achieved by using the unsecured ssh port identified on the vulnerable machine.
Identification of vulnerable ports to potentially gain unauthorized access to the "Target 1" system.

`ssh Michael@192.168.1.110`

![image](https://github.com/duffian/SIEM_SOC/blob/b1ad487330827e1fdd4a7fa298a8a63e6a605ca5/images/8_sshmichael.png)

**Exploitation - WordPress Susceptible to Enumeration**

    - What tool or technique did you use to exploit the vulnerability?

WPScan enumeration

`wpscan --url 192.168.1.110/wordpress -e u`

![image](https://github.com/duffian/SIEM_SOC/blob/0ec4844e2ccdd26a87831fc1e3ef47458b2cb65f/images/9_enumeration.png) 
    - What did the exploit achieve?

Acquisition of usernames associated with IP addresses.


![image](https://github.com/duffian/SIEM_SOC/blob/0ec4844e2ccdd26a87831fc1e3ef47458b2cb65f/images/10_enum.png)

**Exploitation - Weak User Password**

    - What tool or technique did you use to exploit the vulnerability?

Brute forcing passwords using hydra

`$ hydra -l michael -t 4 -P /usr/share/wordlists/rockyou.txt 192.168.1.110 ssh`


    - What did the exploit achieve?

Discovered the login password for user “michael” allowing ssh access into Target 1 Machine

![image](https://github.com/duffian/SIEM_SOC/blob/a89a67f44005945181d2897d97e6466921eec59a/images/11_hydra.png) 


**Exploitation - No File Security**

    - What tool or technique did you use to exploit the vulnerability?

Simple directory exploration.

    - What did the exploit achieve?

Privelege escalation: login data granted root access to Target 1 mysql data.

`michael@target1:/var/www/html/wp-config.php`

![image](https://github.com/duffian/SIEM_SOC/blob/a89a67f44005945181d2897d97e6466921eec59a/images/12_rootcreds.png)
![image](https://github.com/duffian/SIEM_SOC/blob/a89a67f44005945181d2897d97e6466921eec59a/images/13_pe.png)






**Avoiding Detection**

Identifying monitoring data and understanding how these data points might be used in mitigation helps to maintain unauthorized access, increase the duration of malicious activity, and avoid detection.

Monitoring - Identify alerts, metrics, and thresholds 

Mitigation - How can you execute the same exploit without triggering the alert? Are there alternative exploits that may perform better? 
 
![image](https://github.com/duffian/SIEM_SOC/blob/dcdcb2395afc40104128b2e63049f37ae94e1a84/images/15_introalerts.png) 

**Stealth Exploitation of Poorly Secured SSH Port**

    - Monitoring Overview
        - Alerts that detect this exploit:
		  - Port scanning traffic alerts
		  - Alerts monitoring for Unauthorized access through ssh port
        - Metric = `WHEN sum () of http.request.bytes OVER all documents`
        - Threshold = `IS ABOVE 3500 FOR THE LAST 1 minute`


    - Mitigation Detection
        - How can you execute the same exploit without triggering the alert? 
		  - Stealth scan
		  - Fast scan
		  - Limited-time scan
Stealth scan:
`nmap -sS`

![image](https://github.com/duffian/SIEM_SOC/blob/dcdcb2395afc40104128b2e63049f37ae94e1a84/images/16_nmap_stlth.png) 

Fast scan:
`nmap -F`
![image](https://github.com/duffian/SIEM_SOC/blob/dcdcb2395afc40104128b2e63049f37ae94e1a84/images/17_ssh_stlth.png)

**Stealth Exploitation of WordPress Susceptible to Enumeration**

    - Monitoring Overview
        - Alerts that detect this exploit:
		  - Alerts monitoring traffic from suspicious sources.
		  - Alerts monitoring traffic from non-white-listed IPs.
        - Metric = `WHEN count() GROUPED OVER top 5 ‘http.response.status_code`
        - Threshold = `IS ABOVE 400`

    - Mitigation Detection
        - How can you execute the same exploit without triggering the alert?
		  - Scan a WordPress site using random user agents and passive detection.
`wpscan --url 192.168.1.110/wordpress --stealthy -e u`
		

![image](images/adn19.png)
![image](https://github.com/duffian/SIEM_SOC/blob/dcdcb2395afc40104128b2e63049f37ae94e1a84/images/adn19.png)

**Stealth Exploitation of Weak User Password**

    - Monitoring Overview
        - Alerts that detect this exploit: CPU Usage Monitoring
		  - Alerts monitoring abnormal CPU usage according to time.
		  - Alerts monitoring abnormally high CPU usage.
        - Metric = `WHEN max() OF system.process.cpu.total pct OVER all documents`
        - Threshold = `IS ABOVE 0.5` (or norm for company)

    - Mitigation Detection
        - How can you execute the same exploit without triggering the alert? 
`$ hydra -l michael -t 4 P /usr/share/wordlists/rockyou.txt 192.168.1.110 ssh`
  - -t limits tasks per attempt
`$ hydra -l michael -w 5 P /usr/share/wordlists/rockyou.txt 192.168.1.110 ssh`
  - -w defines max wait time
  
![image](https://github.com/duffian/SIEM_SOC/blob/e65bdfaa5b51d75846e8a35e70dc7db5d75ab504/images/20_hydratasklimit.png) 
![image](https://github.com/duffian/SIEM_SOC/blob/e65bdfaa5b51d75846e8a35e70dc7db5d75ab504/images/21_hydrawaittimelimit.png)

**Stealth Exploitation of No File Security**

    - Monitoring Overview
        - Alerts that detect this exploit: Alerts monitoring traffic from;
		  - suspicious/malicious sources
		  - non-white-listed IPs
		  - unauthorized accounts
		  - root user logins
        - Metric = `user.name: root AND source.ip: 192.168.1.90 AND destination.ip: 192.168.1.110 OR event.action: ssh_login OR event.outcome: success`
        - Threshold = `IS ABOVE 1`

    - Mitigation Detection
        - How can you execute the same exploit without triggering the alert? 
		  - Remove evidence of instrusion/unauthorized access
		  - Mask source IP
		- Are there alternative exploits that may perform better? If attempts to elevate privileges to sudo are restricted and if root login data is secured with up-to-date password hashes, malicious actors will have a much more difficult time gaining root or elevated permissions.

![image](https://github.com/duffian/SIEM_SOC/blob/e65bdfaa5b51d75846e8a35e70dc7db5d75ab504/images/22_xtraalerts.png) 

**Maintaining Access**
Maintain access by writing a script to add an unauthorized user to the target system.



Python Script to Add User
`sudo python -c 'import.pty;pty.spawn("/bin/bash")'`
![image](https://github.com/duffian/SIEM_SOC/blob/1e64581adce910196643460e232d35489b2fee22/images/23_pythscrp_adduser.png) 
![image](https://github.com/duffian/SIEM_SOC/blob/1e64581adce910196643460e232d35489b2fee22/images/24_pythscrp_adduser.png) 
Could also write a script to install a backdoor shell listening for the attacker's instructions.












## Defensive Security

The objective is to configure Kibana alerts and make sure they are working correctly. Here we ensure the latest exploits and vulnerabilities are accounted for.

**Alerts Overview**

Identify the metric the alert monitors and the threshold that metric fires at.
![image]() [S28]

**Alert: CPU Usage Monitor**

![image]() [S29]

**Alert: Excessive HTTP Errors**

![image]() [S30]

**Alert: HTTP Request Size Monitor**

![image]() [S31]


**Hardening**
Explain how to patch the target against the vulnerabilities. Explain why the patch works and how to install the patch.

**Hardening Against on Target 1**

![image]() [S35]

**Hardening Against on Target 1**

![image]() [S36]

**Hardening Against on Target 1**

![image]()  [S37]














## Network Analysis

The objective is to analyze network traffic to identify suspicious or malicious activity.

**Traffic Profile and Behavioral Analysis**

![image]() [s39-40]

**Normal Activity - Web Traffic**

![image]() [s42-44]

**Normal Activity - DNS**

![image]() [s45-47]

**Malicious Activity**

    - Time Thieves

![image]() [s49]

    - Malicious File Download

![image]() [s50]

    - Vulnerable Windows Machines

![image]() [s51]

    - Illegal Downloads

![image]() [s52]











## Works Cited ##

![image]() [s56]







`THIS TEXT IS IN CODE FORMAT`


![image]()

