---
title: Splunk Challenge
published: true
categories: Sans-Challenge
---

**1- What is the short host name of Professor Banas' computer?**

We can know the host name of Professor Banas’s computer from the Soc team Chat. 

Answer: sweetums.

**2- What is the name of the sensitive file that was likely accessed and copied by the attacker? Please provide the fully qualified location of the file. (Example: C:\temp\report.pdf)**

In this challenge I used Bana’s hostname in splunk search to identify sensitive file that was likely accessed and copied by the attacker. And from the hint from SOC team. He said that he very close from (Santa). So in search we used his name and santa and I found the file.

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/Splunk_1.PNG)

Answer: C:\Users\cbanas\Documents\Naughty_and_Nice_2019_draft.txt

**3- What is the fully-qualified domain name(FQDN) of the command and control(C2) server? (Example: badguy.baddies.com)**

as Alice says (on of the SOC team) using Microsoft sysmon as source type in splunk search to find C2 server by.

> index=main sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/Splunk_2.PNG)

Answer: 144.202.46.214.vultr.com

**4- What document is involved with launching the malicious PowerShell code? Please provide just the filename. (Example: results.txt) **

this very intersted part and here is the steps to solve it:
1. using powershell souce type and reverse the result from the oldest Event
> sourcetype="WinEventLog:Microsoft-Windows-Powershell/Operational" | reverse
1. find two intersted process id and conver it them to hex
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/Splunk_3_1.PNG)
1. using sourcetype=WinEventLog EventCode=4688 to uncover what launched those processes and then using process id that had been converted to hex.
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/Splunk_3_2.PNG)

Answer: 19th Century Holiday Cheer Assignment.docm

**5- How many unique email addresses were used to send Holiday Cheer essays to Professor Banas? Please provide the numeric value. (Example: 1)**

for this challenge splunk a plugin called StoQ. its an automation framework that we use to analyze all email messages
so by collect all the eamil it will be 21
Answer: 21

** 6- What was the password for the zip archive that contained the suspicious file?**

while to looking to unique email which is spam email bradly.buttercups@eifu.org that used to hack carl.banas@faculty.elfu.org.
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/Splunk_4.PNG)

Answer: 123456789

** 7- What email address did the suspicious file come from?**

as we notice that the suspicious email came from bradly.buttercups@eifu.org

Answer: bradly.buttercups@eifu.org

**Challenge Question**
**What was the message for Kent that the adversary embedded in this attack?**

here is the steps to find the message for kent:
1. we should look on all file path that come from the attacker which is bradly.buttercups@eifu.org
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/Splunk_5.PNG)
1. get the core.xml file path
> /home/ubuntu/archive/f/f/1/e/a/ff1ea6f13be3faabd0da728f514deb7fe3577cc4/core.xml
1. go to StoQ Archive which stores the raw artifacts in their entirety in the archive and get the file and open it in chrome borwser.
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/Splunk_6.PNG)

and this was the last amazing Challenge
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/Splunk_7.PNG)

