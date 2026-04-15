# TITAN SMALL BUSINESS SERVICES: SECURITY ARCHITECTURE DOCUMENT (SAD)
**Operator:** Wesley Martinez-Close
**Date:** April 15, 2026

## 1. Perimeter Hardening (UFW & SSH)
* **SSH Status:** 

Disabled root login and password authentication by modifying /etc/ssh/sshd_config
i.e. (setting PermitRootLogin no and PasswordAuthentication no). 
Restarted the SSH service to enforce change
* **Firewall Logic:** 

Firewall Logic: > * Default Policy: Deny (incoming), Allow (outgoing)
Open Ports: >  22/tcp: Secure Shell (SSH) access and  8080/tcp: Web Application Frontend
Closed Ports: All other incoming ports are implicitly denied by the default policy to minimize the attack surface.


## 2. The Automated Auditor (Python)
* **Script Logic:**

import os

# Define the target IP (Ensure TITAN-DC01 is actually ON for this test)
dc_ip = "192.168.1.81"
log_path = "/var/log/dc_audit.log"

# Pings the DC 4 times (-c 4)
# We use > /dev/null to keep the console clean
response = os.system(f"ping -c 4 {dc_ip} > /dev/null 2>&1")

# Open the log file in append mode ('a')
with open(log_path, "a") as log_file:
    if response == 0:
        log_file.write("DC is UP\n")
    else:
        log_file.write("DC is DOWN\n")

* **Telemetry Path:** `/var/log/sys_audit.log`

## 3. Containerized App (Docker)
* **Network Isolation:** Isolated the Redis database by placing it on a private bridge network (backend_network) without mapping any 
ports to the host machine. This ensures only the Nginx frontend container can communicate with the database.

* **Stack Health:** [Paste output of `docker compose ps`]
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                   NAMES
abc68ddb873f   redis:latest   "docker-entrypoint.s…"   7 minutes ago   Up 7 minutes   6379/tcp                                wclose-backend-1
4caf9e74bca1   nginx:latest   "/docker-entrypoint.…"   7 minutes ago   Up 7 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   wclose-frontend-1

## 4. Executive Summary
The outpost has been hardened using a multi-layered security approach. By disabling password-based SSH 
and implementing a strict UFW firewall, the external attack surface is limited to only two essential ports. Internal security is 
further bolstered by container isolation and an automated auditing script that provides consistent telemetry on system health.
