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
