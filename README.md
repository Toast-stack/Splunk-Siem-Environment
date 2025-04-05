# Splunk-SIEM-Environment
A complete SIEM (Security Information and Event Management) solution leveraging Splunk to monitor SSH activity logs, detect threats, and visualize security insights. This project is tailored for cybersecurity enthusiasts and professionals aiming to learn the intricacies of SIEM configurations and log analysis.

## Overview
This SIEM setup utilizes Splunk to ingest, parse, and analyze SSH logs (`auth.log`) for real-time threat detection and incident response. Designed within a controlled virtualized environment, the project replicates attacker-victim scenarios, enabling hands-on experience with log analysis, dashboards, and alerts.

## Environmental Details
This project leverages a dual-VM setup to replicate an attacker-victim scenario, providing a practical and hands-on learning experience.

### **1. Victim Machine (Ubuntu VM)**
- **Operating System**: Ubuntu 20.04 LTS
- **Role**: Acts as the log source, hosting SSH logs (`auth.log`) for Splunk ingestion.
- **Purpose**:
    - Captures login attempts and security events from its SSH service.
    - Demonstrates system logging and potential exploitation by attacker.
- **Configuration**:
    - SSH server enabled with default settings.
    - Intentionally weak credentials to facilitate brute force attempts (testing purpose only).
    - VirtualBox Guest Additions installed for clipboard sharing and seamless interaction with the host.
 
### **2. Attacker Machine (Kali Linux VM)**
- **Operating System**: Kali Linux
- **Role**: Simulates malicious actor targeting the victim machine.
- **Tools Used**:
    - **Hydra**:
      ```bash
      hydra -l root -P wordlist.txt ssh://<victim-IP>
      ```
    - **Nmap**:
      ```bash
      nmap -p 22<victim-IP>
      ```
- **purpose**:
  - Replicates attacker behavior and brute force patterns.
  - Helps analysts observe and analyze malicious activity in logs.
 
## Features
- Splunk integration with SSH logs for real-time analysis.
- Configurable dashboards to visualize failed login attempts and detect brute force activity.
- Alerts to notify admins of suspicious IP addresses or failed login trends.
- Flexible field extractions for targeted searches and custom queries.

## Installation and Setup
### **1.Prerequisites**
  - Install Splunk Enterprise on the host machine and ensure it's accessible via `http://localhost:8000`.
  - Install Splunk Universal Forwarder on the victim VM.
### **2.Configure the Universal Forwarder**
  1. Install the Splunk Universal Forwarder:
     ```bash
     sudo dpkg -i splunkforwarder-<version>.deb
     ```
  2. Accept the license agreement and start the service:
     ```bash
     sudo /opt/splunkforwarder/bin/splunk start --accept-license
     ```
  3. Add the SSH log file (`auth.log`) for monitoring:
     ```bash
     sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
     ```
  4. Connect the forwarder to the host Splunk instance:
     ```bash
     sudo /opt/splunkforwarder/bin/splunk add forward-server <host-IP>:9997
     ```
### **3.Configure Splunk on the Host Machine**
  1. Enable port `9997` for data ingestion:
     - go to **Settings → Data Inputs → Forwarding and Receiving → Configure Receiving** in Splunk Web.
     - Add and enable port `9997`.
    
  2. Search for ingested logs to confirm the setup:
     ```spl
     index=* sourcetype="auth"
     ```
## **Visualizations**
### Failed Login Attempts by IP (Bar Chart)
Query:
```spl
index=* sourcetype="auth" "Failed password"
| stats count by src_ip
| sort -count
```
  - X-axis: IP addresses (`scr_ip`).
  - Y-axis: Number of failed attempts.

### Login Trends Over Time (Line Chart)
Query:
```spl
index=* sourcetype="auth" "Failed password"
| timechart count
```
  - X-axis: Time.
  - Y-axis: Number of failed login attempts.

### Event Distribution by Host (Pie Chart)
Query:
```spl
index=* sourcetype="auth" "Failed password"
| stats count by host
| sort -count
```
  - **Slices**: Hostnames (`host`).
  - **Values**: Event counts.

## **Alerts**
Configure alerts within Splunk for specific log events:
  1. **Failed Login Spike**:
     - Trigger when a single IP has 10+ failed login attempts within 10 minutes.
  2. **Suspicious IP Block**:
     - Automate blocking flagged IPs using firewall rules.
    
## **Future Enhancements**
- Integrate GeoIP lookups to determine the geographical origin of suspicious IPs.
- Enable automated blocking of flagged IPs using Splunk scripts and firewall rules.
- Add anomaly detection dashboards to highlight irregular patterns.

## **Acknowledgments**
This project draws inspiration from real-world cybersecurity workflows and leverages Splunk as a learning tool for aspiring SOC Analysts and cybersecurity professionals.
