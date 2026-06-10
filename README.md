# 🧠 Analyze Memory Dump with Volatility - CTF Challenge

## 📋 Overview

**Analyze Memory Dump with Volatility** is an interactive, browser-based Capture The Flag (CTF) challenge designed for cybersecurity training. This challenge focuses on memory forensics using the Volatility framework. Participants analyze a simulated Windows memory dump to identify malicious processes, detect code injection, uncover network connections, and extract malware configurations.

## 🎯 Learning Objectives

By completing this CTF, participants will learn:

- **Memory Profile Identification**: Determine the correct OS profile using imageinfo
- **Process Analysis**: Compare pslist vs psscan to find hidden processes
- **Network Analysis**: Use netscan to identify C2 connections
- **Code Injection Detection**: Use malfind to detect injected code
- **Malware Extraction**: Dump processes and extract embedded configurations

## 🛠️ Challenge Tasks (5 Total)

| Task | Description | Skill Focus |
|------|-------------|-------------|
| **Task 1** | Determine memory profile (Win7SP1x64) | Memory Analysis |
| **Task 2** | Find hidden process (malware.exe) | Process Detection |
| **Task 3** | Identify C2 connection IP (185.130.5.253) | Network Analysis |
| **Task 4** | Detect code injection in explorer.exe | Injection Detection |
| **Task 5** | Extract C2 domain from malware config | Malware Analysis |

## 🚀 Quick Start

### Prerequisites
- A modern web browser (Chrome, Firefox, Edge, Safari)
- No server required - runs entirely in the browser
- No installation needed

### Access the Challenge
1. Open the HTML file directly in your browser
2. Enter your name
3. Use the password: `45_2026`
4. Complete all 5 tasks to capture the flag

### Hosting on GitHub Pages
1. Fork or clone this repository
2. Go to repository Settings > Pages
3. Select the branch (usually `main`) and save
4. Access via `https://your-username.github.io/repository-name`

## 🎮 How to Play

### Login
```
Password: 45_2026
Name: Enter any name (progress is saved locally)
```

### Game Features

- **Interactive Volatility Terminal**: Clickable command buttons simulating Volatility framework
- **Command Output Display**: Realistic Volatility output with color-coded findings
- **Process Comparison**: Side-by-side analysis of pslist vs psscan output
- **Network Connection Display**: netscan output with highlighted suspicious connections
- **Injection Detection Results**: malfind output showing injected code locations
- **Malware Config Extraction**: strings output revealing C2 domains
- **Answer Validation**: Immediate feedback on submitted answers
- **Progress Tracking**: Local storage saves your progress across sessions

### Completing Tasks
1. Read each task description carefully
2. Click Volatility commands to run analysis
3. Analyze the command outputs for suspicious findings
4. Type your answer in the input field
5. Click "Submit" to validate
6. Complete all 5 tasks to reveal the flag

## 🏆 Flag

```
FLAG{MEMORY_FORENSICS}
```

The flag is revealed only after completing all 5 tasks successfully.

## 📊 Challenge Details

### Available Volatility Commands

```
volatility> imageinfo    # Determine OS profile
volatility> pslist       # List running processes
volatility> psscan       # Scan for hidden processes
volatility> netscan      # Network connections
volatility> malfind      # Detect code injection
volatility> procdump     # Dump process executable
volatility> clear        # Clear terminal
```

### imageinfo Output

```
Suggested Profile(s): Win7SP1x64, Win7SP0x64, Win2008R2SP0x64
AS Layer1: WindowsAMD64PagedMemory
PAE type: No PAE
DTB: 0x187000L
KDBG: 0xf8000280a0a0L
Number of Processors: 4
⚠ Best profile: Win7SP1x64
```

### pslist Output (Normal Processes)

```
System (PID: 4)
smss.exe (PID: 256)
csrss.exe (PID: 344)
winlogon.exe (PID: 400)
services.exe (PID: 488)
lsass.exe (PID: 500)
svchost.exe (PID: 612)
explorer.exe (PID: 1680)
cmd.exe (PID: 2240)
⚠ Note: Some processes may be hidden. Use psscan.
```

