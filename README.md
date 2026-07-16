# SPLUNK-ENTERPRISE-DEPLOYMENT
A hands-on Splunk SIEM home lab focused on security monitoring, Windows event log analysis, threat detection, and incident response. This project demonstrates the deployment of Splunk Enterprise, log collection, security event investigation, and the identification of suspicious activities within a virtualised environment.


##  Virtual Security Lab Project


Table of Contents

   1. Introduction

   2. Lab Architecture Overview

   3. Ubuntu VM Preparation

   4. Splunk Enterprise Installation

   5. Splunk Receiving Configuration

   6. Windows Server (Win-DC) Forwarder Setup

   7. Windows Client (Client-PC) Forwarder Setup

   8. pfSense Syslog Integration

   9. Testing & Validation

   10. Troubleshooting & Lessons Learned

   11. Conclusion

   12. Appendix (Configs & Credentials)

# 1. Introduction
### Project Overview

This project documents the deployment of Splunk Enterprise as a Security Information and Event Management (SIEM) solution within a virtualized home lab environment. The primary objective was to centralize log collection from multiple endpoints and network infrastructure components for comprehensive security monitoring and analysis.

### Project Goals

| Goal                        | Description                                          | Status     |
| --------------------------- | ---------------------------------------------------- | ---------- |
| Deploy Splunk Enterprise    | Install and configure Splunk on Ubuntu Server        | ✅ Complete |
| Configure Log Collection    | Set up receiving ports for Windows event logs        | ✅ Complete |
| Deploy Universal Forwarders | Install and configure forwarders on Windows machines | ✅ Complete |
| Integrate pfSense Logs      | Forward firewall syslog data to Splunk               | ✅ Complete |
| Test & Validate             | Verify log ingestion, searches, and event visibility | ✅ Complete |
| Document Project            | Create comprehensive deployment documentation        | ✅ Complete |



# 2. Lab Architecture Overview
- Lab Environment Overview

The lab environment consists of multiple virtual machines running on Oracle VirtualBox, interconnected through a pfSense firewall on an internal network.


###  Tools & Technologies Used

| Component                            | Technology                 | Version     |
| ------------------------------------ | -------------------------- | ----------- |
| Virtualization Platform              | Oracle VirtualBox          | 7.x         |
| SIEM Platform                        | Splunk Enterprise          | 10.4.1      |
| Operating System (Splunk Server)     | Ubuntu Server              | 22.04 LTS   |
| Operating System (Domain Controller) | Windows Server             | 2019 / 2022 |
| Operating System (Client Endpoint)   | Windows 10/11              | -           |
| Firewall                             | pfSense                    | 2.7.x       |
| Log Forwarder                        | Splunk Universal Forwarder | 9.x         |




### Network Topology

The lab is built on Oracle VirtualBox, utilizing an internal network (192.168.1.0/24) managed by a pfSense firewall acting as the gateway.

Figure 1: Virtual Lab Network Architecture

                  ┌─────────────────────────────────────────────────────────────────────┐
                  │                    VIRTUAL LAB NETWORK ARCHITECTURE                 │
                  │                                                                     │
                  │  ┌─────────────────────────────────────────────────────────────┐    │
                  │  │                   pfSense Firewall                          │    │
                  │  │                   IP: 192.168.1.1                           │    │
                  │  │                                                             │    │
                  │  │  ┌──────────────────────────────────────────────────────┐   │    │
                  │  │  │              INTERNAL NETWORK (192.168.1.0/24)       │   │    │
                  │  │  └──────────────────────────────────────────────────────┘   │    │
                  │  └──────────────────────┬──────────────────────────────────────┘    │
                  │                         │                                           │
                  │          ┌──────────────┼──────────────────┬──────────────────┐     │
                  │          │              │                  │                  │     │
                  │          ▼              ▼                  ▼                  ▼     │
                  │  ┌────────────┐ ┌────────────┐  ┌────────────┐  ┌────────────┐      │
                  │  │  UBUNTU    │ │  WIN-DC    │  │  Client-PC │  │  Kali      │      │
                  │  │  Splunk    │ │  Domain    │  │  Windows   │  │  Linux     │      │
                  │  │  Server    │ │ Controller │  │  Endpoint  │  │  Attacker  │      │
                  │  │            │ │            │  │            │  │            │      │
                  │  │ IP: .109   │ │ IP: .101   │  │ IP: .103   │  │ IP: .105   │      │
                  │  │ Ports:     │ │  ┌──────┐  │  │  ┌──────┐  │  │            │      │
                  │  │  8000(Web) │ │  │UF v9 │  │  │  │UF v9 │  │  │            │      │
                  │  │  9997(Recv)│ │  └──┬───┘  │  │  └──┬───┘  │  │            │      │
                  │  │  5514(Sys) │ │     │      │  │     │      │  │            │      │
                  │  └─────┬──────┘ └─────┼──────┘  └─────┼──────┘  └────────────┘      │
                  │        │               │                 │                          │
                  │        │    ┌──────────┴─────────────────┴──────────┐               │
                  │        │    │   LOG DATA FLOW (Port 9997)           │               │
                  │        └────►   Windows Event Logs + Sysmon Logs    │               │
                  │             └───────────────────────────────────────┘               │
                  │                                                                     │
                  │             ┌───────────────────────────────────────┐               │
                  │             │   SYSLOG FLOW (Port 5514)             │               │
                  │             │   pfSense Firewall Logs               │               │
                  │             └───────────────────────────────────────┘               │
                  └─────────────────────────────────────────────────────────────────────┘



