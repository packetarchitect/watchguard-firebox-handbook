# Firebox Admin Setup — Local Management Interface

> **WatchGuard Firebox / Fireware — Locally Managed**
>
> This lesson covers WSM, Policy Manager, Firebox System Manager (FSM), Fireware Web UI, Fireware CLI, management credentials, timeout behavior, diagnostics, Policy Checker, authentication testing, and a practical local-management workflow.

---

## 1. Local Management Toolset

```text
                         LOCALLY MANAGED FIREBOX
                                  |
          +-----------------------+-----------------------+
          |                       |                       |
          v                       v                       v
        WSM                  Fireware Web UI          Fireware CLI
 WatchGuard System Manager       Browser                 SSH
          |                       |                       |
          +----------+------------+-----------+-----------+
                     |                        |
                     v                        v
              Policy Manager             Diagnostics
                     |
                     v
             Configuration (.xml)
```

WatchGuard identifies Fireware Web UI, WSM, and the Fireware CLI as Firebox management/monitoring tools. WSM is particularly useful when managing multiple Fireboxes.

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/overview/fireware/intro_to_fireware_c.html

---

# 2. WSM — WatchGuard System Manager

## What is WSM?

**WSM = WatchGuard System Manager.**

WSM is a Windows-based management application used to connect to WatchGuard devices and launch administration/monitoring tools.

From WSM you can launch tools such as:

- Policy Manager
- Firebox System Manager (FSM)
- HostWatch
- Ping
- Other Firebox management tools
- Management Server tools, when a Management Server is configured

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/centralized_management/tools_start_wsm.html

### IMAGE — WSM Main Window

![WSM main window](images/01_wsm_menu.png)

The screenshot shows the important WSM options:

- `Connect to Server...`
- `Connect to Device...`
- `RapidDeploy`

---

# 3. Connect to Device vs Connect to Server

This is a key WSM concept.

## Connect to Device

**Connect to Device = connect directly to a Firebox.**

```text
Your PC
   |
   | WSM
   v
Firebox
   |
   +--> Policy Manager
   +--> Firebox System Manager
   +--> Device status
```

### Basic steps

1. Open WSM.
2. Select **File > Connect to Device**.
3. Enter the Firebox IP address/hostname.
4. Enter the appropriate Device Management credentials.
5. Select the authentication server if required.
6. Log in.

**Official:** https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/fireware/installation/firebox_connect_wsm.html

---

## Connect to Server

**Connect to Server = connect WSM to a WatchGuard Management Server.**

It does **not** mean connecting directly to the Firebox.

```text
WSM
 |
 | Connect to Server
 v
WatchGuard Management Server
 |
 +---- Firebox A
 +---- Firebox B
 +---- Firebox C
```

This is useful for centralized management environments.

### Basic steps

1. Open WSM.
2. Select **File > Connect to Server**.
3. Enter/select the Management Server hostname/IP.
4. Enter the Management Server credentials/passphrase.
5. Log in.

**Official:** https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/fireware/management_server/mgmt_server_connect_wsm.html

### Easy memory trick

| WSM option | Connects to | Purpose |
|---|---|---|
| **Connect to Device** | Firebox | Direct Firebox management |
| **Connect to Server** | Management Server | Centralized management |

---

# 4. Passphrase vs Password

WatchGuard commonly uses the term **passphrase** for management credentials.

For practical purposes, it is the credential used to authenticate a management account, but WatchGuard distinguishes different passphrases for different accounts/services.

## Status Passphrase

The built-in `status` account is associated with the **Device Monitor** role.

```text
READ-ONLY
```

It can review configuration/status information but cannot save configuration changes.

## Configuration Passphrase

The built-in `admin` account is associated with the **Device Administrator** role.

```text
READ + WRITE
```

It can make and save configuration changes.

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/wsc/wg_passphrases_about_c.html

### Built-in accounts

| Account | Role | Access |
|---|---|---|
| `admin` | Device Administrator | Read/write |
| `status` | Device Monitor | Read-only |

