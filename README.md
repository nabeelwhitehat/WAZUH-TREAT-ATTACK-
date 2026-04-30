# 🛡️ WAZUH-VT-THREATATTACK

> **An Integrated Framework for Automated Threat Intelligence & Incident Response**  
> **Author:** Nabeel IV | **Supervisor:** Muhammed Sinan | **Institution:** TechByHeart

---

## 📌 Overview

A fully functional Wazuh SIEM environment deployed on Kali Linux using Docker, integrated with multiple threat intelligence platforms and automated response mechanisms. The system provides centralized security monitoring, real-time threat detection, and automated incident response without manual intervention.

---

## 🔧 Integrations

| Component | Purpose |
|-----------|---------|
| **Wazuh SIEM/XDR** | Central security monitoring and event management platform |
| **VirusTotal API** | Automated malware detection via file hash analysis (70+ AV engines) |
| **AlienVault OTX** | Real-time threat intelligence for malicious IP/domain reputation checks |
| **MITRE ATT&CK** | Maps detected events to attacker tactics and techniques |
| **File Integrity Monitoring (FIM)** | Real-time detection of unauthorized file changes using MD5/SHA1/SHA256 |
| **Active Response** | Automated mitigation — firewall blocking, process termination, host isolation |

---

## 🖥️ System Requirements

**Wazuh Stack (Single-Node):**
- OS: Linux or Windows (AMD64)
- CPU: 4+ cores
- RAM: 8+ GB
- Disk: 50+ GB

**Wazuh Agent:**
- OS: Linux or Windows (AMD64)
- CPU: 2+ cores
- RAM: 1+ GB
- Disk: 10+ GB

---

## ⚙️ Implementation Overview

### Environment Setup
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io -y
sudo systemctl start docker && sudo systemctl enable docker
sudo usermod -aG docker $USER
sudo apt install docker-compose -y
```

### Deploy Wazuh Stack
```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.10.0
cd wazuh-docker/single-node

# Generate SSL certificates
docker compose -f generate-indexer-certs.yml run --rm generator

# Start containers
docker-compose up -d
```

### Install & Configure Wazuh Agent
```bash
sudo apt install wazuh-agent -y
sudo nano /var/ossec/etc/ossec.conf   # Set <address>127.0.0.1</address>
sudo systemctl enable wazuh-agent && sudo systemctl start wazuh-agent
```

### Access Dashboard
```
URL: http://127.0.0.1
Username: admin
Password: SecretPassword
```

---

## 🧩 Key Features Configured

### File Integrity Monitoring (FIM)
Monitors `/home/kali/Downloads` (or custom directory) in real-time using `check_all` and `whodata` — detecting file additions, modifications, and deletions with full user/process attribution.

### VirusTotal Integration
On any FIM event, Wazuh extracts the file hash and queries VirusTotal's database. EICAR test files were used to validate detection — alerts appeared at **rule level 12** with Rule ID `87105`.

### AlienVault OTX Integration
A custom Python script (`custom-alienvault.py`) queries the OTX API for source IP reputation. Simulated alerts with test IPs confirmed successful integration and pulse-count reporting.

### MITRE ATT&CK Mapping
Simulated techniques detected and mapped:

| Simulation | MITRE Technique | ID |
|------------|-----------------|-----|
| Access `/etc/shadow` | Privilege Escalation | T1548.003 |
| Add hidden root user | Persistence | T1136.001 |
| Repeated failed logins | Brute Force | T1110 |

### Active Response
Triggers `firewall-drop` automatically when VirusTotal alert (Rule ID `87105`) fires — blocks the malicious source for **600 seconds**. Requires `NET_ADMIN` and `NET_RAW` Docker capabilities.

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>87105</rules_id>
  <timeout>600</timeout>
</active-response>
```

---

## ✅ Threat Validation Results

All integrations were tested with simulated threats and verified through the Wazuh Dashboard:

| Test | Method | Result |
|------|--------|--------|
| FIM | File create/modify/delete in monitored directory | ✅ Alerts generated |
| VirusTotal | EICAR test file dropped in monitored path | ✅ 62 engines flagged |
| AlienVault OTX | Simulated alert with test IP | ✅ Pulse count returned |
| MITRE ATT&CK | Shadow access, backdoor user, brute force | ✅ Techniques mapped |
| Active Response | EICAR file trigger | ✅ Host blocked (132 events logged) |

---

## 📁 Project Structure

```
wazuh-docker/
└── single-node/
    ├── docker-compose.yml
    └── config/
        └── wazuh_cluster/
            └── wazuh_manager.conf   # VirusTotal, Active Response config
/var/ossec/
    ├── etc/ossec.conf               # Agent config, FIM, AlienVault, audit logs
    └── integrations/
        └── custom-alienvault.py     # OTX threat intelligence script
```

---

## ⚠️ Disclaimer

This project was built in an isolated lab environment for educational purposes under the supervision of TechByHeart. Do not deploy or test against systems without explicit authorization.

---

*Project Report — WAZUH-VT-THREATATTACK | Nabeel IV | April 2026*
