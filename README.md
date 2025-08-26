# 🐞 Bug Report Samples

This repository contains Redacted bug reports from real-world findings. All company names, domains, and sensitive details have been anonymized. 
Reports are shared strictly for educational and professional demonstration purposes

---

## 📂 Repository Structure

- **/security-reports/** → Security-related bugs (e.g., XSS, SQLi, authentication bypass)  
- **/ui-ux-reports/** → User interface and usability issues  
- **/functional-reports/** → Logic or workflow bugs  

Each report follows a **structured format**:
1. **Title / Summary**  
2. **Description**  
3. **Steps to Reproduce**  
4. **Expected Result**  
5. **Actual Result**  
6. **Impact / Risk**  
7. **Evidence (screenshots, PoC code, etc.)**  
8. **Recommendation / Fix**  

---

## 📑 Example

```markdown
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
