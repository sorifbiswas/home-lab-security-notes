# DVWA: Damn Vulnerable Web Application

**Lab machine:** `target-lab` (192.168.56.30, Host-only, **no internet**)
**Goal:** Understand the most common web vulnerabilities by seeing them in a deliberately weak app, and, more importantly, learn how to fix each one.
**Status:** Draft

---

## Background
DVWA is a PHP and MySQL web application that is **intentionally vulnerable**. It exists for legal practice. It must **never** be put on the internet or on a network I do not control, because real attackers could use it as a foothold.

## Safe setup rules
- Run it only on the isolated Host-only network (see [network-design](../01-lab-setup/network-design.md)).
- Confirm isolation: from the target VM, `ping 8.8.8.8` must fail.
- Follow the **official installation guide** in the DVWA GitHub project (`digininja/DVWA`), because the steps change between versions. A Docker image is also offered by the project.
- Change nothing about its intentionally weak default credentials except on a VM that stays isolated.

## Security levels
DVWA has four levels: **Low, Medium, High, Impossible**. The best learning trick: open **View Source** at each level and compare. Low shows the mistake. **Impossible shows the correct fix.**

## Modules: what is wrong and how to fix it
| Module | The weakness (concept) | The fix |
|--------|------------------------|---------|
| Brute Force | Login allows unlimited guesses | Rate limiting, account lockout, MFA, strong passwords |
| Command Injection | User input reaches a shell command | Avoid the shell, use safe APIs, allow-list inputs |
| CSRF | A site accepts requests a user did not mean to send | Anti-CSRF tokens, `SameSite` cookies, re-authentication |
| File Inclusion | User controls which file the server loads | Allow-list of file names, never build paths from input |
| File Upload | Server accepts and runs dangerous files | Check type and content, rename, store outside the web root, no execute rights |
| SQL Injection | User input becomes part of a database query | **Parameterized queries** (prepared statements) |
| Weak Session IDs | Predictable session identifiers | Long random IDs from a secure generator, `HttpOnly` and `Secure` cookies |
| XSS (reflected, stored, DOM) | User input shown as page code | Output encoding, Content Security Policy |
| Content Security Policy | Weak CSP lets bad scripts run | A strict CSP without unsafe allowances |

## The key fix for injection, in code
The mistake: **building a query by pasting user input into a string**.
```python
# WRONG: input becomes part of the SQL code
query = f"SELECT * FROM users WHERE name = '{name}'"
```
The fix: **parameters**, so input is always treated as data, never as code.
```python
# RIGHT: the database driver keeps data and code separate
cursor.execute("SELECT * FROM users WHERE name = %s", (name,))
```
The same principle fixes command injection (pass arguments as a list, never through a shell string) and XSS (encode output for the place where it is shown).

## My log (fill in for each module)
| Module | Level | What I observed | Why it happened | Fix from the Impossible source |
|--------|-------|-----------------|-----------------|-------------------------------|
| SQL Injection | Low | | | |

## Detect it as a defender
- Web server logs: look for odd characters, very long parameters, and repeated errors from one address.
- A **web application firewall (WAF)** can block common patterns, but the real fix is secure code.
- Compare what I see in `/var/log/apache2/access.log` (or the web server in use) with what I did in the browser.

## What I learned
- Almost every injection flaw is the same mistake: mixing **data** with **code**.
- Fixing the root cause (parameterized queries, output encoding) beats trying to filter bad input.

## Defensive takeaway
Validate input, **encode output**, use parameterized queries, apply least privilege to the database account, keep software updated, and learn the **OWASP Top 10**.
