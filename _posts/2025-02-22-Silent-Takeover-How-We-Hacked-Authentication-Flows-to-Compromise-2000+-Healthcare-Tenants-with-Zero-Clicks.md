---
title: "Silent Takeover: How We Hacked Authentication Flows to Compromise 2000+ Healthcare Tenants with Zero Clicks"
date: 2023-12-16 
categories: [Bug Bounty, Cybersecurity]
tags: [bugbounty-writeup, Infosec, Authentication, VAPT]
---

## Introduction

During a recent VAPT (Vulnerability Assessment and Penetration Testing) engagement, my friend **Padsala Tushal** and I discovered a critical authentication flaw in a [Redacted] healthcare and biotech inventory management platform. This platform, used by research labs, universities, and biotech companies worldwide, was still in active development but already handling sensitive patient data, lab inventory systems, and clinical trial records.

What started as a simple black-box assessment turned into one of the most significant authentication vulnerabilities we’ve ever encountered — a flaw affecting over 2,000 tenants and enabling account takeover (ATO) without user interaction. In this write-up, we’ll dissect how we exploited a chain of misconfigurations to bypass 2FA, SAML, and LDAP, ultimately logging in as any user on any tenant with zero clicks.

---

## Phase 1: The JavaScript Clue — Decoding Authentication Logic

### Finding the Authentication Map

During static analysis, we uncovered a minified JavaScript file (`auth.prod.js`) loaded on the login page. Buried in the code was a critical function:

```javascript
// auth.prod.js
const authMethods = {
  1: "basic",
  2: "ldap",
  3: "2fa",
  4: "saml"
};
```

This code revealed four authentication methods: **Basic Auth**, **LDAP**, **2FA**, and **SAML**. But how did the platform decide which method to use for a tenant?

### Intercepting the Authentication Decision

By proxying traffic through Burp Suite, we observed a request to:

```
GET /api/auth/config
Response: {"auth_mode": "ldap"}  
```

The `role=2` parameter aligned with the JavaScript’s `2: "ldap"` mapping. Our hypothesis: If we could manipulate this response, we could force the platform to use a weaker authentication method.

---

## Phase 2: Forcing Basic Auth — From Credential Harvesting to Full Tenant Compromise

### Step 1: Hijacking the Authentication Flow

Using Burp Suite’s **Match and Replace** rule, we modified the server’s response from `{"auth_type": "ldap"}` to `{"auth_type": "basic"}`. Instantly, the login page changed:

- **Before**: A corporate LDAP login button.
- **After**: A basic username/password form.

### Step 2: OSINT Goldmine — Docker Hub Leaks

#### How a Forgotten Docker Image Became Our Treasure Trove

While we now had a way to force Basic Auth, we needed valid credentials. Enter **Padsala Tushal’s** Recon expertise.

#### The Docker Hub Discovery

Padsala Tushal uncovered a public Docker Hub profile linked to the platform’s developers. Among the repositories was one named `inventory-management`, hosting three image tags:

- `inventory-management:latest` (latest build)
- `inventory-management:staging`
- `inventory-management:v1.2` (older, deprecated version)

#### Cloning and Decompiling the Image

We pulled the `v1.2` image (older versions often have hardcoded secrets!):

```bash
docker pull developer123/inventory-management:v1.2  
docker save -o inventory-v1.2.tar developer123/inventory-management:v1.2
```

Extracting the TAR archive, we found a Java JAR file: `inventory_dev.jar`.

#### Reverse-Engineering the JAR

Using **Jadx-GUI**, we decompiled the JAR and stumbled upon a critical file:

```yaml
# application.yaml
username: admin
password: Redacted
```

This file contained hardcoded credentials and a password pattern that matched other files in the repository.

### Step 3: Cracking the Credential Pattern

Using the pattern & password:

- **Username**: Redacted (from the Docker image)
- **Password**: Redacted

**Result**: Successful login into multiple tenants. We could now access patient records, lab equipment controls, and clinical trial datasets.

---

## Phase 3: The SAML Nuclear Option — 0-Click Takeover of 2000+ Tenants

