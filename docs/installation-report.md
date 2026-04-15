# SCADA/ICS Laboratory Environment — Installation Report

**Date:** April 8, 2026  
**Project:** Master's Thesis "Simulation and Analysis of Cyberattacks Against ICS/SCADA Systems and Development of a Defense Plan"  
**Institution:** G.S. Rakovski Military Academy, Cybersecurity Program

---

## 1. Laboratory Architecture

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
┌──────────┼────────────────────────────────────────┼─────────────┐
│          ▼                                        ▼             │
│   192.168.100.121                          192.168.100.154      │
│                                                                 │
│                 HOME NETWORK (192.168.100.0/24)                 │
│                      (Internet + SSH Access)                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. RPi 5 — SCADA Server

### 2.1 Hardware
- **Model:** Raspberry Pi 5 4GB
- **Case:** Argon ONE V3 M.2
- **Storage:** NVMe SSD 119GB
- **OS:** Raspberry Pi OS Lite 64-bit (Debian Trixie)

### 2.2 Network Configuration
| Interface | IP Address | Purpose |
|-----------|------------|---------|
| eth0 | 192.168.10.20 | OT network to PLC |
| wlan0 | 192.168.100.121 | SSH access + internet |

### 2.3 Installed Software

#### Docker Containers (docker-compose.yml)
```yaml
services:
  mysql:
    image: arm64v8/mysql:8.0
    container_name: scada-mysql
    command: >
      --default-authentication-plugin=mysql_native_password
      --log_bin_trust_function_creators=1
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: scadalts
      MYSQL_USER: scadalts
      MYSQL_PASSWORD: scadalts
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"
    restart: unless-stopped

  influxdb:
    image: influxdb:2.7
    container_name: influxdb
    environment:
      DOCKER_INFLUXDB_INIT_MODE: setup
      DOCKER_INFLUXDB_INIT_USERNAME: admin
      DOCKER_INFLUXDB_INIT_PASSWORD: admin12345
      DOCKER_INFLUXDB_INIT_ORG: scada-lab
      DOCKER_INFLUXDB_INIT_BUCKET: sensors
      DOCKER_INFLUXDB_INIT_ADMIN_TOKEN: scada-lab-token-2024
    volumes:
      - influxdb_data:/var/lib/influxdb2
    ports:
      - "8086:8086"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana_data:/var/lib/grafana
    ports:
      - "3000:3000"
    restart: unless-stopped

volumes:
  mysql_data:
  influxdb_data:
  grafana_data:
```

#### Scada-LTS (Native Installation)
- **Java:** OpenJDK 21 (pre-installed in Debian Trixie)
- **Tomcat:** 9.0.102 (manual installation in /opt/tomcat9)
- **Scada-LTS:** v2.7.8 (WAR file)
- **Additional libraries:** JAXB for Java 21 compatibility:
  - jaxb-api-2.3.1.jar
  - jaxb-runtime-2.3.1.jar
  - activation-1.1.1.jar
  - istack-commons-runtime-3.0.7.jar
  - mysql-connector-j-8.0.33.jar

