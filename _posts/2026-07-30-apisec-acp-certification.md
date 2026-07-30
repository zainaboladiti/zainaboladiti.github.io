---
layout: post
title: "How I Prepared for the APIsec Certified Practitioner (ACP) Exam"
date: 2026-07-30 10:00:00 +0100
categories: [AppSec, API Security, Certification]
tags: [api-security, cybersecurity, certification, appsec, penetration-testing, APISecurity, InfoSec, application security, owasp top 10, API, security]
layout: post
publish: true
comments: true # <-- THIS LINE ENABLES GISCUS COMMENTS
image:
  path: /assets/img/acp.png   
---

Late last year, I joined CyberSafe Foundation for a 16 week API Security training program. Out of 250 beneficiaries in the cohort, I graduated among the top 50, and that earned me a voucher to sit the ([APIsec Certified Practitioner (ACP) exam](https://au.apisec.ai/courses/apisec-certified-practitioner)). I recently sat the exam and passed, and I wanted to write down for anyone else considering this path.

The APIsec Certified Practitioner certification is awarded to professionals in API security, application security, application security testing, and penetration testing. APIs sit behind most of what we use day to day, and they often carry application logic and sensitive data, including PII, which makes them a real target for attackers. Getting the security right matters if we want to keep building and shipping at the pace we do.

## How I Studied

The course is broken into modules (API Security Fundamentals, the OWASP API Security Top 10, API Authentication and Authorization, API Documentation Best Practices, and Securing API Servers), and I did not try to absorb everything at once.

A few things I did:

**Condensed notes and flashcards per module** instead of rereading the same material over and over, I rewrote each module into shorter notes and flashcards, organised around the key concepts and the real world examples that came with them. Rewriting something in your own words is a good way to find out what you actually understood versus what you only recognised.

**Going back through the official quizzes** I used the end of module quizzes as a way to test myself.

**Anchoring concepts to real breaches** vulnerability categories like Broken Object Level Authorization or Server Side Request Forgery are much easier to remember once you have seen a real case attached to them, rather than just a definition on its own.

## A Few Things That Stuck With Me

- Broken Object Level Authorization has been the top API vulnerability on the OWASP list for years, and it is often surprisingly simple to exploit, sometimes it just takes changing an ID in a request from one number to another.
- Very few organizations test their APIs for security specifically. Most testing effort still goes into confirming that an API works, not whether it can be misused.
- Good documentation is a security practice, not just a developer convenience. Writing clear documentation early tends to surface missing or inconsistent authentication before an API ever reaches production.

## If You Are Considering This Path

Understand the reasoning behind an answer rather than memorizing it. Use the practice material to find your weak spots, not just to confirm your strong ones. Break the syllabus into manageable pieces instead of trying to cover it all in one sitting. And do not skip the foundational modules to rush toward the more advanced topics. The fundamentals are what everything else in API security is actually built on.
