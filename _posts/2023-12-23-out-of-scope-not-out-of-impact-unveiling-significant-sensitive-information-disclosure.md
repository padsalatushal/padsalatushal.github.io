---
title: "Out-of-Scope, Not Out-of-Impact: Unveiling Significant Sensitive Information Disclosure"
date: 2023-12-23 
categories: [Bug Bounty, Cybersecurity]
tags: [bugbounty-writeup, Infosec]
---

## Introduction

As a bug bounty hunter, navigating the complexities of subdomain vulnerabilities often leads to intriguing discoveries. One such revelation, though initially deemed out of scope, unearthed a significant vulnerability with unforeseen implications.

## The Discovery

I was invited in private program on yeswehack. The program have small scope not wildcard scope. i struggle to get time to hunt mannually because of collage exam. but still i ran my recontool for getting recon data like subdomains and sensitive information disclosure on wildcard domain. i knew that it is better to look in wildcard subdomains because program scope and assest are small. it is not big funcationlity webapp. Reconscript took bit long time on my vps.

The next day i check recon output and i found wp-config.php.old file exposes critical database credentials and cryptographic keys, which could potentially be exploited to gain unauthorized access to sensitive data or manipulate the target’s WordPress installation.

![image_of_poc](/assets/img/2023-12-23-out-of-scope-not-out-of-impact-unveiling-significant-sensitive-information-disclosure/1.webp)

i knew that it is out of scope finding but still i reported with well written impact. and after few days i got accepted as medium severity and i got half of half of normal reward amount.

![image_of_yeswehackresponce](/assets/img/2023-12-23-out-of-scope-not-out-of-impact-unveiling-significant-sensitive-information-disclosure//2.webp)

## Timeline
- 18/11/2023: Discovery and Reported

- 20/11/2023: Under Review

- 01/12/2023: Accepted and got Reward

Suggestions are most welcome as always. I will try to keep posting my findings. If you got anything from it, you can press the clap icon below, and don’t forget to follow me on [Twitter](https://twitter.com/padsalatushal) & [LinkedIn](https://www.linkedin.com/in/padsalatushal/) as well. See you all next time. :)