### IP Address Plan
            
| Device            | Hostname                  | IP Address    | Role                                 |
| ----------------- | ------------------------- | ------------- | ------------------------------------ |
| Firewall          | pfSense                   | 192.168.1.1   | Gateway / Firewall                   |
| Splunk Server     | Ubuntu-Splunk-SIEM-Server | 192.168.1.109 | Splunk Enterprise SIEM               |
| Domain Controller | WIN-DC.lab.local          | 192.168.1.101 | Windows Server / Log Source          |
| Client Machine    | CLIENT-PC                 | 192.168.1.103 | Windows Endpoint / Log Source        |
| Attacker Machine  | Kali Linux                | 192.168.1.105 | Security Testing / Attack Simulation |




# 3. Ubuntu VM Specifications

The Splunk Enterprise server runs on Ubuntu Server 22.04 LTS, provisioned as a virtual machine within the Oracle VirtualBox environment. The following specifications were carefully selected to ensure optimal performance while maintaining efficient resource utilization within the lab environment.


| Setting          | Value                     |
| ---------------- | ------------------------- |
| Operating System | Ubuntu Server 22.04 LTS   |
| RAM              | 8 GB                      |
| Storage          | 40 GB                     |
| Network Adapter  | Internal Network (LabLAN) |
| Hostname         | Ubuntu-Splunk-SIEM-Server |


### Static IP Configuration

A static IP was configured to ensure the Splunk server is always reachable.

Network Configuration File: /etc/netplan/50-cloud-init.yaml
      
      
      
    ```yaml
    network:
      ethernets:
        enp0s3:
          dhcp4: false
           addresses:
            - 192.168.1.109/24
          gateway4: 192.168.1.1
          nameservers:
            addresses:
              - 8.8.8.8
              - 1.1.1.1
      version: 2
    ```

### Commands Executed:

      sudo netplan apply
      ip a
### System Updates

    sudo apt update
    sudo apt upgrade -y

### Guest Additions Installation

      # Insert Guest Additions CD from VirtualBox menu
      sudo ./VBoxLinuxAdditions.run
      sudo reboot

Figure 2: Ubuntu Static IP Verification
<img width="1290" height="787" alt="ip a Output" src="https://github.com/user-attachments/assets/b34c1d36-33c1-4ea1-a978-4aaf02276ee5" />




    Terminal showing ip a command output

    Confirm IP: 192.168.1.109/24

    Confirm interface: enp0s3


# 4. Splunk Enterprise Installation
 ### Download Splunk Enterprise

Download Details:

    Version: Splunk Enterprise 10.4.1

    Package: .deb (Debian/Ubuntu compatible)

    Source: https://www.splunk.com/en_us/download/splunk-enterprise.html

