# 📡 WLAN-SOC: Wireless Traffic Analysis and Visualization

A lightweight, automated Security Operations Center (SOC) framework designed for passive wireless network traffic inspection. 

This platform pairs the **Suricata NIDS Engine** with a multi-threaded **Python Flask backend** to parse network telemetry and stream live security anomalies to a modern Tailwind CSS web dashboard.

---

## 🛠️ System Architecture
```bash
                                                [ Wireless Client ]
                                                         │
                                                         ▼ (Packets)
                                       ┌─────────── wlan0 / eth0 ───────────┐
                                       │  Suricata IDS Engine (Sniffing)    │
                                       └─────────────────┬──────────────────┘
                                                         │ Writes alerts to
                                                         ▼
                                       ┌───────────── eve.json ─────────────┐
                                       │  Flask API Orchestrator Backend    │
                                       └─────────────────┬──────────────────┘
                                                         ├─► Serves Web UI Dashboard (Port: 2000)
                                                         └─► Spawns Ngrok Safe Tunnel
                                                         │
                                                         ▼
                                                [ Remote Public URL ]

```

| Layer | Component | Technology | Description |
| :--- | :--- | :--- | :--- |
| **Ingestion** | IDS Engine | Suricata | Sniffs raw packets on the configured interface and outputs JSON alert telemetry. |
| **Backend** | Orchestrator | Python 3 + Flask | Parses engine logs, manages core processes, and exposes a REST API. |
| **Exposition** | Secure Edge | Ngrok Agent | Automatically exposes the local dashboard port safely to the internet. |
| **Frontend** | Dashboard | HTML5 / JS / Tailwind | Client-side engine that pulls real-time analytics using live API polling. |

---


## 📋 Pre-Requisites & Requirements

Before deploying, ensure your host platform meets the following foundational dependencies:

* **Operating System:** Linux (Kali Linux, Ubuntu, or Debian-based distributions).
* **Privileges:** Complete `sudo` root execution rights.
* **Hardware:** A network interface capable of promiscuous or monitor mode (`wlan0`, `eth0`).
* **Dependencies Installed by Agent:** `suricata`, `python3`, `pip3`, `curl`, `jq`.

---

## Step-by-Step Deployment Guide

Follow these steps in order to provision and start the automated monitoring cluster.

### Step 1: Add your Ngrok Authentication Token

1. Retrieve your Auth Token from your personal [Ngrok Dashboard](https://dashboard.ngrok.com).
2. Authenticate the agent under `sudo` so the system service can access it:

```bash
sudo ngrok config add-authtoken <YOUR_NGROK_AUTH_TOKEN>
```

### Step 2: Deploy Custom Suricata Configuration (suricata.yaml)

This repository includes a customized suricata.yaml file optimized for wireless log extraction.
🔍 What modifications are inside this custom YAML?
   
Eve-Log JSON Stream Enabled: It explicitly ensures that eve.json is turned on with alert types enabled so that the Flask backend can parse them.

   Optimized Local Rule Paths: It points directly to `/etc/suricata/rules/local.rules` as the primary classification engine, bypassing massive public lists that slow down initial testing.


🛠️ Deployment Instructions:

Copy the repository's custom YAML file directly over the default system installation file:
Bash
```bash
sudo cp config/suricata.yaml /etc/suricata/suricata.yaml
```

Step 3: Implement Testing Signatures (local.rules)

This repository provides a tailored local.rules file containing optimized detection rules to verify that the end-to-end telemetry mapping works perfectly.
🔍 What modifications are inside this custom rules file?

The rules are custom-built with specific sid tags (1,000,001 to 1,000,004) to prevent collisions with default public signatures. They also use the explicit classtype:not-suspicious; classification so they safely trigger the UI counters without flagging false critical events during setup.

Ensure the rules directory exists on your system host:
   Bash

      sudo mkdir -p /etc/suricata/rules

Copy the rules file from this repository into Suricata's active rule pathway:
   Bash

      sudo cp config/local.rules /etc/suricata/rules/local.rules

If you want to view or manually append signatures to this file, you can modify it at any time:
   Bash

    sudo nano /etc/suricata/rules/local.rules



Step 4: Fine-Tune Web Dashboard Ports

The repository includes a core configuration component under the config/ workspace directory to tell the backend engine exactly where to run.

Open config/config.json and verify your interface framework targets:
    JSON

    {
        "interface": "wlan0",
        "port": 2000,
        "backup_port": 2001
    }

   Note: If you want to sniff ethernet traffic instead of wireless, change "interface" to eth0. If you want to use a custom dashboard port, change "port" to your desired integer (e.g., 1212).

Step 5: Execute the Automated System Installer

The install.sh engine cleanly sets up backend packages, ensures conflicting web servers (like Apache2) are shut down to prevent port binding issues, and registers the background Systemd daemon tracking logic.

   Mark the script as executable:
    Bash

      chmod +x install.sh

Run the deployment sequence:
Bash

    sudo ./install.sh

🚦 Managing the Framework Service

The cluster runs seamlessly inside a single Systemd unit profile named wlan-soc.service.
Start the Service
Bash

      sudo systemctl start wlan-soc

Enable Automatic Boot-Time Execution
Bash

      sudo systemctl enable wlan-soc

Check Live Telemetry Logs & URLs

When checking the status of the service, pay close attention to the console printout logs. The script dynamically lists your local endpoint URL and your public Ngrok tunnel URL right in the output:
Bash

      sudo systemctl status wlan-soc

Expected Terminal View Output:
Plaintext

====================================================
[+] Dashboard UI and endpoints running on: [http://0.0.0.0:2000](http://0.0.0.0:2000)
====================================================
====================================================
[+] NGROK PUBLIC ACCESS URL: [https://xxxx-xx-xx-xx.ngrok-free.app](https://xxxx-xx-xx-xx.ngrok-free.app)
====================================================

Tail the Live Output Logs
Bash

      sudo journalctl -u wlan-soc -f

🔒 Safe Service Termination

Stopping the service automatically invokes an integrated cleanup routine that terminates background packet-sniffing tasks, drops the open Ngrok socket connection, and frees system network ports instantly.

To shut down the entire monitoring engine cleanly, run:
Bash

      sudo systemctl stop wlan-soc
