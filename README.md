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

## 1. Introduction
1.1 Project Overview

This project documents the deployment of Splunk Enterprise as a Security Information and Event Management (SIEM) solution within a virtualized home lab environment. The primary objective was to centralize log collection from multiple endpoints and network infrastructure components for comprehensive security monitoring and analysis.

1.2 Project Goals

| Goal                        | Description                                          | Status     |
| --------------------------- | ---------------------------------------------------- | ---------- |
| Deploy Splunk Enterprise    | Install and configure Splunk on Ubuntu Server        | ✅ Complete |
| Configure Log Collection    | Set up receiving ports for Windows event logs        | ✅ Complete |
| Deploy Universal Forwarders | Install and configure forwarders on Windows machines | ✅ Complete |
| Integrate pfSense Logs      | Forward firewall syslog data to Splunk               | ✅ Complete |
| Test & Validate             | Verify log ingestion, searches, and event visibility | ✅ Complete |
| Document Project            | Create comprehensive deployment documentation        | ✅ Complete |


 1.3 Lab Environment Overview

The lab environment consists of multiple virtual machines running on Oracle VirtualBox, interconnected through a pfSense firewall on an internal network.


 1.4 Tools & Technologies Used

| Component                            | Technology                 | Version     |
| ------------------------------------ | -------------------------- | ----------- |
| Virtualization Platform              | Oracle VirtualBox          | 7.x         |
| SIEM Platform                        | Splunk Enterprise          | 10.4.1      |
| Operating System (Splunk Server)     | Ubuntu Server              | 22.04 LTS   |
| Operating System (Domain Controller) | Windows Server             | 2019 / 2022 |
| Operating System (Client Endpoint)   | Windows 10/11              | -           |
| Firewall                             | pfSense                    | 2.7.x       |
| Log Forwarder                        | Splunk Universal Forwarder | 9.x         |



1.5 Network Topology

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



1.6 IP Address Plan
            
| Device            | Hostname                  | IP Address    | Role                                 |
| ----------------- | ------------------------- | ------------- | ------------------------------------ |
| Firewall          | pfSense                   | 192.168.1.1   | Gateway / Firewall                   |
| Splunk Server     | Ubuntu-Splunk-SIEM-Server | 192.168.1.109 | Splunk Enterprise SIEM               |
| Domain Controller | WIN-DC.lab.local          | 192.168.1.101 | Windows Server / Log Source          |
| Client Machine    | CLIENT-PC                 | 192.168.1.103 | Windows Endpoint / Log Source        |
| Attacker Machine  | Kali Linux                | 192.168.1.105 | Security Testing / Attack Simulation |




# 2.1 VM Specifications

The Splunk Enterprise server runs on Ubuntu Server 22.04 LTS, provisioned as a virtual machine within the Oracle VirtualBox environment. The following specifications were carefully selected to ensure optimal performance while maintaining efficient resource utilization within the lab environment.


            Setting	                                Value
            Operating System	                    Ubuntu Server 22.04 LTS
            RAM	                                    8 GB
            Storage	                                40 GB
            Network Adapter	                        Internal Network (LabLAN)
            Hostname	                            Ubuntu-Splunk-SIEM-Server

2.2 Static IP Configuration

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

Commands Executed:

      sudo netplan apply
      ip a
2.3 System Updates

    sudo apt update
    sudo apt upgrade -y

2.4 Guest Additions Installation

      # Insert Guest Additions CD from VirtualBox menu
      sudo ./VBoxLinuxAdditions.run
      sudo reboot

Figure 2: Ubuntu Static IP Verification
<img width="1290" height="787" alt="ip a Output" src="https://github.com/user-attachments/assets/b34c1d36-33c1-4ea1-a978-4aaf02276ee5" />




    Terminal showing ip a command output

    Confirm IP: 192.168.1.109/24

    Confirm interface: enp0s3


## 3. Splunk Enterprise Installation
3.1 Download Splunk Enterprise

Download Details:

    Version: Splunk Enterprise 10.4.1

    Package: .deb (Debian/Ubuntu compatible)

    Source: https://www.splunk.com/en_us/download/splunk-enterprise.html

Download Command:


    cd ~/Downloads
    
    wget -O splunk-10.4.1-5a009d941268-linux-amd64.deb \
    "https://download.splunk.com/products/splunk/releases/10.4.1/linux/splunk-10.4.1-5a009d941268-linux-amd64.deb"
    ```

3.2 Installation Process

        # Install curl dependency
          sudo apt install curl -y
        
        # Install the .deb package
        sudo dpkg -i splunk-*.deb


3.3 First-Time Startup

      # Start Splunk with license acceptance
      sudo /opt/splunk/bin/splunk start --accept-license --answer-yes
Administrator credentials were created during the initial setup.

3.4 Enable Boot-Start

To ensure Splunk Enterprise starts automatically whenever the Ubuntu server boots, the boot-start feature was enabled. This allows the SIEM platform to remain available after system restarts without requiring manual intervention.

Command Executed:

      sudo /opt/splunk/bin/splunk enable boot-start




3.5 Verify Installation

| Component      | Status     | Verification                         |
| -------------- | ---------- | ------------------------------------ |
| Splunk Process | Running    | `sudo /opt/splunk/bin/splunk status` |
| Web Interface  | Accessible | `http://192.168.1.109:8000`          |
| Service        | Enabled    | Boot-start configured                |


Figure 4: Splunk Web Interface Login Page

<img width="1077" height="640" alt="Splunk WebUI" src="https://github.com/user-attachments/assets/e192e06c-6013-44ab-8dc6-6b4b097ba975" />
<img width="1057" height="657" alt="SplunkDASHBOARD" src="https://github.com/user-attachments/assets/6a12ec18-288b-45f8-8964-2c8039f3f840" />


##  4. Splunk Receiving Configuration
4.1 Enable Port 9997 for Forwarder Traffic

Navigation:
Settings → Forwarding and receiving → Configure receiving → Add new

Configuration Details:

    Port: 9997

    Description: Universal Forwarder receiving port

4.2 Verify Receiving Port
Command: 

    sudo netstat -tulpn | grep 9997

Expected Output:

    tcp 0 0 0.0.0.0:9997 0.0.0.0:* LISTEN 1234/splunkd

4.3 Configure Syslog Receiving Port

Navigation:
Settings → Data inputs → UDP → New

Configuration Details:

    Setting	        Value
    Port	        5514
    Source Type	    syslog
    Index	        main  
    Host	        IP





Note: Port 5514 was used instead of the privileged port 514 to avoid permission issues.



      | Setting          | Value             |
      | ---------------- | ----------------- |
      | Receiving Port   | 5514              |
      | Data Source Type | syslog            |
      | Splunk Index     | main              |
      | Host Identifier  | Source IP Address |


# 🛡️ SPLUNK ENTERPRISE DEPLOYMENT
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![License](https://img.shields.io/badge/License-MIT-blue)
