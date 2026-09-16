# RIG Exploit Kit → KPOT Stealer: SOC Incident Analysis

A Tier 1 SOC-style investigation of a real-world drive-by-download infection chain: a compromised host redirected through a malvertising landing page into the **RIG Exploit Kit**, which chained two browser-side vulnerabilities to silently drop **KPOT Stealer**, an information-stealing trojan that exfiltrated browser credentials and host telemetry to an external C2 server.

This repository documents the full investigation — from raw packet capture to a completed incident report with IOCs, containment actions, and remediation steps — reflecting the actual workflow of triaging a NIDS alert end-to-end.

> **Training data source:** The packet capture used here is a publicly available training exercise (2019-06-22) from [malware-traffic-analysis.net](https://www.malware-traffic-analysis.net/), used for educational SOC/incident-response practice. No real organizations or individuals were affected.

---

## Incident Summary

| Field | Detail |
|---|---|
| Incident ID | INC-2019-06-22 |
| Date/Time Detected | 2019-06-22 23:48:05 UTC |
| Severity | High |
| Category | Malware / Drive-by Download / C2 Activity |
| Affected Host | `BANGKOK-8AC2-PC` (`10.0.76.109`), user `edris.haight` |
| Root Cause | Outdated Adobe Flash Player (28.0.0.126) + unpatched VBScript engine |
| Disposition | **True Positive** — confirmed exploitation and active C2 exfiltration |

**What happened, in order:**

1. The user browsed to `letsdoitquick.site` (`91.235.129.60`), which silently redirected the browser to `37.46.135.170`
2. That host served the **RIG Exploit Kit**: obfuscated JavaScript/VBScript targeting **CVE-2018-8174** (VBScript engine memory corruption), alongside a malicious Flash object targeting **CVE-2018-4878**
3. Successful exploitation triggered a silent drive-by download of an executable payload
4. The payload — identified as **KPOT Stealer** — executed immediately and began exfiltrating browser-stored credentials and system telemetry to `fghjkmgru34.site` (`8.209.83.76`) via `POST /gate.php`

---

## Repository Structure

```
.
├── docs/
│   └── Incident_Report.docx     Full SOC incident report (IOCs, containment,
│                                 remediation, and the lab's question/answer sheet)
├── evidence/
│   ├── IOCs.md                  Structured indicator list (IPs, domains, hashes, CVEs)
│   └── full_analysis_output.txt Raw packet-level findings (conversations, DNS, HTTP)
└── pcap/
    └── 2019-06-22-traffic-analysis-exercise.pcap   Original capture
```

---

## Methodology

Packet analysis was performed without relying on a pre-built parser — the capture was inspected at the protocol level to independently verify every finding in the incident report:

1. **Traffic volume triage** — identifying the two most active conversations by byte count immediately surfaced both the exploit-kit delivery host and the C2 channel, before reading any packet content
2. **DNS reconstruction** — extracting every query/response pair to map out which domains were contacted and what they resolved to, separating legitimate ad-tech/browsing noise (Google, Facebook, Bing) from the two suspicious `.site` domains
3. **HTTP request reconstruction** — parsing raw TCP payloads to rebuild full HTTP requests, revealing the RIG EK landing page's characteristic URI structure (Base64-like blobs paired with nonsense English-word parameters) and the exact `gate.php` C2 exfiltration endpoint
4. **Cross-referencing** — confirming the independently-derived findings matched the incident report's stated IOCs, timestamps, and host details exactly

See `evidence/full_analysis_output.txt` for the complete raw output this methodology produced.

---

## Key Indicators of Compromise

| Type | Value |
|---|---|
| EK Delivery IP | `37.46.135.170` |
| C2 IP | `8.209.83.76` |
| C2 Domain | `fghjkmgru34.site` |
| Malware SHA-256 | `39be5610259ffade85599720ee0af31187788a00791f1e4cb0cd05ef00105eda` |
| CVEs Exploited | CVE-2018-8174, CVE-2018-4878 |

Full IOC list with file hashes for every stage-artifact (Flash object, VBScript, C2 response) is in [`evidence/IOCs.md`](evidence/IOCs.md).

---

## Containment & Remediation (as executed)

- [x] Host `BANGKOK-8AC2-PC` isolated from the internal network segment
- [x] `37.46.135.170` and `8.209.83.76` blocked at the perimeter firewall
- [x] `fghjkmgru34.site` blocked at DNS/proxy level
- [ ] Password reset for `edris.haight` and all browser-cached credentials
- [ ] Full host wipe and re-image
- [ ] Fleet-wide Adobe Flash Player removal/patch audit for CVE-2018-8174

---

## Detection Signatures

```
ET CURRENT_EVENTS RIG EK URI Struct
ET TROJAN KPOT Stealer Exfiltration M2
```

---

## Skills Demonstrated

- NIDS alert triage (Snort/Suricata via Security Onion)
- Packet capture analysis (protocol-level HTTP/DNS reconstruction)
- Exploit kit identification and CVE correlation
- Malware family identification via C2 traffic pattern (`gate.php` beaconing)
- IOC extraction and structured documentation
- Incident report writing to a professional SOC template
- Containment and remediation planning

---

## License

Incident report content is original work. The packet capture is a publicly redistributed educational training resource — see [malware-traffic-analysis.net](https://www.malware-traffic-analysis.net/) for their usage terms.
