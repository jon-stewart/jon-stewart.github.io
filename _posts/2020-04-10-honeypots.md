---
title:  "Honeypots"
tags:
  - honeypots
toc: true
toc_sticky: true
related: false
---

Running a vulnerable server for fun and profit

Honeypots are by design unprotected and vulnerable - deployed with the goal of enticing malicious actors to attack.  The value in running a honeypot comes from the recording of attacker activities.  With the correct design and deployment, a honeypot can capture every stage of an attack - from exploitation through to action on objectives.

The information captured comprises Tactics, Techniques, and Procedures ([TTP](https://posts.specterops.io/whats-in-a-name-ttps-in-info-sec-14f24480ddcc?gi=ca00fb8ba42f)), the tools used, and attacker motivations.  From this it is possible to produce Indicators of Compromise (IoC) which can be used to identify compromised systems and ensure that coverage is in place to protect infrastructure.  TTP can even constitute a fingerprint for threat actors or groups - used to track campaigns and tool development.  In this way the honeypot acts as a proactive tool - indirectly aiding defence and reducing the asymmetry of information between attacker and defender.

# Considerations

There are considerations to be made in regard to the design and operation of honeypot.

- **Honeypot are noisy.**  A honeypot will attract a tsunami of traffic which will largely consist of scanning and automated low effort attacks.  The challenge is finding value in the resulting noise.  Or perhaps the value of honeypot data is in its use as 'anti-intelligence', a filter for other traffic.
- **Centralised data management.**  Filtering is only going to be possible if honeypot data is stored in a centralised location - never mind the fact that writing data to local logs files will not scale as more honeypot are employed.  Consider using data pipelines and the ELK stack.
- **Honeypot should be isolated.** The database and visualisation software will constitute a support infrastructure for the honeypot that should be secured and kept isolated, preferably on a different account.  Hosting companies will close accounts if complaints are received about questionable activities originating from the honeypot.
- **Deployment locations can have a significant effect on the data collected.**  The IP addresses associated with countries may be included or excluded from certain attacks.  A Cisco switch honeypot will not be believable if running on cloud hosting.
- **Collecting malware can be a liability.**  Malware must be safely and securely stored - illegal in some countries - and isolated from other infrastructure to minimise the risk of accidental execution.  Samples will be as numerous as attack traffic and you will want to filter out duplicates - I suggest the use of [SSDeep](https://ssdeep-project.github.io/ssdeep/index.html) with its fuzzy hashing.

# Design

Categorising honeypot by the level of interaction they offer attackers is a great way of approaching honeypot construction.  Interestingly the transition from low to high interaction overlaps nicely with honeypot believability.

## Low Interaction

Low Interaction honeypots superficially emulate the fingerprints and vulnerable aspects of an application.  Importantly, attacks have no effect beyond that of being recorded.  The benefits of this level of interaction are low cost and safety.  The software tends to be lightweight - quick to create and cheap to operate - and outside of developer error it is not possible to lose control of the honeypot.

![low interaction honeypot](/assets/images/honeypot/lowInteractionPot.png)

However minimal interaction has a drawback in that a careful attacker can ensure that the honeypot captures nothing of value.  Some attacks will stop if the vulnerability does not act as expected and the use of multi-stage payloads - with a few tricks - will leave an initial stage URL that will return 404 Page Not Found when later accessed.

## Medium Interaction

The next step - medium interaction - is to emulate an OS shell, selection of tools, and filesystem.  This offers attackers a sandbox to play in while ensuring that they cannot cause any real trouble.  The hope is that the emulated tools will be used, recording attacker procedures.

![medium interaction honeypot](/assets/images/honeypot/medInteractionPot.png)

Emulation will always falls short, with the quirks and limitations being used to fingerprint and identify the honeypot.  A further problem is that code execution is (typically) prevented - like low interaction honeypots, it is possible to protect the later stages of an attack.

## High Interaction

Why emulate when it is possible to hook a real OS to the vulnerable paths.  This offers the most believability in post-compromise.  Attacks should be directed to Virtual Machine or Docker containers, with automation used to periodically recycle/reset state.  VM snapshots or memory dump can be used to help in analysis.

![high interaction honeypot](/assets/images/honeypot/highInteractionPot.png)

But beware, there is great danger in giving attackers unrestricted OS access.  Potentially greater reward comes with the higher risk. The honeypot could be used in the propagation of malware and the exploitation of real vulnerable systems.  This could lead to the loss of a hosting account or your job if your organisations customers are attacked.  It is also important that the OS contains what the attacker expects; the vulnerable application that was compromised and state changes they may have previously made.

## Pure

Pure honeypots are real - not necessarily vulnerable - systems that allow attackers full run of the OS with network and kernel level monitoring to record activities.  These are the most believable of targets.

![pure honeypot](/assets/images/honeypot/purePot.png)

Honeypot tend to suffer from a common limitation - they only catch attacks against known vulnerabilities.  It is only with pure honeypot that there is any hope of catching a zero day exploit - assuming sufficient monitoring is in place to catch it.  These are going to be long running, high risk, high maintenance systems - with potential for the greatest rewards.