WatchGuard recommends changing default passphrases whenever a Firebox is newly set up or factory-reset.

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/basicadmin/firebox_security_best_practices.html

---

# 5. What is WSM Timeout?

The WSM **Timeout** is the amount of time, in seconds, that WSM waits for data from the device/server before reporting that it cannot connect.

Example:

```text
Timeout = 30 seconds

WSM attempts connection
        |
        | waits for response
        v
    30 seconds
        |
        v
No response -> timeout message
```

### Why increase it?

A higher timeout can help when connecting across:

- Slow WAN links
- High-latency links
- Congested networks
- Remote Fireboxes

### Important

**WSM Timeout is NOT the administrator login/session timeout.**

It is a **connection-response timeout**.

**Official:** https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/fireware/installation/firebox_connect_wsm.html

---

# 6. Policy Manager

## What is Policy Manager?

Policy Manager is the WatchGuard configuration editor.

It is used to:

- Create configurations
- Open configurations
- Modify firewall policies
- Modify network/security settings
- Save configuration changes to the Firebox

Firebox configuration files use the `.xml` extension.

**Official:** https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/basicadmin/config_files_open_about_wsm.html

### IMAGE — Policy Manager

![Policy Manager](images/02_policy_manager.png)

The screenshot shows policies such as:

- FTP-proxy
- HTTP-proxy
- HTTPS-proxy
- WatchGuard Web UI
- DNS
- Ping
- WatchGuard management
- Network Traversal
- Outgoing

---

# 7. Local Configuration vs Live Firebox

A critical concept:

```text
Policy Manager
      |
      | Edit
      v
Configuration
      |
      | Save To Firebox
      v
Live Firebox
```

A change made to a locally stored configuration does **not** become active on the Firebox until the configuration is saved to the device.

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/system_status/config_pages_about_web.html

### Interview question

**Q: I changed a firewall policy in Policy Manager. Is it immediately active?**

**A:** No. The configuration must be saved to the Firebox before the change takes effect.

---

# 8. Does the Firebox Allow Only One Admin Connection?

This needs a correction to the simplified statement in the lesson.

It is better **not** to memorize:

> "A Firebox only allows one admin connection."

WatchGuard's current documentation describes **configuration locking and role-based administration**.

For a managed device, when Policy Manager is opened for that device, WSM places a configuration lock to prevent simultaneous configuration changes by another user. The lock is released when Policy Manager closes or switches to another device.

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/content/en-US/Fireware/policies/policy_mgr_open_wsm.html

For locally managed Fireboxes, Fireware also supports multiple Device Administrators. In Fireware Web UI, when more than one Device Administrator can connect, the configuration file can be locked while one administrator is making changes.

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/role-based_admin/device-rba_users-roles_c.html

### Therefore

Your observation is useful, but the technically safer explanation is:

> **WatchGuard does not simply use a universal "one admin session only" rule. Management access and concurrent configuration changes are controlled through roles and configuration-lock behavior, depending on the management interface and management architecture.**

---

# 9. Firebox System Manager (FSM)

**FSM = Firebox System Manager.**

FSM is primarily used to monitor and administer the operational status of a connected Firebox.

It can provide:

- System status
- Fireware version
- Uptime
- Feature key information
- Certificates
- ARP cache operations
- Alarms
- FireCluster controls
- Backup/restore functions
- Reboot/shutdown
- Monitoring tools

**Official:** https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/fsm/fsm_about_wsm.html

### How to open FSM

```text
WSM
 |
 v
Device Status
 |
 v
Select Firebox
 |
 v
Firebox System Manager
```

**Official:** https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/fsm/fsm_start_wsm.html

### IMAGE — FSM Status Report

![FSM status report](images/03_fsm_status_report.png)

Use this view to establish:

- Device/model
- Fireware version
- Serial number
- CPU information
- System time
- Uptime
- Fault information

---

# 10. FireWatch / Network Discovery

Network Discovery can show devices connected to Firebox interfaces, including:

