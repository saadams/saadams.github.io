---
title: Analyzing SambaSPY RAT on and Beyond LetsDefend
date: 2025-09-05
categories: [malware analysis, research]
tags: [malware analysis, itw]      # TAG names should always be lowercase
---



# Analyzing SambaSPY RAT on and Beyond LetsDefend




## Challenge Info:


LetsDefend Scenario:


"Your organization has discovered an infection on one of its systems involving a malicious Java application. This malware performs environment checks to ensure it is not running inside a virtual machine and targets systems with specific configurations. Once the required conditions are met, it extracts files and executes malicious components that could compromise sensitive data or system integrity. The stealthy nature of the malware and its ability to evade detection pose a serious threat, requiring immediate action to secure the network and prevent further compromise."


<https://app.letsdefend.io/challenge/samba-spy>




> Note: this is designed to be an analysis of the malware sample provided, not simply a solution to the CTF style challenge; it will go beyond the answers requested in the challenge. However the answers will be provided within the analysis.








## Background Information About SambaSPY:


SambaSPY RAT is linked to Brasil and was detected back in September of 2024 and it was designed to only affect Italy/Italians (or anyone using an Italian keyboard layout). The sample was distributed via a phishing campaign in which the users would contain a malicious attachment. If this malicious attachment was unzipped it would execute a stager which would check that certain conditions were met and if so, it would launch the malware.


The malware had two different variants:




![alt text](image-36.png)


In this case we are dealing with variant B which we will see later on.


Source:
<https://thehackernews.com/2024/09/new-brazilian-linked-sambaspy-malware.html>




## Analysis:


> Note: By default Lets Defend uses a vm on their platform to let us analyze the provided sample and solve the challenge. I wanted to use my own vm so I found the sample with the corresponding hash.


![alt text](image-7.png)






#### Code Extraction:


The malware is written in Java and the code must be extracted from within the `jar` file.


![alt text](image-8.png)


![alt text](image-9.png)


 The `class` contained is the compiled binary file.


![alt text](image-10.png)


> This can also be found within the `MANIFEST.MF` file.




The `class` file can be opened and analyzed using Ghidra.


![alt text](image-11.png)






#### VM Detection:


Starting in main there is a conditional check which checks whether or not the malware is being run in a vm.


![alt text](image-12.png)


The `isRunningInVM` function checks a variety of OS name properties and uses them to determine if the system is a VM.


![alt text](image-13.png)


The function getSystemProperty is using `System.getProperty` with the param of `"os.name"`


![alt text](image-14.png)


The result of System.getProperty("os.name") is used with the conditional that iterates through the list of names checking if the return value contains any matches to any of the following:


![alt text](image-15.png)


The next method involves using the command `wmic get baseboard manufacturer` which is used to determine the motherboard manufacturer.




![alt text](image-16.png)


If the motherboard name matches any of the names in the list it determines that it is being run under a VM.


---


#### Trigger Condition Check:


Next the malware checks for a trigger condition which in this case based on the printed string has to do with the language of the system being Italian.




![alt text](image-17.png)




![alt text](image-18.png)


If the system is not Italian the program continues and does not trigger the payload.




#### Payload Dropping:


Next it unpacks the zip folder and extracts the payload.


![alt text](image-19.png)


This code will extract the files from the zip folder


![alt text](image-20.png)


![alt text](image-21.png)


The zip file contains a "png" file `prodotto.png`(meaning "product")


![alt text](image-23.png)


Then the code searches for this file and attempts to use `java -jar` to execute the file.


![alt text](image-24.png)




The `png` file itself is flagged as being a malicious RAT written in java. 


![alt text](image-25.png)


Within the behavior tab in VirusTotal you will find indicators of malicious communication with a domain.


![alt text](image-26.png)


That information can be verified with further analysis.




#### RAT Analysis:


Extracting the “png” (`jar`) file reveals a large number of folders and files most of which are useless.


![alt text](image-27.png)


The information stored within the `MANIFEST.MF` will give us info on what class file we need to start.


![alt text](image-28.png)


Searching for that main class file.


![alt text](image-29.png)


After decompiling the file we can see that it contains a section of code that is used for network communication with a specified URL.


![alt text](image-30.png)


Within the class initializer there are multiple large strings of what look like encrypted domain names or potential commands.


![alt text](image-31.png)




Due to the obfuscated nature of this file the easiest way to analyze this will be to dynamically analyze it using `FakeNet`.


> This allows the domain name to be decoded.


![alt text](image-32.png)


Running the binary results in a connection to the following domain.


![alt text](image-33.png)


This also matches what was reported in the VirusTotal results.


Searching this URL on VirusTotal results in a large number of hits to various resolved IP addresses and has a large list of malicious files associated with the URL.


![alt text](image-34.png)


![alt text](image-35.png)




Based on a dynamic execution report the file also modifies register keys to auto run the program and maintain persistence.


![alt text](image-37.png)


> This sets the program to be run everytime the user logs into windows.


Based on the information and analysis above it is likely that this is a malicious file that attempts to hide itself, maintain persistence, as well as establishes communications with a C2.








# Conclusion:


This malware was detected uniquely targeting Italian speaking individuals and being distributed via malicious email attachments. Based on the above results we can see that this malicious file given the right conditions will infect a victim's machine with a RAT.



































