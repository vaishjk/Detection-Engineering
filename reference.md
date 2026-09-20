# Day 1: MITRE ATT&CK Primer

MITRE ATT&CK is a public catalog of how real attackers behave. It gives defenders, red teams and threat reports a shared vocabulary. It has four levels:

- **Tactic:** the attacker's goal at that stage (the *why*), like Credential Access.
- **Technique:** the general method (the *how*), like Brute Force.
- **Sub-technique:** a specific variant, like Password Spraying.
- **Procedure:** a named group or tool actually doing it, like Fox Kitten brute-forcing RDP.

The current Enterprise matrix (v19) has 15 tactics. Defense Evasion is now called Stealth, and Defense Impairment is a new tactic for disabling or degrading defenses. Older guides still say 14 tactics.

## The five techniques at a glance

| Technique | Tactic | Sub-techniques | In one line |
|---|---|---|---|
| [T1498 Network Denial of Service](https://attack.mitre.org/techniques/T1498/) | Impact | 2 | Fill the pipe so real users can't get through |
| [T1071 Application Layer Protocol](https://attack.mitre.org/techniques/T1071/) | Command and Control | 5 | Hide attacker traffic inside normal protocols |
| [T1110 Brute Force](https://attack.mitre.org/techniques/T1110/) | Credential Access | 4 | Get into accounts by trying passwords at scale |
| [T1078 Valid Accounts](https://attack.mitre.org/techniques/T1078/) | Initial Access, Persistence, Privilege Escalation, Stealth | 4 | Log in with a real account so nothing looks wrong |
| [T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/) | Initial Access | None | Break in through a flaw in something exposed to the Internet |

## Notes on each technique

### T1498: Network Denial of Service (Impact)

**In plain words.** The goal is to use up the bandwidth a service relies on so real users can't reach it. MITRE's own example is 10 Gbps of traffic aimed at a server behind a 1 Gbps link. It can come from one machine or thousands (a DDoS). There are two flavors: a **direct flood**, where traffic goes straight at the target, and **reflection amplification**, where small spoofed requests are bounced off third-party servers so they send much bigger replies to the victim. Spoofed source IPs are a big reason these floods are hard to filter by source address.

**What it looks like.** From the victim's side: link saturation, sudden packet-rate spikes, and lots of traffic arriving from amplification ports. MITRE's detection ideas take the attacker's side instead: a script or tool sending huge outbound volume, or flooding tools like hping3 and nping.

**How to reduce it.** Once a flood is bigger than your link, you can't fix it locally. The traffic has to be dropped upstream by your ISP, a CDN or a DoS-mitigation provider. That is the one mitigation MITRE lists, along with having a continuity plan.

**Real examples.** APT28 ran a DDoS against the World Anti-Doping Agency in 2016, and the Lucifer malware can launch TCP, UDP and HTTP floods.

**Don't confuse it with** T1499 Endpoint Denial of Service, which targets the host or service itself instead of the network link.

<!-- Add your own hands-on notes here (no employer details) -->

### T1071: Application Layer Protocol (Command and Control)

**In plain words.** Attackers talk to their compromised machines using ordinary protocols such as web traffic, DNS, mail, file transfer and message brokers, so the traffic blends in and slips past filters. Commands and their results ride inside the normal protocol. Inside a network, the same idea uses SMB, SSH or RDP between pivot points.

**What it looks like.** Beacons: regular check-ins, often with a little jitter. Also lopsided flows, an unexpected process making HTTP or DNS requests, odd ports, unusually long or random-looking DNS names, and heavy TXT lookups. MITRE's detection ideas point to abnormal processes using common protocols with high outbound volume, beacon-like intervals, and tunneling such as DNS-over-HTTPS.

**The hard part.** Legitimate software beacons too (updaters, telemetry agents, CDNs). So the timing pattern alone isn't enough. Add context: which process made the connection, how rare the destination is in your environment, and how old the domain is.

**How to reduce it.** MITRE lists only two mitigations, filtering network traffic and network intrusion prevention. Prevention is limited here, so detection carries most of the weight.

**Real examples.** TeamTNT and Siloscape used IRC for C2, Velvet Ant used reverse SSH tunnels, and Sliver can run C2 over WireGuard.

<!-- Add your own hands-on notes here (no employer details) -->

### T1110: Brute Force (Credential Access)

**In plain words.** Getting into an account by trying passwords at scale, either against a live login or offline against stolen hashes. The four sub-techniques are really four different shapes:
- **Password Guessing:** many passwords against one account.
- **Password Cracking:** offline against stolen hashes, so it never touches your logs.
- **Password Spraying:** one or two common passwords across many accounts, kept slow to stay under lockout limits.
- **Credential Stuffing:** username and password pairs leaked from other sites, replayed against yours.

**What it looks like.** Failed logons (Windows 4625, sshd "Failed password", failed IdP sign-ins), often followed by a success. The shape tells you the variant: many failures on one account is guessing, few failures across many accounts is spraying. That's why a per-account threshold misses spraying, so count the distinct accounts targeted per source instead. MITRE's detection ideas focus on failures followed by a success and on attempts across many users in short windows.

**How to reduce it.** MFA is the big one. Also lockout and conditional access (MITRE warns that a too-strict lockout can become a denial of service, since attackers can lock real users out), a sensible password policy, and resetting accounts known to appear in breached credential sets.

**Real examples.** Fox Kitten brute-forced RDP, Chaos and Kinsing went after SSH, and CrackMapExec can test credentials across a network range.

**Why it matters next.** Brute force is a common way to get the valid account used in T1078.

<!-- Add your own hands-on notes here (no employer details) -->

### T1078: Valid Accounts (Initial Access, Persistence, Privilege Escalation, Stealth)

**In plain words.** No exploit and no malware, just a real account: stolen, guessed, leaked, or a default password nobody changed. Because the login works and looks normal, this is one of the hardest techniques to spot. It sits under four tactics because a valid account gets the attacker in (Initial Access), keeps them in (Persistence), can carry higher privileges (Privilege Escalation) and hides them in plain sight (Stealth). The sub-techniques are default, domain, local and cloud accounts. MITRE also calls out abusing inactive accounts, like a former employee's, because nobody is around to notice odd activity.

**What it looks like.** Nothing on the wire is malicious, so the signal is context: a login from a new place or at a strange time, an unusual logon type, a service account logging in interactively, impossible travel, or repeated MFA prompts. MITRE's analytics cover Windows, Linux SSH and sudo, IdP sign-in logs, and Kubernetes configs used from unexpected places.

**How to reduce it.** MFA on all account types, conditional access, turning off legacy authentication that can't do MFA, and regularly auditing accounts (removing the ones nobody needs, and reviewing privileged ones).

**Real examples.** Volt Typhoon relies mainly on valid credentials for persistence. Akira, BlackByte and Play have used VPN credentials, and FIN10 got in through VPNs protected by a single factor.

<!-- Add your own hands-on notes here (no employer details) -->

### T1190: Exploit Public-Facing Application (Initial Access)

**In plain words.** The attacker exploits a weakness in something exposed to the Internet to get their first foothold: web applications, databases, services like SMB or SSH, VPNs and other edge devices. The weakness can be a software bug, a temporary glitch or a misconfiguration. It has no sub-techniques. In cloud or container setups, exploiting the app can lead on to the cloud metadata API or the container host. MITRE points to the OWASP Top 10 and CWE Top 25 for the most common weaknesses.

**What it looks like.** MITRE's detection idea chains four signals: unusual requests to a public endpoint, then a spike in 4xx/5xx errors, then the web server process spawning a shell or something like curl or wget, then an outbound connection to a new host. The chain matters, because scanners hit public apps all day and mostly fail, so the request alone is noisy.

**How to reduce it.** A WAF, fast patching of Internet-facing software, scanning your external attack surface, keeping public servers segmented from the rest of the network, running services with least privilege, and restricting outbound traffic from public servers. That last one doesn't stop the exploit, but it limits what the attacker can do afterward.

**Real examples.** APT29 exploited flaws in Citrix, Pulse Secure VPN, FortiGate and Zimbra. ShadowRay attacked exposed Ray servers used for AI workloads. In the Anthropic AI-orchestrated campaign, the adversary used Claude Code to deploy an exploit for an SSRF flaw.

<!-- Add your own hands-on notes here (no employer details) -->

## How the five fit together

A rough story: T1190 or T1110 gets an attacker in, T1078 lets them stay as a normal-looking user, T1071 gives them a quiet channel back home, and T1498 is what a noisy attacker does when the goal is disruption.

*Contains material derived from MITRE ATT&CK®, a registered trademark of The MITRE Corporation. Source: https://attack.mitre.org (ATT&CK v19).*
