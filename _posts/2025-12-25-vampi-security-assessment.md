---
title: "Breaking Into APIs: A Security Assessment of VamPI"
date: 2025-12-25 17:39:00
categories: [AppSec, API Security]
tags: [api-security, bola, idor, data privacy, APISecurity, InfoSec, application security, owasp top 10, devsecops, API, VAmPI]
layout: post
comments: true # <-- THIS LINE ENABLES GISCUS COMMENTS
---
Application Programming Interfaces (APIs) have become the backbone of modern web applications, handling everything from user authentication to sensitive data transfers. However, poorly secured APIs can become critical vulnerabilities that expose entire systems to attackers.

In this technical report, I document my security assessment of VamPI (Vulnerable API), an intentionally vulnerable application designed for security training. I systematically tested for three critical security flaws that when chained together allowed me to escalate from zero access to complete administrative control of the system.

The assessment focused on identifying vulnerabilities aligned with the OWASP API Security Top 10, specifically targeting authentication bypass, authorization failures, and data exposure issues.

> **Note:** This assessment was conducted in a controlled lab environment using VamPI, a deliberately vulnerable application created for educational purposes. No real systems were exploited.

## Assessment Overview

**Target Application:** VamPI ([Vulnerable API](https://github.com/erev0s/VAmPI))

**Environment:** Local Docker Container  
**Methodology:** Black-box penetration testing  

### Testing Environment Setup

```bash
# Pull the VamPI Docker image
docker pull erev0s/vampi

# Run the container on port 5000
docker run -p 5000:5000 erev0s/vampi
```

## Executive Summary

During this assessment, I identified three critical vulnerabilities that could enable complete system compromise:

| Finding                                  | Severity           | Impact                      |
| ---------------------------------------- | ------------------ | --------------------------- |
| Debug Endpoint Exposure                  | **Critical (9.2)** | Full database disclosure    |
| Mass Assignment Privilege Escalation     | **Critical (9.4)** | Unauthorized admin creation |
| Broken Authorization on Password Updates | **Critical (9.6)** | Complete account takeover   |

**Real-World Impact:** In a production environment, these vulnerabilities would allow attackers to:
- Steal all user credentials from the database
- Create unlimited administrator accounts
- Take over any user account, including system administrators
- Access and manipulate all system data
- Potentially violate GDPR and other data protection regulations

## The Attack Chain: From Zero to Admin

Let me walk you through how I progressed from having no access to gaining complete control of the system.

### Step 1: Discovery and Reconnaissance

Every security assessment begins with understanding the target. I started by:

1. **Mapping available endpoints** through API documentation
2. **Testing authentication requirements** on each endpoint
3. **Probing for hidden or debug endpoints**

This reconnaissance phase revealed several interesting endpoints, including one that would become my entry point.

---

## Finding #1: The Debug Endpoint That Exposed Everything

While exploring the API, I discovered an unauthenticated debug endpoint at `/users/v1/_debug`. Out of curiosity, I sent a simple GET request to this endpoint.

**What I Expected:** An error message or authentication challenge  
**What I Got:** The entire user database

### The Vulnerability

The debug endpoint returned complete user information without requiring any authentication

![App running locally3](/assets/img/vampi/debug.png)

**Why This Matters:**
- No authentication required - anyone could access this endpoint
- Passwords stored in plaintext (not hashed)
- Clear identification of which accounts had admin privileges
- Email addresses exposed for potential phishing attacks

### Additional Discovery: Exposed System Secrets

While testing for SQL injection vulnerabilities, I sent a malformed request to `/users/v1/name2' OR 1=1`. Instead of handling the error gracefully, the API returned detailed error information including:

```json
{
  "EVALEX": "true",
  "SECRET": "AVx0JrifAQ9blN5w0oMd"
}
```
![App running locally3](/assets/img/vampi/sqli.png)

This revealed:
- **Interactive debugger enabled** in the running environment
- **Hardcoded application secret** used for signing authentication tokens

### The Impact

This single vulnerability represents a complete security failure:

1. **Credential Theft:** Attackers gain access to all user passwords
2. **Token Forgery:** With the exposed secret, attackers can create valid authentication tokens for any user
3. **Account Takeover:** Direct login to any account using stolen credentials

**CVSS Score:** 9.2 (Critical) - No authentication required and complete confidentiality breach
**OWASP Categories:** API8:2023 (Security Misconfiguration), API3:2023 (Excessive Data Exposure)

### Security Remediation

**Immediate Actions:**
1. Remove all debug endpoints from production environments
2. Use environment variables to control debug mode (never enable in production)
3. Rotate all exposed secrets immediately
4. Implement authentication on all endpoints that return user data

**Long-term Solutions:**
1. Store passwords using strong hashing algorithms
2. Store secrets in secure vaults (AWS Secrets Manager, HashiCorp Vault)
3. Implement response filtering to return only necessary data
4. Add automated security scanning to detect exposed endpoints

---

## Finding #2: Creating Admin Accounts Through Mass Assignment

With knowledge of the database structure from the debug endpoint, I turned my attention to the user registration endpoint at `POST /users/v1/register`.

Standard registration requests looked like this:

```json
{
  "username": "newuser",
  "password": "password123",
  "email": "user@example.com"
}
```

But what if I added an extra parameter?

### The Exploit

I modified the registration request to include an `admin` parameter

```json
{
  "username": "testuser",
  "password": "pass1",
  "email": "test@example.com",
  "admin": true
}
```

**Result:** The API accepted the parameter and created an account with administrative privileges.

![App running locally3](/assets/img/vampi/admin.png)

### Understanding Mass Assignment

Mass assignment occurs when an application automatically binds user input to internal object properties without validation. In this case:

1. The registration endpoint accepted all JSON parameters from the request
2. It directly mapped these parameters to the user object
3. No validation checked whether the `admin` parameter should be allowed
4. The privileged property was assigned without authorization

**Analogy:** Imagine a hotel check-in form where you can write anything you want, including "Room Type: Presidential Suite" - and the hotel just gives it to you without verifying you paid for it.

### Verification

I verified the successful privilege escalation by:

1. Checking the debug endpoint - my new account appeared with `"admin": true`
   ![App running locally3](/assets/img/vampi/created.png)
2. Logging in with the account and receiving an admin-level authentication token
3. ![App running locally3](/assets/img/vampi/login.png)
4. Testing admin-only functionality - all succeeded

### The Impact

This vulnerability enables instant privilege escalation:

- **Unlimited Admin Creation:** Attackers can create as many admin accounts as needed
- **Bypassed Controls:** No approval process or authorization checks
- **Persistent Access:** Attacker-controlled admin accounts remain active until manually discovered
- **Complete System Control:** Admin accounts typically have unrestricted access to all functions and data

**CVSS Score:** 9.4 (Critical) - Complete privilege escalation, no special permissions required
**OWASP Categories:** API3:2023 (Broken Object Property Level Authorization), API5:2023 (Broken Function Level Authorization)

### Security Recommendation

**Immediate Actions:**
1. Implement server-side property whitelisting to prevent mass assignment
2. Never allow privileged properties in user-supplied input
3. Enforce role-based access control for privilege assignment

**Code Example (Python/Flask):**

```python
# BAD: Direct mass assignment
@app.route('/register', methods=['POST'])
def register():
    user = User(**request.json)  # Dangerous
    db.session.add(user)
    return {"status": "created"}

# GOOD: Explicit whitelisting
@app.route('/register', methods=['POST'])
def register():
    data = request.json
    user = User(
        username=data.get('username'),
        password=hash_password(data.get('password')),
        email=data.get('email')
        # admin field is NOT included - must be set separately with proper authorization
    )
    db.session.add(user)
    return {"status": "created"}
```

**Additional Measures:**
- Audit and alert on all privilege changes
- Implement administrative approval for role assignments
- Log all registration attempts with source IP and parameters

---

## Finding #3: Taking Over Any Account

With my newly created admin account, I tested whether authorization controls were properly implemented on other sensitive endpoints. I targeted the password update endpoint at `PUT /users/v1/{username}/password`.

### The Vulnerability

The endpoint allowed authenticated users to change passwords for *any* username specified in the URL path, without verifying:
- Whether the authenticated user owned that account
- Whether the target account had higher privileges
- Whether the action should be allowed at all

### The Attack

**Step 1:** Obtained an authentication token by logging in as `testuser` (my mass-assigned admin account)
![App running locally3](/assets/img/vampi/login2.png)

**Step 2:** Changed another user's password

```
PUT /users/v1/name1/password
Authorization: Bearer [testuser_token]

{
  "password": "testpass"
}
```

**Response:** `{"status": "password changed"}`

**Step 3:** Verified by logging in as `admin` with the new password - success
![App running locally3](/assets/img/vampi/debug2.png)

### Escalating to Full System Compromise
After changing the admin password, I:
1. Logged in as the original admin using the new password
2. Verified full administrative access
3. Demonstrated persistence by deleting the original admin account entirely
![App running locally3](/assets/img/vampi/deleted.png)
![App running locally3](/assets/img/vampi/deleted2.png)

### Understanding Broken Function Level Authorization (BFLA)

This vulnerability type (BFLA) occurs when:
- An API endpoint performs a sensitive function (password change, deletion, privilege modification)
- The endpoint checks *if* a user is authenticated, but fails to check *whether that specific user should be allowed* to perform the action

**Analogy:** It's like a building that checks if you have *any* key card to enter, but doesn't verify whether your specific card should open *this particular* door.

### The Impact

This vulnerability enables complete account takeover

1. **Lateral Movement:** Change passwords for any user account
2. **Privilege Preservation:** Lock out legitimate administrators by changing their passwords
3. **Evidence Destruction:** Delete compromised accounts to hide tracks
4. **Persistent Control:** Maintain access even if initial entry point is closed

**CVSS Score:** 9.6 (Critical)  
**OWASP Category:** API5:2023 (Broken Function Level Authorization)

### Security Recommendation

**Immediate Actions:**

1. **Implement proper authorization checks**

```python
@app.route('/users/<username>/password', methods=['PUT'])
@require_authentication
def change_password(username):
    current_user = get_authenticated_user()
    
    # Check if user can modify this account
    if current_user.username != username and not current_user.is_admin:
        return {"error": "Unauthorized"}, 403
    
    # Additional check: prevent modifying higher-privileged accounts
    target_user = User.query.filter_by(username=username).first()
    if target_user.is_admin and not current_user.is_admin:
        return {"error": "Cannot modify admin accounts"}, 403
    
    # Proceed with password change
    target_user.set_password(request.json['password'])
    db.session.commit()
    
    # Notify user of password change
    send_notification(target_user.email, "Password changed")
    
    return {"status": "password changed"}
```

2. **Add security notifications**
   - Email users when their password is changed
   - Provide a mechanism to report unauthorized changes

3. **Implement comprehensive logging:**
   - Log all password modification attempts
   - Include source IP, timestamp, actor, and target account
   - Set up alerts for suspicious patterns

---

## The Complete Attack Chain

When combined, these three vulnerabilities create a trivial path to complete system compromise:

```
1. Access /users/v1/_debug
   → Discover all usernames and passwords
   
2. POST /users/v1/register with {"admin": true}
   → Create unauthorized admin account
   
3. PUT /users/v1/admin/password
   → Change original admin password
   
4. Login as admin
   → Full system control achieved
   
```

This demonstrates why security must be comprehensive - a chain is only as strong as its weakest link.

---

## Comprehensive Remediation Strategy

### Immediate Actions (0-7 Days) - Critical Priority

1. **Remove Debug Endpoints**
   - Disable all debug functionality in production
   - Use environment-based configuration
   - Implement feature flags for debug mode

2. **Rotate All Secrets**
   - Change all exposed API keys and secrets immediately
   - Invalidate all existing authentication tokens
   - Force password resets for all users

3. **Fix Mass Assignment**
   - Implement parameter whitelisting on all endpoints
   - Remove ability to set privileged fields through user input
   - Add validation layers

4. **Implement Authorization**
   - Add ownership verification to all data access
   - Enforce role-based access control
   - Verify permissions server-side (never trust client data)

5. **Hash Passwords**
   - Replace plaintext passwords with hashes
   - Force all users to reset passwords
   - Implement minimum password requirements

### Short-Term Actions (1-4 Weeks)

1. **Comprehensive RBAC Implementation**
   - Define clear role hierarchies
   - Create permission matrices for all endpoints
   - Document which roles can perform which actions

2. **Security Logging and Monitoring**
   - Log all authentication attempts (success/failure)
   - Track privilege escalation events
   - Monitor for suspicious patterns
   - Set up real-time alerts

3. **Response Filtering**
   - Define Data Transfer Objects (DTOs) for each endpoint
   - Never return sensitive fields unnecessarily
   - Implement field-level permissions

### Medium-Term Actions (1-3 Months)

1. **Secure Development Practices**
   - Mandatory security code reviews
   - Automated security scanning in CI/CD pipeline (SAST/DAST)
   - Regular penetration testing schedule
   - Security training for development team

2. **Input Validation Framework**
   - JSON schema validation for all requests
   - Sanitization of user inputs
   - Type checking and bounds validation

3. **Incident Response**
   - Establish incident response procedures
   - Define escalation paths
   - Create runbooks for common security events
   - Regular incident response drills

---

## Key Takeaways for Developers

### 1. Never Expose Debug Endpoints in Production

Debug functionality is invaluable during development but catastrophic in production. Use environment variables and feature flags to ensure debug code never reaches production systems.

### 2. Implement Defense in Depth

Security should exist at multiple layers:
- **Application Layer:** Authentication, authorization, input validation
- **Data Layer:** Encryption, hashing, access controls

### 3. Trust Nothing from the Client

All user input is potentially malicious:
- Validate everything server-side
- Never trust client-side validation alone
- Implement parameter whitelisting, not blacklisting
- Assume attackers will send unexpected data

### 4. Authorization is Not Authentication

Authentication answers "Who are you?"  
Authorization answers "Are you allowed to do this?"

Both are essential, and neither is sufficient alone.

### 5. Follow the Principle of Least Privilege

Users and services should have only the minimum permissions necessary:
- Default to denying access
- Explicitly grant only required permissions
- Regularly audit and revoke unnecessary privileges

---

## Tools Used in This Assessment

- **Postman** for API testing and request manipulation
- **Docker** for Container deployment and management
- **VSCode:** for documentation and code review

## Conclusion

This assessment of VamPI revealed critical vulnerabilities that, while demonstrated in a training environment, mirror real-world security flaws found in production APIs. The combination of excessive data exposure, mass assignment, and broken authorization created a clear path from zero access to complete system compromise.

API security requires a comprehensive approach. Individual vulnerabilities can be severe, but when combined, they create attack chains that dramatically increase risk.

For organizations deploying APIs, security cannot be an afterthought. Proper authentication, authorization, input validation, and secure configuration must be built into every API from the start.

**Interested in discussing API security or any cybersecurity related project?** Feel free to connect with me on [LinkedIn](https://www.linkedin.com/in/zainab-oladiti/).

---

*Disclaimer: This report documents security testing conducted in a controlled lab environment for educational purposes. Never attempt to test or exploit vulnerabilities in systems you don't own or have explicit permission to assess. Unauthorized access to computer systems is illegal.*