- IP address
- MAC address
- Hostname
- Operating system
- Open ports

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/system_status/network-discovery_web.html

### IMAGE — Network Discovery

![Network Discovery](images/06_network_discovery.png)

### Practical use

If a user says:

> "There is an unknown device on our LAN."

Network Discovery can help identify:

```text
IP
MAC
Hostname
OS
Open ports
```

### Performance note

Network Discovery consumes resources. On large networks, enable it only when needed and limit scans to the networks you actually need to monitor.

---

# 11. Fireware Web UI

Fireware Web UI is the browser-based management interface.

Typical URL:

```text
https://<Firebox-IP>:8080
```

The default Firebox Trusted interface is commonly:

```text
https://10.0.1.1:8080
```

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/content/en-US/Fireware/system_status/connecting_about_web.html

### IMAGE — Web UI Dashboard

![Fireware Web UI](images/05_webui_dashboard.png)

Typical sections include:

```text
Dashboard
System Status
Network
Firewall
Subscription Services
Authentication
VPN
System
```

---

# 12. Web UI vs Policy Manager

| Feature | Web UI | Policy Manager |
|---|---|---|
| Browser based | Yes | No |
| Monitor status | Yes | Limited |
| Configure policies | Yes | Yes |
| Edit configuration | Yes | Yes |
| Local XML workflow | Limited | Yes |
| Some advanced/legacy settings | Limited | More complete |

WatchGuard documents several configuration tasks that are available in Policy Manager but not in Fireware Web UI.

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/system_status/config_pages_about_web.html

---

# 13. Firewall Policies

### IMAGE — Firewall Policies

![Firewall Policies](images/08_firewall_policies.png)

A policy can be understood as:

```text
SOURCE
   |
   v
PROTOCOL / SERVICE / PORT
   |
   v
DESTINATION
   |
   v
ACTION
```

When troubleshooting traffic, determine:

1. Where traffic originates.
2. Where it is going.
3. Protocol.
4. Source/destination ports.
5. Which policy matches it.
6. What action the policy takes.

---

# 14. Policy Checker

### IMAGE — Policy Checker

![Policy Checker](images/09_policy_checker.png)

Policy Checker is one of the most useful troubleshooting tools.

It can test how the Firebox handles traffic for a specified:

- Interface
- Protocol
- Source IP
- Destination IP
- Source port
- Destination port

It can identify the policy that manages the tested traffic.

**Official:** https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/fireware/policies/policy_checker_web.html

### Example

```text
Source IP:
10.0.10.50

Destination:
203.0.113.50

Protocol:
TCP

Destination Port:
443
```

The goal is to answer:

> **Which Firebox policy handles this traffic?**

This is particularly useful when troubleshooting unexpected allow/deny behavior or policy-order issues.

---

# 15. Diagnostics

### IMAGE — Web UI Diagnostics

![Web UI Diagnostics](images/07_webui_diagnostics.png)

In Fireware Web UI:

```text
System Status
     |
     v
Diagnostics
     |
     v
Network
```

Useful tools include:

- Ping
- Traceroute
- DNS Lookup
- TCP Dump

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/system_status/stats_diagnostics_tasks_web.html

## Ping

Tests whether the Firebox can reach a destination.

```text
ping 8.8.8.8
```

## Traceroute

Shows the route/hops toward a destination.

Useful when:

```text
Destination unreachable
```

and you need to find where the path breaks.

## DNS Lookup

Useful when:

```text
IP connectivity works
but hostname access fails
```

## TCP Dump

Useful when you need packet-level evidence.

Example:

```text
-i eth0 port 443
```

It can help determine whether traffic arrives, leaves, and receives a response.

WatchGuard also supports saving the resulting capture as a PCAP.

---

# 16. Authentication Servers

### IMAGE — Authentication Servers

![Authentication servers](images/10_authentication_servers.png)

The authentication area can contain services such as:

- Firebox-DB
- RADIUS
- LDAP
- Active Directory
- AuthPoint, depending on deployment/version

### Why integrate authentication?

