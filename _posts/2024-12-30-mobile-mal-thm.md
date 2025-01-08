---
title: Mobile Malware Analysis
date: 2024-12-15
categories: [malware analysis, writeups]
tags: [malware analysis, mobile]      # TAG names should always be lowercase
---








# Mobile Malware Analysis on Try Hack Me




## Challenge info:


![alt text](/assets/imgs/mma-imgs/image.png)


* Intro:


   * "It's incredible how often our computers are in the scope of cyber attacks. Antivirus has become an indispensable shield to provide us with a more secure environment, since we are exposed to destructible malware and cyber attacks. Inside our pockets, we have computers so powerful, but much smaller, we must be equally attentive on our phones, because we can suffer equally damaging attacks, sometimes even worse, because they can store relevant information such as private conversations and important accounts."






## 1: An Unknown Land


"It is important to look at the past to understand why things are as they are today.


A new technology, due to the lack of exploration, appears to be extremely reliable.
Every system is reliable, until someone proves otherwise.


You will need to do some research in order to answer the questions in this task."




* What is known as the first malware created to affect mobile devices?
   * This question simply requires some research. After some quick googling we can find the answer which is `cabir`


   ![alt text](/assets/imgs/mma-imgs/image-1.png)


* What technology does this worm use to multiply?
   * This question can be answered by viewing the following post.
   > <https://medium.com/threat-intel/mobile-malware-infosec-history-70f3fcaa61c8>


![alt text](/assets/imgs/mma-imgs/image-2.png)


* What operating system did it infect?
   * This can also be found in the same article


![alt text](/assets/imgs/mma-imgs/image-3.png)


* What message did it show on the screen of the infected mobile phone?
   * This can also be found in the article


![alt text](/assets/imgs/mma-imgs/image-4.png)


## 2: Small size, a lot of destruction


After starting the attackbox we can begin answering the questions.




* Deploy the machine & use MobSF to scan the file named "TWFsd2FyZQ.apk" that is located on the Desktop.
   * No answer needed.




* What is the format of the file?
   * This is pretty easy to see as the file is a `.apk` file.


* The sample's size is 10,1 bytes, so it seems that it is not a complex application.
   * No answer needed.


* Decode the name of the sample.
   * We can figure this out by popping the file name into cyberchef. I assumed base64 as the encoding method and that was right.
   ![alt text](/assets/imgs/mma-imgs/image-5.png)


* Which is the target platform?
   * I assumed due to the file type that this would be a android sample and that was correct.


## 3: Digging Deeper


"Let's make a deeper analysis.


VirusTotal is an incredible service, this web site can give us the power of analyzing a package with the database of more than seventy Anti-Virus, and the result is fast and accurate.


To analyze the file in VirusTotal, you will need the file hash, you can get it by using the powershell cmdlet "Get-FileHash" or you can analyze the file with MobSF and it will show the file hash (we will get back to this tool in the next task)."






First let's get the file hash...


![alt text](/assets/imgs/mma-imgs/image-6.png)






VT scan:


![alt text](/assets/imgs/mma-imgs/image-7.png)




* What does Avast-Mobile can tell us about this software?
   *  At first I used the avast-mobile analysis on VT but I found the answer to be under just avast.
   ![alt text](/assets/imgs/mma-imgs/image-8.png)


* What program was used to create the malware?
   * In the sample results on VT we can see references to `metasploit` and `meterpreter`. `Meterpreter` is a payload in `metasploit.` `Metasploit` was used to create the malware.


* What is the package name?
   * This can be found in the VT details.


![alt text](/assets/imgs/mma-imgs/image-9.png)


* What is the SHA-1 signature?
   * This can also be found in the VT details.


![alt text](/assets/imgs/mma-imgs/image-10.png)


* What is the unique XML file?
   * This can be found in the relations section of the VT page.


![alt text](/assets/imgs/mma-imgs/image-11.png)


* How many permissions are there inside?
   * We can find this by inspecting the VT page for the xml file.


![alt text](/assets/imgs/mma-imgs/image-13.png)


![alt text](/assets/imgs/mma-imgs/image-12.png)


* Which permission allows the application to take pictures with the camera?
   * We can find this by either googling the android manifest permissions or byt taking a closer look at the perms in the file.


   ![alt text](/assets/imgs/mma-imgs/image-14.png)




* What is the message left by the community?
   * This can be found in the community tab in the VT page for the apk file.


![alt text](/assets/imgs/mma-imgs/image-15.png)


## 4: MobSFing the sample


"Let's use MobSF(Mobile Security Framework) to make a deeper analysis of this file, MobSF is a software created to make a security focused analysis of Android and IOS files. It can check for misconfigurations, leaked data and much more in a mobile program.


