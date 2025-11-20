<img width="1536" height="1024" alt="CybersecurityHoneyPot" src="https://github.com/user-attachments/assets/6ed45f46-4706-4bdb-9728-debe1b091250" />

# Honey-Pot-Creation-for-Home-Lab
Designed, built, and automated a production-grade honeypot environment using OpenCanary on MX Linux as part of a home SOC lab. The system was engineered for stealth, persistence, and eventual seamless integration with Elastic Stack (via Kali Purple) for centralized monitoring and analysis.

### 🧠 Project: MX Linux Honeypot with OpenCanary + Elastic Stack
**Role:** Security Engineer / SOC Lab Developer  
**Date:** October 12-13, 2025  

#### Summary
Built a persistent and stealthy honeypot on MX Linux using **OpenCanary** to simulate network services and capture attacker behavior. Integrated the deployment with **Elastic Stack (via Kali Purple)** for real-time analysis and visualization of intrusion data.

#### Key Highlights
- Deployed OpenCanary v0.9.6 inside a controlled Python virtual environment (`venv`) with SysVinit for automatic boot startup.
- Implemented least-privilege execution using `cap_net_bind_service` to bind low ports (21, 22, 80) without root escalation.
- Authored a custom `/etc/init.d/opencanary` service script with:
  - runtime directory creation for `/var/run/opencanary`
  - automatic log rotation via `/etc/logrotate.d/opencanary`
  - recovery and restart logic for persistent uptime.
- Configured UFW firewall, validated listener ports, and verified service resiliency across reboots.
- Created a **one-page rebuild and operations checklist** for rapid redeployment and maintenance.
- Prepared environment for syslog forwarding to **Kali Purple / Elastic Kibana** for threat event visualization.
- Designed supporting network segmentation plan (pfSense + VLANs) for future containment and monitoring expansion.

#### Outcome
Delivered a robust, lightweight honeypot environment that boots automatically, operates securely under SysVinit, and seamlessly integrates into a home SOC architecture for hands-on defense and detection practice.




---

# 🛡️ Creating a Honeypot

**By Bruce Thornton**
**10/12–13/2025**

---

# 📘 Professional Project Summary

### **Project Title:**

**MX Linux Honeypot Integration with OpenCanary and Eventual Elastic Stack Configuration**

### **Role:**

Security Engineer / SOC Lab Developer

### **Duration:**

October 12–13, 2025

### **Overview**

Designed, built, and automated a production-grade honeypot environment using **OpenCanary on MX Linux** as part of a home SOC lab. The system was engineered for stealth, persistence, and seamless integration with an eventual **Elastic Stack / Kibana** deployment (via Kali Purple) for centralized monitoring and analysis.

### **Key Achievements**

* Deployed and configured **OpenCanary v0.9.6** in a secure Python virtual environment using **SysVinit** (MX Linux is non-systemd).
* Implemented **least-privilege execution** and Linux capabilities (`cap_net_bind_service`) to safely bind to privileged ports (21, 22, 80).
* Automated full startup lifecycle using custom SysV service scripts with log rotation, PID management, and tmpfs runtime fixes.
* Tuned honeypot listeners (FTP, SSH, HTTP) for realistic attack simulation without interfering with legitimate daemons.
* Developed a lightweight rebuild checklist for rapid redeployment (venv provisioning, package setup, init script registration, UFW rules).
* Prepared system for **syslog forwarding** to Elastic/Kibana dashboards through Kali Purple.
* Built groundwork for **VLAN and pfSense segmentation** to safely isolate and simulate attacks.

### **Outcome**

Achieved a stable, resource-efficient honeypot that:

* auto-starts at boot
* provides actionable telemetry to Elastic Stack
* and forms the core of a scalable, modular cybersecurity home lab

---

# 🖥️ Selecting the Environment

This honeypot was deployed on:

**Lenovo ThinkCentre M93P Tiny Desktop**

* Intel Core i5-4570T
* 8GB RAM
* 256GB SSD
* Windows 10 Pro (Renewed)

MX Linux was flashed onto a USB drive and installed cleanly onto the machine.

---

# 🛠️ MX Linux + OpenCanary

## **Single-Page Rebuild Guide**

---

## **1️⃣ Install MX Linux (clean base)**

