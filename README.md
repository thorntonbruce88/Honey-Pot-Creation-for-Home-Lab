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
