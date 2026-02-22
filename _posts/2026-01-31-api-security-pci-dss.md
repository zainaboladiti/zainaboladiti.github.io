---
title: "What API Developers Need to Know About PCI DSS and Financial API Security"
date: 2026-01-31 17:39:00
categories: [AppSec, API Security, Compliance]
tags: [api-security, data privacy, APISecurity, InfoSec, application security, owasp top 10, API, security, pci-dss, financial-security, compliance, payment-apis]
layout: post
publish: true
comments: true # <-- THIS LINE ENABLES GISCUS COMMENTS
image:
  path: /assets/img/pci_dss_api_security.png
---


Payment Card Industry Data Security Standard (PCI DSS) compliance is not optional for organisations that handle credit card information. The latest version, PCI DSS 4.0.1, brings APIs into focus with explicit security requirements that recognise the central role APIs play in modern payment processing systems. For developers building financial APIs, understanding these requirements is essential not just for compliance, but for building secure systems that protect customer data and maintain trust.

## Why PCI DSS Matters for API Development

PCI DSS establishes a baseline of technical and operational requirements designed to protect cardholder data in any environment where it is stored, processed, or transmitted. The standard applies to all organisations that accept payment cards, regardless of size or transaction volume. This includes merchants, payment processors, service providers, and any third-party vendors that could impact the security of cardholder data.

### The Evolution of PCI DSS for Modern APIs

For API developers, PCI DSS 4.0.1 represents a fundamental shift in how compliance frameworks address modern application architecture. Previous versions focused primarily on traditional web applications, but the current standard explicitly acknowledges APIs as critical components of the payment ecosystem. This recognition stems from the reality that APIs have become the primary means of connecting payment systems, integrating third-party services, and enabling real-time transaction processing across distributed systems.

The full PCI DSS 4.0.1 deadline came into effect on 31 March 2025, making compliance mandatory for all organisations handling payment card data. This means API developers working in financial services, retail, hospitality, travel, and any other industry that processes card payments must ensure their APIs meet these requirements.

### Consequences of Non-Compliance

The consequences of non-compliance extend beyond regulatory penalties. Data breaches resulting from PCI violations can lead to massive fines, legal action, loss of payment processing privileges, and severe reputational damage.

Target breach in 2013 which exposed payment card data for 41 million customers and personal information for an additional 60 million, resulted from attackers who gained access through a third-party HVAC vendor's compromised credentials[^1]. The attackers used these credentials to access Target's network through a vendor portal, then moved laterally to compromise point-of-sale systems. The breach resulted in an $18.5 million settlement and approximately $202 million in total costs[^2]. Whilst the initial access wasn't through an API, the incident demonstrated how inadequate access controls and network segmentation can allow attackers to move from vendor systems to production environments.

More recently, in 2018, British Airways suffered a data breach when attackers injected malicious JavaScript into the airline's payment pages, harvesting personal data for over 400,000 customers
including payment card details for over 200,000 individuals[^3]. The UK Information Commissioner's Office (ICO) initially proposed a fine of £183 million but reduced it to £20 million after considering the company's response and mitigating factors[^4]. The breach involved attackers who gained access through compromised third-party supplier credentials, modified JavaScript on payment pages, and redirected customer payment data to attacker-controlled servers.

More importantly, organisations that fail to protect payment data lose customer trust, which can be far more costly than any financial penalty.

## Key PCI DSS Requirements for APIs

PCI DSS 4.0.1 contains numerous security controls, but several requirements have relevance for API development and security. Understanding these requirements helps developers build compliant systems from the ground up rather than attempting to introduce security after development.

### Requirement 4: Protect cardholder data with strong cryptography during transmission over open, public networks

**API Risk Addressed:** This requirement addresses data interception during transmission; when APIs transmit Primary Account Numbers (PANs) or sensitive authentication data over networks, attackers can intercept this traffic if it is not properly encrypted. Man-in-the-middle attacks, where attackers position themselves between the client and server to intercept communications, become possible without strong encryption.

