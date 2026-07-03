# Splunk Firewall & Network Monitoring Dashboard

A network security monitoring project built in Splunk to detect firewall-level threats using simulated log data. The dashboard tracks port scans, RDP/SSH brute force attacks, blocked suspicious IPs, and data exfiltration attempts.

---

## Dashboard Preview

[📄 View Firewall Monitoring Dashboard](./Firewall%20Monitoring.pdf)

## Tools Used

- [Splunk Enterprise (Free Trial)](https://www.splunk.com/en_us/download/splunk-enterprise.html)
- SPL (Splunk Processing Language)
- Python (for generating the dataset)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)

---

## Project Structure

```
splunk-firewall-monitoring/
│
├── firewall_logs.csv            # Firewall log dataset (2,374 records)
├── Firewall Monitoring.pdf      # Dashboard screenshot/export
└── README.md
```

---

## Dataset — firewall_logs.csv

**Total Records:** 2,374
**Date Range:** May 1, 2026 — June 30, 2026

| Field | Description |
|---|---|
| event_id | Unique ID for each log event |
| timestamp | Date and time of the event |
| firewall_name | Which firewall logged this event |
| action | Allow / Block / Drop |
| direction | Inbound / Outbound |
| protocol | TCP / UDP / HTTPS / SSH / RDP / DNS etc. |
| source_ip | IP where the traffic originated |
| source_port | Port used by the source |
| destination_ip | Target IP address |
| destination_port | Target port |
| bytes_sent | Data sent in bytes |
| bytes_received | Data received in bytes |
| duration_sec | How long the connection lasted |
| country | Country of the source IP |
| alert_flag | Attack label assigned to the event |
| rule_triggered | Firewall rule that matched |

---

## How to Load the Data in Splunk

**Step 1 — Upload the dataset**
- Go to **Settings > Add Data > Upload**
- Select `firewall_logs.csv`
- Click **Create New Sourcetype** and name it `firewall_logs`
- Set index to `main`
- Click **Review > Submit**

**Step 2 — Verify data loaded**
```spl
index=main sourcetype=firewall_logs | head 10
```

**Step 3 — Create a new dashboard**
- Go to **Dashboards > Create New Dashboard**
- Name it `Firewall Monitoring`
- Add panels using the SPL queries below

---

## SPL Queries

### 1. Total Firewall Events (KPI)
```spl
index=main sourcetype=firewall_logs
| stats count as Total_Firewall_Events
```

### 2. Total Blocked Events (KPI)
```spl
index=main sourcetype=firewall_logs action=Block
| stats count as Total_Blocked_Events
```

### 3. Total Alert Flags (Table)
```spl
index=main sourcetype=firewall_logs
| stats count as Total_Alert_Flags by alert_flag
| sort - Total_Alert_Flags
```

### 4. Total Allowed Events (KPI)
```spl
index=main sourcetype=firewall_logs action=Allow
| stats count as Total_Allowed_Events
```

### 5. Traffic By Action (Pie Chart)
```spl
index=main sourcetype=firewall_logs
| stats count by action
```

### 6. Traffic Trend Over Time (Line Chart)
```spl
index=main sourcetype=firewall_logs
| timechart span=1d count by action
```

### 7. Port Scan Detection Table
```spl
index=main sourcetype=firewall_logs alert_flag=Port_Scan
| stats dc(destination_port) as Ports_Scanned, count as Total_Packets by source_ip, country
| where Ports_Scanned > 20
| sort - Ports_Scanned
```

### 8. Top 10 Blocked Source IPs
```spl
index=main sourcetype=firewall_logs action=Block
| stats count as Blocked_Count by source_ip, country
| sort - Blocked_Count
| head 10
```

### 9. RDP Brute Force Over Network — Port 3389
```spl
index=main sourcetype=firewall_logs destination_port=3389
| stats count as RDP_Attempts by source_ip, destination_ip, action, country
| sort - RDP_Attempts
```

### 10. SSH Brute Force Over Network — Port 22
```spl
index=main sourcetype=firewall_logs destination_port=22
| stats count as SSH_Attempts by source_ip, destination_ip, action, country
| sort - SSH_Attempts
```

### 11. Traffic Blocked By Country (Bar Chart)
```spl
index=main sourcetype=firewall_logs action=Block
| stats count as Blocked_Events by country
| sort - Blocked_Events
```

### 12. Protocol Usage (Bar Chart)
```spl
index=main sourcetype=firewall_logs
| stats count by protocol
| sort - count
```

### 13. Inbound VS Outbound Traffic (Table)
```spl
index=main sourcetype=firewall_logs
| stats count by direction, action
```

---

## MITRE ATT&CK Mapping

| Alert Flag | Tactic | Technique |
|---|---|---|
| Port_Scan | Discovery | [T1046 — Network Service Scanning](https://attack.mitre.org/techniques/T1046/) |
| RDP_BruteForce | Lateral Movement | [T1021.001 — Remote Desktop Protocol](https://attack.mitre.org/techniques/T1021/001/) |
| SSH_BruteForce | Lateral Movement | [T1021.004 — SSH](https://attack.mitre.org/techniques/T1021/004/) |
| Blocked_Suspicious_IP | Initial Access | [T1190 — Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/) |
| Possible_Data_Exfiltration | Exfiltration | [T1041 — Exfiltration Over C2 Channel](https://attack.mitre.org/techniques/T1041/) |

---

## What I Learned

- How port scans look in firewall logs — one IP hitting dozens of ports in seconds is a clear pattern
- Why RDP (port 3389) and SSH (port 22) are the most targeted ports in real attacks
- How to spot data exfiltration at the network level using bytes_sent values
- Writing SPL queries to detect threats from raw firewall data

---

## References

- [Splunk SPL Documentation](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/WhatsInThisManual)
- [MITRE ATT&CK — Discovery](https://attack.mitre.org/tactics/TA0007/)
- [MITRE ATT&CK — Lateral Movement](https://attack.mitre.org/tactics/TA0008/)
- [MITRE ATT&CK — Exfiltration](https://attack.mitre.org/tactics/TA0010/)
- [LetsDefend SOC Analyst Path](https://app.letsdefend.io/)

---

## Author

**Omkar Kotkar**
[GitHub](https://github.com/OMKARGIT2006) 
