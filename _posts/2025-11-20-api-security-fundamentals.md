---
title: "API Security, an Essential Knowledge for Modern Development"
date: 2025-11-20 10:00:00 +0000
categories: [AppSec, API Security]
tags: [api, api-security, application security, owasp top 10]
layout: post
publish: true
comments: true # <-- THIS LINE ENABLES GISCUS COMMENTS
---

Recently, I completed the API Security Fundamentals course on APISec University and the experience has been eye-opening. The course covered everything from basic concepts to attack vectors, and I wanted to share some of the key insights that every developer and security professional should understand.

Application Programming Interface (APIs) are modern applications backbone, they connect mobile apps to servers and allow third-party services integration. However, the connectivity comes with a significant challenge that makes APIs prime targets for attackers.

Unlike traditional web applications, APIs are designed to be discoverable and accessible which makes them easy targets. API attack pattern differs from conventional security threats: rather than following a kill chain, API attacks often find a vulnerability, exploit it and cause a breach. This makes API security testing critical.

## Common API Vulnerabilities

The OWASP API Security Top 10 [1] provides a framework for understanding the most critical risks:

**Broken Object Level Authorization (BOLA)** remains one of the most dangerous flaws. This occurs when an application fails to properly verify that a user should have access to specific data. Imagine a banking app where User 1 can change account number in a request and view User 2's transaction history. The consequences can range from privacy violations to fraudulent transactions.

**Broken Object Property Level Authorization (BOPLA)** involves exposing more data than necessary or allowing unauthorized modification of object properties. Applications often return entire data objects when only a few fields are needed. This excessive data exposure gives attackers information they can use for further attacks or direct data theft.

**Unrestricted Resource Consumption** exploitation affects Availability which is one of the important elements of the CIA Triad. When APIs don't implement proper resource constraints, attackers can flood endpoints with requests, harvest massive amounts of data or cause denial of service that takes application offline.

**Unrestricted Access to Sensitive Business Flows** occurs when attackers manipulate the intended workflow of an application. For example, an e-commerce platform where someone could bypass payment steps or a ticket booking system where bots purchase all available seats instantly.

**Server-Side Request Forgery (SSRF)** allows attackers to manipulate API server URLs to make malicious requests, potentially accessing internal systems that should never be exposed to the outside world.

## Building Security Strategies

Addressing API security requires more than fixing individual vulnerabilities, three pillars form the foundation of effective API security:

![Pillars of API Security](/assets/img/apisec/pillar.png)

**Governance** ensures that security is built into APIs from the start. This means establishing clear processes for API development, enforcing security standards and maintaining consistency across. Without governance, each development team might implement security differently, creating gaps.

**Testing** must be thorough and continuous. Security testing should examine not just whether authentication works, but whether business logic can be abused, whether data access controls are properly enforced and whether edge cases have been considered. Testing for flaws is important.

**Monitoring** provides the eyes and ears needed in production environment. Runtime protection involves filtering malicious traffic and validating that security controls work as intended. Effective monitoring detects volumetric attacks, identifies unusual patterns and provides the data needed for incident response.

## Threat Modeling, The Security Blueprint

![Pillars of API Security](/assets/img/apisec/threat-model.png)

Before writing a single line of code, threat modeling helps understand what you're protecting. Start by identifying APIs, the business flows they enable, the data they handle and all possible access paths. Then assess where vulnerabilities might exist, examine how likely different attacks are and understand what the impact would be if those attacks succeeded. This structured approach allows developing targeted mitigation strategies rather than applying generic security measures that might miss critical risks.

## Practical Prevention Measures

Prevention starts with good practices.     
- Implement least privilege principle by granting users and services only the access they need.
- Validate and sanitise all inputs, don't trust data just because it comes from an authenticated user.
- Avoid mass assignment, use data minimisation by returning only the fields required for each use case.
- Review authorisation rules carefully and test thoroughly before deployment.

## Why API Security Matters

These breaches demonstrate that API security failures lead to some of the most significant data exposures in recent years. The cost isn't just financial—it includes reputation damage, regulatory penalties, and loss of customer trust. As APIs continue to proliferate, security can't be an afterthought.

The fundamentals covered here provide a starting point, but API security is a continuous journey. Stay informed about emerging threats, regularly audit your APIs, and foster a security-conscious culture within your development teams. Your APIs are the doors to your data—make sure only the right people can open them.

## References
[1] OWASP API Security Top 10. Available at: https://owasp.org/API-Security/editions/2023/en/0x11-t10/