Instead of maintaining every user locally:

```text
User
 |
 v
Firebox
 |
 +--> Active Directory
 +--> RADIUS
 +--> LDAP
```

This allows the Firebox to use existing organizational identity infrastructure.

---

# 17. Authentication Troubleshooting

If authentication fails, do not immediately assume the password is wrong.

Use:

```text
1. Can Firebox reach authentication server?
          |
          v
2. Is DNS working?
          |
          v
3. Is required connectivity available?
          |
          v
4. Does the server connection test succeed?
          |
          v
5. Does the user authentication succeed?
```

This separates:

```text
NETWORK PROBLEM
```

from:

```text
AUTHENTICATION PROBLEM
```

---

# 18. Fireware CLI

The Fireware CLI provides command-line administration and diagnostics.

Network-based CLI access uses SSH on TCP **4118** by default for trusted/optional networks.

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/overview/fireware/intro_to_fireware_c.html

### IMAGE — PuTTY

![PuTTY CLI connection](images/11_putty_cli_connection.png)

The uploaded example shows:

```text
Host:
10.0.10.1

Port:
4118

Connection:
SSH
```

Typical connection:

```bash
ssh admin@10.0.10.1 -p 4118
```

Use the actual management IP and credentials for your Firebox.

---

# 19. CLI Diagnostics

### IMAGE — Fireware CLI

![Fireware CLI](images/12_fireware_cli.png)

A useful starting point is:

```text
diagnose ?
```

This can expose diagnostic areas supported by the Fireware version.

The CLI is useful when:

- Web UI is unavailable
- WSM is unavailable
- You need detailed diagnostics
- You need to troubleshoot remotely
- You need a command not exposed in the GUI

**Official CLI reference:** https://www.watchguard.com/help/docs/fireware/12/en-US/CLI/CLI_Reference_v12_10.pdf

---

# 20. Practical Local Management Setup

## Step 1 — Connect the laptop

Example:

```text
Firebox Trusted:
10.0.10.1/24

Laptop:
10.0.10.10/24
```

The laptop must be able to reach the Firebox Trusted interface.

## Step 2 — Test connectivity

```bash
ping 10.0.10.1
```

If it fails, check:

- Cable
- Interface status
- VLAN
- IP address
- Subnet
- Firewall/network path
- Firebox state

## Step 3 — Open Web UI

```text
https://10.0.10.1:8080
```

Log in with the correct Device Management account.

## Step 4 — Connect WSM to the Firebox

```text
WSM
 |
 +--> File
      |
      +--> Connect to Device
```

Enter:

```text
IP / Hostname
Username
Passphrase
Authentication Server
Timeout
```

## Step 5 — Open FSM

```text
WSM
 |
 +--> Device Status
      |
      +--> Firebox
           |
           +--> FSM
```

Check:

- Model
- Fireware version
- Uptime
- Faults
- System information

## Step 6 — Open Policy Manager

Use Policy Manager to inspect or modify configuration.

## Step 7 — Save deliberately

```text
Edit
 |
 v
Review
 |
 v
Save To Firebox
 |
 v
Live configuration
```

---

# 21. Troubleshooting Workflow

```text
START
  |
  v
Can I reach the Firebox?
  |                 |
 NO                 YES
  |                  |
  v                  v
L1/L2/IP          Can I login?
troubleshooting     |       |
                  NO        YES
                   |         |
                   v         v
             Credentials   Can traffic
             / access      pass?
                            |   |
                           NO   YES
                            |
                            v
                     Policy Checker
                     Diagnostics
                     Logs / TCP Dump
```

---

# 22. Example Troubleshooting Scenario

### Problem

Client:

```text
10.0.10.50
```

cannot reach:

```text
203.0.113.50:443
```

### Step 1 — Verify client addressing

Confirm:

```text
IP
Subnet
Gateway
DNS
```

### Step 2 — Test from Firebox

Use:

```text
Ping
Traceroute
DNS Lookup
```

### Step 3 — Run Policy Checker

Test:

