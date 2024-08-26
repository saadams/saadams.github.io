---
title: Malware Categories, The ATT&CK Framework and Research/Report Stratagies
date: 2024-07-05 
categories: [malware analysis]
tags: [malware analysis, windows, reverse engineering]      # TAG names should always be lowercase
---


# Malware Categories, Research Tactics and the MITRE ATT&CK Framework.

Below common terms and categories of malware will be covered then research strategies and industry frameworks will be discussed. Various malware types will be listed and briefly described, the MITRE ATT&CK framework will then be described and research strategies will follow.



## Malware Categories: 

* Viruses:

    * Viruses are malicious programs that infect legitimate software when a user opens the executable or document. The main goal of viruses is to propagate within a local machine rather than spread like a worm.

        Worms are self-replicating malware that spread over networks and propagate between machines. They can use various methods to spread; they can also install backdoors for remote access.

* Trojans (Trojan Horses):

    * Trojans disguise themselves as legitimate software to trick users into downloading and executing them.

* Ransomware:

    * Ransomware encrypts files on a victim's computer and demands payment (often in cryptocurrency) to decrypt them. It can cause significant data loss or financial damage to individuals and organizations.

* Spyware:

    * Spyware covertly gathers user information without their knowledge. It can be used to track keystrokes, capture screenshots, monitor browsing habits, and collect sensitive data, which is then sent to a C&C server.

* Infostealer/Password stealer (PWS):

    * Similar to spyware but has the express purpose to steal credentials.

* Bankers:
    * A type of infostealer focused around stealing financial credentials and gaining access to money. Often these will include methods of intercepting 2FA and modifying financial data/information to redirect money transfers and payments. 

* Adware:

    * Adware automatically displays or downloads advertisements on a user's device. It can slow down system performance, change browser settings, and lead to a poor user experience.

* Rootkits:

    * Rootkits are stealthy malware that gain privileged access (root/administrator level) to a computer or network. They hide their presence from detection tools and can be used to maintain unauthorized access or control. These can operate in kernel mode and be used to hide modules.


* Keyloggers:

    * Keyloggers record keystrokes on a computer, enabling attackers to capture sensitive information such as usernames, passwords, credit card numbers, and other personal data.

* Botnets:

    * Botnets are networks of infected computers (bots) that are remotely controlled by attackers (botmasters). They can be used to send spam emails, launch distributed denial-of-service (DDoS) attacks, or perform other malicious activities.

* Backdoors:

    * Can be known as Remote Access Trojens (RATs)  They are often installed by other malware and allow attackers to control infected systems remotely.

* Cryptojacking:

    * Cryptojacking involves using malware to hijack a victim's computer resources (CPU, GPU) to mine cryptocurrencies without their consent. This can slow down the system and increase electricity usage.
    
* Fileless Malware:

    * Fileless malware operates in memory, leaving little or no trace on the hard disk. It can exploit vulnerabilities in software or use legitimate system tools to execute malicious actions, making detection and removal challenging.

* Polymorphic Malware:

    * Polymorphic malware changes its code and appearance each time it infects a new file or system. This variation helps evade detection by antivirus software that relies on signature-based detection.

* Macro Viruses:

    * Macro viruses infect documents and spreadsheets that support macros. When the infected document is opened the macro executes malicious actions.

* Bootkits:

    * Bootkits infect the master boot record (MBR) or volume boot record (VBR) of a computer's hard drive. They load before the operating system and can control the system at a very low level, making them difficult to detect and remove.

* Scareware:

    * Scareware deceives users into believing their computer is infected with malware or has other problems. It prompts them to purchase fake antivirus software or services to resolve non-existent issues, thereby generating profit for the attackers.

* Wiper Malware:

    * Wiper malware is designed to permanently destroy or overwrite data on a victim's system. 

* Ghostware or self-deleting malware:

    * This type of malware removes traces of itself after achieving its objectives, making detection and analysis difficult. 

* Hybrid Malware:

    * It is common that malware combines techniques of different types of malware to maximize its effectiveness. 




## The MITRE ATT&CK Framework:

From <https://attack.mitre.org/> (2024):

“MITRE ATT&CK® is a globally-accessible knowledge base of adversary tactics and techniques based on real-world observations. The ATT&CK knowledge base is used as a foundation for the development of specific threat models and methodologies in the private sector, in government, and in the cybersecurity product and service community.”

The MITRE ATT&CK framework allows security experts to communicate in a global knowledge database on attack techniques which are grouped into tactics. The site also provides examples of how these techniques are used by attackers and in malware. 

### Breaking Down the Terms:

* Tactic:
    * The goal of the attack and the reason why the attack/action is being done.
* Technique:
    * The way in which the goal is achieved
* Sub-technique:
    * A more detailed description of how a technique is performed.
* Procedure:
    * The full implementation of the techniques used.
* TTPs:
    * Tactics,techniques and procedures. Summarizes the goals and methods used by an attacker.
* Group:
    * Actions/attackers performed by an entity or “group” known by some given name.
* Mitigation:
    * Actions used to prevent an attack.
* Matrix:
    * A combo of TTPs in a given industry sector.
* Industrial Control Systems (ICS):
    * An electronic control system used to control an industrial process.


The framework focuses on three major matrices: enterprise, mobile and ICS. Enterprise is a very common matrix to use so let's break it down.


