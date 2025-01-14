---
title: HTB Runner
date: 2024-12-12
categories: [web sec, writeup]
tags: [pentesting, web, Hack The Box]      # TAG names should always be lowercase
---


# Hack the Box Runner:


## Box info:


Link: <https://app.hackthebox.com/machines/Runner>


"About Runner
Runner is a medium difficulty Linux box that contains a vulnerability ([CVE-2023-42793](https://nvd.nist.gov/vuln/detail/CVE-2023-42793)) in `TeamCity`. This vulnerability allows users to bypass authentication and extract an API token, which can be used to enable debug features for executing system commands. By gaining access to a `TeamCity` docker container and compressing the `HSQLDB` database files, we can extract credentials for the user `matthew` and find an `SSH` key for `john`. After cracking the password, we can authenticate on the host filesystem. Upon inspecting the `/etc/hosts` file, we discover a running `Portainer` instance. Using `matthew&amp;#039;s` credentials, we access the subdomain externally. While authenticated, we find that we can create imgs/images, but our privileges are limited. After checking the version of `runc` on the host, we exploit a vulnerability ([CVE-2024-21626](https://nvd.nist.gov/vuln/detail/CVE-2024-21626)) through the imgs/image build function of `Portainer`, which allows us to create a SUID bash file on the host."


* Difficulty: Medium




## Recon:


First we can start out by pinging the machine to make sure it's up.


`ping <ip>`
lets

![alt text](/assets/imgs/htb-runner/image.png)




Perfect, now let's run a `namp` scan of the machine I am going to use `-sC -sV`.




```
 nmap -sV -sC 10.10.11.13                    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-12 14:11 EST
Nmap scan report for 10.10.11.13
Host is up (0.15s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp   open  http        nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://runner.htb/
8000/tcp open  nagios-nsca Nagios NSCA
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.13 seconds


```


Lets add `runner.htb` to the hosts file and then checkout the http site:


`sudo vi etc/hosts`


then add


`10.10.11.13 runner.htb`






Now the page can be visited at `http://runner.htb/` or the machine ip.




![alt text](/assets/imgs/htb-runner/image-1.png)




For starters `gobuster` can be run against this site to look for subdomains and dirs.


I am using the top subdomains 20000 lists from seclists


<https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-20000.txt>




`gobuster vhost -w  wordlist.txt -u http://runner.htb --append-domain`


This resulted in no results found....




After getting stuck for a bit I shifted to using a bigger subdomain list:


<https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/bitquark-subdomains-top100000.txt>




`gobuster vhost -w  subdomains_100000.txt -u http://runner.htb --append-domain`




![alt text](/assets/imgs/htb-runner/image-4.png)


This results in the subdomain `TeamCity` being found.




I later found out you can make custom wordlists with a tool called `cewl`


<https://github.com/digininja/CeWL>


`cewl http://runner.htb -w runner_wordlist.txt`




`gobuster vhost -w  runner_wordlist.txt -u http://runner.htb --append-domain`


![alt text](/assets/imgs/htb-runner/image-2.png)




This worked significantly faster than using a giant wordlist.






After we get the domain `TeamCity.runner.htb` we have to add it to our hosts file like we did before.


`sudo vi etc/hosts` then add `10.10.11.13 TeamCity.runner.htb`




Now we can view the page which appears to be a login.




![alt text](/assets/imgs/htb-runner/image-3.png)






## Exploiting:




If we search the web for the version number posted at the bottom of the page we can find multiple examples of vulns being reported.




![alt text](/assets/imgs/htb-runner/image-5.png)




![alt text](/assets/imgs/htb-runner/image-6.png)




With this info we can use the exploit script posted on exploit DB:


`python3 jetbrains_ex.py -u "http://teamcity.runner.htb" `




![alt text](/assets/imgs/htb-runner/image-9.png)




We can now use the creds to sign in:




![alt text](/assets/imgs/htb-runner/image-8.png)






Here is a link to more info about the TeamCity vuln: <https://www.sonarsource.com/blog/teamcity-vulnerability/>




The `TLDR` of the vuln is that you can send requests to the site and as long as the path ends in `/RPC2` you can avoid auth checks.


An example:


`curl <site_url>/app/rest/users/id:1/tokens/RPC2 -X POST`


`id:1` - is for the admin user.


returns:


"Token already exists"


We can send a delete request and then re-post:


`curl <site_url>/app/rest/users/id:1/tokens/RPC2 -X DELETE`


`curl <site_url>/app/rest/users/id:1/tokens/RPC2 -X POST`


it will return a token that can be used to authenticate and create an admin account.




Returning to the site we can examine the users page:




![alt text](/assets/imgs/htb-runner/image-10.png)








I initially tried to simply change this "John" user's password and then ssh in with it but that did not work.


![alt text](/assets/imgs/htb-runner/image-15.png)






We can also find the option to backup the server.




![alt text](/assets/imgs/htb-runner/image-11.png)






We can download this and then poke around.




![alt text](/assets/imgs/htb-runner/image-12.png)






![alt text](/assets/imgs/htb-runner/image-13.png)


> You might be able to crack those hashes but I quickly found a ssh key






![alt text](/assets/imgs/htb-runner/image-14.png)










We should be able to use this to ssh into the machine.


![alt text](/assets/imgs/htb-runner/image-16.png)


We have to change the perms on the key but we should get in after that


`chmod 600 id_rsa`


and we get in.




![alt text](/assets/imgs/htb-runner/image-17.png)




We also have our first flag:


![alt text](/assets/imgs/htb-runner/image-18.png)






## Enum:




First I tried `ps -ef` with no success.


![alt text](/assets/imgs/htb-runner/image-19.png)




Next I used `netstat -lantp`:


![alt text](/assets/imgs/htb-runner/image-20.png)




After viewing all the listening ports I tried to `curl` each one.


```
curl localhost:8111
Authentication required
To login manually go to "/login.html" page


```


`curl localhost:9000`


Results in a large dump of html lets checkout port 9000.


> You can also figure this out by inspecting the `/etc/hosts` file.


Lets setup some port forwarding and then checkout the page...


`ssh -L 9000:127.0.0.1:9000 john@10.10.11.13 -i id_rsa`


Then navigating to the local `127.0.0.1:9000` we can find the page.


![alt text](/assets/imgs/htb-runner/image-21.png)


## Cracking
Well it appears we need a login so maybe I should have cracked those hashes earlier... anyway lets do it now.


![alt text](/assets/imgs/htb-runner/image-22.png)


lets clean these up and add them to a file


![alt text](/assets/imgs/htb-runner/image-25.png)


I also removed the accounts I made while exploiting:


![alt text](/assets/imgs/htb-runner/image-23.png)


> This should speed things up...


I was originally using hashcat... but my kali vm was too weak.


![alt text](/assets/imgs/htb-runner/image-24.png)




So I used john and added some ram.




![alt text](/assets/imgs/htb-runner/image-26.png)




After entering those creds we are in.


![alt text](/assets/imgs/htb-runner/image-27.png)




## Vuln Searching:


This next part kinda sucked and I was stuck googling portanier vulns... After looking around I googled the docker version "`Docker version 25.0.3, build 4debf41  vulns`"


I found this:


<https://www.docker.com/blog/docker-security-advisory-multiple-vulnerabilities-in-runc-buildkit-and-moby/>




Here we can find a bunch of vulns for docker.


![alt text](/assets/imgs/htb-runner/image-28.png)


> As well while googling I found the vuln listed in several write-ups of this box.


![alt text](/assets/imgs/htb-runner/image34.png)




The RUNC vuln we need to use is detailed here:


<https://securityvulnerability.io/vulnerability/CVE-2024-21626>




<https://nitroc.org/en/posts/cve-2024-21626-illustrated/#exploit-via-setting-working-directory-to-procselffdfd>




In order to exploit this we need to make a container. We are going to use the teamcity image.


![alt text](/assets/imgs/htb-runner/image-30.png)


After that we set the working dir to `/proc/self/fd/8`.


![alt text](/assets/imgs/htb-runner/image-29.png)


Once the container is made we can log into a shell as root and claim the flag after navigating the somewhat weird filesystem.


![alt text](/assets/imgs/htb-runner/image-32.png)




![alt text](/assets/imgs/htb-runner/image-33.png)





## Conclusion:
This box was pretty fun, but the second half was definitely trickier than the first half. I am not well versed on the details of the second vulnerability, but the article linked above gives a really great breakdown of how the vulnerability works.












