

# Investigating a Joomla Web Server Attack with Splunk

## Case summary
(4 sentences: scan → brute force → login → upload)

## Scenario
Hands-on lab where I played the role of a junior SOC analyst, investigating an attack on a company web server in Splunk using the BOTS v1 dataset (Immersive Labs, Cyber Million programme). I followed the attack through the Cyber Kill Chain stages, using Splunk to search each one.

## Tools
Splunk (SPL)

## Investigation

### 1. Reconnaissance
- **Search:** `imreallynotbatman.com | top src_ip`
- **Found:**
- **Why it's malicious:**

### 2. Exploitation
- **Search:** `index="botsv1" earliest="0" source="stream:HTTP" http_method=POST passwd "username=admin"`
- **Found:**
- **Why it's malicious:**

### 3. Delivery
- **Search:** `index="botsv1" earliest="0" source="stream:HTTP" http_method=POST *.exe`
- **Found:**
- **Why it's malicious:**

## Indicators of Compromise
| Type | Value | What it was |
|------|-------|-------------|
| | | |

## Lessons learn't
-
