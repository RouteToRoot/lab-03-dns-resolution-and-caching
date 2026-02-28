# SOC Lab 03 — DNS Resolution & Caching

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Lab Objectives](#lab-objectives)
3. [Environment Overview](#environment-overview)
4. [Workflow](#workflow)
5. [Analysis](#analysis)
6. [Detection Engineering Insights](#detection-engineering-insights)
7. [Evidence](#evidence)
8. [Conclusions](#conclusions)
9. [Next Steps](#next-steps)

---

## Executive Summary
This lab examines how DNS resolution works on a Windows system and how caching influences query results.  
DNS is a critical network service and a common source of indicators in SOC operations. Understanding how to inspect DNS behavior enhances an analyst’s ability to detect malicious domains, unusual lookup behavior, and command-and-control activity.

This lab demonstrates:
- DNS lookups using the default system resolver  
- DNS lookups using Cloudflare (1.1.1.1)  
- The impact of DNS caching on repeated queries  

---

## Lab Objectives
- Flush DNS cache to ensure fresh resolution  
- Perform DNS lookups with `nslookup`  
- Compare responses between internal and external resolvers  
- Observe DNS caching mechanics  
- Capture evidence of DNS queries and TTL-related behavior  

---

## Environment Overview
**Host OS:** Windows 10 / Windows 11  
**Tools Used:**  
- `nslookup` (built-in Windows DNS tool)  
- `ipconfig` (DNS cache management)  
- Windows DNS Client Resolver  

---

## Workflow

### 1. Flush Local DNS Cache
Clears existing cached entries to ensure the first lookup resolves externally.

**Command:**
```bash
ipconfig /flushdns
```

**Evidence:**  
See `screenshots/flushdns-and-nslookup-google.png`

---

### 2. Perform Baseline DNS Lookup (Default Resolver)
Query `google.com` using the system’s configured DNS server.

**Command:**
```bash
nslookup google.com
```

**Purpose:**
- Establish baseline resolver behavior  
- Identify which DNS server the system is using  
- Record returned A/AAAA records  

**Evidence:**  
See `screenshots/flushdns-and-nslookup-google.png`

---

### 3. Compare with Cloudflare Resolver (1.1.1.1)
Perform the same lookup using Cloudflare’s DNS server.

**Command:**
```bash
nslookup google.com 1.1.1.1
```

**Purpose:**
- Compare results between resolvers  
- Observe response differences (load balancing, geo-routing)  
- Identify resolver-specific TTL behavior  

**Evidence:**  
See `screenshots/nslookup-google-com.png`

---

### 4. Lookup Additional Domain (cloudflare.com)
Query `cloudflare.com` through Cloudflare’s resolver.

**Command:**
```bash
nslookup cloudflare.com 1.1.1.1
```

**Purpose:**
- Validate resolver consistency  
- Observe patterns across multiple domains  
- Document DNS caching and TTL behavior  

**Evidence:**  
See `screenshots/nslookup-google-cloudflare-1.1.1.1.png`

---

## Analysis

### DNS Behavior Observed
- The first lookup after flushing cache produced a fresh DNS resolution.  
- Using Cloudflare’s resolver returned consistent and globalized A/AAAA records.  
- Repeated queries performed **after** caching showed faster responses and “Non-authoritative answer” output.  
- TTL determines how long cached results persist before a fresh resolution occurs.  

### Key Findings
- DNS caching significantly reduces resolution time.  
- Responses may differ slightly between resolvers due to geographic load-balancing.  
- Public DNS servers like 1.1.1.1 often respond with globally distributed IPs.  
- Observing DNS output is essential in forensic investigations and detection tuning.

---

## Detection Engineering Insights

### Threat Detection
- Many malware families use DNS to reach C2 infrastructure.  
- DNS tunneling and exfiltration often appear as unusual query patterns.  
- Failed lookups or rapid-fire DNS bursts may indicate reconnaissance or bot activity.

### Incident Response
- DNS logs help analysts reconstruct an attacker’s timeline.  
- Differentiating cached vs. new lookups can indicate whether a domain was contacted recently.  
- Resolver logs support correlation with network proxy or EDR alerts.

### Baselining
- Normal systems frequently query major services (Google, Microsoft, CDNs).  
- Rare, newly registered, or suspicious TLDs stand out immediately.  

---

## Evidence

All screenshots are stored in the `/screenshots` directory:

- `flushdns-and-nslookup-google.png` — Flushed DNS + baseline google.com lookup  
- `nslookup-google-com.png` — google.com lookup using Cloudflare (1.1.1.1)  
- `nslookup-google-cloudflare-1.1.1.1.png` — cloudflare.com lookup using Cloudflare  

Each screenshot validates hands-on execution of the DNS workflow.

---

## Conclusions
This lab demonstrated:

- How Windows resolves DNS queries  
- The impact of DNS caching  
- Differences between internal and external DNS resolvers  
- TTL behavior in repeated lookups  
- Practical skills applicable to SOC detection, threat hunting, and IR  

These fundamentals are essential for identifying malicious domain activity and analyzing DNS-based attacker techniques.

---

## Next Steps
To expand your DNS analysis skills:

- Capture DNS traffic in Wireshark  
- Compare DNS behavior on Linux using `dig`  
- Investigate DNS over HTTPS (DoH)  
- Explore DNS tunneling detection techniques  
- Analyze DNS logs in a SIEM (ELK, Splunk, Sentinel)

This lab establishes foundational DNS knowledge for network detection and forensic workflows.