The security control used to meet this requirement is Transport Layer Security (TLS) with strong cipher suites. APIs handling payment data must:
- Enforce HTTPS for all connections and reject unencrypted HTTP traffic
- Use TLS 1.2 or higher (TLS 1.3 is recommended)
- Implement cipher suites that use strong encryption algorithms
- Disable weak protocols like SSL, TLS 1.0, and TLS 1.1

Modern API gateways like AWS API Gateway support enhanced TLS security policies that enforce these standards. The TLS 1.3-only policy ensures that all connections use the latest, most secure version of TLS, which provides improved performance and security compared to earlier versions. Certificate pinning can provide additional protection against man-in-the-middle attacks by ensuring clients only accept certificates from expected sources.

### Requirement 6: Develop and Maintain Secure Systems and Software

**Requirement 6.2.3:** Bespoke and custom software is reviewed prior to being released to production or customers to identify and correct potential security vulnerabilities.

**API Risk Addressed:** This requirement addresses the risk of releasing vulnerable code into production as APIs often contain security flaws like broken authentication, broken object-level authorisation, excessive data exposure, lack of resource limiting, broken function-level authorisation, mass assignment, security misconfiguration, injection vulnerabilities, improper assets management, and insufficient logging and monitoring. These correspond to the OWASP API Security Top 10, which represents the most critical API security risks[^5].

The 2018 British Airways breach illustrates the consequences of failing to properly review and monitor code security. Attackers compromised third-party supplier credentials and modified JavaScript on payment pages to harvest payment card data[^3]. Proper pre-deployment security reviews and continuous monitoring could have detected the unauthorised code modifications.

Security control involves implementing a secure software development lifecycle (SDLC) that includes:
- **Code Reviews:** Peer review of all API code changes, with focus on authentication logic, authorisation checks, input validation, and data handling
- **Static Application Security Testing (SAST):** Automated scanning of API source code to identify potential vulnerabilities such as SQL injection points, weak cryptography, or hardcoded credentials
- **Dynamic Application Security Testing (DAST):** Testing running APIs to identify runtime vulnerabilities, configuration issues, and authentication flaws
- **API-Specific Security Testing:** Using tools designed for API testing that can detect issues like broken object-level authorisation, where users can access resources belonging to other users by manipulating object IDs

For example, tools like OWASP ZAP, Burp Suite, or commercial API security platforms can be integrated into CI/CD pipelines to automatically test APIs before deployment.

**Requirement 6.2.4:** Bespoke and custom software is reviewed to address common software attacks.

**API Risk Addressed:** This requirement specifically addresses business logic flaws and attacks on access control mechanisms. Unlike generic vulnerabilities that automated tools can detect, business logic flaws require understanding the application's intended behaviour. For APIs, this particularly addresses scenarios like:
- Authorisation bypasses where attackers access resources they should not have permission to view or modify
- Parameter manipulation where attackers modify request parameters to escalate privileges or access other users' data
- Race conditions in financial transactions where attackers can exploit timing to duplicate payments or transfers

The security control involves implementing comprehensive input validation, output encoding, and robust authentication and authorisation

**Requirement 6.3.2:** An inventory is maintained of bespoke and custom software, and third-party software components.

**API Risk Addressed:** This requirement addresses the risk of shadow APIs and unknown vulnerabilities in the API attack surface. Shadow APIs are undocumented or forgotten APIs that remain active in production environments. These often have weaker security controls because developers may not be aware they exist or may consider them "internal only" even though they are exposed to the internet. These undocumented APIs become attractive targets for attackers because they often receive less security scrutiny than officially documented APIs.

The security control involves implementing automated API discovery tools that continuously scan for new or modified API endpoints and maintaining accurate documentation

For example, tools like Kong, or Postman can help maintain API catalogues, whilst dependency scanning tools like Snyk or OWASP Dependency-Check can track third-party components and alert teams to vulnerabilities.

## How Compliance Influences Secure API Design

PCI DSS compliance drives architectural decisions that ultimately create more secure and resilient API systems. When developers design APIs with compliance requirements in mind from the beginning, they build better products that are both secure and easier to maintain.

### Tokenisation and Data Minimisation

