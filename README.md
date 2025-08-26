# Bug Report 

This repository contains **redacted bug reports** from real-world findings.   
All company names, domains, and sensitive details have been anonymized.   
Reports are shared strictly for educational and professional demonstration purposes

---

## Report Structure

Each report follows a **structured format**:
1. **Title / Summary**  
2. **Description**  
3. **Steps to Reproduce**  
4. **Expected Result**  
5. **Actual Result**  
6. **Impact**
7. **Recommendation**
8. **reference** 

---

## Reports

---

### Report Title: [SQL Injection in Login Form]

**Description:**  
The login form is vulnerable to SQL injection due to unsanitized user input.

**Steps to Reproduce:**  
1. Go to `/login`  
2. Enter username: `' OR 1=1 --`  
3. Enter any password  
4. Submit  

**Expected Result:**  
The system should reject invalid input.

**Actual Result:**  
Login succeeds, granting unauthorized access.

**Impact:**  
Critical – allows authentication bypass and data exposure.

**Recommendation:**  
Use parameterized queries (prepared statements) and validate user inputs.

**Reference:**  
Use parameterized queries (prepared statements) and validate user inputs.

---

### Vulnerability: Security Misconfiguration – No Rate Limiting Protection

**Description:**  
We discovered that there is no rate limit protection on the login interface:  
`[REDACTED_URL]/path/login`  

This allows attackers to **brute-force user accounts** on your website, which may lead to denial-of-service attacks and credential stuffing.  


**Steps to Reproduce:**  
1. Open a browser and visit the login page.  
2. Log in using any email as the username and any password as the password.  
3. Intercept the request with Burp Suite.  
4. Send the request to Intruder.  
5. Add payloads for both email and password fields.  
6. Start the attack.  

*Note:* An attacker can obtain a list of user emails from public sources such as review websites (e.g., https://www.g2.com/products/[REDACTED]/reviews) or from breached credential sets available online.  


**Expected Result:**  
- The login interface should detect repeated failed login attempts and block or slow down brute-force attempts.  
- After multiple incorrect login attempts, the account or IP should be temporarily locked, challenged with CAPTCHA/OTP, or rate-limited.  

**Actual Result:**  
- The login interface accepts **unlimited login attempts** without restriction.  
- No lockout, CAPTCHA, or rate-limiting is applied, allowing attackers to brute-force credentials continuously.  


**Impact:**  
An attacker can exploit this vulnerability to launch DoS attacks against the system, leading to service disruptions or downtime. This can cause financial losses, reputational damage, and other negative consequences.  

If an intruder successfully brute-forces credentials, they gain full account access and legitimate login credentials. This can enable the attacker to:  
- Extract sensitive information (bank details, transaction history, etc.)  
- Perform unauthorized transactions using the compromised account  
- Reuse valid credentials on other websites (credential stuffing)  

**Recommendations:**  
- Implement CAPTCHA after a few failed login attempts  
- Introduce account lockouts or timeouts after repeated failures  
- Apply request and API rate limiting  
- Enforce OTP or MFA for additional protection  

---

### Vulnerability: Broken Authentication – Password Reset Link Remains Active

**Description:**  
When a user creates an account on `[REDACTED_URL]` and requests a password reset, a password reset link is sent to their email.  
If the user changes their password directly from within their account (via the Account Settings page) **without using the reset link**, the previously issued reset link **remains active**.  

As a result, if the user later accesses the old password reset link, they can still reset their password.  
This behavior poses a security risk because an attacker who intercepts or obtains an old reset link could still take over the account, even after the user has already changed their password.  

**Steps to Reproduce:**  
1. Sign up for an account on `[REDACTED_URL]`.  
2. Request a password reset for your account.  
3. Without using the reset link sent to your email, log in to your account.  
4. From the Account Settings page (`[REDACTED_URL]/profile`), change your password.  
5. Return to your email inbox and click the old password reset link received in Step 2.  
6. Observe that the reset link is still valid and allows password change.  


**Expected Result:**  
- Any previously issued password reset links should be **invalidated immediately** once the user:  
  - Changes their password from within the account, or  
  - Successfully resets their password via a reset link.  
- Attempting to use an expired/invalid reset link should result in an error such as:  
  *“This password reset link is no longer valid.”*  

**Actual Result:**  
- The old reset link remains **active and functional**, even after the user has changed their password from the account settings.  
- This allows reuse of outdated reset links, enabling unauthorized password changes.  

**Impact:**  
- An attacker who gains access to an old password reset link (via email compromise, leaked logs, or man-in-the-middle attacks) can reset the victim’s password at any time.  
- This leads to **account takeover**, exposing sensitive data and enabling fraudulent activities.  


**Recommendations:**  
- Immediately **invalidate all active reset tokens** once a password change occurs (either via account settings or another reset flow).  
- Enforce **single-use password reset links**.  
- Add an **expiration time** (e.g., 15 minutes) to reset links.  
- Monitor and log password reset token usage for anomaly detection.  

---

### [Bug] TRACE HTTP Method Permitted by Web Server

**Overview:**  
The affected web servers were discovered to allow the **TRACE HTTP method**.  
This method is intended for debugging and reflects user-supplied input (from the HTTP request header) back into the response header.  

When **Cross-Site Scripting (XSS)** vulnerabilities are present, the TRACE method can be abused to bypass the **HttpOnly cookie attribute**, in an attack known as **Cross-Site Tracing (XST)**.  
If successfully exploited, this could allow a remote attacker to gain access to sensitive information stored within the HTTP response headers, such as authentication session tokens.  

For further details on Cross-Site Tracing, see:  
🔗 [OWASP: Cross Site Tracing (XST)](https://owasp.org/www-community/attacks/Cross_Site_Tracing)  

---

**Bug Stats:**  
- **Assets Affected:**  
  - `https://gtm.website.com/`  
  - `https://gtm.stage.website.com/`  
- **Severity:** Medium  

---

**Steps to Reproduce:**  
1. Open a terminal.  
2. Execute the following commands:  
```bash
curl -vX TRACE "https://gtm.stage.website.com/"
curl -vX TRACE "https://gtm.website.com/"
```
3. Observe that the server responds and reflects the request headers back in the response.