### psscan Output (Reveals Hidden Process)

```
...all pslist processes...
malware.exe (PID: 31337) ⚠ HIDDEN (DKOM)
⚠ FOUND HIDDEN PROCESS: malware.exe - unlinked from ActiveProcessLinks!
```

### netscan Output

```
Proto  | Local Address      | Foreign Address    | State        | PID   | Owner
-------|--------------------|--------------------|--------------|-------|----------
TCPv4  | 192.168.1.100:49158| 185.130.5.253:443  | ESTABLISHED  | 31337 | malware.exe
TCPv4  | 192.168.1.100:49159| 8.8.8.8:53         | CLOSED       | 612   | svchost.exe
TCPv4  | 0.0.0.0:445        | 0.0.0.0:0          | LISTENING    | 4     | System
⚠ Suspicious: PID 31337 → 185.130.5.253:443 (C2 communication)
```

### malfind Output

```
Process: System (PID: 4) - No injected code detected
Process: svchost.exe (PID: 612) - No injected code detected
Process: explorer.exe (PID: 1680)
  [!] INJECTED CODE FOUND at 0x400000
  Protection: PAGE_EXECUTE_READWRITE
  Tag: VadS (private memory)
  Data: 4D 5A 90 00... (MZ header - embedded PE)
⚠ Code injection detected in explorer.exe!
```

### procdump & strings Output

```
$ volatility -f memory.dmp --profile=Win7SP1x64 procdump -p 31337
Process: malware.exe | Result: OK: executable.31337.exe

$ strings executable.31337.exe | grep -E "(http|https|\.com|\.net|\.org)"
https://badguy.attacknet.org/api/register
https://badguy.attacknet.org/api/command
https://badguy.attacknet.org/api/upload
⚠ C2 Domain extracted: badguy.attacknet.org
```

## 🔍 Investigation Walkthrough

### Task 1: Determine Profile
The suggested profile is **Win7SP1x64**. This is critical because:
- Using wrong profile produces incorrect results
- Profile determines memory structure parsing
- KDBG scan identifies the Windows version
- Multiple suggestions provided, use the first/best match
- Win7SP1x64 indicates Windows 7 Service Pack 1 64-bit

### Task 2: Find Hidden Process
**malware.exe** is hidden from pslist but detected by psscan:
- pslist walks the linked list of active processes
- psscan scans physical memory for process signatures
- Hidden processes unlink from ActiveProcessLinks
- DKOM (Direct Kernel Object Manipulation) technique used
- PID 31337 is suspicious (1337 = "leet" speak for "elite")

### Task 3: C2 Connection
The suspicious external IP is **185.130.5.253**:
- Connected on port 443 (HTTPS) from malware.exe
- ESTABLISHED state indicates active connection
- Known malicious IP in threat intelligence
- Encrypted communication over standard HTTPS port
- Multiple API endpoints suggest structured C2 protocol

### Task 4: Code Injection
**explorer.exe** has injected code:
- Injected PE file starts with MZ header
- Located in private memory (VadS tag)
- Executable memory protection (PAGE_EXECUTE_READWRITE)
- Common technique to hide malware in legitimate process
- explorer.exe is a frequent target for injection

### Task 5: Extract C2 Domain
The C2 domain is **badguy.attacknet.org**:
- Three API endpoints: /api/register, /api/command, /api/upload
- Register endpoint for victim registration
- Command endpoint for receiving C2 commands
- Upload endpoint for data exfiltration
- HTTPS protocol for encrypted communication

## 🎨 Visual Features