The requirement for data encryption influences how APIs handle sensitive information throughout their lifecycle. Rather than passing cardholder data directly through API layers, compliant architectures often implement tokenisation, where sensitive data is replaced with non-sensitive tokens. The actual cardholder data remains in a secure vault that meets strict access controls, whilst APIs work with tokens that are useless if intercepted.

This design pattern significantly reduces the scope of PCI compliance because most API endpoints never touch actual cardholder data. For example:

1. When a customer enters payment information, the frontend sends it directly to a secure tokenisation service
2. The tokenisation service returns a token (for example, `tok_1234567890abcdef`)
3. The application's APIs work exclusively with this token
4. Only the payment processing service, which exists in a tightly controlled PCI-compliant environment, exchanges the token for the actual card data when processing transactions

This architecture means that most of the application's API infrastructure falls outside the scope of PCI DSS requirements, simplifying compliance and reducing risk.

### Secure Software Development Practices

The emphasis on secure software development practices encourages adoption of modern DevSecOps principles where security is integrated throughout the development pipeline rather than being a final checkpoint. API specifications become security contracts that define not just what the API does, but what security controls it enforces.

Automated testing validates that every endpoint properly:
- Authenticates users using secure mechanisms
- Authorises access based on roles and permissions
- Validates input data to prevent injection attacks
- Handles errors securely without leaking sensitive information
- Logs security-relevant events without logging sensitive data

This integration of security into the development process ensures that security is not an afterthought but a fundamental aspect of how APIs are designed, built, and deployed.

### Continuous Monitoring and Observability

The requirement for continuous monitoring influences how organisations architect their observability infrastructure. APIs must emit detailed logs that capture security-relevant events without logging sensitive cardholder data. This means implementing careful log filtering and masking to ensure compliance.

Rate limiting, anomaly detection, and behavioural analysis become standard features rather than optional enhancements. When unusual patterns emerge, such as a spike in failed authentication attempts or unexpected access to sensitive endpoints, automated systems can flag or block the activity before damage occurs.

For example, monitoring might detect:
- A user suddenly accessing APIs from multiple geographic locations simultaneously
- An account making API calls at unusual times
- Rapid sequential attempts to access different customer records
- Unusual patterns of API calls that suggest automated scanning or data harvesting

These patterns can trigger automatic responses such as requiring additional authentication, temporarily blocking the account, or alerting security teams for investigation.

### API Governance and Inventory Management

The inventory requirement drives API governance practices that maintain visibility across the entire API landscape. Organisations implement API catalogues that document every endpoint, what data it handles, who has access, and what security controls are in place.

This visibility enables risk-based prioritisation where APIs handling cardholder data receive extra scrutiny and protection compared to those handling only non-sensitive information. It also ensures that when vulnerabilities are discovered, security teams can quickly identify all affected APIs and deploy patches systematically.

## Conclusion

For API developers working in financial services or any industry handling payment data, PCI DSS compliance represents both a challenge and an opportunity. The challenge lies in implementing comprehensive security controls across complex API ecosystems while the opportunity comes from building secure-by-design systems that protect customer data, maintain regulatory compliance, and establish trust with users and partners.

By treating PCI DSS requirements not as regulatory burdens but as guidelines for building better APIs, developers can create payment systems that are both compliant and secure. The security practices driven by PCI DSS; strong encryption, comprehensive input validation, strong authentication and authorisation, continuous monitoring, and regular security testing create APIs that are resilient against attacks and worthy of customer trust.

As payment processing continues to shift towards API-first architectures and as attackers become increasingly sophisticated, the importance of proper API security only grows. Developers who invest in understanding PCI DSS requirements and implementing proper security controls position themselves as valuable professionals who can build the secure payment systems that modern commerce depends upon.

## References

[^1]: [Target Data Breach Settlement - Multi-State AG Settlement](https://www.bankinfosecurity.com/target-reaches-185-million-breach-settlement-states-a-9942)

[^2]: [Target Data Breach Cost Analysis](https://www.huntress.com/threat-library/data-breach/target-data-breach)

[^3]: [British Airways Data Breach - Wikipedia](https://en.wikipedia.org/wiki/British_Airways_data_breach)

[^4]: [ICO Fines British Airways £20m for Data Breach](https://www.gdprregister.eu/news/british-airways-fine/)

[^5]: [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
