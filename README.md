# Windows Firewall Log Analysis on Windows 11
*A Windows-native approach using built-in tools*

---

## 1. Introduction

### 1.1 Objective
The objective of this project is to extract, inspect, and analyze Windows Firewall logs on a Windows 11 system using only native, built-in tools—primarily Windows PowerShell and default command-line utilities. The goal is to identify patterns in allowed traffic, detect blocked connection attempts, and highlight any activity that may indicate unauthorized access or potential data exfiltration.

### 1.2 Scope of Analysis
This analysis focuses exclusively on:
- The default Windows Firewall log (`pfirewall.log`)
- Metadata recorded by the firewall (not full packet contents)
- Local, endpoint-based visibility rather than network-wide monitoring
- Analysis performed on a single Windows 11 machine

No third-party applications, external log collectors, or commercial security tools are required. All investigation is performed within the constraints of Windows’ native capabilities.

### 1.3 Why Windows Firewall Logs Matter
Windows Firewall logs provide valuable insight into how a system communicates over the network. They can help:
- Reveal inbound connection attempts that were blocked by the firewall
- Show which outbound connections were allowed
- Identify unusual traffic patterns or activity outside normal behavior
- Support incident response and security investigations
- Serve as evidence of reconnaissance, intrusion attempts, or misuse

Even though Windows Firewall logs are lightweight and metadata-focused, they can act as a first layer of defense and an essential audit trail for detecting suspicious or unauthorized network activity.

### 1.4 Windows-Native Approach
Because the project relies only on built-in tools, it demonstrates:
- How to enable, access, and parse logs without external software
- How PowerShell can be used to filter and analyze large log files
- How investigators can perform meaningful security analysis using default Windows installations

This makes the methodology practical for real-world environments where installing additional software may be restricted or monitored.

---

## 2. Windows Firewall Logging Basics

Windows Firewall logging is **not always enabled by default** on Windows 11. If the log file or folder is missing, it usually means logging is turned off. This section explains what the logs contain, where they are stored, and how to enable them using only native Windows tools.

---

### 2.1 What Gets Logged
Windows Firewall logs record **metadata**, not full network packets. Each entry may include:

- Timestamp (date & time)
- Action (**ALLOW** or **DROP**)
- Source IP and Port
- Destination IP and Port
- Protocol (TCP, UDP, ICMP)
- Traffic Direction (Inbound/Outbound)
- Interface details

This data is sufficient to detect:
- Blocked connection attempts
- Outbound communication patterns
- Suspicious or unusual network behavior

> ✅ **Placeholder:** *Screenshot of sample log snippet in Notepad*

---

### 2.2 Default Log Location

If logging is enabled, the log file is typically stored at:

```text
C:\Windows\System32\LogFiles\Firewall\pfirewall.log
````

If this folder or file does **not** exist, logging is most likely **disabled**, and you’ll need to enable it.

> ✅ **Placeholder:** *Screenshot of File Explorer showing the log folder*

---

### 2.3 How to Enable Firewall Logging (Windows 11, Native Tools)

Logging can be activated using either the **GUI** or **PowerShell**—both built into Windows.

#### 2.3.1 Method 1: Enable Logging via Windows Firewall GUI

**Step 1: Open Advanced Firewall Settings**

1. Press **Start**
2. Type `wf.msc`
3. Press **Enter**

This launches **Windows Defender Firewall with Advanced Security**.

---

**Step 2: Open Firewall Properties**

1. In the right panel, select **Windows Defender Firewall Properties**

---

**Step 3: Enable Logging**

You will see three profiles:

* **Domain**
* **Private**
* **Public**

For each profile you care about:

1. Select the profile (e.g., **Private Profile**)

2. Click **Customize** under **Logging**

3. In the logging dialog, configure:

   * **Log dropped packets:** `Yes`
   * **Log successful connections:** `Yes` (recommended)
   * **Log file path:**

     ```text
     C:\Windows\System32\LogFiles\Firewall\pfirewall.log
     ```

     ✅ If permission issues occur, choose a folder you own, such as:

     ```text
     C:\Users\<YourName>\Documents\pfirewall.log
     ```

4. Adjust **Maximum file size** if desired

5. Click **OK**

6. Click **Apply**

✅ The log file will be created automatically once traffic occurs.

> ✅ **Placeholder:** *Screenshot: Logging settings window showing “Log dropped packets = Yes” and log path*

---

#### 2.3.2 Method 2: Enable Logging Using PowerShell (Native & Fast)

Open **PowerShell as Administrator** and run:

**Enable logging for dropped packets:**

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -LogDroppedPackets Enabled
```

**Enable logging for allowed connections:**

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -LogSuccessfulConnections Enabled
```

**Set log file path (default location):**

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -LogFileName "C:\Windows\System32\LogFiles\Firewall\pfirewall.log"
```