```text
Source:
10.0.10.50

Destination:
203.0.113.50

Protocol:
TCP

Destination Port:
443
```

### Step 4 — Identify the matching policy

Investigate the policy identified by Policy Checker.

### Step 5 — Capture packets

If policy matching appears correct:

```text
TCP Dump
```

Check whether packets:

```text
arrive
  |
  v
are processed
  |
  v
leave
  |
  v
receive response
```

---

# 23. Secure Management

Management access should not be exposed unnecessarily.

WatchGuard recommends limiting management access and recommends VPN-based remote management rather than broadly exposing management interfaces to the Internet.

Preferred:

```text
Admin
  |
  v
VPN
  |
  v
Trusted management network
  |
  v
Firebox
```

Avoid:

```text
Internet
   |
   v
Any-External
   |
   v
Firebox Web UI
```

**Official:** https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/system_status/connect_webui_external.html

---

# 24. Which Tool Should I Use?

| Situation | Tool |
|---|---|
| Connect directly to Firebox | **WSM — Connect to Device** |
| Connect to Management Server | **WSM — Connect to Server** |
| Build/change configuration | **Policy Manager** |
| Monitor device status | **FSM** |
| Browser-based administration | **Fireware Web UI** |
| Ping/traceroute/DNS/TCP dump | **Web UI Diagnostics / FSM diagnostics** |
| Determine matching firewall policy | **Policy Checker** |
| Discover LAN devices | **Network Discovery** |
| Advanced CLI troubleshooting | **Fireware CLI** |
| Remote secure administration | **VPN + management interface** |

---

# 25. Interview Questions

### Q1. What is WSM?

**Answer:** WSM stands for WatchGuard System Manager. It is a Windows management application used to connect to and administer WatchGuard Fireboxes and, where deployed, WatchGuard Management Servers.

### Q2. Connect to Device vs Connect to Server?

**Answer:** Connect to Device connects WSM directly to a Firebox. Connect to Server connects WSM to a WatchGuard Management Server.

### Q3. Status vs Configuration passphrase?

**Answer:** The status passphrase is associated with the read-only Device Monitor role. The configuration passphrase is associated with the Device Administrator role and permits configuration changes.

### Q4. What is WSM Timeout?

**Answer:** The amount of time WSM waits for data from the Firebox or Management Server before reporting a connection timeout. It is not an admin session timeout.

### Q5. What is Policy Manager?

**Answer:** A WatchGuard configuration tool used to create, inspect, modify, and save Firebox configuration files.

### Q6. Are Policy Manager changes immediately live?

**Answer:** No. The configuration must be saved to the Firebox.

### Q7. What is FSM?

**Answer:** Firebox System Manager is used to monitor and administer the operational status of a connected Firebox.

### Q8. What is Policy Checker?

**Answer:** It tests how the Firebox handles specified traffic and helps identify the policy that manages that traffic.

### Q9. What diagnostics are available?

**Answer:** Ping, traceroute, DNS Lookup, and TCP Dump are key network diagnostics.

### Q10. What port does the Fireware CLI use?

**Answer:** TCP **4118** by default for network-based SSH CLI access from trusted/optional networks.

---

# 26. One-Minute Revision

```text
WSM
 |
 +--> Connect to Device = Firebox
 |
 +--> Connect to Server = Management Server
 |
 +--> Launch Policy Manager
 |
 +--> Launch FSM

Policy Manager
 |
 +--> Configuration
 +--> Policies
 +--> Save To Firebox

FSM
 |
 +--> Status
 +--> Monitoring
 +--> Diagnostics
 +--> Operational controls

Web UI
 |
 +--> https://<IP>:8080
 +--> Dashboard
 +--> Policies
 +--> Diagnostics
 +--> Authentication
 +--> Network

CLI
 |
 +--> SSH
 +--> TCP 4118
 +--> Diagnostics

Credentials
 |
 +--> status = read-only
 +--> admin = read/write

Timeout
 |
 +--> WSM response wait time
 +--> NOT admin session timeout
```