### The SAML Deep Dive

After exploiting the Basic Auth bypass, Padsala Tushal and I shifted focus to SAML. The platform’s SSO flow was complex, but we spent some time dissecting it:

#### Normal SAML Workflow:

1. User clicks “Login with SAML”.
2. Redirected to the Identity Provider (IdP), e.g., Azure AD.
3. After successful login, the IdP sends a SAML response to the platform.
4. The platform validates the response and logs the user in.

But during testing, we noticed an oddity. After the SAML response was processed, the platform made a server-side POST request:

```
POST /saml/v1/assertion  
Body: {"user_id": 101}  
Response: {"token": "eyJhbGciOiJSUzI1NiIsIn..."}  
```

### The Lightbulb Moment

The `userId` parameter was a simple integer. We wondered:

- What if we could control this value?
- Does the platform validate if the user actually authenticated via SAML?

#### Testing with Burp Intruder:

1. Intercepted the POST request.
2. Set `userId` to `§101§` (payload position).
3. Launched an attack with payloads from 1 to 1000.

**Result**: Every response contained a valid JWT token for the specified `userId`. We could impersonate any user by incrementing the `userId`!

### Bypassing SAML Entirely: The 0-Click Trick

But there was a catch: to trigger this endpoint, we needed a valid SAML response first. Or did we?

#### The Hidden URL Pattern

While testing random endpoints, we stumbled on a URL that skipped the SAML flow entirely:

```
https://{tenant}.redacted-portal.com/login?id=101
```

Appending `?id=101` to the login page:

1. Skipped the IdP redirect.
2. Directly called `/Saml/v1/assertion` with the provided `id`.
3. Returned a token for `userId=101` without any authentication.

**How?** The platform had no validation to check if the user actually completed SAML authentication. The `id` parameter was trusted blindly.

---

## Phase 4: The Devastating Scale — Mass Compromise of 2000+ Tenants

### Step 1: Mapping the Attack Surface

Padsala Tushal, with his expertise in recon, uncovered **2,000+ tenant subdomains**, including:

- `lab-ny.healthportal.com`
- `cure-cancer.healthportal.com`

40% of these tenants used SAML or LDAP.

### Step 2: Automated Exploitation

We wrote a script to:

1. Force SAML authentication for each tenant.
2. Iterate through `uid` values (1–1000) to hijack accounts.

**Impact**:

- Attackers could log in as any user (admins, researchers, doctors).
- Delete/modify clinical trial data.
- Leak patient health records.
- Sabotage lab equipment configurations.

---

## The Bigger Picture: Lessons Learned

1. **Authentication Type Switching is Dangerous**: Never trust client-side input to dictate auth methods.
2. **SAML Implementations are Fragile**: Always validate assertions and enforce IdP-side session checks.
3. **Default Credentials in Developer Artifacts**: These are a ticking time bomb.
4. **Missing Authorization Checks**: The token endpoint didn’t verify if the user authenticated via SAML.
5. **Incremental User IDs**: User accounts were assigned predictable numeric IDs.

---

## Final Words

We hope you enjoyed this deep dive into one of the most critical authentication flaws we’ve ever uncovered. This discovery was possible only because of the relentless teamwork between me and my brother-in-arms, **Padsala Tushal** — shoutout to his recon expertise for mapping the attack surface at scale!

This is just the beginning. In **Part 2**, we’ll dive deeper into another great writeup on Authentication & Authorization flows.

---

### Follow Us for More

Stay connected for more ethical hacking insights, bug bounty tips, and real-world case studies:

- **Mayur Pandya**: [LinkedIn](https://linkedin.com/in/mayurpandya) | [Twitter/X](https://twitter.com/mayurpandya)
- **Padsala Tushal**: [LinkedIn](https://linkedin.com/in/padsalatushal) | [Twitter/X](https://twitter.com/padsalatushal)

If you found this write-up useful, share it with your network! Together, let’s make the internet safer — one vulnerability at a time. 🛡️

Happy Hacking!  
**Mayur & Tushal**
