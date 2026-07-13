---
title: "AI Innovation Challenge at Worldline: Internal Hackathon and Strategic AI Projects"
date: 2026-07-13 10:00:00 +0200
categories: [AI]
tags: [Worldline, Hackathon, AI Projects, Internal Innovation, Multi-Agent Systems, Agents]
toc: true
---
# Worldline AI innovation

## Hackaton

For the month of july, Worldline launches the AI innovation challenge. This internal event happens in a context where companies want to be part of the AI transition. Team of maximum 5 people are brought to propose innovated and internal AI ideas. From the first to 10 july, teams pick different subject that they want to handle and submit a compelling pitch explaining the challenge, the solution, and why the team will succeed. Then, they have a selection part where experts select the team entering the final sprint at the end of july during 3 days as a hackaton format. 

Teams must choose one of the six strategic domains:

- Software Delivery
Developer tooling, engineering velocity, code generation, testing, CI/CD optimization.
- Internal Operations
Finance, HR, and back‑office workflows.
- Customer Experience
Onboarding, servicing, dispute resolution, and customer interaction processes.
- Payment & Money Movement
Transaction processing, reconciliation, settlements.
- IT & Infrastructure Operations
Platform reliability, network operations, incident management, monitoring.
- Risk, Compliance & Fraud
Regulatory compliance, fraud detection, AML/KYC, audit.

My Team combines technical diversities with different domain specializations across three countries from France, Sweden to Romania and is composed by:

- Alexandra-Catalina, project manager in Romania
- Baptiste Guillot, information security officer in France
- Mattias Westlund, security engineer in Sweden
- Julien Jimenez, security chapter lead in France

### Project 1: question form agent ([https://van-tran-19.github.io/posts/ai-agents/](https://van-tran-19.github.io/posts/ai-agents/))

**The Everyday Problem:**
Worldline teams spend hours answering client security and compliance questionnaires covering policies, risk management, IAM, physical controls, and third‑party oversight. Banks and regulated institutions rely on these questionnaires to assess Worldline as a critical ICT provider, but today the process is fully manual: Sales, Compliance, and Security teams search internal documentation, repeat answers across hundreds of questions, and risk delays, inconsistencies, and errors.

**Solving this matters** because it accelerates deal cycles, strengthens customer trust, reduces compliance risk, and cuts operational cost.

**Success looks like** faster, consistent, validated responses powered by a multi‑agent GenAI system trained on internal documentation and historical questionnaires. 

### Project 2: ReconAI

**The Everyday Problem:**
Cybercriminals are heavily automated. They use powerful AI scripts to rapidly scan company networks for weaknesses. Meanwhile, Worldline's ethical hackers (like Elmedhi [https://elmehdilaassiri.github.io/](https://elmehdilaassiri.github.io/)) are still doing their homework by hand.

Before our security team can even try to find a vulnerability, they spend two to three days running manual scans, reading lines of messy computer code, and copy-pasting IP addresses into Excel. They are wasting valuable time organizing data while the bad guys are moving at lightspeed (with the arrival of GLM5-2, the new open source cyber security weapon of China [1]).

**How the AI Fixes It:**

**ReconAi** splits the boring work among five specialized AI agents:

```
[The Orchestrator] ➔ Launches the automated scanning tools.
       ↓
[The Analyst]      ➔ Translates raw, unreadable code into plain English.
       ↓
[The Cartographer] ➔ Draws a visual map of our entire digital perimeter.
       ↓
[The Strategist]   ➔ Points out exactly where the weakest spots are.
       ↓
[The Reporter]     ➔ Packages everything into a neat report.
```

**The Win:** 

It cuts down the prep work from **days to hours**. Our hackers can stop acting like data organizers and start doing what they do best: finding and fixing security flaws before real attackers do.

### Project 3: AccessGuard

**The Everyday Problem:**
Every few months, corporate rules say we must audit who has access to Worldline’s critical systems. We need to make sure former employees don't still have active accounts and that nobody has more digital keys than they actually need to do their job.

Today, a manager is handed a spreadsheet with **5,000 lines of data** and told to review it. Human eyes naturally glaze over after line 50. It’s slow, universally hated, and highly dangerous if a manager accidentally misses an unauthorized account.

**How the AI Fixes It:**

**AccessGuard** acts like an **AI Detective**. Instead of forcing a manager to read 5,000 lines, the AI reads the entire document in seconds, cross-references it with company policies, and hands the manager a tiny, curated list. 

**The Win:** We turn a painful, weeks-long headache into a quick 10-minute check, ensuring Worldline stays completely secure and audit-ready.

## Sources

1. [https://www.secureworld.io/industry-news/china-glm-5.2-mythos-vulnerability-detection](https://www.secureworld.io/industry-news/china-glm-5.2-mythos-vulnerability-detection)
