---
title: "Understanding OWASP API01:2023 - Broken Object Level Authorization (BOLA)"
date: 2025-12-06 21:00:00 +0000
categories: [AppSec, API Security]
tags: [api-security, bola, idor, data privacy, APISecurity, InfoSec, application security, owasp top 10, devsecops]
layout: post
publish: true
comments: true # <-- THIS LINE ENABLES GISCUS COMMENTS
image:
  path: /assets/img/bola1.png
---
## Introduction
Broken Object Level Authorization, commonly known as BOLA, sits at the top of the OWASP API Security Top 10 list as API01:2019 and API01:2023. It is the most common vulnerability found in APIs today, affecting applications across every industry from healthcare to finance to social media.

At its core, BOLA is a simple concept with devastating consequences. It happens when an API fails to properly validate whether a user should have access to a specific resource or object. The API checks if the user is logged in, but it doesn't verify if they should be accessing that particular piece of data. It's like a security guard who checks that visitors have a badge to enter the building but never verifies which floors or rooms they're actually allowed to access.


## Understanding BOLA Technical Details

To really understand BOLA, we need to distinguish between authentication and authorization. Authentication answers the question "who are you?" while authorization answers "what are you allowed to do?" Many APIs handle authentication well but fail catastrophically at authorization, specifically at the object level.

Modern APIs use some form of unique identifier to reference specific objects. These might be database IDs, user account numbers, order numbers or record identifiers. A typical API endpoint might look like this:

```
GET /api/users/1234
GET /api/orders/5678
GET /api/documents/9012
```

The numbers in these URLs are direct object references. They point to specific records in a database. The vulnerability occurs when the API receives a request like "GET /api/orders/5678" and only checks whether the requesting user is authenticated, without verifying whether that user actually owns order 5678 or has permission to view it.

An attacker can exploit this by simply changing the object ID in their requests. If they can access order 5678, what happens when they try 5679, 5680 or 5681? In a vulnerable system, they can enumerate through all orders in the database, viewing details they have no business accessing.

What makes BOLA dangerous is its simplicity, hackers don't need sophisticated hacking tools or deep technical knowledge. A basic understanding of how APIs work and a browser's developer tools are often enough to discover and exploit these vulnerabilities.

## How BOLA Happens in Development

Understanding why BOLA vulnerabilities make it into production requires looking at the development process. Several factors contribute to these issues:

**Assumption of Trust**, developers sometimes assume that because a user is authenticated and the request is coming from their official app or website, the data in that request can be trusted. This leads to code that doesn't validate object ownership.

**Framework Misuse**, many web frameworks make authentication easy but leave authorization up to the developer. It's simple to add a middleware that checks if someone is logged in, but implementing granular object-level checks requires additional code that's often overlooked.

**Time Pressure**, developers might implement the "happy path" where users only access their own data through the UI, without considering what happens if someone manipulates the API requests directly.

**Incomplete Testing**, security testing often focuses on authentication bypass and SQL injection, with less attention paid to authorization logic. Standard test cases might verify that authenticated users can access the endpoint without checking whether they can access other users' data.

### BOLA Real-World Attack: The Optus Data Breach

In September 2022, Optus, Australia's second-largest telecommunications provider, suffered one of Australia’s worst data breaches, potentially affecting up to 9.8 million current and former customers that's nearly 40% of the population. Reported personal data included names, dates of birth, contact information, and for a subset of individuals, identification document numbers (passport or driver’s licence).

Investigations and regulatory filings indicate the breach stemmed from a publicly exposed API, a legacy “test” endpoint on a sub-domain that retained internet-facing status. A coding error introduced in 2018 disabled crucial access controls, and though the main domain was patched in 2021, the vulnerable sub-domain remained live.

According to the regulatory filing by ACMA, the attacker used a low-sophistication method: by supplying a user identifier (likely predictable/sequential), they retrieved user records one after another. This behavior mirrors a Broken Object Level Authorization (BOLA) or IDOR vulnerability. Some post-mortem commentary also suggests a lack of rate limiting and use of sequential identifiers (rather than unguessable tokens), which facilitated large-scale data extraction.

The breach underscores that even large organisations can fall victim to very basic API-security failures: mis-configured access controls, exposure of dormant or test endpoints, predictable identifiers and lack of proper lifecycle management.

## Prevention Strategies for BOLA

Preventing BOLA requires a multi-layered approach that combines secure coding practices, architectural decisions and ongoing security testing.

**Implement Authorization Checks Everywhere**: every single API endpoint that accesses a specific object must verify that the requesting user has permission to access that object. This check should happen on the server-side, developers should never rely on client-side validation. The pattern should be: authenticate the user, identify the requested object, verify the user's permission for that specific object, then return or deny the data.

**Use Indirect Object References**: instead of exposing database IDs or sequential numbers in APIs, developers should use random UUIDs or tokens that don't reveal anything about the data structure.

**Implement Role-Based Access Control (RBAC)**: before the development phase, define clear roles and permissions in system. A user might be an owner, viewer or editor of a resource. API should check both that the user is authenticated and that their role permits the requested action on that specific object.

**Apply the Principle of Least Privilege**: users should only be able to access the minimum data necessary for their legitimate use. If a customer service representative only needs to see basic account information, the API shouldn't even allow requests for sensitive financial details, regardless of authorization.

**Log and Monitor Access Patterns**: system should implement comprehensive logging of who accesses what and when. Set up monitoring to detect unusual patterns like a user suddenly accessing far more objects than normal, sequential ID enumeration, or attempts to access objects outside their normal scope.

**Conduct Regular Security Testing**: include BOLA testing in security assessment process. This should involve both automated scanning with tools designed to detect authorization issues and manual penetration testing where security professionals attempt to access resources they shouldn't.

**Implement Rate Limiting**: even with proper authorization checks, rate limiting provides defense in depth. If an attacker does find a vulnerability, rate limiting can slow down or prevent mass data extraction.

**Security Training for Developers**: ensure development team understands the difference between authentication and authorization, and why object-level checks are critical. Real-world examples make excellent training material.

## Conclusion

Broken Object Level Authorization remains the number one API security risk because it's both common and impactful. The Optus breach demonstrated that even major organisations with significant resources can fall victim to this fundamental vulnerability. The exploitation is straightforward, the impact is severe and the data exposed can affect millions of people.

By implementing thorough authorization checks, using indirect object references, following the principle of least privilege and conducting regular security testing, organisations can protect their APIs and their users' data from this threat.

As APIs continue to power more of our digital infrastructure, securing them against BOLA and other vulnerabilities isn't just a technical requirement, it's a fundamental responsibility to the users who trust us with their data.


## References:
- https://www.theguardian.com/business/2022/sep/29/optus-data-breach-everything-we-know-so-far-about-what-happened
- https://en.wikipedia.org/wiki/2022_Optus_data_breach
- https://www.theregister.com/2024/06/21/optus_data_breach_faulty_api
- https://www.comcourts.gov.au/file/Federal/P/VID429/2024/3981938/event/31836639/document/2300547
- https://securityscorecard.com/blog/5-lessons-from-the-optus-data-breach-for-telecom-and-third-party-risk/ 
- OWASP API Security Top 10 2023: https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