After installation and first boot:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git python3-venv python3-pip libcap2-bin ufw
```

---

## **2️⃣ Create virtual environment & install OpenCanary**

```bash
sudo mkdir -p /opt/opencanary
cd /opt/opencanary
sudo python3 -m venv venv
sudo chown -R $USER:$USER venv
source venv/bin/activate
pip install --upgrade pip setuptools==68.0.0
pip install opencanary==0.9.6
deactivate
```

---

## **3️⃣ Initialize config**

```bash
sudo mkdir -p /etc/opencanaryd
sudo /opt/opencanary/venv/bin/python3 -m opencanaryd --copyconfig
sudo chown -R opencanary:opencanary /etc/opencanaryd
```

---

## **4️⃣ Grant Python access to privileged ports**

```bash
sudo setcap 'cap_net_bind_service=+ep' "$(readlink -f /opt/opencanary/venv/bin/python)"
```

---

## **5️⃣ Install SysV service script**

```bash
sudo nano /etc/init.d/opencanary
```

Paste the script below:

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          opencanary
# Required-Start:    $network $remote_fs
# Required-Stop:     $network $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: OpenCanary honeypot (Twisted)
# Description:       Starts OpenCanary using twistd and the opencanary.tac file
### END INIT INFO

NAME="opencanary"
VENV="/opt/opencanary/venv"
TWISTD="$VENV/bin/twistd"
PIDDIR="/var/run/opencanary"
PIDFILE="$PIDDIR/opencanary.pid"
LOGFILE="/var/log/opencanary.log"

TAC="$($VENV/bin/python - <<'PY'
import os
try:
    import opencanary
    p = os.path.join(os.path.dirname(opencanary.__file__), "opencanary.tac")
    print(p if os.path.isfile(p) else "")
except Exception:
    print("")
PY
)"
[ -z "$TAC" ] && TAC="/opt/opencanary-src/bin/opencanary.tac"

DAEMON_OPTS="-y $TAC -l $LOGFILE --pidfile $PIDFILE"

do_start() {
    if [ ! -x "$TWISTD" ]; then
        echo "twistd not found at $TWISTD"; return 1
    fi
    if [ ! -f "$TAC" ]; then
        echo "TAC not found at $TAC"; return 1
    fi

    echo "Starting $NAME..."

    mkdir -p "$PIDDIR"
    chown opencanary:opencanary "$PIDDIR"
    touch "$LOGFILE"
    chown opencanary:opencanary "$LOGFILE"

    start-stop-daemon --start --oknodo --background \
      --pidfile "$PIDFILE" --chuid opencanary --exec "$TWISTD" -- $DAEMON_OPTS
}

do_stop() {
    echo "Stopping $NAME..."
    if [ -f "$PIDFILE" ]; then
        start-stop-daemon --stop --oknodo --pidfile "$PIDFILE"
        rm -f "$PIDFILE"
    else
        pkill -f "twistd .*opencanary.tac" 2>/dev/null || true
    fi
}

do_status() {
    if [ -f "$PIDFILE" ] && ps -p "$(cat "$PIDFILE" 2>/dev/null)" >/dev/null 2>&1; then
        echo "$NAME running (pid $(cat "$PIDFILE"))"; return 0
    fi
    pgrep -f "twistd .*opencanary.tac" >/dev/null 2>&1 && { echo "$NAME running"; return 0; }
    echo "$NAME not running"; return 3
}

case "$1" in
  start)   do_start ;;
  stop)    do_stop ;;
  restart) do_stop; sleep 1; do_start ;;
  status)  do_status ;;
  *) echo "Usage: /etc/init.d/$NAME {start|stop|restart|status}"; exit 1 ;;
esac
exit $?
```

Finalize setup:

```bash
sudo dos2unix /etc/init.d/opencanary 2>/dev/null || true
sudo chmod +x /etc/init.d/opencanary
sudo update-rc.d -f opencanary remove
sudo update-rc.d opencanary defaults
```

---

## **6️⃣ Create service user & log dirs**

```bash
sudo useradd -r -s /usr/sbin/nologin opencanary 2>/dev/null || true
sudo mkdir -p /var/run/opencanary /var/log
sudo touch /var/log/opencanary.log
sudo chown -R opencanary:opencanary /opt/opencanary /etc/opencanaryd /var/run/opencanary /var/log/opencanary.log
```

---

## **7️⃣ Start & verify**

```bash
sudo service opencanary start
sudo service opencanary status
sudo tail -f /var/log/opencanary.log
sudo ss -tulpen | grep -E ':21|:22|:80' || true
```

Expected result:
`twistd` should be listening on **ports 21, 22, 80**.

---

## **8️⃣ Firewall & network**

```bash
sudo ufw allow 21/tcp
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status
```

---

## **9️⃣ Auto-start check**

```bash
sudo reboot
sudo service opencanary status
```

Should show:

```
opencanary running (pid XXXX)
```

---

## 🔟 Syslog → Kali Purple (Elastic Stack)

Edit:

```bash
sudo nano /etc/opencanaryd/opencanary.conf
```

Set:

```json
"syslog": { 
  "enabled": true,
  "host": "KALI_IP",
  "port": 514
}
```

---

## **11️⃣ Test the Honeypot**

From a Windows device (victim):

```powershell
curl http://<HONEYPOT_IP>/
telnet <HONEYPOT_IP> 21
ssh <HONEYPOT_IP> -p 22
```

Back on MX:

```bash
sudo tail -f /var/log/opencanary.log
```

---

## **12️⃣ Log rotation (recommended)**

```bash
sudo tee /etc/logrotate.d/opencanary >/dev/null <<'EOF'
/var/log/opencanary.log {
  weekly
  rotate 8
  compress
  missingok
  notifempty
  create 0640 opencanary opencanary
  postrotate
    /usr/sbin/service opencanary restart > /dev/null 2>&1 || true
  endscript
}
EOF
```

---

# ✅ Enjoy!


