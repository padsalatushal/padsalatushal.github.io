---
title: "From Disclosure to High Severity: Leveraging Dyte API Key for Maximum Impact"
date: 2024-01-02
categories: [Bug Bounty, Cybersecurity]
tags: [bugbounty-writeup, Infosec]
---

## Introduction
![image_of_intro](/assets/img/2024-01-02-from-disclosure-to-high-severity-leveraging-dyte-api-key-for-maximum-impact/1.webp)

As a bug bounty hunter engaged in a VDP program, I recently uncovered a JavaScript file that unveiled a trove of sensitive API keys and tokens. Among them was the Dyte API key, an unfamiliar entity in my exploration. This discovery revealed a mix of widely recognized keys like Google Developer, Stripe API, and Firebase credentials — each holding significant sensitivity. Curiously, while scouring through resources like the Keyhacks GitHub repo, I found no information on exploiting the Dyte API key, particularly the DYTE_ORG_ID and DYTE_KEY.

In this article, I aim to delve into the intricacies of exploiting the Dyte API key, exploring its vulnerabilities and potential impact.

So the javascript file look like this:

![image_of_js_file](assets/img/2024-01-02-from-disclosure-to-high-severity-leveraging-dyte-api-key-for-maximum-impact/2.webp)

I did a quick Google search to find out more about what the Dyte API key is all about. Turns out, Dyte offers a solution for real-time video and voice integration into websites, mobile apps, and desktop applications. It’s all about making those video and voice calls.

## Exploitation
I stumbled upon the Dyte API documentation at https://docs.dyte.io/guides/rest-apis/quickstart.

In Dyte’s API documentation, I discovered that their APIs rely on API Keys for request authentication through HTTP Basic Auth. The process involves sending an HTTP request that includes an Authorization header. This header contains the term ‘Basic,’ followed by a space and a base64-encoded string comprising the organizationId and apiKey. This method ensures secure validation of requests made to Dyte’s APIs.

![image_of_api_docs](assets/img/2024-01-02-from-disclosure-to-high-severity-leveraging-dyte-api-key-for-maximum-impact/3.webp)

I promptly encoded the values of the Dyte organization ID and Dyte key using Base64 in the format ‘organizationId:apiKey’, maintaining the specified order.

Now i have Authorization header value now it is time to test . After i craft following curl request with authoriazaiton header to create a simple test meeting.

```bash
curl — request POST \
 — url https://api.dyte.io/v2/meetings \
 — header ‘Authorization: Basic BASE64_VALUE’ \
 — header ‘Content-Type: application/json’ \
 — data ‘{
 “title”: “Sample meeting”,
 “preferred_region”: “ap-south-1”,
 “record_on_start”: false
}’
```

![image_of_poc](assets/img/2024-01-02-from-disclosure-to-high-severity-leveraging-dyte-api-key-for-maximum-impact/4.webp)


# Timeline
- 14/12/2023 : Discover and Reported

- 16/12/2023: Changed the state to Triaged

- 19/12/2023: Changed the state to Resolved

Suggestions are most welcome as always. I will try to keep posting my findings. If you got anything from it, you can press the clap icon below, and don’t forget to follow me on [Twitter](https://twitter.com/padsalatushal) & [LinkedIn](https://www.linkedin.com/in/padsalatushal/) as well. See you all next time. :)


