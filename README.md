# SCADA/ICS Security Lab

## Master's Thesis: "Simulation and Analysis of Cyberattacks Against ICS/SCADA Systems and Development of a Defense Plan"

**Institution:** G.S. Rakovski Military Academy  
**Program:** Cybersecurity  
**Date:** April 2026

---

## 📋 Overview

This project is a laboratory environment for simulating an ICS/SCADA system, used to research cyberattacks and develop defensive measures.

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      OT NETWORK (192.168.10.0/24)               │
│                                                                 │
│   ┌─────────────┐         Ethernet         ┌─────────────┐     │
│   │   RPi 5     │◄───────────────────────►│   RPi 4     │     │
│   │   SCADA     │       Switch             │    PLC      │     │
│   │192.168.10.20│                          │192.168.10.10│     │
│   └──────┬──────┘                          └──────┬──────┘     │
│          │                                        │             │
└──────────┼────────────────────────────────────────┼─────────────┘
           │ WiFi                                   │ WiFi
           │                                        │
    192.168.100.121                          192.168.100.154
```

## 🔧 Hardware

### RPi 5 — SCADA Server
- Raspberry Pi 5 4GB
- Argon ONE V3 M.2 Case
- NVMe SSD 119GB

### RPi 4 — PLC Simulator
- Raspberry Pi 4B
- microSD 16GB
- Sensors:
  - DHT22 (Temperature/Humidity)
  - HC-SR04 (Ultrasonic)
  - LED Indicator
  - Relay Module

## 💻 Software Stack

### RPi 5 (SCADA)
| Component | Version | Port |
|-----------|---------|------|
| Scada-LTS | 2.7.8 | 8080 |
| Grafana | latest | 3000 |
| InfluxDB | 2.7 | 8086 |
| MySQL | 8.0 | 3306 |
| Tomcat | 9.0.102 | 8080 |

### RPi 4 (PLC)
| Component | Version | Port |
|-----------|---------|------|
| OpenPLC Runtime | v3 | 8080 |

## 📁 Project Structure

```
scada-ics-security-lab/
├── README.md
├── docs/
│   └── installation-report.md
├── docker/
│   └── docker-compose.yml
├── hardware/
│   └── wiring-diagram.md
├── plc/
│   └── programs/
├── scada/
│   └── config/
└── security/
    ├── attacks/
    └── defense/
```

## 🚀 Quick Start

See [docs/installation-report.md](docs/installation-report.md) for full installation instructions.

### Start SCADA Server (RPi 5)
```bash
cd ~/scada
docker compose up -d
sudo systemctl start tomcat9
```

### Access Points
| Service | URL | Login |
|---------|-----|-------|
| Scada-LTS | http://192.168.100.121:8080/Scada-LTS | admin / admin |
| Grafana | http://192.168.100.121:3000 | admin / admin |
| InfluxDB | http://192.168.100.121:8086 | admin / admin12345 |
| OpenPLC | http://192.168.100.154:8080 | openplc / openplc |

## 🔌 GPIO Pinout (RPi 4)

| Component | GPIO | Pin |
|-----------|------|-----|
| DHT22 DATA | GPIO4 | 7 |
| LED | GPIO14 | 8 |
| Relay IN | GPIO15 | 10 |
| HC-SR04 TRIG | GPIO17 | 11 |
| HC-SR04 ECHO | GPIO27 | 13 |

## ⚠️ Attack Scenarios

| # | Attack | MITRE ATT&CK |
|---|--------|--------------|
| 1 | Reconnaissance (Modbus Scanning) | T0846 |
| 2 | Man-in-the-Middle | T0830 |
| 3 | Replay Attack | T0867 |
| 4 | Denial of Service | T0814 |
| 5 | Modbus Function Code Abuse | T0855 |
| 6 | False Data Injection | T0856 |
| 7 | Firmware Manipulation | T0839 |

## 🛡️ Defense Measures

- TLS Encryption
- Snort IDS Rules
- Firewall Configuration
- Network Segmentation
- Zeek Network Monitoring

## 📚 References

- [Scada-LTS](https://github.com/SCADA-LTS/Scada-LTS)
- [OpenPLC Project](https://openplcproject.com/)
- [MITRE ATT&CK for ICS](https://attack.mitre.org/techniques/ics/)

## 📄 License

This project is created for educational purposes only.