### Enterprise Matrix Terms:

* Reconnaissance:
    * How an attacker is gathering information to be used to launch an attack. 
* Resource Development:
    * What attacks need to establish required resources based on what they collected during recon. This can include creating or buying resources such as email accounts, domains, software, etc…
* Initial Access:
    * How the attack is breaching the network or service. The entry attack vector of the attacker to gain a foothold in the system or network. Common example is spear phishing 
* Execution:
    * The process of the attacker running malicious code to achieve their goal.
* Persistence:
    * The act of the attacker attempting to maintain a foothold in the network/system. Ex. Bootkits, code/process injection, cron jobs, etc…
* Privilege Escalation:
    * Actions an attacker takes to elevate their privilege/permissions within a system or network. Ex: gaining access to a root or admin user/account.
* Defense Evasion:
    * The actions an attacker takes to remain undetected while they are attempting to complete their operation.
* Credential Access:
    * The techniques an attacker uses to attempt to steal credentials.
* Discovery:
    * The techniques used by an attacker to gather information about the victim system and network.

### Additional Terms:
* Command and Control:
    * The way an attack may interact/communicate with a victim's system. Ex: C2C server.
* Exfiltration:
    * How attackers move data or information out of a system.
* Lateral Movement:
    * How an attack can propagate or move through a victim's network to other machines.
* Impact:
    * The impact or effects that an attacker could cause.
* Zero Day:
    * An attack that is not previously known or patched and therefore has no easy solution for securing the vulnerable system. Sometimes you will get evasion techniques that can sometimes be used to minimize risk to a vulnerable system until a patch is released.
* Indicator of Compromise  (IoC):
    * Pieces of evidence that allude to a system being compromised. Indicator examples: Virus signatures, MD5 hashes, Known urls/domains, system logs, etc…
* Advanced Persistent Threat (APT):
    * A highly sophisticated tailored threat/attack that targets a specific entity. These will often use zero day exploits or abstract and complex techniques that are difficult to detect. In malware these are often compiled per victim and can be very hard to identify with standard IoCs or AVs.
* In the Wild (ITW):
    * Examples of an attack in the “wild.” ie: a real world example.


> The ATT&CK framework website is an extremely helpful tool/resource and provides huge amounts of information. <https://attack.mitre.org/>



## Research and Analysis Methods:

### Analysis Workflow:
* Triage:  
    * Gathering the most amount of information about the sample
        * PE header 
        * Is the sample packed?
        * Check for relevant public information about the sample if available.
* Behavioral analysis:
    * Searching for information about the capabilities of the sample.
        * Unpack the sample
        * If needed you will need to unpack the sample before you can continue to static analysis.
* Static analysis:
    * Use disassemblers/decompilers to examine the code of the sample.
        * Searching for strings, WinAPI calls, etc…
        * Goal is to get an understanding of what the sample is doing and how it is doing it.
* Dynamic analysis:
    * Using debuggers to confirm suspected functionality of a sample.
        * Can be used to examine communications,APIs, payload delivery, etc…
        * Similar goals as static analysis but can be a costly time consuming process so it is primarily only used when necessary.

### Reporting Methodology:


#### Know Your Audience:

Reverse engineering is a highly technical and time consuming process and oftentimes you will need to prioritize sections of your analysis to make sure your deliverable result is the best it can be. 

Your level of technicality and what you focus on in your report will vary based on who you are presenting it to. 

Some examples:

* Using an AV:
    * Focus on finding unique aspects of the malware to allow for predictable consistent detection with no false positives.
* Reporting for threat intelligence:
    * Focus on finding IoCs and artifacts left by the malware. 
* Reporting to non-tech roles:
    * Use high level language and focus mainly on the impact of the attack. 


Regardless of who you are presenting to you can omit overly technical language or details if it will lead to a waste of time. When talking about technical aspects try to keep it brief and avoid falling into a “rabbit-hole.”  Never present information based on a “hunch,” always have facts/evidence in the sample to support what you are reporting.

#### Reporting Structure:
	
* Technical:
    * Include:
        * Hashes
        * Compilation dates
        * File size and type
        * ITW filenames
        * AV information on the sample
    * Modules:
        * What is used?
        * Persistence techniques
        * Network Comms information
        * Protocols used ex: TCP
        * Encryption used?
        * C&C information
        * Anti-reversing techniques used
    * IoCs
    * Impact
    *  Mitigation if possible
* Non-Technical:
    * High level functionality
    * Impact
    * Targets
    * Advasiarys/Groups involved
    * Similar examples
    * Language used in sample
    * Compilation dates



## References/Resources:



* Mastering Malware Analysis by Alexey Kleymenov and Amr Thabet
  * <https://www.packtpub.com/en-us/product/mastering-malware-analysis-9781803240244>
* Practical Malware Analysis by Michael Sikorski and Andrew Honig
  * <https://nostarch.com/malware>

Here is an additional example from Blackhat USA that talks about how the ATT&CK framework can be used. 

><https://i.blackhat.com/USA-19/Wednesday/us-19-Nickels-MITRE-ATTACK-The-Play-At-Home-Edition.pdf>


Here is a short article about what to include in a threat report focusing on malware.

> <https://zeltser.com/malware-analysis-report/>


