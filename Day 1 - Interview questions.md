Six interview questions with sample answers

Use these as a guide and put them in your own words. Where you have real examples from your work, use them.

1. What is MITRE ATT&CK, and how would you use it as a detection engineer?

It's a knowledge base of real-world adversary behavior. Behaviors are organized by goal (tactic), method (technique) and variant (sub-technique), and each comes with real examples. I use it in three ways. First, as a shared vocabulary, so a rule, an incident report and a threat brief all mean the same thing by "T1078." Second, as a map, where I tag each detection with the techniques it covers, find gaps, and prioritize what threat actors targeting us actually use. Third, to drive hunt hypotheses and purple-team tests. It describes behaviors rather than IPs and hashes, so detections built on it last longer. One caveat is that a technique tag isn't coverage. A rule only counts as coverage once it's tested against positive and negative logs.

2. Explain tactic vs. technique vs. sub-technique vs. procedure, with an example.

Tactic is the why, the goal, like Credential Access. Technique is the how, like Brute Force (T1110). Sub-technique is the specific variant, like Password Spraying (T1110.003). Procedure is a named actor or tool doing it, like Fox Kitten brute-forcing RDP credentials, which is listed on the technique page. A technique can sit under several tactics too. Valid Accounts (T1078) appears under Initial Access, Persistence, Privilege Escalation and Stealth.

3. Why does Valid Accounts (T1078) appear under four tactics, and how do you detect it when the login is legitimate?

A working credential gets an attacker in, keeps them in, lets them escalate if the account is privileged, and looks like normal activity, all without malware. Since the login succeeds, I detect on context and baselines: unusual source, geography or time, abnormal logon types, service accounts logging in interactively, new devices, impossible travel, and repeated MFA prompts. I'd combine signals, because impossible travel alone is noisy (VPNs, mobile carriers). For mitigation: MFA, conditional access, disabling legacy authentication, and removing dormant accounts.

4. How do you tell brute force, password spraying and credential stuffing apart in logs?

I look at the shape: how many accounts, how many passwords, and how many sources.

Guessing: many passwords against one account, so many failures on a single user. A per-account threshold catches it.
Spraying: a few common passwords across many accounts, slowly. Per-account counts stay low and lockout thresholds are evaded, so I count distinct accounts targeted per source and time window.
Stuffing: leaked username/password pairs, mostly one attempt per account, often from distributed IPs, with a higher success rate. I look for distributed sources, client fingerprint anomalies, and successes after a failure.
Cracking: offline, so there are no logs on my side. The mitigation is strong password hashing and policy.

MFA covers all of them, and rate limiting or bot management helps with stuffing.

5. How would you tell a real C2 beacon over HTTPS or DNS (T1071) from a CDN, update service or telemetry agent?

I start with the behavior: regular intervals with some jitter, similar request sizes, a long-lived pattern, and a destination that's rare in my environment. Legitimate agents also beacon, so I add context. Which process made the connection (endpoint telemetry), how old and reputable the domain is, TLS fingerprints (JA3/JA4) and certificate details, and whether the traffic is asymmetric. For DNS, I look at long or high-entropy subdomains, heavy TXT use and NXDOMAIN rates. I allowlist known-good agents by process plus destination, then validate suspects with pivots like passive DNS and certificates.

6. Walk me through designing a detection for exploitation of a public-facing web app (T1190).

I follow one path.

Behavior: The attacker sends crafted input to an Internet-facing app and gets code execution, often followed by a web shell and a callback.
Telemetry: Web and WAF logs, process-creation events on the server, and egress flow logs.
Logic: I don't alert on the request alone, because scanners hit constantly and mostly fail. I correlate an exploit-like request, then errors, then the web server process spawning a shell or curl/wget, then a new outbound connection. Post-exploitation behavior is harder for an attacker to change than a payload string.
Testing: I replay a known-vulnerable app in a lab for positive cases, and use normal deploys and admin activity for negative cases.
Tuning: I allowlist known deployment scripts and expected child processes.
Rollout: I start in shadow mode, then alert with a runbook and an owner.
Upkeep: I track precision, alert volume and time to triage.

Mitigations are a WAF, patching, and restricting outbound traffic from public servers.

You can also expect a T1498 question, since it's your daily work. Use your own real examples for it rather than a generic answer, and don't claim production experience you don't have.
