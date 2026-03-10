# Windows Endpoint Telemetry & SOAR Automation

Windows Endpoint Telemetry System using Sysmon + Splunk Indexer + UF + Generating Alerts + SPL Queries + Dashboards + Atomic Red Team Attacks + Writing Sysmon XML Rules + Python SOAR Mini-Clone.

## System Architecture & Components

### 1. Telemetry & Detection (Sysmon + Splunk)
The foundation of the project is robust telemetry and detection.
*   **Sysmon**: Configured with custom XML rules to capture relevant endpoint telemetry.
    *   [Installation Guide](Window/Sysmon/Insatallion.md)
*   **Splunk**: Acts as the SIEM, ingesting logs via Universal Forwarder (UF), running SPL queries, and hosting dashboards.
    *   [Windows UF Installation](Window/SplunkUF/Installtion.md)
    *   [Splunk Custom Alert](Linux/Splunk/customAlert.md)

![Alt text](Image.png)
### 2. Python SOAR Mini-Clone
A custom automation pipeline that processes alerts triggered by Splunk.

**Pipeline Steps:**
1.  **Ingest**: Receives the Alert JSON payload from Splunk.
2.  **Extraction**: Parses the payload to extract artifacts (IPs, Hashes, URLs).
3.  **Enrichment**: Queries external APIs (VirusTotal, Shodan, AbuseIPDB) for reputation data.
4.  **Scoring**: Calculates a severity score based on enrichment data.
5.  **AI Analysis**: Queries an LLM to format the analysis, map to MITRE ATT&CK, and suggest response actions.
    *   *Note: AI is used primarily for formatting and structure, as it may not be reliable for critical risk assessment.*
6.  **Response**: Automatically creates a Jira ticket with all gathered context.

**Scripts:**
- [Alert Script](SoarWorkflow/soar_flow.py) - Main controller for the SOAR pipeline.
- [Testing Script](SoarWorkflow/soar_testing.py) - Demo/testing pipeline (reads from `stdin`).

**Environment Setup (`SoarWorkflow/.env`):**
```env
nv_api=<your-nvidia-nim-api-key>
ABUSEIPDB_API_KEY=<your-abuseipdb-key>
VIRUS_TOTAL_API_KEY=<your-virustotal-key>
```

**Quick Demo (no Splunk needed):**
```bash
cd SoarWorkflow
cat demo.txt | python soar_testing.py
```
This pipes a sample alert JSON into the pipeline and runs the full ingest → enrich → AI analysis → Jira ticket flow.

### 3. Attack Simulation & Detection (Atomic Red Team)
Simulated attacks used to test and validate the detection pipeline.

#### Windows Attacks
*   **T1003.001 (OS Credential Dumping)**
    *   [Analysis](AtomicRedteam/T1003.001-Playbook/Analysis.md)
    *   [Attack Execution](AtomicRedteam/T1003.001-Playbook/Detection.md)
*   **T1053.005 (Scheduled Task)**
    *   [Analysis](AtomicRedteam/T1053.005-Playbook/Analysis.md)
    *   [Attack Execution](AtomicRedteam/T1053.005-Playbook/Detection.md)
*   **T1059 (Command and Scripting Interpreter)**
    *   [Guide](AtomicRedteam/T1059-Playbook/Guide.md)
*   **T1110 (Password Spray and Brute Force Login)**
    *   [Guide](/Linux/T1110-Playbook/Setup_AND_security_event_analysis.md)
    *   [Attack Execution](/Linux/T1110-Playbook/pre_access.md)

#### Active Directory Attacks
*   **T1558.003 (Kerberoasting)**
    *   [Attack Execution](/AtomicRedteam/AD-ATTACKS/T1558.003-Playbook/Analysis.md)
    *   [Guide](/AtomicRedteam/AD-ATTACKS/T1558.003-Playbook/Detection.md)
*   **T1550.002 (Pass the Hash)**
    *   [Attack Execution](/AtomicRedteam/AD-ATTACKS/T1550.002&003-Playbook/Analysis.md)
    *   [Guide](/AtomicRedteam/AD-ATTACKS/T1550.002&003-Playbook/Detection.md)
*   **T1550.003 (Pass the Ticket)**
    *   [Attack Execution](/AtomicRedteam/AD-ATTACKS/T1550.002&003-Playbook/Analysis.md)
    *   [Guide](/AtomicRedteam/AD-ATTACKS/T1550.002&003-Playbook/Detection.md)

### 4. Integration Guides
- [Jira Python Connection](Jira/Python_Connection.md)
- [Jira Setup](Jira/Setup_Guide.md)

### 5. Known Limitations
This is a lab/portfolio project. The following gaps are documented to show awareness of production requirements:

*   **No rate limiting** — VirusTotal free tier is 4 requests/min, AbuseIPDB is 1,000/day. High alert volume would exhaust quotas or get the IP blocked.
*   **Alert spam / no deduplication** — Same IP firing 50 alerts creates 50 Jira tickets. Production needs a cooldown window or hash-based dedup.
*   **Silent enrichment failures** — If VT or AbuseIPDB is down, the pipeline continues with incomplete data and the LLM analysis is less reliable.
*   **No queue/async processing** — Alerts are processed synchronously, so a slow API call blocks the whole pipeline.
*   **LLM non-determinism** — Same alert can produce slightly different MITRE mappings on repeated runs.