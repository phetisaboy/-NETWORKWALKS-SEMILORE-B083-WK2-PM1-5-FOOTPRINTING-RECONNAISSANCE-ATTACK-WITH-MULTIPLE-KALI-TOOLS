# NETWORKWALKS-SEMILORE-B083-WK2-PM1-5-FOOTPRINTING-RECONNAISSANCE-ATTACK-WITH-MULTIPLE-KALI-TOOLS

# Footprinting & Network Scanning: Penetration Testing Project

**Pentester:** Aboderin Semilore Gold
**Program:** Networkwalks, Cybersecurity (Batch B083)
**Date:** 28 September 2026
> The full report (W2-PM-FINAL) is submitted separately. This README summarises the work.

## Overview
This project covers the first two phases of a penetration test: footprinting (information gathering) and network scanning, using tools in Kali Linux and Zenmap.

## Scope and Authorization
| Target | Permission |
|---|---|
| networkwalks.com | Practice target specified in the program task |
| phetboy.name.ng | My own domain |
| 127.0.0.1 (localhost) | My own machine |

Only test systems you own or have written permission to test.

## Part 1: Footprinting (networkwalks.com)
| Tool | Command | Purpose | Finding |
|---|---|---|---|
| whois | `whois networkwalks.com` | Domain registration details | Name servers: NS6135/NS6136.HOSTGATOR.COM. Owner hidden behind a privacy proxy service. DNSSEC unsigned. [PASTE registrar + creation/expiry dates] |
| whatweb | `whatweb networkwalks.com` | Fingerprint web technologies | Apache server running WordPress 7.1.2 with the Download Manager plugin. Uses jQuery 3.7.1. IP 192.232.216.135. Redirects HTTP to HTTPS. |
| nslookup | `nslookup networkwalks.com` | Resolve domain to IP | [Resolves to IP 192.232.216.135 (matches whatweb). Answered by DNS server 8.8.8.8 (non-authoritative).] |
| curl | `curl -I https://networkwalks.com` | Read HTTP headers | [HTTP/2 200 OK. Apache (version hidden). Secure, HttpOnly cookie. WordPress API exposed. Missing security headers (HSTS, X-Frame-Options, CSP).] |
| wafw00f | `wafw00f networkwalks.com` | Detect a WAF | Behind a ModSecurity (SpiderLabs) WAF, detected with 2 requests (wafw00f v2.4.2). |
| dnsrecon | `dnsrecon -d networkwalks.com` | Enumerate DNS records | [A record: 192.232.216.135. NS: ns6135/ns6136.hostgator.com. MX: mail.networkwalks.com (same IP). SPF record present. No DNSSEC. Name servers expose BIND version 9.16.23-RH.] |

Screenshots: `1-whois.png`, `2-whatweb.png`, `3-nslookup.png`, `4-curl.png`, `5-wafw00f.png`, `6-dnsrecon.png`

### Comparison with my own domain
| | networkwalks.com | phetboy.name.ng |
|---|---|---|
| Web server | Apache | GitHub.com |
| WAF / CDN | ModSecurity (SpiderLabs) | Fastly CDN |
| Hosting | WordPress site | GitHub Pages |

## Part 2: Network Scanning (Zenmap / Nmap)
- **Target:** 127.0.0.1, the IP address provided in class (localhost, so the scan ran against my own computer)
- **Command:** `nmap 127.0.0.1` (run in Zenmap)
- **Open ports:** 135/tcp (msrpc) and 445/tcp (microsoft-ds). 98 other scanned ports were closed.
- **Meaning:** Windows RPC and SMB file sharing are running on the scanned machine. Both are normal on Windows but should be firewalled from untrusted networks, since SMB has had serious vulnerabilities such as the one used by WannaCry.
  
Screenshots: `7-Nmap-scanning.png`, `8-Nmap-topology.png`
## Recommendations
- Limit the technology and version details exposed in HTTP headers.
- Keep the web server, WordPress, and plugins updated, and hide version numbers.
- Enable security headers (HSTS, X-Frame-Options, CSP).
- Enable DNSSEC where possible.
- Firewall SMB (445) and RPC (135) from untrusted networks.

## Conclusion
This project showed me how much an outsider can learn about a website before touching it. Using Kali Linux tools (whois, whatweb, nslookup, curl, wafw00f and dnsrecon), I collected public information about networkwalks.com: who registered it and which name servers it uses, that it runs on Apache with WordPress, which IP address it resolves to, and that a ModSecurity firewall sits in front of it. Running the same tools on my own domain, phetboy.name.ng, showed a very different setup: it is hosted on GitHub Pages behind the Fastly CDN. This taught me that the same tool can reveal very different things depending on how a site is built and protected.

For network scanning, I used Zenmap and Nmap on my own machine (127.0.0.1) and found ports 135 (RPC) and 445 (SMB) open. I learned that an open port is not automatically a problem, but services like SMB should be restricted because of past attacks such as WannaCry.

The biggest lessons for me were that footprinting is passive and useful, that every finding should be explained (what I did, what I saw, what it means, and how to reduce the risk), and that a tool like whois or wafw00f only helps if I understand the result. I also had real problems along the way, like the RPM file that would not open on Windows and a whois connection error, and fixing them taught me to read errors carefully. Finally, I learned that all of this work must stay inside an authorised scope: targets I own or am allowed to test.

## Disclaimer
This work was done for educational purposes on targets I own or am authorised to test.
