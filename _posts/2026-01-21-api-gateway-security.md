---
title: "Why API Gateways Are Critical for Securing Public APIs"
date: 2026-01-21 17:39:00
categories: [AppSec, API Security, Cloud Security, DevSecOps]
tags: [api-security, data privacy, APISecurity, InfoSec, application security, owasp top 10, devsecops, API, api-gateway, aws, security, rate-limiting]
layout: post
publish: true
comments: true # <-- THIS LINE ENABLES GISCUS COMMENTS
image:
  path: /assets/img/api_gateway_security.png
---


In modern software architecture, APIs serve as the connective tissue between different systems, enabling applications to communicate and share data. However, as APIs become more prevalent, they also become attractive targets for attackers. This is where API gateways become essential components of any security strategy.

## The Risks of Exposing APIs Without Gateways

When APIs are exposed directly to the internet without an API gateway, organisations face several critical security challenges. Direct exposure means that backend services receive requests without any intermediary layer to validate, filter, or monitor traffic. This creates multiple vulnerabilities that attackers can exploit.

First, without a gateway, there is no centralised point for enforcing authentication and authorisation. Each service must implement its own security controls, leading to inconsistent protection across the API landscape. This increases the likelihood of configuration errors and security gaps. A single misconfigured endpoint can become an entry point for unauthorised access to sensitive data.

Capital One breach in 2019 exemplifies the consequences of inadequate security; a former AWS employee exploited a misconfigured web application firewall to access sensitive data for over 100 million customers through vulnerable APIs. The breach resulted in a $190 million settlement and significant reputational damage[^1]. The attacker gained unauthorised access by exploiting a server-side request forgery (SSRF) vulnerability, demonstrating how improperly secured APIs without proper gateway controls can become critical vulnerabilities.

Second, direct API exposure makes systems vulnerable to denial-of-service attacks. Without rate limiting or throttling mechanisms, a malicious actor can overwhelm backend services with excessive requests, causing system outages and degrading service for legitimate users. The cost implications can be significant, as cloud resources scale to handle the attack traffic, leading to unexpected infrastructure expenses.

Third, visibility becomes a major problem. Without a gateway, tracking who accesses what resources, when, and how becomes difficult. This lack of visibility makes it nearly impossible to detect anomalous behaviour, conduct security audits, or investigate incidents after they occur. Organisations essentially operate blind to their API security posture.

## How API Gateways Protect Backend Services

API gateways act as reverse proxies that sit between clients and backend services, providing a single entry point for all API traffic. This architecture delivers multiple layers of protection that work together to create a robust security posture.

### Authentication and Authorisation

Authentication and authorisation form the first line of defence. Modern API gateways support multiple authentication mechanisms, including API keys, OAuth 2.0, JWT tokens, and integration with identity providers like Amazon Cognito or third-party systems. For example, AWS API Gateway can validate tokens before requests reach backend services, ensuring that only authenticated users with proper permissions can access protected resources. This centralised authentication simplifies security management and ensures consistent policy enforcement.

### Request Validation

Request validation provides another critical security layer. API gateways can inspect incoming requests to ensure they conform to expected patterns and schemas. This helps prevent injection attacks, malformed requests, and attempts to exploit vulnerabilities in backend systems. By validating requests at the gateway level, organisations can reject malicious traffic before it consumes backend resources.

### Transport Layer Security (TLS) Enforcement

TLS enforcement ensures that all communication between clients and the gateway remains encrypted. API Gateway supports enhanced TLS security policies that enforce modern encryption standards, including TLS 1.3-only configurations and cipher suites that meet regulatory requirements like PCI DSS and FIPS compliance. These policies protect sensitive data from interception during transmission.

### Web Application Firewall (WAF) Integration

Integration with WAF adds protection against common web attacks. When an API gateway connects with WAF, it can filter requests based on IP addresses, geographic regions, request patterns, and known attack signatures. This helps block SQL injection attempts, cross-site scripting attacks, and bot traffic before these threats reach application logic.