**If you get a permission error**, set the log to your Documents folder instead:

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -LogFileName "$env:USERPROFILE\Documents\pfirewall.log"
```

✅ Once traffic occurs, the log file will appear automatically at the chosen path.

---

### 2.4 Where to Check After Enabling

**Default path:**

```text
C:\Windows\System32\LogFiles\Firewall\
```

**User Documents path (if set):**

```text
C:\Users\<YourName>\Documents\
```

If the file is still missing, it may mean:

* No firewall events have occurred yet
* You configured the wrong profile (Public/Private/Domain)
* PowerShell wasn’t run as **Administrator**
* The log path points to a folder that doesn’t exist

---

### 2.5 Log Field Overview

| Field     | Meaning                 |
| --------- | ----------------------- |
| date      | Event date              |
| time      | Event time              |
| action    | `ALLOW` / `DROP`        |
| protocol  | `TCP` / `UDP` / `ICMP`  |
| src-ip    | Source IP address       |
| dst-ip    | Destination IP address  |
| src-port  | Source port number      |
| dst-port  | Destination port number |
| direction | `in` / `out`            |
| status    | Additional info         |

> ✅ **Placeholder:** *Screenshot or snippet highlighting key fields from the header and a few rows*

---

### 2.6 Limitations of Firewall Logs

Windows Firewall logs **do not** capture:

* Packet payloads
* Application names
* Session duration
* User identity

Logs can also:

* Overwrite when size limit is reached
* Be empty if logging was recently enabled
* Miss events that occur before logging was turned on

Despite these limitations, they are still useful for:

* Detecting intrusion attempts
* Spotting unusual outbound activity
* Identifying blocked scans or probing patterns

> ✅ **Optional Placeholder (Graph Later):**
> *Bar chart comparing number of Allowed vs Dropped events (to be generated using PowerShell and a charting tool if needed)*

---

## 3. Accessing the Firewall Log

### 3.1 Log File Path

* Confirm the configured log path (from Section 2.3).
* Note any custom locations (e.g., user Documents folder).

### 3.2 Checking Log Contents Manually (Notepad)

* Open `pfirewall.log` using Notepad or another text viewer.
* Scroll to the bottom to see the latest entries.

> ✅ **Placeholder:** *Screenshot of `pfirewall.log` opened in Notepad*

### 3.3 Viewing Logs Using PowerShell `Get-Content`

* Use `Get-Content` or `Get-Content -Tail` to view recent entries.
* Introduce `-Wait` to watch logs in real time (like `tail -f`).

### 3.4 Backing Up / Exporting the Log Safely

* Copy the log file to another folder before analysis.
* Optionally rename it with a timestamp (e.g., `pfirewall_YYYYMMDD.log`).

---

## 4. Understanding Log Format & Fields

### 4.1 Header & Metadata

* Structure of the log header (e.g., `#Version:`, `#Fields:`).
* How the header defines the meaning and order of fields.

### 4.2 Core Fields (date, time, action, protocol, src-ip, dst-ip, etc.)

* Detailed explanation of each commonly used field.
* How these map to network concepts (source/destination, client/server, etc.).

### 4.3 Allowed vs Blocked Entries

* How `ALLOW` vs `DROP` appears in the `action` field.
* Why both are important for investigation.

### 4.4 Inbound vs Outbound Indicators

* How direction (`in`/`out`) is represented.
* Examples of inbound vs outbound log lines.

> ✅ **Placeholder:** *Snippet with one inbound and one outbound entry annotated*

---

## 5. Filtering & Extracting Data (Native Tools)

### 5.1 Using `findstr` for Keyword Filtering (CMD)

* Filter by `DROP`, IP, port, protocol using `findstr`.
* Save filtered output to a new `.log` or `.txt` file.

### 5.2 Using PowerShell `Select-String`

* Use `Select-String` for more flexible pattern searching.
* Filter by date, IP, or action.

### 5.3 Filtering by Criteria

* **Date/Time** (e.g., events within a specific day or hour)
* **Action** (`ALLOW` / `DROP`)
* **IP Address** (source/destination)
* **Port** (e.g., 3389, 80, 443)
* **Protocol** (TCP/UDP/ICMP)

### 5.4 Exporting Filtered Results to a File

* Redirect PowerShell or CMD output to `.txt` or `.csv`.
* Prepare filtered datasets for further manual or scripted analysis.

> ✅ **Placeholder:** *Screenshot or snippet of a filtered log file*

---

## 6. Basic Analysis of Allowed Traffic

### 6.1 Identifying Frequent Destinations

* Count how often the system connects to specific external IPs.
* Identify top N destination IPs.

### 6.2 Identifying Common Ports & Protocols

* Determine frequently used ports (e.g., 80, 443, 53).
* Spot unusual ports for the host’s normal behavior.

### 6.3 Noticing Off-Hours Traffic