- **Interactive Terminal**: Green-on-black Volatility terminal with clickable commands
- **Color-coded Output**: Info (blue), Warnings (yellow), Danger (red), Output (green)
- **Command Buttons**: Styled buttons for each Volatility plugin
- **Process Comparison**: Hidden process highlighted in red in psscan output
- **Network Analysis**: Suspicious connection with red danger styling
- **Injection Results**: Code injection findings with danger formatting
- **Progress Indicators**: Visual completion status for each task
- **Glowing Flag Animation**: Celebratory golden flag reveal
- **Toast Notifications**: Non-intrusive success/error messages with hints
- **Dark Theme**: Gold-accented UI for memory forensics theme

## 💾 Data Storage

- **Progress**: Saved in browser's `localStorage`
- **Persistence**: Progress survives page refreshes
- **Privacy**: All data stays on the user's device
- **Reset**: Clear browser data to reset progress

## 🛡️ Memory Forensics Detection Indicators

### High-Fidelity Indicators:
- Processes hidden from pslist but visible in psscan
- Code injection in legitimate processes (explorer.exe, svchost.exe)
- Network connections from hidden processes
- Executable memory regions with MZ headers
- Unlinked processes from ActiveProcessLinks

### Medium-Fidelity Indicators:
- Unusual process parent-child relationships
- Processes with high entropy memory regions
- Suspicious TCP connections to uncommon ports
- Processes with abnormally high handle counts

### Volatility Plugin Reference:
```
imageinfo    - Profile identification
pslist       - Active process listing
psscan       - Physical memory process scan
netscan      - Network connection enumeration
malfind      - Code injection detection
procdump     - Process executable dumping
dlllist      - Loaded DLL enumeration
handles      - Open handle enumeration
cmdline      - Process command line
timeliner    - Timeline generation
```

## 📁 File Structure

```
analyze-memory-dump-volatility/
│
├── index.html          # Main CTF challenge file
├── README.md           # This documentation
└── (no other files required)
```

## 🔧 Technical Implementation

- **Pure Frontend**: HTML5, CSS3, JavaScript (Vanilla)
- **No Dependencies**: Zero external libraries
- **Responsive Design**: Works on desktop and mobile
- **Terminal Simulation**: Interactive command execution with scrollable output
- **Storage**: Browser localStorage API
- **Gamification**: Progress tracking, badge system, visual rewards
- **Command Parser**: Simulated Volatility plugin outputs

## 📊 Memory Forensics Workflow

```
1. Initial Assessment
   ├── Run imageinfo for profile
   ├── Run pslist for process listing
   └── Note system specifications

2. Process Analysis
   ├── Compare pslist vs psscan
   ├── Identify hidden processes
   ├── Check process parent-child relationships
   └── Analyze command lines

3. Network Analysis
   ├── Run netscan for connections
   ├── Identify external IPs
   ├── Correlate connections to processes
   └── Map C2 infrastructure

4. Malware Detection
   ├── Run malfind for injection
   ├── Check suspicious memory regions
   ├── Dump malicious processes
   └── Extract embedded configurations

5. Reporting
   ├── Document all findings
   ├── Extract IOCs
   ├── Generate timeline
   └── Recommend remediation
```

## 🎓 Educational Use Cases

- **Cybersecurity Training Programs**
- **SOC Analyst Onboarding**
- **Memory Forensics Workshops**
- **Blue Team Exercises**
- **Incident Response Training**
- **Academic Courses** (Digital Forensics, Malware Analysis)
- **Self-paced Learning**
- **DFIR Training**

## 🔄 Version History

- **v1.0** - Initial release
  - 5 tasks with validation
  - Interactive Volatility terminal with 7 commands
  - Simulated memory dump analysis outputs
  - Process hiding via DKOM detection
  - Code injection in explorer.exe detection
  - Local storage progress tracking
  - Student login system

## 👥 Target Audience

- Security Operations Center (SOC) Analysts
- Incident Response Team Members
- Digital Forensics Examiners
- Malware Analysts
- Threat Hunters
- Cybersecurity Students
- IT Security Professionals
- Blue Team Practitioners

---

**Happy Memory Forensics! 🧠**