## Rate Limiting as a Security Control

Whilst many developers think of rate limiting primarily as a performance optimisation technique, it serves as a fundamental security control that protects APIs from abuse and attack. Understanding this dual nature is essential for proper API security architecture.

### How Rate Limiting Works

Rate limiting implements the token bucket algorithm, which controls both the steady-state request rate and the ability to handle traffic bursts. In this model, each request consumes a token from a bucket, the bucket has a maximum capacity (the burst limit) and tokens are added back at a fixed rate (the rate limit). When the bucket is empty, requests are throttled and receive a 429 Too Many Requests error response.

### Protection Against Attack Vectors

From a security perspective, rate limiting protects against several attack vectors:

**Brute Force Attacks** where attackers attempt to guess credentials or API keys through repeated requests. Credential stuffing attacks, which use leaked username and password combinations across multiple services, are alsomitigated because rate limits prevent attackers from testing large volumes of credentials quickly. Organisations that implement proper rate limiting can significantly reduce the success rate of automated authentication attacks.

**Denial-of-Service Protection** by limiting the number of requests any single client can make. Rate limiting prevents individual actors from monopolising system resources and this ensures that legitimate users maintain access even when the system is under attack.

### Cost Control as a Security Dimension

Cost control represents an often-overlooked security dimension of rate limiting. In cloud environments where infrastructure scales automatically, an attack that floods the API with requests can trigger massive scaling that results in unexpected costs. Without proper rate limiting, organisations can face substantial bills after attackers exploit their unprotected APIs, generating millions of requests that trigger auto-scaling.

Rate limiting acts as a financial safeguard by capping the maximum number of requests that can be processed, thereby limiting the potential cost of an attack. This is why rate limiting should be viewed not just as a performance feature, but as a critical security control that protects both system availability and organisational resources.

## Real-World Attack Prevention

Consider an attack scenario where threat actors discover an unprotected API endpoint that exposes user data. Without an API gateway, they could query this endpoint repeatedly to enumerate all users in the system. However, with proper gateway security controls in place, multiple defensive layers activate.

First, authentication requirements block unauthorised access entirely. If the attacker somehow obtains valid credentials, rate limiting restricts how much data can be extracted before the system flags and blocks the suspicious activity. Request validation might detect and reject malformed queries designed to bypass access controls. Logging and monitoring capture the attack attempts, enabling security teams to respond and investigate.

The 2021 Peloton API vulnerability demonstrated the importance of proper API security controls. Security researcher Jan Masters from Pen Test Partners discovered that Peloton's APIs exposed user data without adequate authentication and authorisation, allowing anyone to access private user information including age, gender, location, and workout statistics by making unauthenticated requests[^2] [^3]. An API gateway with proper authentication and rate limiting would have prevented this unauthorised data access.

This layered defence demonstrates why API gateways have become essential infrastructure for any organisation exposing APIs to external consumers. They transform API security from a patchwork of individual service protections into a cohesive, centrally managed security architecture that can adapt to emerging threats whilst maintaining performance and availability.

## Conclusion

Organisations that treat API gateways as foundational security components rather than optional features position themselves to build secure, scalable, and resilient API ecosystems that can support business growth whilst protecting sensitive data and maintaining user trust.

As APIs continue to proliferate across every industry, the security controls provided by API gateways become increasingly critical. Whether protecting against brute force attacks, preventing denial-of-service incidents, or ensuring compliance with regulatory requirements, API gateways provide the centralised security enforcement point that modern distributed systems require.

## References

[^1]: [Capital One Data Breach Settlement - Official Settlement Website](https://www.capitalonesettlement.com/)

[^2]: [Peloton API Vulnerability - TechCrunch](https://techcrunch.com/2021/05/05/peloton-bug-account-data-leak/)

[^3]: [Peloton API Security Incident - Pen Test Partners](https://www.pentestpartners.com/security-blog/tour-de-peloton-exposed-user-data/)