* Compare normal working hours vs late-night / early-morning activity.
* Flag unexpected allowed connections outside expected times.

### 6.4 Detecting Unusual Outbound Patterns

* Sudden uptick in allowed outbound connections.
* Repeated connections to rarely seen or new external IPs.

> ✅ **Placeholder (Graph):** *Simple bar chart of top destination ports or IPs*

---

## 7. Analysis of Blocked Traffic

### 7.1 Identifying Repeated Block Attempts

* Look for multiple `DROP` events from the same source IP.
* Possible indication of port scanning or brute-force attempts.

### 7.2 Possible Port Scanning Indicators

* Many different destination ports from the same IP.
* Multiple rapid `DROP` events over a short period.

### 7.3 Blocked Inbound Requests from External IPs

* External hosts trying to access internal services.
* Blocked attempts towards sensitive ports.

### 7.4 Blocked Traffic to Sensitive Ports

* Attempts to connect on ports like 3389 (RDP), 22 (SSH via port forwarding), etc.
* Interpretation of risk based on environment.

---

## 8. Suspicious / Unauthorized Activity Indicators

### 8.1 Failed Repeated Inbound Attempts

* Many blocked inbound attempts may indicate targeted attacks.

### 8.2 Sudden Spikes in Outbound Connections

* Potential malware or exfiltration tools phoning home.

### 8.3 Outbound Traffic to Unknown External IPs

* Destinations not normally contacted by the host.
* Especially if located in unusual geographic regions (conceptual, based on IP lookups).

### 8.4 Activity During Non-Working Hours

* Host generating traffic at odd hours may be compromised or misused.

### 8.5 Traffic on Uncommon Ports

* Outbound or inbound traffic using unusual or high-numbered ports.
* May be tunneling or custom C2 channels.

---

## 9. Potential Data Exfiltration Clues

### 9.1 High Volume Outbound Connections

* Large number of outbound sessions or bursts of activity.

### 9.2 Persistent Connections to a Single External Host

* Long-lasting or frequently repeated connections to the same IP.

### 9.3 Outbound Traffic on Non-Standard Ports

* Data sent over ports not typically used by the system’s normal applications.

### 9.4 Correlating Firewall Logs with File Transfer Activity (Conceptual)

* Conceptual description of how firewall logs might align with:

  * File copy actions
  * Large uploads
  * Backups to external services

---

## 10. Summarizing Findings

### 10.1 Key Observations

* Summary of high-level trends (allowed vs blocked, time patterns, etc.).

### 10.2 Legitimate vs Suspicious Traffic

* Distinguishing normal activity from anomalies.

### 10.3 Evidence of Unauthorized Access Attempts

* Highlight IPs, ports, and timestamps that indicate possible attacks.

### 10.4 Evidence of Possible Exfiltration

* Any indicators from Section 9 that suggest data may have been transferred out.

---

## 11. Reporting & Documentation

### 11.1 Recommended Report Structure

* Executive summary
* Methodology (how logs were collected & analyzed)
* Key findings
* Detailed evidence
* Recommendations

### 11.2 Screenshots & Log Snippets

* Limited, clear screenshots to support findings.
* Sanitized log snippets with sensitive data redacted.

### 11.3 How to Present Conclusions

* Clear, non-technical summary for management.
* Technical appendix for security teams.

---

## 12. Mitigation & Recommendations

### 12.1 Firewall Rule Adjustments

* Tighten rules for exposed services.
* Limit inbound connections to specific IP ranges where possible.

### 12.2 Hardening Windows Firewall Settings

* Ensure logging remains enabled.
* Enforce stricter profiles for Public networks.
* Disable unused services.

### 12.3 Monitoring Strategy Using Native Tools

* Regular review of logs.
* Scheduled tasks to rotate or archive logs.
* Basic automated checks using PowerShell (conceptual).

### 12.4 When to Escalate or Investigate Further

* When clear signs of compromise appear.
* When unusual traffic persists despite mitigation.

---

## 13. Limitations

### 13.1 Metadata-Only Logging

* No payload analysis; only connection metadata is available.

### 13.2 No Full Traffic Capture

* For full packet-level investigation, additional tools (not covered here) are required.

### 13.3 Potential Log Overwrites

* Logs can roll over and overwrite older entries if size is too small.

### 13.4 Dependency on Configuration

* If logging is disabled or misconfigured, visibility is lost.

---

## 14. Appendix

### 14.1 Sample Log Snippets

* Example lines showing ALLOW, DROP, inbound, outbound.

### 14.2 Useful Native Commands (PowerShell / CMD)

* `Get-Content`, `Select-String`, `findstr`, basic PowerShell one-liners.

### 14.3 Default Ports Reference

* Common service ports (80, 443, 3389, 53, etc.).

### 14.4 Glossary of Terms

* Definitions of key networking and firewall terms used in the report.

---