This tool can be used for static and dynamic analysis, in this room we will focus only in the static analysis but you are free to install it in a virtual machine you own to understand more how the application works, you can install it in GitHub - https://github.com/MobSF/Mobile-Security-Framework-MobSF.


The machine is configured to start MobSF when deployed, if you accidentally close the web page you can visit the MobSF page by visiting the link http://127.0.0.1:8000 inside the deployed machine. Press the "Upload & Analyze" button and select the file we have been working on."


* What is the programming language used to create the program?
  
   * I honestly guessed this question before confirming it as it was an android sample so I guessed java and it was correct. We can also easily see that confirmed below.


   ![alt text](/assets/imgs/mma-imgs/image-21.png)




* How many signatures does the package have?
   * We can find this in the signer certificate section of the MobSF scan.


![alt text](/assets/imgs/mma-imgs/image-16.png)




* Application is signed with v1 signature scheme, what is it vulnerable to on Android <7.0?
   * We can find this below the signer certificate section.


   ![alt text](/assets/imgs/mma-imgs/image-17.png)


"MobSF gives all the code decompiled. Just a base of programming make us able to understand a little bit of what is happening. "


"This malware is used to create a connection with the victim that is called a reverse shell."




* What is the App name?
   * I found this by viewing the full report for `zenbox` in the VirusTotal behavior section previously. You can also find it in mobSF.


   ![alt text](/assets/imgs/mma-imgs/image-18.png)


   ![alt text](/assets/imgs/mma-imgs/image-20.png)




* It looks like  there is a function calling for the package manager, so it can see all the installed applications. What function is that?
   * This can be found in the `payload.java` file.


   ![alt text](/assets/imgs/mma-imgs/image-22.png)


* The flag "android:allowBackup" allows the user to backup application data via USB debugging. It is recommended that this be set as "False", even if by default it is "True". What is the severity of this configuration?
   * We can find this by looking at the manifest analysis section of the scan.


![alt text](/assets/imgs/mma-imgs/image-23.png)


## 5: It doesn't smell good!


* What is the SHA-256 hash of the file?
   * We can find this immediately in the MobSF report.


![alt text](/assets/imgs/mma-imgs/image-24.png)




* After finding the sample on VirusTotal, what does the "Avast" anti-virus engine recognizes it as?
   * After inputting this sample into VT we can see the following.
   * This is a sample of pegasus spyware which is a pretty nasty and notorious piece of spyware.


![alt text](/assets/imgs/mma-imgs/image-25.png)


* With what we have, try to find out the name of the sample.


   * Based on the above I used the name `pegasus` and it worked.


* It seems like it is a very dangerous malware and has a big history of destruction. This became news for spying journalists, what year was that?


   * After some brief research we can find that the first reported sighting of the spyware was in 2016 but it didn't garner much attention until 2017.


* If we search the name we found of the malware in MITRE ATT&CK (https://attack.mitre.org/), we can find some interesting information. What is the ID of the MITRE ATT&CK that is associated with our sample?
   * If we search on MITRE ATT&CK we find the following.


   ![alt text](/assets/imgs/mma-imgs/image-26.png)


* What technique has the ability to exploit OS vulnerabilities to escalate privileges?
   * If we look at the techniques listed on the page we can find the following technique.


  ![alt text](/assets/imgs/mma-imgs/image-28.png)


"Now, let's go back to the MobSF analysis."


* There is a permission that when accepted, allows the application to access the list of accounts in the Accounts Service. What is the status shown by MobSF regarding this permission. (android.permission.GET.ACCOUNTS)
   * If we search for "accounts" in the permissions section of the MobSF scan we can find the permission and the status associated with it.


![alt text](/assets/imgs/mma-imgs/image-29.png)


* What org.eclipse.paho.client file refers to properties of Portuguese from Brazil (pt-br)?


   * We can find this in the files section of the MobSF report.


   ![alt text](/assets/imgs/mma-imgs/image-30.png)




"This software has several features that make the identification and the processes it performs to explore the target, harder to handle, even when it is being analyzed."


* The malware has a special appeal for its safety and its internal components, reducing the risk of compromise. It has a functionality for its cryptographic operations with the feature of a random bit generation service. How can it be identified?


   * We can find this in the NIAP section of the MobSF report.


![alt text](/assets/imgs/mma-imgs/image-31.png)




## Conclusion:


This was a pretty simple room detailing the basics of mobile malware analysis focusing on MobSF.


![alt text](/assets/imgs/mma-imgs/image-32.png)





