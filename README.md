# 📡 WLAN-SOC: Wireless Traffic Analysis and Visualization

A lightweight, automated Security Operations Center (SOC) framework designed for passive wireless network traffic inspection. 

This platform pairs the **Suricata NIDS Engine** with a multi-threaded **Python Flask backend** to parse network telemetry and stream live security anomalies to a modern Tailwind CSS web dashboard.

---

&nbsp;

## 🛠️ System Architecture

```text
                                   [ Wireless Client ]
                                           │
                                      ▼ (Packets)
                    ┌──────────────── wlan0 / eth0 ────────────────┐
                    │       Suricata IDS Engine (Sniffing)         │
                    └─────────────────────┬────────────────────────┘
                                          │ Writes alerts to
                                          ▼
                    ┌─────────────── eve.json ─────────────────────┐
                    │       Flask API Orchestrator Backend         │
                    └─────────────────────┬────────────────────────┘
                                          ├─► Serves Web UI Dashboard (Port: 2000)
                                          └─► Spawns Ngrok Safe Tunnel
                                          │
                                          ▼
                                 [ Remote Public URL ]
```
<br>

### 🗺️ Operational Component Breakdown

| Layer | Component | Technology | Description |
| :--- | :--- | :--- | :--- |
| 📥 **Ingestion** | IDS Engine | `Suricata` | Sniffs raw packets on the configured interface and outputs JSON alert telemetry. |
| ⚙️ **Backend** | Orchestrator | `Python 3` + `Flask` | Parses engine logs, manages core processes, and exposes a REST API. |
| 🌐 **Exposition**| Secure Edge | `Ngrok Agent` | Automatically exposes the local dashboard port safely to the internet. |
| 📊 **Frontend** | Dashboard | `HTML5` / `JS` / `Tailwind` | Client-side engine that pulls real-time analytics using live API polling. |
---
## 📋 Pre-Requisites & Requirements

Before deploying, ensure your host platform meets the following foundational environment metrics.
🖥️ Base Environment

   > Operating System: Linux (Kali Linux, Ubuntu, or Debian-based distributions).

   > Privileges: Complete sudo root execution rights.

  > Hardware: A network interface capable of promiscuous or monitor mode (wlan0, eth0).
<br>
🐍 Python Global Package Setup

  > [!CAUTION]
  > CRITICAL SERVICE REQUIREMENT <br>
    Because the background system daemon (systemd) executes the orchestrator script under the global root user space, all required Python dependencies must be installed globally.<br>Run the command below before triggering the installer to prevent immediate ModuleNotFoundError crashes.

Bash
```
sudo pip3 install flask flask-cors --break-system-packages
```
---

## 🚀 Chronological Step-by-Step Deployment Guide

Follow these configuration steps in exact sequential order to spin up the automated monitoring cluster.
#### Step 1: Add your Ngrok Authentication Token

  > Install and retrieve your custom Auth Token from your personal Ngrok Dashboard.

  > Authenticate your local tracking agent:

Bash
```
sudo ngrok config add-authtoken <YOUR_NGROK_AUTH_TOKEN>
```
<br>

#### Step 2: Deploy Custom Suricata Configuration

  > This repository includes a customized suricata.yaml file pre-optimized specifically for local wireless network log extraction and interface handling.


Bash
```
sudo cp suricata/suricata.yaml /etc/suricata/suricata.yaml
```

  > Eve-Log JSON Stream Enabled: Explicitly ensures that eve.json output targets are toggled on with alert structures active so the Flask parser can query events.

  > Targeted Local Rule Paths: Maps directly to your custom `/etc/suricata/rules/local.rules` asset as the absolute tracking priority, stripping away heavy public lists that slow down initial testing loops.
<br>

#### Step 3: Implement Testing Signatures (local.rules)

  > This repository provides a tailored local.rules file containing optimized detection rules to verify that the end-to-end telemetry pipeline is working smoothly.

  > Ensure the core rules storage directory tree exists on your host:

Bash
```
sudo mkdir -p /etc/suricata/rules
```

  > Move the rule configuration files from your repository workspace to Suricata's rule matrix:

Bash
```
sudo cp suricata/local.rules /etc/suricata/rules/local.rules
```
>[!TIP]
> The rules are custom-built using specialized sid values ranging from 1,000,001 to 1,000,004 to prevent logic or database collisions with default public signatures. <br>They utilize the classtype:not-suspicious; classification tag to safely trigger frontend counters without generating false system security flags during initial setups.

  > If you wish to view or manually append custom operational rules to this engine layer later, edit the live path directly:

Bash
```
sudo nano /etc/suricata/rules/local.rules
```
<br>

#### Step 4: Fine-Tune Web Dashboard Ports

  > The ecosystem tracks configuration variables inside a centralized data file under the config/ directory to manage system interface bindings.

  > Open config/config.json and adjust the variables to line up with your local hardware targets:
    JSON
```
    {
        "interface": "wlan0",
        "port": 2000,
        "backup_port": 2001
    }
```

  >[!TIP]
  > If running an Ethernet-based monitoring architecture instead of an over-the-air asset, update the "interface" entry string from wlan0 to eth0.
  >To bind your dashboard tracking console onto a separate network layer, adjust the target "port" integer to any preferred alternative (e.g., 1212).
<br>

#### Step 5: Execute the Automated System Installer

  > The install.sh sequence installs missing software packages, automatically kills off conflicting web servers (such as native Apache2 instances) to keep port access clear, and safely mounts the background monitoring engine profile.

  > Grant the script executable and Kick off the automated deployment:

Bash
```
chmod +x install.sh
sudo ./install.sh
```
<br>
<br>

## 🚦Managing the Framework Service

  > The cluster runs seamlessly inside a single background Systemd unit profile named wlan-soc.service.
 
 🟢 Start the Service Core

Bash
```
sudo systemctl start wlan-soc
```

  🔄 Enable Automatic Boot-Time Persistence

Bash
```
sudo systemctl enable wlan-soc
```

   🌐 Retrieving Active Access Endpoints

  > [!IMPORTANT]
  > When evaluating system initialization metrics, pay close attention to the terminal console printouts. <br>
  The orchestrator script dynamically pulls your local listener socket and your secure public Ngrok proxy tunnel URL directly into the status block:

Bash
```
sudo systemctl status wlan-soc
```

<br>

> [!TIP]
> **Active Network Gateways:**
> * **Local Console:** `http://localhost:2000`
> * **Public Ngrok Access:** `https://xxxx-xx-xx-xx.ngrok-free.app`

<br>

  🪵 Tail the Live Output Telemetry Stream

  > Keep a live console session running to monitor telemetry traffic passing through the analytics engine processing loops:

Bash
```
sudo journalctl -u wlan-soc -f
```

  🔒 Safe Service Termination

  > [!WARNING]
  > Stopping the service via Systemd triggers an automated teardown script sequence.<br>
  This operation instantly drops the open Ngrok socket connection, stops background sniffing tasks, and frees system network ports to prevent interface locks.

  > To halt execution processing lines safely, run:

Bash
```
sudo systemctl stop wlan-soc
```