### Download Command:


    cd ~/Downloads
    
    wget -O splunk-10.4.1-5a009d941268-linux-amd64.deb \
    "https://download.splunk.com/products/splunk/releases/10.4.1/linux/splunk-10.4.1-5a009d941268-linux-amd64.deb"
    ```

### Installation Process

        # Install curl dependency
          sudo apt install curl -y
        
        # Install the .deb package
        sudo dpkg -i splunk-*.deb


### First-Time Startup

      # Start Splunk with license acceptance
      sudo /opt/splunk/bin/splunk start --accept-license --answer-yes
Administrator credentials were created during the initial setup.

### Enable Boot-Start

To ensure Splunk Enterprise starts automatically whenever the Ubuntu server boots, the boot-start feature was enabled. This allows the SIEM platform to remain available after system restarts without requiring manual intervention.

Command Executed:

      sudo /opt/splunk/bin/splunk enable boot-start




### Verify Installation

| Component      | Status     | Verification                         |
| -------------- | ---------- | ------------------------------------ |
| Splunk Process | Running    | `sudo /opt/splunk/bin/splunk status` |
| Web Interface  | Accessible | `http://192.168.1.109:8000`          |
| Service        | Enabled    | Boot-start configured                |


Figure 4: Splunk Web Interface Login Page

<img width="1077" height="640" alt="Splunk WebUI" src="https://github.com/user-attachments/assets/e192e06c-6013-44ab-8dc6-6b4b097ba975" />
<img width="1057" height="657" alt="SplunkDASHBOARD" src="https://github.com/user-attachments/assets/6a12ec18-288b-45f8-8964-2c8039f3f840" />


#  5. Splunk Receiving Configuration
### Enable Port 9997 for Forwarder Traffic

Navigation:
Settings → Forwarding and receiving → Configure receiving → Add new

Configuration Details:

    Port: 9997

    Description: Universal Forwarder receiving port

### Verify Receiving Port
Command: 

    sudo netstat -tulpn | grep 9997

Output:

    tcp 0 0 0.0.0.0:9997 0.0.0.0:* LISTEN 1234/splunkd

### Configure Syslog Receiving Port

Navigation:
Settings → Data inputs → UDP → New

- Configuration Details:


## 📥 Syslog Input Configuration

> **Port:** `5514`  
> **Source Type:** `syslog`  
> **Index:** `main`  
> **Host:** Source IP Address


Note: Port 5514 was used instead of the privileged port 514 to avoid permission issues.

Figure 5: Syslog UDP Input Configuration
<img width="1292" height="785" alt="Splunk DATAinput 5514" src="https://github.com/user-attachments/assets/db07ff97-2dd5-4ac5-807c-bc08271a0e91" />

# 6. Windows Server (Win-DC) Forwarder Setup
## Download Universal Forwarder

    Download Source: https://www.splunk.com/en_us/download/universal-forwarder.html

    Package: Windows .msi (64-bit)

    Version: 9.x

### Installation Process

Installation Steps:

    Run the downloaded .msi file

    Accept license agreement → Clicked Next

    Set credentials:
                     Username:
                     
                     Password
Deployment Server: Left blank

Receiving Indexer: 192.168.1.109:9997

Complete installation

### Configure inputs.conf

File Location:
text

C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf

