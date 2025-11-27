<img width="1536" height="1024" alt="CybersecurityHoneyPot" src="https://github.com/user-attachments/assets/6ed45f46-4706-4bdb-9728-debe1b091250" />
🍯 Honey-Pot-Creation-for-Home-Lab

# T-Pot Honeypot Deployment on Ubuntu 22.04

## 📖 Project Overview
This repository documents the full lifecycle of deploying the [T-Pot honeypot framework](https://github.com/telekom-security/tpotce) on a freshly installed Ubuntu 22.04.5 LTS system.  
The goal was to evaluate T-Pot’s reliability in a home lab environment, focusing on Docker stability, apt keyring integrity, and secure SSH access.

---

## 🖥️ Environment Setup

- **Host OS**: Ubuntu 22.04.5 LTS (clean install)
- **Kernel**: `TOKEN_KERNEL_VERSION`
- **Architecture**: `TOKEN_ARCHITECTURE`
- **Network Interface**: `TOKEN_INTERFACE` with LAN IP `TOKEN_LAN_IP`
- **SSH Port**: `TOKEN_SSH_PORT` (reassigned by T-Pot installer)
- **Dashboard Port**: `TOKEN_DASHBOARD_PORT`

---

## ⚙️ Docker Installation & Keyring Repair

Initial Docker installation failed due to:
- Duplicate entries in `/etc/apt/sources.list.d/docker.list`
- Conflicting `signed-by` clauses (`docker.asc` vs `docker.gpg`)
- GPG key mismatch (`NO_PUBKEY TOKEN_DOCKER_KEY_ID`)

### ✅ Resolution Steps
1. **Cleaned repo file**:
   ```bash
   sudo nano /etc/apt/sources.list.d/docker.list
   ```
   Replaced with:
   ```
   deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable
   ```

2. **Re-imported Docker GPG key**:
   ```bash
   sudo rm -f /etc/apt/keyrings/docker.gpg
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
   sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   sudo chmod a+r /etc/apt/keyrings/docker.gpg
   ```

3. **Verified Docker install**:
   ```bash
   sudo apt update
   sudo apt install docker-ce docker-ce-cli containerd.io -y
   sudo systemctl status docker
   sudo docker info | grep "Storage Driver"
   ```
   Result: Docker running with `TOKEN_STORAGE_DRIVER`.

---

## 🛡️ T-Pot Installation

Cloned the official repo and ran the installer:
```bash
git clone https://github.com/telekom-security/tpotce
cd tpotce
sudo ./install.sh
```

### ✅ Installer Highlights
- Pulled honeypot containers: `cowrie`, `dionaea`, `conpot`, `heralding`, `suricata`, `wordpot`, etc.
- Reassigned SSH to port `TOKEN_SSH_PORT`
- Exposed dashboard on port `TOKEN_DASHBOARD_PORT`
- Created default user `TOKEN_DEFAULT_USER` with password `TOKEN_DEFAULT_PASSWORD`

---

## 🔍 Post-Install Verification

- **SSH Access**:
  ```bash
  ssh -p TOKEN_SSH_PORT TOKEN_DEFAULT_USER@TOKEN_LAN_IP
  ```
- **Running Containers**:
  ```bash
  sudo docker ps
  ```
- **Dashboard Access**:
  ```
  https://TOKEN_LAN_IP:TOKEN_DASHBOARD_PORT
  ```
  Default creds: `TOKEN_DEFAULT_USER / TOKEN_DEFAULT_PASSWORD` (change immediately).

---

## 🔍 Verification

After installation and reboot, verify that T-Pot is running as a managed service:

```bash
sudo systemctl status tpot
```

### ✅ Example Healthy Output
```text
● tpot.service - T-Pot
     Loaded: loaded (/etc/systemd/system/tpot.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-11-27 20:23:44 UTC; 5min ago
    Process: 1171 ExecStartPre=/usr/bin/docker compose -f /home/TOKEN_USER/tpotce/docker-compose.yml down -v (code=exited, status=0/SUCCESS)
   Main PID: 1196 (docker)
      Tasks: 19 (limit: 9247)
     Memory: 56.2M
        CPU: 1.415s
     CGroup: /system.slice/tpot.service
             ├─1196 /usr/bin/docker compose -f /home/TOKEN_USER/tpotce/docker-compose.yml up
             └─1213 /usr/libexec/docker/cli-plugins/docker-compose compose -f /home/TOKEN_USER/tpotce/docker-compose.yml up

Nov 27 20:29:12 boyhowdy docker[1213]: ewsposter            |  => Starting Ipphoney Honeypot Modul.
Nov 27 20:29:12 boyhowdy docker[1213]: ewsposter            |  => Starting Log4Pot Honeypot Modul.
Nov 27 20:29:12 boyhowdy docker[1213]: ewsposter            |  => Starting Mailoney Honeypot Modul.
Nov 27 20:29:12 boyhowdy docker[1213]: ewsposter            |  => Starting Medpot Honeypot Modul.
Nov 27 20:29:12 boyhowdy docker[1213]: ewsposter            |  => Starting Wordpot Honeypot Modul.
```

### 🔑 What to Look For
- **Loaded** → Service file exists and is enabled (will start on boot).  
- **Active (running)** → T-Pot is currently running.  
- **Main PID** → Managed by Docker Compose.  
- **Logs** → Honeypot modules (Cowrie, Dionaea, Conpot, Heralding, Suricata, Wordpot, etc.) are being initialized.

---

### 🧭 Additional Checks
- List containers:
  ```bash
  sudo docker ps
  ```
- Access dashboard:
  ```
  https://TOKEN_LAN_IP:TOKEN_DASHBOARD_PORT
  ```
- Confirm SSH honeypot responds on port `22` (while management SSH is on `TOKEN_SSH_PORT`).

---


## 🧠 Skills Demonstrated
- Advanced troubleshooting of apt keyring and repo conflicts
- Docker daemon recovery and storage driver validation
- Secure SSH reconfiguration and multi-port access management
- Systematic verification of honeypot container health and dashboard availability

---

## 📌 Notes
- `TOKEN_STORAGE_DRIVER` was used successfully despite prior kernel compatibility concerns.
- All honeypot containers deployed without port binding conflicts.
- Future hardening steps include rotating `TOKEN_DEFAULT_USER` credentials and isolating honeypot traffic via VLAN segmentation.

---

## 📷 Screenshots
<img width="1584" height="799" alt="image" src="https://github.com/user-attachments/assets/8ed74f98-6f54-4197-a4cb-2196930b3061" />

<img width="1590" height="793" alt="image" src="https://github.com/user-attachments/assets/738db769-183a-41d7-8cb6-4323e2e24e96" />


---

### 🔑 Token Legend
- `TOKEN_KERNEL_VERSION` → your kernel version  
- `TOKEN_ARCHITECTURE` → system architecture (e.g., x86_64)  
- `TOKEN_INTERFACE` → network interface name (e.g., eno1)  
- `TOKEN_LAN_IP` → your LAN IP address  
- `TOKEN_SSH_PORT` → reassigned SSH port (e.g., 64295)  
- `TOKEN_DASHBOARD_PORT` → dashboard port (e.g., 64297)  
- `TOKEN_DOCKER_KEY_ID` → Docker GPG key ID (e.g., 7EA0A9C3F273FCD8)  
- `TOKEN_STORAGE_DRIVER` → storage driver (e.g., overlayfs)  
- `TOKEN_DEFAULT_USER` → default user created by installer (e.g., tsec)  
- `TOKEN_DEFAULT_PASSWORD` → default password (e.g., tsec)  

---

# Security Policy

## 🔒 Supported Versions
This repository documents a deployment of **T-Pot honeypot framework** on Ubuntu 22.04.  
Security practices described here apply to the following environment:

- **OS**: Ubuntu LTS (22.04 and newer)
- **Docker**: Latest stable release
- **T-Pot**: Current release from [telekom-security/tpotce](https://github.com/telekom-security/tpotce)

---

## ⚠️ Responsible Use
T-Pot is a **honeypot framework** designed to attract, log, and analyze malicious traffic.  
It should **never** be deployed on production systems or networks without proper segmentation.

- Run only in **isolated lab environments** or controlled research networks.
- Do not expose honeypots to sensitive internal systems.
- Treat all captured data as potentially malicious.

---

## 🛡️ Hardening Recommendations
After installation, apply the following security measures:

1. **Change Default Credentials**
   - Replace `TOKEN_DEFAULT_USER` / `TOKEN_DEFAULT_PASSWORD` immediately.
   - Use strong, unique passwords.

2. **Restrict Access**
   - Limit SSH (`TOKEN_SSH_PORT`) and dashboard (`TOKEN_DASHBOARD_PORT`) access to trusted IP ranges.
   - Use firewalls (e.g., `ufw` or `iptables`) to enforce rules.

3. **Network Segmentation**
   - Place honeypots in a dedicated VLAN or subnet.
   - Prevent lateral movement into production networks.

4. **System Updates**
   - Regularly run:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```
   - Keep Docker and T-Pot images up to date.

5. **Monitoring & Logging**
   - Review container logs with:
     ```bash
     sudo docker logs <container-id>
     ```
   - Forward logs to a SIEM or secure storage for analysis.

---

## 📢 Reporting Vulnerabilities
If you discover a security issue in this repository or deployment process:

- **Do not open a public GitHub issue.**
- Contact the maintainer directly via GitHub profile email.
- Provide:
  - Steps to reproduce
  - Affected components
  - Suggested remediation

---

## ✅ Disclosure Policy
- Vulnerabilities will be acknowledged within **7 days**.
- Fixes or mitigations will be documented in the repository.
- Critical issues will trigger an immediate update to the `README.md` and deployment notes.

---

## 📌 Notes
This repository is for **educational and research purposes only**.  
The maintainer assumes no liability for misuse of the instructions or framework.  
Always deploy honeypots responsibly and in compliance with local laws and organizational policies.


```