---

# 27. Hands-On Lab Checklist

- [ ] Connect laptop to Firebox Trusted interface
- [ ] Configure laptop IP
- [ ] Ping Firebox
- [ ] Open `https://<Firebox-IP>:8080`
- [ ] Login as Device Administrator
- [ ] Verify Fireware version
- [ ] Open WSM
- [ ] Connect to Device
- [ ] Test/understand WSM timeout
- [ ] Open FSM
- [ ] Review Status Report
- [ ] Review system information
- [ ] Open Policy Manager
- [ ] Inspect existing policies
- [ ] Make a harmless test configuration change
- [ ] Confirm it is not live until saved
- [ ] Save configuration to Firebox
- [ ] Open Web UI Diagnostics
- [ ] Run Ping
- [ ] Run DNS Lookup
- [ ] Run Traceroute
- [ ] Review Firewall Policies
- [ ] Run Policy Checker
- [ ] Review Authentication Servers
- [ ] Test authentication-server connectivity
- [ ] Connect to CLI using SSH/4118
- [ ] Run `diagnose ?`
- [ ] Record useful commands

---

# 28. Image Placement / Notes

The following screenshots from your uploaded lesson are already included in the accompanying package:

1. `01_wsm_menu.png` — WSM main menu
2. `02_policy_manager.png` — Policy Manager
3. `03_fsm_status_report.png` — FSM Status Report
4. `04_fsm_diagnostics.png` — FSM diagnostics
5. `05_webui_dashboard.png` — Fireware Web UI
6. `06_network_discovery.png` — Network Discovery
7. `07_webui_diagnostics.png` — Web UI Diagnostics
8. `08_firewall_policies.png` — Firewall Policies
9. `09_policy_checker.png` — Policy Checker
10. `10_authentication_servers.png` — Authentication Servers
11. `11_putty_cli_connection.png` — PuTTY/SSH CLI
12. `12_fireware_cli.png` — Fireware CLI

The earlier deployment-option screenshots are better kept in the previous **Deployment Options / RapidDeploy** lesson rather than repeated here.

---

# Official References

- https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/overview/fireware/intro_to_fireware_c.html
- https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/fireware/installation/firebox_connect_wsm.html
- https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/fireware/management_server/mgmt_server_connect_wsm.html
- https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/basicadmin/config_files_open_about_wsm.html
- https://www.watchguard.com/help/docs/help-center/en-US/content/en-US/Fireware/policies/policy_mgr_open_wsm.html
- https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/fsm/fsm_start_wsm.html
- https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/fsm/fsm_about_wsm.html
- https://www.watchguard.com/help/docs/help-center/en-US/content/en-US/Fireware/system_status/connecting_about_web.html
- https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/system_status/stats_diagnostics_tasks_web.html
- https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/fireware/policies/policy_checker_web.html
- https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/system_status/network-discovery_web.html
- https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/wsc/wg_passphrases_about_c.html
- https://www.watchguard.com/help/docs/fireware/12/en-US/CLI/CLI_Reference_v12_10.pdf

---

# Key Takeaways

1. **WSM = WatchGuard System Manager.**
2. **Connect to Device = direct Firebox connection.**
3. **Connect to Server = Management Server connection.**
4. **Status = read-only; admin/configuration = read/write.**
5. **WSM Timeout is a connection-response timeout, not an admin-session timeout.**
6. **Policy Manager builds/changes configuration; changes must be saved to become live.**
7. **FSM focuses on device status, monitoring, and operational administration.**
8. **Web UI provides browser-based management, normally over HTTPS/8080.**
9. **Policy Checker identifies how a specified flow is handled by the policy set.**
10. **Diagnostics provide Ping, Traceroute, DNS Lookup, and TCP Dump.**
11. **CLI provides an additional troubleshooting path over SSH/TCP 4118.**
12. Do not reduce the management model to "only one admin can connect"; understand **roles and configuration locking**.
13. Prefer **VPN-based remote management** over broadly exposing management interfaces to the Internet.
