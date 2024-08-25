---
title: "Subdomain Takeover in Azure Trafficmanager for Fun & Profit"
date: 2023-12-16 
categories: [Bug Bounty, Cybersecurity]
tags: [bugbounty-writeup, Infosec]
---

## Introduction

In the dynamic world of cybersecurity, where vulnerabilities lurk in unexpected corners, the concept of subdomain takeovers has become a compelling arena for exploration. This article delves into a real-world scenario involving the Company’s infrastructure, unraveling the intricacies of subdomain takeovers within Azure Traffic Manager.

**The fundamentals seemed clear**: identify a dangling domain and claim it, then showcase a Proof of Concept (PoC). Yet, the execution proved more intricate than anticipated. I invested significant time in understanding the process of uploading a PoC across various Microsoft Azure Trafficmanager services. Given the scarcity of comprehensive resources available online and the inherent confusion in navigating this terrain, I felt compelled to document my journey to aid others facing similar challenges.

## So How did I find subdomain takeovers?

I was doing recon on a private target with a huge scope and assets on Bugcrowd. I fired up my VPS, collected the domains in scope, added them to a file, and started my recon.

First, I did Subdomain Enumeration with `subfinder` and `assetfinder`:

```bash
subfinder -dL domains.txt -all -recursive -o subs.txt
cat domains.txt | assetfinder --subs-only | tee -a subs2.txt
```

Then, I started combining and filtering them. After that, I did an HTTP Probing with `httpx`:

```bash
cat subs.txt subs2.txt | sort -u | tee -a all-subs.txt
cat all-subs.txt | httpx | tee -a live-subs.txt
```

After checking live subdomains manually, I found 3 subdomains that were giving 404. I ran the dig command on them:

  - securemftpptemp.target.com -> azsu-tm-core-ngfw-emftpreprod-002.trafficmanager.net

  - securemfttemp.target.com -> azsu-tm-core-ngfw-emft-002.trafficmanager.net
  
  - ukras1.target.com -> azsu-tm-c-eucprod-infra-pulse-test.trafficmanager.net

I checked [Can-i-take-over-xyz GitHub repository](https://github.com/EdOverflow/can-i-take-over-xyz) for verifying if those subdomains were vulnerable, but it didn’t have documentation for takingover Azure Trafficmanager services. After a quick Google search, I found an insightful article about taking over Azure services: [GODIEGO's Guide to Azure Subdomain Takeovers](https://godiego.co/posts/STO-Azure/).

The domain pointed to a Trafficmanager CNAME that didn’t seem to be registered. To verify, I went to the Azure portal and tried registering it.

![image_of_azure_portal](/assets/img/2023-12-16-subdomain-takeover-azure-trafficmanager/1.webp)

After successfully registering the Azure Trafficmanager profile, I set its outgoing endpoint to my VPS IP, which was running an HTTP server with my PoC code.

After a few minutes, I ran the dig command again to check.

![image_of_dig_command](/assets/img/2023-12-16-subdomain-takeover-azure-trafficmanager/2.webp)

Then I quickly checked the subdomain and it worked.

![image_of_poc](/assets/img/2023-12-16-subdomain-takeover-azure-trafficmanager/3.webp)

I did the same process for the other two subdomains.

I quickly reported to a private program on Bugcrowd.

## Timeline
- 01/12/2023: Discovered and took over the subdomains
- 02/12/2023: Reported to Bugcrowd
- 05/12/2023: Changed the state to **Triaged**
- 08/12/2023: Changed the state to **Resolved**

Suggestions are most welcome as always. I will try to keep posting my findings. If you got anything from it, you can press the clap icon below, and don’t forget to follow me on [Twitter](https://twitter.com/padsalatushal) & [LinkedIn](https://www.linkedin.com/in/padsalatushal/) as well. See you all next time. :)