#### Tomcat systemd service (/etc/systemd/system/tomcat9.service)
```ini
[Unit]
Description=Apache Tomcat 9
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk-arm64"
Environment="CATALINA_HOME=/opt/tomcat9"
Environment="CATALINA_BASE=/opt/tomcat9"
Environment="CATALINA_PID=/opt/tomcat9/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms256M -Xmx512M"

ExecStart=/opt/tomcat9/bin/startup.sh
ExecStop=/opt/tomcat9/bin/shutdown.sh

Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### 2.4 Services and Access
| Service | URL | Login |
|---------|-----|-------|
| Scada-LTS | http://192.168.100.121:8080/Scada-LTS | admin / admin |
| Grafana | http://192.168.100.121:3000 | admin / admin |
| InfluxDB | http://192.168.100.121:8086 | admin / admin12345 |
| MySQL | localhost:3306 | scadalts / scadalts |
| Tomcat | http://192.168.100.121:8080 | — |

### 2.5 SSH Access
```bash
ssh pi@192.168.100.121
# or
ssh pi@rpi-scada.local
```

---

## 3. RPi 4 — PLC Simulator

### 3.1 Hardware
- **Model:** Raspberry Pi 4B
- **Storage:** microSD 16GB
- **OS:** Raspberry Pi OS Lite 64-bit

### 3.2 Network Configuration
| Interface | IP Address | Purpose |
|-----------|------------|---------|
| eth0 | 192.168.10.10 | OT network to SCADA |
| wlan0 | 192.168.100.154 (DHCP) | SSH access + internet |

#### Static IP Configuration for eth0
```bash
sudo nmcli con add type ethernet con-name "OT-Network" ifname eth0 ipv4.addresses 192.168.10.10/24 ipv4.method manual
sudo nmcli con up "OT-Network"
```

### 3.3 Installed Software

#### OpenPLC Runtime v3
- **Installation:**
```bash
cd ~
git clone https://github.com/thiagoralves/OpenPLC_v3.git
cd OpenPLC_v3
./install.sh rpi
```
- **Installation time:** ~45 minutes
- **Auto-start:** Yes (systemd service)

### 3.4 Services and Access
| Service | URL | Login |
|---------|-----|-------|
| OpenPLC | http://192.168.100.154:8080 | openplc / openplc |

### 3.5 SSH Access
```bash
ssh pi@192.168.100.154
# or
ssh pi@rpi-plc.local
```

---

## 4. Important Installation Notes

### 4.1 Issue: scadalts/scadalts Docker Image
**Problem:** The official Scada-LTS Docker image is amd64 only and does NOT work natively on ARM64 (Raspberry Pi).

**Solution:** Native installation with Tomcat 9 + WAR file instead of Docker.

### 4.2 Issue: Java 21 Incompatibility
**Problem:** Debian Trixie only has Java 21, but Scada-LTS was written for Java 8/11 and uses `javax.xml.bind` (JAXB), which was removed in Java 11+.

**Error:**
```
java.lang.ClassNotFoundException: javax.xml.bind.ValidationEventHandler
```

**Solution:** Add JAXB libraries to /opt/tomcat9/lib/:
- jaxb-api-2.3.1.jar
- jaxb-runtime-2.3.1.jar
- activation-1.1.1.jar
- istack-commons-runtime-3.0.7.jar

### 4.3 Issue: Tomcat 9 Missing in Debian Trixie
**Problem:** Debian Trixie only has Tomcat 10/11, which use Jakarta EE namespace instead of javax.*, incompatible with Scada-LTS.

**Solution:** Manual download and installation of Tomcat 9.0.102 from Apache archive.

---

## 5. Management Commands

### RPi 5 — SCADA
```bash
# Docker containers
cd ~/scada
docker compose up -d      # Start
docker compose down       # Stop
docker ps                 # Status

# Tomcat/Scada-LTS
sudo systemctl start tomcat9
sudo systemctl stop tomcat9
sudo systemctl restart tomcat9
sudo systemctl status tomcat9

# Logs
sudo tail -f /opt/tomcat9/logs/catalina.out
```

### RPi 4 — PLC
```bash
# OpenPLC
sudo systemctl start openplc
sudo systemctl stop openplc
sudo systemctl restart openplc
sudo systemctl status openplc
```

---

## 6. Next Steps (TODO)

### Hardware for RPi 4 (PLC)
- [ ] Connect DHT22 sensor (temperature/humidity)
- [ ] Connect HC-SR04 sensor (ultrasonic)
- [ ] Connect relay for outputs
- [ ] Connect LED indicators
- [ ] Breadboard wiring

### Software Configuration
- [ ] Create PLC program in OpenPLC Editor
- [ ] Configure Raspberry Pi GPIO driver in OpenPLC
- [ ] Setup Modbus TCP server in OpenPLC
- [ ] Add Modbus TCP data source in Scada-LTS
- [ ] Create data points in Scada-LTS
- [ ] Connect Grafana to InfluxDB
- [ ] Create Grafana dashboard

### Monitoring and Security
- [ ] Install Snort on RPi 5
- [ ] Install Zeek on RPi 5
- [ ] Configure Modbus traffic rules
- [ ] Test the 7 attacks from the thesis

---

## 7. Useful Links

- **Scada-LTS GitHub:** https://github.com/SCADA-LTS/Scada-LTS
- **Scada-LTS Wiki:** https://github.com/SCADA-LTS/Scada-LTS/wiki
- **OpenPLC Project:** https://openplcproject.com/
- **OpenPLC GitHub:** https://github.com/thiagoralves/OpenPLC_v3

---

## 8. File Structure

### RPi 5
```
/home/pi/scada/
└── docker-compose.yml

/opt/tomcat9/
├── bin/
├── conf/
├── lib/
│   ├── mysql-connector-j-8.0.33.jar
│   ├── jaxb-api-2.3.1.jar
│   ├── jaxb-runtime-2.3.1.jar
│   ├── activation-1.1.1.jar
│   └── istack-commons-runtime-3.0.7.jar
├── logs/
├── webapps/
│   ├── Scada-LTS/
│   └── Scada-LTS.war
└── ...

/etc/systemd/system/tomcat9.service
```

### RPi 4
```
/home/pi/OpenPLC_v3/
├── webserver/
├── start_openplc.sh
└── ...

/usr/lib/systemd/system/openplc.service
```

---

*Document generated on April 8, 2026*
