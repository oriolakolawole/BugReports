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

### Vulnerability: TRACE HTTP Method Permitted by Web Server

**Overview:**  
The affected web servers were discovered to allow the **TRACE HTTP method**.  
This method is intended for debugging and reflects user-supplied input (from the HTTP request header) back into the response header.  

When **Cross-Site Scripting (XSS)** vulnerabilities are present, the TRACE method can be abused to bypass the **HttpOnly cookie attribute**, in an attack known as **Cross-Site Tracing (XST)**.  
If successfully exploited, this could allow a remote attacker to gain access to sensitive information stored within the HTTP response headers, such as authentication session tokens.  

For further details on Cross-Site Tracing, see:  
🔗 [OWASP: Cross Site Tracing (XST)](https://owasp.org/www-community/attacks/Cross_Site_Tracing)  

- **Assets Affected:**  
  - `https://gtm.website.com/`  
  - `https://gtm.stage.website.com/`  
- **Severity:** Medium  


**Steps to Reproduce:**  
1. Open a terminal.  
2. Execute the following commands:  
```bash
curl -vX TRACE "https://gtm.stage.website.com/"
curl -vX TRACE "https://gtm.website.com/"
```
3. Observe that the server responds and reflects the request headers back in the response.

**Expected Result**
- The server should reject **TRACE** requests.  
- Response should return an error such as:  
  - `405 Method Not Allowed`  
  - `501 Not Implemented`  

**Actual Result**
- The server **accepts TRACE requests** and reflects back request headers in the response.  
- This behavior could be chained with XSS to perform **Cross-Site Tracing (XST)** attacks.  

**Impact**
- Attackers may bypass **HttpOnly** cookie protection.  
- Sensitive session data can be exposed if combined with XSS.  
- Expands the overall **attack surface** of the web server.  

**Remediation Procedure**
- Restrict supported HTTP methods to only those necessary for business operations (commonly: `GET` and `POST`).  
- Disable the **TRACE** method in the server configuration. For example, in Apache:  
```apache
TraceEnable off
```
- Ensure proper input validation and sanitization to reduce the risk of injection attacks.

---

### Vulnerability: Credential Stuffing via Brute Force Attack & Information Disclosure

**Description:**  
The login interface at:  
`[REDACTED_URL]/login`  
does not enforce rate limiting, allowing brute-force and credential stuffing attacks.  

Additionally, the server’s login responses disclose **whether an email is registered or not**, enabling attackers to enumerate valid user accounts.  

**Steps to Reproduce:**  
1. Open a browser and visit the login page.  
2. Enter any email as the username and any password as the password.  
3. Intercept the login request using Burp Suite.  
4. Send the request to Intruder.  
5. Add payloads for the email address field and start the attack.  
6. Observe the following responses:  
   - If the email is **not registered** → Response: *“You do not have a business account.”*  
   - If the email **is registered** → Response: *“Your credentials are incorrect.”*  

*Note:* An attacker can obtain a list of emails from breached databases or exposed datasets and use them to brute-force accounts.  

**Expected Result:**  
- The login interface should:  
  - Apply **rate limiting** to block brute-force attempts.  
  - Provide a **generic error message** for both invalid and valid email inputs, such as:  
    *“Incorrect email address or password.”*  

**Actual Result:**  
- The system:  
  - Allows **unlimited login attempts** without lockout or CAPTCHA.  
  - Discloses whether an email is registered via distinct error messages.  

**Impact:**  
- Attackers can enumerate valid user accounts through the error message discrepancy.  
- Attackers can perform credential stuffing/brute-force attacks against known emails.  
- If successful, attackers gain unauthorized access to user accounts, enabling them to:  
  - Extract sensitive information (bank details, private messages, credit card data, etc.).  
  - Perform fraudulent actions or send phishing/spam from compromised accounts.  
  - Reuse valid credentials on other platforms (credential stuffing).  

**Recommendations:**  
- Implement CAPTCHA and/or OTP after several failed login attempts.  
- Enforce **rate limiting** (e.g., max 5–20 attempts per IP/user within a short timeframe).  
- Standardize login error messages to avoid information disclosure. Example:  
  - *“Incorrect email address or password.”*  
- Monitor login attempts and block suspicious IPs showing brute-force patterns.  

---

### Vulnerability: Broken Authentication & Session Management

**Description:**  
When a user changes their password from one browser/device, active sessions on other browsers/devices remain valid.  
Instead of **expiring old sessions**, the application **updates them silently**, allowing continued access without reauthentication.  

This indicates a flaw in **session invalidation** on password change, which falls under **Broken Authentication & Session Management**.  


**Steps to Reproduce:**  
1. Log in from two browsers simultaneously (e.g., Chrome and Firefox).  
2. In Chrome, navigate to **Account Settings → Change Password**.  
3. Observe the session in Firefox.  
4. Firefox session remains active and is silently updated, instead of being expired.  

**Reproduction on multiple systems:**  
1. Log in from two different computers with the same account.  
2. Change the password on **Computer A**.  
3. Check the session on **Computer B**.  
4. Session on Computer B is still valid, instead of being logged out.  

**Expected Result:**  
- On password change, **all active sessions** (other than the one performing the change) should be **invalidated immediately**.  
- The user should be required to log in again with the **new password** on other devices/browsers.  

**Actual Result:**  
- All existing sessions remain **active** after a password change.  
- Old sessions are **silently updated** to use the new password without requiring reauthentication.  

**Impact:**  
- If an attacker has access to a victim’s session (e.g., via stolen cookies or shared access), they will remain logged in even after the victim resets their password.  
- This defeats the purpose of password changes as a recovery mechanism after compromise.  
- Leads to a **serious account takeover risk**.  

**Recommendations:**  
- Invalidate **all active sessions** whenever a password is changed.  
- Require all users to log in again with the new password.  
- Maintain proper session management practices:  
  - Enforce **session token rotation** after password change or privilege escalation.  
  - Track active sessions per user and terminate them on credential reset.  
  - Provide users with a **“Log out from all devices”** option.
 
---

### Vulnerability: Information Disclosure – User Enumeration via Login Responses

**Description:**  
The application’s login interface provides **different error messages** depending on whether the email address is registered or not.  

- **Unregistered email + any password** → Response: *“User was not found”*  
- **Registered email + wrong password** → Response: *“Incorrect username and password”*  

This discrepancy allows attackers to determine which email addresses are valid accounts.  

**Steps to Reproduce:**  
1. Navigate to the sign-in page: `[REDACTED_URL]/sign-in`.  
2. Attempt login with an **unregistered email address**.  
   - Observe the error message: *“User was not found”*.  
3. Attempt login with a **registered email** but an incorrect password.  
   - Observe the error message: *“Incorrect username and password”*.  

**Expected Result:**  
- The login interface should return a **generic error message** regardless of whether the email exists or not.  
- Example: *“Incorrect username and password.”*  

**Actual Result:**  
- The system discloses whether an email is valid by returning distinct error messages.  
- This enables attackers to enumerate registered users.  

**Impact:**  
- Attackers can enumerate valid email accounts by submitting login attempts with different email addresses.  
- Once valid accounts are identified, attackers can perform **brute-force or credential stuffing attacks**.  
- If successful, attackers gain access to accounts and can:  
  - Extract sensitive information (credit card details, personal data, etc.).  
  - Perform fraudulent transactions or malicious activities.  
  - Reuse stolen credentials on other platforms (credential reuse attack).  


**Recommendations:**  
- Standardize error messages to avoid information leakage.  
  - Use a **generic error message** for both invalid and valid emails, such as:  
    *“Incorrect username or password.”*  
- Implement **rate limiting and CAPTCHA** on login attempts to further reduce brute-force risks.  
- Monitor login attempts and flag suspicious activity.  