### Configuration Content:

      [WinEventLog://Application]
      index = endpoint
      disabled = false
      
      [WinEventLog://Security]
      index = endpoint
      disabled = false
      
      [WinEventLog://System]
      index = endpoint
      disabled = false
      
      [WinEventLog://Microsoft-Windows-Sysmon/Operational]
      index = endpoint
      disabled = false
      renderXml = true
      source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

### Restart Forwarder Service

Method  - GUI:

   Open Services (services.msc)

   Find SplunkForwarder

   Right-click → Restart

### Test Connectivity

PowerShell Command:
powershell

      Test-NetConnection 192.168.1.109 -Port 9997

Expected Result:
text

TcpTestSucceeded: True

### Verify Logs in Splunk

Search Command:
text

      index=endpoint host=WIN-DC
Figure : Windows Forwarder Configuration File
<img width="860" height="652" alt="forwarder Input conf" src="https://github.com/user-attachments/assets/52c5499f-d7c9-4b1c-adde-c46bf00f5da0" />

Figure : Splunk Forwarder Service Status
<img width="855" height="662" alt="forwarder status" src="https://github.com/user-attachments/assets/fca286b9-d875-46eb-9380-b3008d289f7d" />

Figure 10: Forwarder Connectivity Test
<img width="856" height="646" alt="Test Net connection" src="https://github.com/user-attachments/assets/348417b0-d953-4d2f-a9cd-594b8d1f1c44" />

# 7. Windows Client (Client-PC) Forwarder Setup
### Installation Process

Same process as Win-DC with following configuration:
Setting	Value
Receiving Indexer	192.168.1.109:9997
                
                  Username	
                  Password	
### inputs.conf Configuration

File Location:
text
         
          C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf

- Configuration Content:

      [WinEventLog://Application]
      index = endpoint
      disabled = false
      
      [WinEventLog://Security]
      index = endpoint
      disabled = false
      
      [WinEventLog://System]
      index = endpoint
      disabled = false
### Service Verification

Steps:

    Open Services (services.msc)

    Verify SplunkForwarder is Running

    Verify Startup Type: Automatic

### Verify Logs in Splunk

Search Command:
text

      index=endpoint host=CLIENT-PC

Figure : Client-PC Universal Forwarder Installation
<img width="612" height="476" alt="Screenshot 2026-07-15 180304" src="https://github.com/user-attachments/assets/539fe191-a76c-4120-b45d-17143da3e190" />


<img width="1004" height="582" alt="Client PC input" src="https://github.com/user-attachments/assets/637ed9c2-f841-4eb8-b21a-6f790526dc66" />

# 8. pfSense Syslog Integration
### Access pfSense Web Interface

    URL: https://192.168.1.1

    Username: 

    Password: 

### Configure Remote Syslog

Navigation:
Status → System Logs → Settings → Remote Logging Options

Configuration Details:
| Setting                | Value                |
| ---------------------- | -------------------- |
| Enable Remote Logging  | ✓ Checked            |
| Remote Log Server      | `192.168.1.109:5514` |
| IP Protocol            | IPv4                 |
| Remote Syslog Contents | Everything           |

- 7.3 Syslog Contents Selected

    ☑ Everything

    ☐ System Events (included in Everything)

    ☐ Firewall Events (included in Everything)

    ☐ DNS Events (included in Everything)

    ☐ DHCP Events (included in Everything)

    ☐ General Authentication Events (included in Everything)

    ☐ VPN Events (included in Everything)

    ☐ Gateway Monitor Events (included in Everything)


- 7.4 Verify Syslog Data

Splunk Search:
text

index=main sourcetype=syslog

Figure : pfSense Syslog Configuration
<img width="1000" height="531" alt="Pfsense Syslog Configuration 0" src="https://github.com/user-attachments/assets/dda0efa9-b538-4e8d-9bc2-a4190694fa94" />


# 9. Testing & Validation
### Generate Windows Event Logs

- Test 1: Failed Login (Event ID 4625)

Steps:

    On WIN-DC: Press Windows Key + L to lock

    Attempt login with incorrect password

    Event ID 4625 should be generated


Expected Event Details:

| Field         | Value         |
| ------------- | ------------- |
| Event ID      | `4625`        |
| Log Name      | Security      |
| Task Category | Logon         |
| Keywords      | Audit Failure |
| Type          | Failed Logon  |

- Test 2: Successful Login (Event ID 4624)

Steps:

    On WIN-DC: Lock session

    Attempt login with correct password

    Event ID 4624 should be generated

### Verified in Splunk

Search for Failed Logins:
text

      index=endpoint EventCode=4625

Search for Successful Logins:
text

      index=endpoint EventCode=4624

Search for All Windows Logs:
text

      index=endpoint

Search for Win-DC Logs Only:
text

      index=endpoint host=WIN-DC

Search for Client-PC Logs Only:
text

      index=endpoint host=CLIENT-PC01

### Generate Firewall Traffic

- Generate Traffic for pfSense Logs:
cmd

ping 8.8.8.8 -t
nslookup google.com

Search for Firewall Logs:
text

      index=main sourcetype=syslog

- Figure : WIN-DC Generated Failed Login
<img width="1920" height="1080" alt="Screenshot 2026-07-13 124226" src="https://github.com/user-attachments/assets/2a868958-0f05-41e2-83ce-a61b623f5bae" />



# 🛡️ SPLUNK ENTERPRISE DEPLOYMENT
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![License](https://img.shields.io/badge/License-MIT-blue)
