# Security Incident Reporting

Personal notes from the Hack The Box Academy *Security Incident Reporting* module, written in my own words.

---

## Table of Contents

1. [Identifying Incidents](#1-identifying-incidents)
2. [Categorising Incidents](#2-categorising-incidents)
3. [Severity Levels](#3-severity-levels)
4. [Incident Reporting Process](#4-incident-reporting-process)
5. [Writing the Incident Report](#5-writing-the-incident-report)
6. [Communication During an Incident](#6-communication-during-an-incident)

---

## 1. Identifying Incidents

Incidents usually appear as alerts, anomalies, or activity that differs from normal behaviour. They are typically spotted through one of three sources:

| Source | Examples |
|---|---|
| Security tools | IDS/IPS, EDR/XDR, SIEM alerts, antivirus detections, NetFlow data |
| People | Employees reporting odd emails, strange system behaviour, or suspicious activity |
| Third parties | Partners, vendors, or customers warning the organisation about a breach or vulnerability |

---

## 2. Categorising Incidents

Giving an incident a category helps the team decide its priority, who should respond, and how to explain it to stakeholders.

| Type | What it is |
|---|---|
| Malware | Malicious software such as viruses, worms, and ransomware |
| Phishing | Deceptive messages, mostly emails, used to steal information or gain access |
| DDoS | Flooding a system or network with traffic so it can't function normally |
| Unauthorised access | Someone getting into systems or data they aren't allowed to access |
| Data leakage | Confidential data being exposed, inside or outside the organisation |
| Physical breach | Someone entering a secure physical area without permission |

---

## 3. Severity Levels

Severity decides how fast the team must respond.

| Level | Meaning | Response |
|---|---|---|
| **P1 – Critical** | Directly threatens core business operations or sensitive data | Act immediately |
| **P2 – High** | Serious threat to operations, but not causing damage yet | High priority |
| **P3 – Medium** | No immediate threat, but shouldn't be left too long | Handle promptly |
| **P4 – Low** | Minor issue or routine anomaly | Handle through normal workflow |

---

## 4. Incident Reporting Process

1. **Detection and acknowledgement** – An alert or report is received and someone confirms they're looking into it.
2. **Preliminary analysis** – A quick check to confirm it's a real incident, then assign a category and severity.
3. **Incident logging** – Record the incident in a ticketing or case system with a unique ID, so all actions and evidence are tracked in one place.
4. **Notify relevant parties** – Inform internal teams and, where needed, external parties (see [Communication](#6-communication-during-an-incident)).
5. **Detailed investigation** – Dig into logs and evidence to work out what happened, how, and what was affected.
6. **Final report** – Write up the full incident report (see [next section](#5-writing-the-incident-report)).
7. **Feedback loop** – Review what worked and what didn't, then improve processes, detections, and defences.

---

## 5. Writing the Incident Report

A good incident report tells the full story of an incident for both technical and non-technical readers. Its main sections are below.

### 5.1 Executive Summary

A short, non-technical overview for management. It should cover:

| Item | What to include |
|---|---|
| Incident ID | The unique identifier for the incident |
| Overview | What happened, the incident type, when it started, how long it lasted, which systems or data were affected, and the current status (ongoing, resolved, or escalated) |
| Key findings | The root cause, any vulnerability (CVE) exploited, and what data was compromised or stolen |
| Immediate actions | What was done right away, e.g. isolating systems or bringing in outside help |
| Stakeholder impact | How customers, employees, and the business were affected, including downtime and financial cost |

### 5.2 Technical Analysis

The detailed, technical explanation of the incident.

- **Affected systems and data** – Which hosts, accounts, and data sets were involved, and how badly.
- **Evidence sources and analysis** – Where the evidence came from (logs, EDR, network captures, disk or memory images) and what it showed.
- **Indicators of Compromise (IoCs)** – Artefacts that point to the attack, such as malicious IPs, domains, file hashes, or suspicious processes. These can be used to detect the same attacker elsewhere.
- **Root cause analysis** – The underlying weakness that allowed the attack, such as an unpatched system, weak credentials, or a successful phishing email.
- **Nature of the attack** – A description of the attacker's methods and goals, ideally mapped to MITRE ATT&CK techniques.

#### Technical Timeline

A chronological list of what happened, which makes the sequence of the attack easy to follow. Where possible, include timestamps for each stage:

1. Reconnaissance
2. Initial compromise
3. Command and control (C2) communication
4. Enumeration
5. Lateral movement
6. Data access and exfiltration
7. Malware activity (including process injection and persistence)
8. Containment
9. Eradication
10. Recovery

### 5.3 Impact Analysis

An assessment of the damage, covering affected operations, compromised data, financial cost, legal or regulatory consequences, and harm to the organisation's reputation.

### 5.4 Response and Recovery

What the team did to stop the attack and get back to normal.

#### Immediate Response

**Revoking access**
- How compromised accounts or systems were identified, and with which tools
- Exactly when the access was detected and when it was cut off
- How access was revoked, e.g. disabling accounts, changing permissions, or adding firewall rules
- What this achieved, such as stopping data theft or further spread

**Containment**
- **Short-term:** isolate affected systems quickly to stop the attacker moving further through the network
- **Long-term:** structural changes like network segmentation or a zero-trust approach
- **Effectiveness:** how well containment limited the damage

#### Eradication

**Removing malware**
- How the malware was found, e.g. EDR alerts or forensic analysis
- The tools or manual steps used to remove it
- How removal was confirmed, e.g. hash checks or rescanning

**Patching**
- How the vulnerability was found, including CVE IDs if relevant
- How patches were tested, deployed, and verified
- A rollback plan in case a patch causes problems

#### Recovery

**Restoring data**
- Check that backups are clean and intact before using them
- Document how data was restored, including any decryption
- Verify the restored data is complete and uncorrupted

**Validating systems**
- Confirm systems are secure before reconnecting them, e.g. updated firewall and IDS rules
- Test that they work correctly in production

**Monitoring**
- Set up closer monitoring to catch the same attack or similar patterns in future
- List the tools used and how they fit with existing systems

#### Lessons Learned

- **Gap analysis:** which defences failed and why
- **Recommendations:** specific improvements, ranked by priority and with timelines
- **Future strategy:** long-term changes to policy, architecture, or staff training

### 5.5 Diagrams

Visuals make complex incidents much easier to understand.

| Diagram | Shows |
|---|---|
| Incident flowchart | How the attack progressed from the entry point through the network |
| Affected systems map | The network layout with compromised systems highlighted, colour-coded by severity |
| Attack vector diagram | The path the attacker took through the defences and what they did along the way |

### 5.6 Appendices

Supporting material that backs up the report and lets others verify its findings. This can include:

- Log files
- Network diagrams (before and after the incident)
- Forensic evidence, such as disk images and memory dumps
- Code snippets or malicious scripts
- Incident response checklist
- Communication records
- Legal and compliance documents
- Glossary of terms and acronyms

---

## 6. Communication During an Incident

### 6.1 Internal Communication

Keeps everyone in the organisation aligned and reduces the risk of leaks or mixed messages.

- **Immediate notification** – Inform key stakeholders as soon as an incident is confirmed.
- **Regular updates** – Share status updates on a set schedule so every team knows the current situation and next steps.
- **Feedback loop** – Give teams a way to share findings, raise concerns, and suggest ideas.

### 6.2 External Communication

Handled carefully, since what's said publicly affects trust and legal standing.

- **Affected parties** – Contact impacted customers, clients, or partners directly.
- **Public statement** – For large incidents, release a clear statement in plain language, without technical jargon.
- **Regulators** – Some laws require notifying a regulator within a set time, e.g. the ICO in the UK.

### 6.3 Choosing Secure Communication Channels

During an incident, the normal communication tools may be compromised, and every message could become evidence. Channels need to be both **secure** and **legally compliant**.

#### Security requirements

| Requirement | Why it matters |
|---|---|
| End-to-end encryption | Incident details (affected systems, exploited flaws) are valuable to the attacker |
| Strong authentication (MFA) | Ensures only authorised people can join the conversation |
| Data integrity (hashing) | Proves messages weren't altered in transit |
| Ephemeral messages | Auto-deleting messages reduce exposure of very sensitive discussions |
| Air-gapped channels | A fully isolated fallback if the main network may be compromised |

#### Legal and regulatory requirements

| Requirement | What it means |
|---|---|
| Data privacy laws | Personal data in incident messages is still protected (e.g. GDPR), so share only what's necessary |
| Breach notification deadlines | Laws set time limits and required content for reporting breaches (e.g. GDPR: 72 hours to notify the regulator) |
| Record-keeping | Some regulations require keeping incident communications, which can conflict with auto-deleting messages |
| Cross-border rules | Data sovereignty laws may restrict sending data to other countries |
| Chain of custody | Record who handled each piece of evidence and when, so it stays admissible in court |

> **Balancing act:** ephemeral messaging improves security, but record-keeping laws may require you to keep those messages. Check what applies before choosing a channel.
