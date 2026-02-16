---
title: "Phishing Analysis Tools" 
date: 2026-2-17 00:01:00 0000+
tags: [WriteUp, Phishing, CyberChef, VirusTotal, Message Header Analyzer,  Message Transfer Agent, URLScan.io, Talos Reputation Center, URL Extractor, Any.Run, TryHackMe ]
categories: [WriteUps, TryHackMe]
image:          
  path: /assets/Imgs/Phishing-Tools/tryhackme.png
---
---

In this room, we will look at various tools that will aid us in analyzing phishing emails.

Room Link : https://tryhackme.com/room/phishingemails3tryoe

Disclaimer: Task 1-6 Skipped as it provides general overview of tools( Do complete before proceeding to task 7) 

---

## Task 7: Phishing Case 1

Task: Use the tools discussed throughout this room (or use your own resources) to help you analyze each email header and email body.

We can use 

1. What brand was this email tailored to impersonate?

![image.png](/assets/Imgs/Phishing-Tools/image.png)

> Answer: NETFLIX
> 

1. What is the From email address?

![image.png](/assets/imgs/Phishing-Tools/image%201.png)

> Answer: [JGQ47wazXe1xYVBrkeDg-JOg7ODDQwWdR@JOg7ODDQwWdR-yVkCaBkTNp.gogolecloud.com](mailto:JGQ47wazXe1xYVBrkeDg-JOg7ODDQwWdR@JOg7ODDQwWdR-yVkCaBkTNp.gogolecloud.com)
> 

1. What is the originating IP? Defang the IP address. 

Use more option on the right side and go on view source. In the source you can see X-Originating - IP : [209.85.167.226].

![image.png](/assets/imgs/Phishing-Tools/image%202.png)

 Now lets Defang the IP .

We can use CyberChef for defang :

![image.png](/assets/imgs/Phishing-Tools/image%203.png)

> Answer: 209[.]85[.]167[.]226
> 

1. From what you can gather, what do you think will be a domain of interest? Defang the domain.

We cab find the information in the Return-path of the mail :

![image.png](/assets/imgs/Phishing-Tools/image%204.png)

We will defang it with CyberChef

> Answer: etekno[.]xyz
> 

1. What is the shortened URL? Defang the URL.

We can easily spot the URL on the red button. Right click on it and  select “Copy Link Location”.

Defang it with CyberChef

> Answer: hxxps[://]t[.]co/yuxfZm8KPg?amp=1_.
> 

---

## Task 8: Phishing Case 2

1. What does AnyRun classify this email as?

![image.png](/assets/imgs/Phishing-Tools/image%205.png)

> Answer: Suspicious Activity.
> 

1.  What is the name of the PDF file?

> 
> 

![image.png](/assets/imgs/Phishing-Tools/image%206.png)

1. What is the SHA 256 hash for the PDF file?

The hash is not directly mentioned so either by clicking on **Text report** or directly on the pdf file(Payment-update.pdf) we can get the value of SHA256

![image.png](/assets/imgs/Phishing-Tools/image%207.png)

![image.png](/assets/imgs/Phishing-Tools/image%208.png)

> Answer: cc6f1a04b10bcb168aeec8d870b97bd7c20fc161e8310b5bce1af8ed420e2c24.
> 

1. What two IP addresses are classified as malicious? Defang the IP addresses. (answer: **IP_ADDR,IP_ADDR**)

In the same **Text report,**  go to “Connections” section. We can see two IP marked as malicious.

![image.png](/assets/imgs/Phishing-Tools/image%209.png)

Defang them using CyberChef

> Answer: 2[.]16[.]107[.]24,2[.]16[.]107[.]83
> 

1. What Windows process was flagged as **Potentially Bad Traffic**?

Now scroll to the “Threats” section. 

![image.png](/assets/imgs/Phishing-Tools/image%2010.png)

> Answer: svchost.exe
> 

---

## Task 9: Phishing Case 3

1. What is this analysis classified as?

![image.png](/assets/imgs/Phishing-Tools/image%2011.png)

> Answer: Malicious activity
> 

1. What is the name of the Excel file?

![image.png](/assets/imgs/Phishing-Tools/image%2012.png)

> Answer: CBJ200620039539.xlsx
> 

1. What is the SHA 256 hash for the file?

![image.png](/assets/imgs/Phishing-Tools/image%2013.png)

> Answer: 5f94a66e0ce78d17afc2dd27fc17b44b3ffc13ac5f42d3ad6a5dcfb36715f3eb
> 

1. What domains are listed as malicious? Defang the URLs & submit answers in alphabetical order. (answer: **URL1,URL2,URL3**)

Look under Network activity section in DNS requests

![image.png](/assets/imgs/Phishing-Tools/image%2014.png)

Defang them using CyberChef

> Answer: biz9holdings[.]com,findresults[.]site,ww38[.]findresults[.]site
> 

1. What IP addresses are listed as malicious? Defang the IP addresses & submit answers from lowest to highest. (answer: **IP1,IP2,IP3**)

Look at HTTP connections just above DNS requests 

![image.png](/assets/imgs/Phishing-Tools/image%2015.png)

Defang it using CyberChef.

> Answer: 75[.]2[.]11[.]242,103[.]224[.]182[.]251,204[.]11[.]56[.]48
> 

1. What vulnerability does this malicious attachment attempt to exploit?

The vulnerability is mentioned in the same report, scroll to the top again 

![image.png](/assets/imgs/Phishing-Tools/image%2016.png)

> Answer: cve-2017-11882
> 

---

## Task 10: Conclusion

1. Read the above

> No answer needed
>