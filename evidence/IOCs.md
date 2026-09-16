# Indicators of Compromise (IOCs)

Case: RIG Exploit Kit → KPOT Stealer | Incident ID: INC-2019-06-22

## Network Indicators

| Type | Value | Role |
|---|---|---|
| IP | `91.235.129.60` | First suspicious site accessed (`letsdoitquick.site`) |
| IP | `37.46.135.170` | RIG Exploit Kit delivery host |
| IP | `8.209.83.76` | KPOT Stealer C2 server |
| Domain | `letsdoitquick.site` | Initial landing/redirector |
| Domain | `fghjkmgru34.site` | KPOT Stealer C2 domain |
| URI pattern | `/?MTQwMjg3...` (Base64-like, nonsense-word parameters) | RIG EK landing page structure |
| URI | `/gQBljYzDJBnrt4JX/gate.php` | KPOT Stealer exfiltration endpoint |

## File Hashes (SHA-256)

| Hash | File |
|---|---|
| `39be5610259ffade85599720ee0af31187788a00791f1e4cb0cd05ef00105eda` | KPOT Stealer executable |
| `39bf8220d772efc49f7a8f0709ac8607af17997d38525eacec1448d5317dcf38` | Malicious Flash (x-shockwave-flash) |
| `96d61a66fda99897f47232a1f3d15fe711fee6726c2e976eed42011948d87049` | `gate.php` response |
| `ca5a37a5c3401ffcd1b7c98c3a22a921c013d1121fe33122e94dd81c382bf9b0` | Malicious VBScript |

## Vulnerabilities Exploited

| CVE | Component | Description |
|---|---|---|
| CVE-2018-8174 | Microsoft VBScript engine | "Double Kill" — memory corruption via obfuscated VBScript, delivered through Internet Explorer/embedded IE engine |
| CVE-2018-4878 | Adobe Flash Player (≤ 28.0.0.126) | Use-after-free vulnerability, delivered via malicious SWF |

## Detection Signatures

- `ET CURRENT_EVENTS RIG EK URI Struct`
- `ET TROJAN KPOT Stealer Exfiltration M2`

## Affected Host

| Field | Value |
|---|---|
| Hostname | `BANGKOK-8AC2-PC` |
| Internal IP | `10.0.76.109` |
| MAC Address | `78:2b:cb:d4:a5:fe` |
| User | `edris.haight` |
