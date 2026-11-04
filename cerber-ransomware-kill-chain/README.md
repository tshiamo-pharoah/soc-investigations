


# Investigating a Cerber Ransomware Attack with Splunk

## 60-second summary
(4 to 5 sentences: what happened, from first contact to ransomware running)

## Scenario
Hands-on lab where I played the role of a junior SOC analyst, investigating an incident in Splunk using the BOTS v1 dataset (Immersive Labs, Cyber Million programme). 
I followed the attack through the Cyber Kill Chain stages, using Splunk to search each one.

## Tools
Splunk (SPL) 

## Investigation

### 1. Delivery
- **Search:** index=botsv1 earliest=0 USB VID
- **Found:** A USB flash drive with serial number 7D961196 was plugged into host we8105desk  while user bob.smith was logged in, incident recorded in the Windows event logs
- **Why it's malicious:** A USB on its own seems normal, but this one was plugged into the same host where malware appeared hiding in AppData, disguised as a real Windows program (osk.exe), with a Run key for persistence. That makes the USB the likely delivery method for the ransomware.


### 2. Installation
- **Search:** index="botsv1" earliest="0" registry *run*
- **Found:** The osk.exe file was hiding in AppData\Roaming  inside a randomly named folder on host we8105desk, and a Run key in the registry made it start every time the PC turned on.
- **Why it's malicious:** The real osk.exe lives in C:\Windows\System32 (where Windows’ own system files are stored), but this one was planted in the user’s AppData folder inside a randomly named folder so the user would not detect it or become suspicious. Planting a program that pretends to be a legitimate Windows program is a trick attackers use to avoid alarming users. The Run key launches the malware every time the computer starts, because memory (RAM) is wiped every time a computer shuts down, which gives the attacker persistent access.

### 3. Command and Control
- **Search:** `...`
- **Found:**
- **Why it's malicious:**

### 4. Actions on Objectives
- **Search:** `...`
- **Found:**
- **Why it's malicious:**

## Indicators of Compromise
| Type | Value | What it was |
|------|-------|-------------|
| IP | | |
| File | | |
| Host | | |

## Lessons learned
-
-
-

