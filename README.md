<img width="1536" height="1024" alt="CybersecurityHoneyPot" src="https://github.com/user-attachments/assets/6ed45f46-4706-4bdb-9728-debe1b091250" />
## Honey-Pot-Creation-for-Home-Lab

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

```

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

