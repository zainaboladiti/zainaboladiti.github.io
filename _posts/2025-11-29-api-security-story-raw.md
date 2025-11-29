---
title: "API Security Story, Massive Data Exposure via Dating App 'Raw'"
date: 2025-11-20 10:00:00 +0000
categories: [AppSec, API Security]
tags: [api-security, bola, idor, dataprivacy, APISecurityStory, InfoSec, application security, owasp top 10]

layout: post
publish: true
comments: true
---

Earlier this year, researchers discovered a serious vulnerability in the API of the dating app Raw. The flaw allowed anyone to access the personal data (names, dates of birth, preferences) and even precise location data of Raw users.

## What Happened

**The API endpoint for fetching user profiles required only a user identifier, no proper authentication or authorisation check.** Once logged in, an attacker could alter the identifier in the request URL and retrieve someone else's profile.

**Among the exposed data was sensitive location information with street-level accuracy.** That meant a malicious actor could see exactly where a user was situated.

This flaw demonstrates how a single, simple API misconfiguration that failed to enforce object-level authorisation can completely undermine user privacy. For a dating app, where users often expect anonymity or at least privacy, leaking identity and precise location is especially dangerous.

It also underlines how easy some of these flaws are to exploit: the attacker did not need advanced tools, just basic HTTP requests and the ability to guess or enumerate user IDs. This is why Broken Object Level Authorisation (BOLA / IDOR) remains one of the most widespread and impactful vulnerabilities in APIs.

## The Broader Context

2025 continues to show that API vulnerabilities are not fringe problems. According to the most recent industry data:

- **APIs have become the primary target for cyber-attacks.** In the first half of 2025, there were over 40,000 API incidents across 4,000 monitored environments.
- **According to the 2025 Global State of API Security report by Traceable, 57% of organisations suffered at least one API-related breach in the past two years.** Traditional defence tools like firewalls, standard gateways often fail to catch modern API threats.

## Lessons for Security Professionals

**Authentication is not enough.** APIs must enforce authorisation at object level. If a request can access data just by changing a resource ID, that's almost certainly a BOLA / IDOR vulnerability.

**Assume APIs are profiled and probed.** Attackers will try to enumerate user IDs, fuzz endpoints, or brute-force predictable IDs.

**Prioritise API security in design and testing.** That means building secrets out, applying strict access checks and running security audits (e.g. object-level access tests) especially for endpoints that expose personal or sensitive data.

## Sources

1. [TechCrunch: Dating app Raw exposed users' location data, personal information](https://techcrunch.com/2025/05/02/dating-app-raw-exposed-users-location-data-personal-information/)
2. [API Security Newsletter: Issue 271 - API Breaches Surge in APAC](https://apisecurity.io/issue-271-api-breaches-surge-in-apac-raw-dating-app-exposes-users-api-credential-missteps-api-sprawl/)
3. [Global Dating Insights: Raw discovers, patches unexpected user data leak](https://www.globaldatinginsights.com/featured/raw-discovers-patches-unexpected-user-data-leak/)
4. [Cyber Material: Raw dating app exposes user data through bug](https://cybermaterial.com/raw-dating-app-exposes-user-data-through-bug/)
5. [Wallarm: API Attack Awareness - Broken Object Level Authorization (BOLA)](https://lab.wallarm.com/api-attack-awareness-broken-object-level-authorization-bola-why-it-tops-the-owasp-api-top-10/)
6. [Imperva: APIs Become Primary Target for Cybercriminals](https://www.imperva.com/company/press_releases/apis-become-primary-target-for-cybercriminals-over-40000-api-incidents-in-first-half-of-2025/)
7. [Traceable: 2025 State of API Security Report](https://www.businesswire.com/news/home/20241030645718/en/Traceable-Releases-2025-State-of-API-Security-Report-API-Breaches-Persist-as-Fraud-Bot-Attacks-and-Generative-AI-Increase-Risks)
