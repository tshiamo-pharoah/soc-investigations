


# Investigating a Cerber Ransomware Attack with Splunk

## Case summary
In this investigation, a USB drive was plugged into user bob.smith’s computer (host:we8105desk) to deliver Cerber ransomware. The malware disguised itself as a real Windows program (osk.exe) in the AppData folder and used a Run key to start every time the PC turned on. It then checked in with the attacker’s servers on the 85.93.x.x network, and ran the vssadmin command to delete Windows backups so the files couldn't be restored.

## Scenario
Hands-on lab where I played the role of a junior SOC analyst, investigating an incident in Splunk using the BOTS v1 dataset (Immersive Labs, Cyber Million programme). 
I followed the attack through the Cyber Kill Chain stages, using Splunk to search each one.

## Tools
Splunk (SPL) 

## Investigation

### 1. Delivery
- **Search:** index=botsv1 earliest=0 USB VID
- **Found:** I found that a USB flash drive with serial number 7D961196 was plugged into host we8105desk while user bob.smith was logged in, recorded in the Windows event logs.
- **Why it's malicious:** A USB on its own seems normal, but this one was plugged into the same host where malware appeared hiding in AppData, disguised as a real Windows program (osk.exe), with a Run key for persistence. That makes the USB the likely delivery method for the ransomware.


### 2. Installation
- **Search:** index="botsv1" earliest="0" registry *run*
- **Found:** I noticed that osk.exe was hiding in AppData\Roaming, inside a randomly named folder.
- **Why it's malicious:** The real osk.exe lives in C:\Windows\System32 (where Windows’ own system files are stored), but this one was planted in the user’s AppData folder inside a randomly named folder so the user would not detect it or become suspicious. Planting a program that pretends to be a legitimate Windows program is a trick attackers use to avoid alarming users. The Run key launches the malware every time the computer starts, because memory (RAM) is wiped every time a computer shuts down, which gives the attacker persistent access.

### 3. Command and Control
- **Search:** index=botsv1 earliest=0 source="/var/log/suricata/eve.json" alert.signature="*cerber*"
- **Found:** I then found out that osk.exe sent traffic to the 85.93.x.x network (including 85.93.63.254 and 85.93.63.255). Suricata IDS flagged Cerber activity with alerts including Cerber Checkin 2 and Cerber Onion Domain Lookup.
- **Why it's malicious:** The Cerber Checkin 2 alert showed me the infected machine was reporting back to the attacker’s server to confirm the infection worked. I also saw .onion domain lookups. A normal office PC has no reason to do that, since .onion sites are hidden Tor sites that ransomware gangs use for things like payment pages.

### 4. Actions on Objectives
- **Search:** index=botsv1 earliest=0 Process_Command_Line="*" (vssadmin OR cmd.exe)` and `index=botsv1 earliest=0 cmd.exe *.tmp`
- **Found:** I found that vssadmin was used on we8105desk, and cmd.exe ran a file called 121214.tmp from Bob's AppData folder. A whoami command was also run.
- **Why it's malicious:** Ransomware uses vssadmin to delete Windows backup copies, so the victim can't restore their files and is pushed to pay. A .tmp file running from AppData is not normal behaviour, and whoami shows someone checking which account they had control of.

## Indicators of Compromise
| Type | Value | What it was |
|------|-------|-------------|
| Host | we8105desk | Bob Smith's infected machine |
| USB serial | 7D961196 | Likely delivery device |
| File | osk.exe (in AppData\Roaming) | Cerber malware disguised as a Windows program |
| File | 121214.tmp | File executed via cmd.exe |
| Network | 85.93.x.x | Cerber C2 traffic |

## Lessons learne't
- I learnt why attackers use startup methods like Run keys. Memory resets when a PC shuts down, so the malware needs a way to start again after every reboot.
- I first picked ProcessID 1216, which was Sysmon's own process, not the malware. The right field was ProcessId (small d), so one capital letter changed the answer.
- Real Windows system programs like osk.exe live in System32. Finding one in AppData was the clue it was malware.
- Sysmon showed osk.exe talking to 85.93.x.x, and Suricata flagged the same network as Cerber. Two logs agreeing turned a hint into evidence.

