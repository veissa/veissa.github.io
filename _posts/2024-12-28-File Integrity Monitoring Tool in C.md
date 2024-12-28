---
title: "FIM : File Integrity Monitoring Tool in C."
date: 2024-12-28 09:20:11 +/-TTTT
categories: [tools, projects]
tags: [malwares,projects, tools]
description: A file integrity monitoring tool that helps detect any unauthorized changes to critical files and ensures the integrity of the system.
---
**COMING SOON AFTER THE FINAL EXAMS**

 > I got bored of ctfs etc, and decided to take a break by building smthng...
 why C and not python? what can I say, I just fell in love with C, a raw && wild love ! just my style!
{: .prompt-info}

### Motivation : 

Idk why I thought of this, but think abt it, guarding only the front door isn't enough, one gotta keep an eye on the inside too, so in case bad luck caught to u, this is essential for detecting and mitigating post-exploitation activities. 

Being able to monitor file changes in real-time is a great feature to have for defending servers and workstations against attacks.

### Why it works ?

Many malware and advanced persistent threats (APTs) rely on modifying system files or configurations. This file integrity monitoring tool helps detect any unauthorized changes to critical files and ensures the integrity of the system.

#### Basic features : 

- Track hash values (MD5, SHA256) of critical files.
- Alert the user when changes are detected (including file additions, deletions, or modifications).
- Provide a dashboard (simple CLI output) showing file statuses.

