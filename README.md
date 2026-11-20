# 🛡️ OfficeGuard RMM — Enterprise Network & Endpoint Management System

**OfficeGuard RMM** is a centralized, self-hosted Remote Monitoring and Management (RMM) & Endpoint Security solution designed to secure an office network, monitor employee productivity, enforce software installation policies, provide instant remote desktop access, and automate Windows credential rotation.

---

## ✨ Key Features

### 1. 👁️ Real-Time Employee Activity & Screen Monitoring
* **Live Screen Streaming:** View real-time screens of any connected office PC directly from the admin dashboard via WebRTC / Socket.io.
* **Automated Screenshots:** Captures periodic screenshots (configurable intervals, e.g., every 15–30 seconds) and stores them securely on the local server.
* **Application & Web Tracking:** Logs active foreground windows, software usage duration, and visited URLs to distinguish between productive work and off-task activities.

### 2. 🚫 Software Installation & Execution Restriction
* **Privilege Enforcement:** Enforces Standard User privileges on staff accounts while isolating Local Administrator credentials.
* **AppLocker / GPO Automation:** Pushes Windows Group Policy and AppLocker rules from the central server to block unauthorized `.exe`, `.msi`, and portable USB applications.
* **Admin Approval Workflow:** Staff cannot install any third-party software without explicit remote authorization or admin credentials.

### 3. 🖥️ Instant Remote Desktop Access
* **Browser-Based Remote Control:** Take full mouse and keyboard control of any office PC directly through the web dashboard without third-party cloud dependencies.
* **RustDesk / WebRTC Integration:** Low-latency peer-to-peer connection over the local office LAN for troubleshooting and IT support.

### 4. 🔐 Centralized Password Management & 30-Day Auto-Rotation
* **Automated Password Rotation:** A server-side Cron Job automatically generates strong, unique passwords every **30 days** for every registered workstation.
* **Zero-Touch Update:** The client agent receives the encrypted payload via secure WebSocket and updates the local Windows account password silently using system-level privileges (`net user`).
* **Encrypted Vault:** All current and historical passwords are encrypted (AES-256) in the database and accessible only by the Master Admin.

### 5. 🌐 Network-Level Security & Firewall Integration
* **Device Inventory & Health:** Tracks IP/MAC addresses, CPU/RAM usage, disk health, and online/offline status in real time.
* **USB & Port Control:** Option to disable USB mass storage devices to prevent unauthorized data transfer.

---

## 🏗️ System Architecture

The system follows a **Client-Server Architecture** optimized for local office networks (LAN) or VPN-connected branches:

```text
+-------------------------------------------------------+
|              CENTRAL ADMIN DASHBOARD                  |
|            (Next.js / React + Tailwind CSS)           |
+---------------------------+---------------------------+
                            | HTTPS / WSS (Socket.io)
                            v
+-------------------------------------------------------+
|               BACKEND & REAL-TIME SERVER              |
|        (Node.js, Express, Socket.io, Node-Cron)       |
|         Database: MongoDB / PostgreSQL (AES-256)      |
+---------------------------+---------------------------+
                            | Encrypted WebSocket / WebRTC
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
+---------------+   +---------------+   +---------------+
|  CLIENT PC 1  |   |  CLIENT PC 2  |   |  CLIENT PC N  |
| Windows Agent |   | Windows Agent |   | Windows Agent |
| (C# / Python) |   | (C# / Python) |   | (C# / Python) |
+---------------+   +---------------+   +---------------+
```

---

## 🛠️ Tech Stack

| Layer | Technologies |
| :--- | :--- |
| **Admin Frontend** | Next.js (React), TypeScript, Tailwind CSS, Shadcn UI |
| **Backend Server** | Node.js, Express.js, Socket.io, Node-Cron |
| **Database & Cache** | MongoDB (Encrypted Vault & Logs), Redis (Live Session State) |
| **Client Agent (Windows)** | C# (.NET 8 Windows Service) or Python (`pywin32`, `mss`, `pyautogui`) |
| **Remote Desktop** | WebRTC / Self-Hosted RustDesk API |
| **Network Security** | pfSense / MikroTik Firewall + Pi-hole (DNS Filtering) |

---

## 📂 Project Directory Structure

```text
officeguard-rmm/
├── admin-dashboard/          # Next.js Web Application for Admin Control
│   ├── src/
│   │   ├── app/              # Dashboard pages (Live View, PCs, Vault, Logs)
│   │   ├── components/       # UI components (RemoteCanvas, PasswordCard)
│   │   └── lib/              # Socket & API client utilities
│   └── package.json
│
├── backend-server/           # Central Node.js & Socket.io Server
│   ├── src/
│   │   ├── config/           # DB & Encryption keys
│   │   ├── controllers/      # Auth, PC Management, Audit Logs
│   │   ├── jobs/             # 30-day Password Rotation Cron Job
│   │   ├── sockets/          # Real-time screen & command handlers
│   │   └── server.js
│   └── package.json
│
└── windows-agent/            # Background Windows Service (.exe)
    ├── AgentService/         # C# / Python system-level service
    │   ├── ScreenCapture/    # Screenshot & WebRTC stream module
    │   ├── ActivityTracker/  # Active window & process logger
    │   ├── PolicyEnforcer/   # Software install blocker & GPO rules
    │   └── PasswordUpdater/  # Local Windows user password updater
    └── installer/            # MSI / InnoSetup silent installer script
```

---

## 🚀 Getting Started

### Prerequisites
* **Server:** Ubuntu Server 22.04 LTS / Windows Server (Node.js v20+, MongoDB v7+)
* **Client Workstations:** Windows 10 / Windows 11 Pro
* **Network:** Static Local IP for the Central Server (e.g., `192.168.1.100`)

### 1. Clone the Repository
```bash
git clone https://github.com/your-username/officeguard-rmm.git
cd officeguard-rmm
```

### 2. Setup Backend Server
```bash
cd backend-server
npm install
cp .env.example .env
# Configure MONGO_URI, JWT_SECRET, and AES_ENCRYPTION_KEY in .env
npm run start
```

### 3. Setup Admin Dashboard
```bash
cd ../admin-dashboard
npm install
cp .env.example .env.local
# Set NEXT_PUBLIC_API_URL=http://192.168.1.100:5000
npm run build && npm run start
```

### 4. Deploy Windows Agent on Staff PCs
1. Build the agent executable from `/windows-agent` with your server's IP address embedded.
2. Run the installer as **Administrator** on each office PC:
   ```powershell
   OfficeGuardAgentSetup.exe /SILENT /SERVER="192.168.1.100:5000" /TOKEN="YOUR_ENROLLMENT_KEY"
   ```
3. The agent will register itself as a persistent **Windows Service** (`OfficeGuardSvc`) running under `NT AUTHORITY\SYSTEM`.

---

## 🔒 Security Best Practices
* **End-to-End Encryption:** Always use WSS (Secure WebSockets) and HTTPS with a local SSL certificate inside the office LAN.
* **Tamper Protection:** Configure the Windows Service recovery options to auto-restart if terminated, and restrict Task Manager / Services access for standard staff users.
* **Network Isolation:** Pair this software with a **pfSense** or **MikroTik** router to block unauthorized VPNs, proxies, and external DNS servers.

---

## ⚠️ Legal & Compliance Notice
> This software is intended strictly for **authorized corporate and office network administration**. Ensure that all employees are informed via company IT policy regarding workstation monitoring and acceptable use guidelines in accordance with local labor and privacy laws.

---

## 📄 License
This project is proprietary and intended for internal enterprise use. All rights reserved.
