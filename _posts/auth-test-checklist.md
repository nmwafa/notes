# Authentication Testing Checklist

## Table of Contents

1. [Registration Flaws](#1-registration-flaws)
   - 1.1 [Duplicate Registration Allowed](#11-duplicate-registration-allowed)
   - 1.2 [Weak Registration Validation](#12-weak-registration-validation)
   - 1.3 [Registration Parameter Manipulation](#13-registration-parameter-manipulation)
   - 1.4 [Username/Email Enumeration During Registration](#14-usernameemail-enumeration-during-registration)
   - 1.5 [Invite/Referral Code Bypass](#15-invitereferral-code-bypass)
   - 1.6 [Account Verification Skipping](#16-account-verification-skipping)
2. [Login Bypasses](#2-login-bypasses)
   - 2.1 [SQL Injection in Login](#21-sql-injection-in-login)
   - 2.2 [No SQL Injection (MongoDB)](#22-no-sql-injection-mongodb)
   - 2.3 [LDAP Injection](#23-ldap-injection)
   - 2.4 [Login Response Manipulation](#24-login-response-manipulation)
   - 2.5 [Remember Me Functionality Flaws](#25-remember-me-functionality-flaws)
   - 2.6 [Login with Weak Default Credentials](#26-login-with-weak-default-credentials)
   - 2.7 [Login Flooding / Rate Limit Bypass](#27-login-flooding--rate-limit-bypass)
3. [Session Management Issues](#3-session-management-issues)
   - 3.1 [Session Fixation](#31-session-fixation)
   - 3.2 [Predictable Session Tokens](#32-predictable-session-tokens)
   - 3.3 [Session Token Exposure](#33-session-token-exposure)
   - 3.4 [Insecure Session Storage](#34-insecure-session-storage)
   - 3.5 [Concurrent Session Handling Flaws](#35-concurrent-session-handling-flaws)
   - 3.6 [Session Hijacking via Cross-Site Scripting (XSS)](#36-session-hijacking-via-cross-site-scripting-xss)
4. [Multi-Factor Authentication (MFA) Bypasses](#4-multi-factor-authentication-mfa-bypasses)
   - 4.1 [MFA Not Enforced for All Users](#41-mfa-not-enforced-for-all-users)
   - 4.2 [MFA Code Brute Force](#42-mfa-code-brute-force)
   - 4.3 [MFA Code Reuse](#43-mfa-code-reuse)
   - 4.4 [MFA Bypass via "Remember This Device"](#44-mfa-bypass-via-remember-this-device)
   - 4.5 [MFA Bypass via OAuth/SSO](#45-mfa-bypass-via-oauthsso)
   - 4.6 [2FA Disable via Password Reset](#46-2fa-disable-via-password-reset)
   - 4.7 [MFA Code Leakage](#47-mfa-code-leakage)
5. [Credential Security Issues](#5-credential-security-issues)
   - 5.1 [Credentials in URL/Logs](#51-credentials-in-urllogs)
   - 5.2 [Weak Password Policy](#52-weak-password-policy)
   - 5.3 [Credential Stuffing Vulnerability](#53-credential-stuffing-vulnerability)
   - 5.4 [Password Equivalent Fields](#54-password-equivalent-fields)
   - 5.5 [Password Change Without Current Password](#55-password-change-without-current-password)
   - 5.6 [Password Hashing Weaknesses](#56-password-hashing-weaknesses)
6. [OAuth & SSO Vulnerabilities](#6-oauth--sso-vulnerabilities)
   - 6.1 [OAuth Redirect URI Manipulation](#61-oauth-redirect-uri-manipulation)
   - 6.2 [OAuth CSRF](#62-oauth-csrf)
   - 6.3 [SSO Signature Verification Bypass](#63-sso-signature-verification-bypass)
   - 6.4 [Account Takeover via SSO](#64-account-takeover-via-sso)
   - 6.5 [OAuth Scope Escalation](#65-oauth-scope-escalation)
7. [Logout & Session Termination Flaws](#7-logout--session-termination-flaws)
   - 7.1 [Incomplete Logout](#71-incomplete-logout)
   - 7.2 [Session Not Terminated on Password Change](#72-session-not-terminated-on-password-change)
   - 7.3 [Session Timeout Not Enforced](#73-session-timeout-not-enforced)
   - 7.4 [Logout CSRF](#74-logout-csrf)
8. [Rate Limiting & Brute Force Protections](#8-rate-limiting--brute-force-protections)
   - 8.1 [No Rate Limiting on Auth Endpoints](#81-no-rate-limiting-on-auth-endpoints)
   - 8.2 [Rate Limiting Bypass via Headers](#82-rate-limiting-bypass-via-headers)
   - 8.3 [Account Lockout Policy Flaws](#83-account-lockout-policy-flaws)
   - 8.4 [CAPTCHA Bypass](#84-captcha-bypass)
9. [Authentication API Security](#9-authentication-api-security)
   - 9.1 [API Authentication Bypass](#91-api-authentication-bypass)
   - 9.2 [JWT Vulnerabilities](#92-jwt-vulnerabilities)
   - 9.3 [Insecure Direct Object References (IDOR) in Auth](#93-insecure-direct-object-references-idor-in-auth)
   - 9.4 [Authentication Headers Exposed](#94-authentication-headers-exposed)
10. [Testing Methodology](#10-testing-methodology)
    - Phase 1: [Reconnaissance](#phase-1-reconnaissance)
    - Phase 2: [Functional Testing](#phase-2-functional-testing)
    - Phase 3: [Attack Patterns](#phase-3-attack-patterns)

---

## 1. Registration Flaws

### 1.1 Duplicate Registration Allowed

**How it works:** System allows multiple accounts with same email/username, or fails to verify uniqueness properly.

**Steps to Find:**
1. Register account with email+1@test.com
2. Attempt to register again with same email
3. Try case variations (Test@test.com vs test@test.com)
4. Test with Unicode homoglyphs
5. Check if email normalization is applied
6. Verify if secondary identifiers (username) can collide

**Impact:** Account confusion, data pollution, impersonation

### 1.2 Weak Registration Validation

**How it works:** Registration accepts invalid email formats, disposable emails, or nonexistent domains.

**Steps to Find:**
1. Register with email: "notanemail"
2. Test with: "test@test" (no TLD)
3. Use disposable email domains (temp-mail.org)
4. Test with + aliasing if restricted
5. Check email verification bypass
6. Verify domain existence validation

**Impact:** Spam accounts, anonymous abuse, verification bypass

### 1.3 Registration Parameter Manipulation

**How it works:** User-controllable parameters during registration allow privilege escalation.

**Steps to Find:**
1. Intercept registration request
2. Add parameters: "role": "admin", "isAdmin": true
3. Modify "email_verified": true
4. Test with "account_type": "premium"
5. Add unexpected fields (mass assignment)
6. Check if user ID can be pre-selected

**Impact:** Privilege escalation, automatic verification, account takeover

### 1.4 Username/Email Enumeration During Registration

**How it works:** System reveals whether a username/email is already taken through error messages or timing.

**Steps to Find:**
1. Register with existing email - check error message
2. Register with new email - compare response
3. Measure response time differences
4. Test JSON responses for existence hints
5. Check if rate limiting applies to registration attempts
6. Verify if enumeration possible via autocomplete/suggestions

**Impact:** User enumeration, reconnaissance, targeted attacks

### 1.5 Invite/Referral Code Bypass

**How it works:** Registration requires invite codes that can be guessed, reused, or bypassed.

**Steps to Find:**
1. Identify invite code parameter
2. Test with empty code
3. Try common codes (admin, test, invite123)
4. Check if codes are sequential or predictable
5. Test code reuse after registration
6. Verify if codes have usage limits

**Impact:** Unauthorized access, bypass of registration restrictions

### 1.6 Account Verification Skipping

**How it works:** Email/SMS verification can be bypassed by modifying requests or accessing features pre-verification.

**Steps to Find:**
1. Register without verifying email
2. Attempt to access protected features
3. Intercept verification confirmation
4. Modify response to indicate success
5. Check if verification status is client-side only
6. Test if old verification links remain valid

**Impact:** Unverified accounts with full access, spam, fraud

---

## 2. Login Bypasses

### 2.1 SQL Injection in Login

**How it works:** Login form vulnerable to SQL injection, allowing authentication bypass.

**Steps to Find:**
1. Test username: `'admin' --`
2. Try: `' OR 1=1 --`
3. Use: `'admin' /*`
4. Test with: `' UNION SELECT 'admin', 'password' --`
5. Check for blind SQL injection via timing
6. Test parameterized inputs in all login fields

**Impact:** Complete authentication bypass, data breach

### 2.2 No SQL Injection (MongoDB)

**How it works:** MongoDB-based login accepts operator injections.

**Steps to Find:**
1. Username: `{'$ne': null}`
2. Password: `{'$ne': null}`
3. Test: `{'$gt': ""}`
4. Try: `'$ne'=1`
5. Use: `username[$regex]=.*`
6. Check for operator injection in JSON bodies

**Impact:** Authentication bypass without credentials

### 2.3 LDAP Injection

**How it works:** Login form vulnerable to LDAP injection attacks.

**Steps to Find:**
1. Username: `*`
2. Password: `*`
3. Test: `admin*`
4. Try: `)(uid=*`
5. Use: `*)(uid=*`
6. Check for verbose LDAP errors

**Impact:** Authentication bypass, directory information disclosure

### 2.4 Login Response Manipulation

**Steps to Find:**
1. Attempt login with invalid credentials
2. Intercept response
3. Modify "success": false to true
4. Change HTTP status code from 401 to 200
5. Modify redirect location to authenticated area
6. Check if client blindly trusts response

**Impact:** Authentication bypass via response tampering

### 2.5 Remember Me Functionality Flaws

**How it works:** "Remember me" tokens are weak, predictable, or stored insecurely.

**Steps to Find:**
1. Check where remember-me token is stored (cookie, localStorage)
2. Analyze token structure (base64? encrypted? plain?)
3. Test token reuse across sessions
4. Check if token tied to IP/user-agent
5. Verify token expiration enforcement
6. Test token theft via XSS

**Impact:** Persistent session hijacking, account takeover

### 2.6 Login with Weak Default Credentials

**How it works:** Default credentials exist and are not changed on first login.

**Steps to Find:**
1. Search for default credentials in documentation
2. Test common combos: admin/admin, admin/password
3. Try vendor-specific defaults
4. Check if password change is enforced on first login
5. Test if default accounts are documented publicly
6. Verify if test accounts remain in production

**Impact:** Easy unauthorized access, privilege escalation

### 2.7 Login Flooding / Rate Limit Bypass

**How it works:** Login attempts are not properly rate-limited, allowing brute force.

**Steps to Find:**
1. Send multiple rapid login attempts
2. Check for rate limiting after N attempts
3. Test with different IPs (X-Forwarded-For spoofing)
4. Rotate user-agents
5. Try different endpoints (API vs web)
6. Check if rate limiting resets on successful login

**Impact:** Credential brute-forcing, account takeover

## 3. Session Management Issues

### 3.1 Session Fixation

**How it works:** Attacker can set victim's session ID to a known value.

**Steps to Find:**
1. Obtain session cookie from login page (pre-auth)
2. Set this cookie in attacker's browser
3. Send login link with session ID parameter
4. Check if session ID changes after login
5. Test if session is accepted in URL
6. Verify session regeneration on privilege change

**Impact:** Session hijacking, account takeover

### 3.2 Predictable Session Tokens

**How it works:** Session tokens generated using weak algorithms (timestamp, sequential, user data).

**Steps to Find:**
1. Collect 100+ session tokens
2. Analyze pattern (length, charset)
3. Check for encoded user data (base64, hash)
4. Test sequential generation
5. Use Burp Sequencer for analysis
6. Check if token tied to time with low entropy

**Impact:** Session hijacking via token prediction

---

### 3.3 Session Token Exposure

**How it works:** Session tokens exposed in URLs, logs, referrers, or via side-channels.

**Steps to Find:**
1. Check if session ID appears in URL parameters
2. Monitor network traffic for token in referrer
3. Test if token visible in browser history
4. Check WebSocket connections for token exposure
5. Verify if token logged on server
6. Test if token exposed in error messages

**Impact:** Session theft, account takeover

### 3.4 Insecure Session Storage

**How it works:** Session tokens stored insecurely on client side.

**Steps to Find:**
1. Check cookie flags (HttpOnly, Secure, SameSite)
2. Verify if token stored in localStorage/sessionStorage
3. Test if accessible via JavaScript
4. Check cookie domain/path scope (too broad)
5. Verify session timeout implementation
6. Test session persistence after browser close

**Impact:** Session theft via XSS, physical access

### 3.5 Concurrent Session Handling Flaws

**How it works:** System allows unlimited concurrent sessions or fails to notify users.

**Steps to Find:**
1. Login from multiple browsers/devices
2. Check if user is notified
3. Test session limits (should be configurable)
4. Verify if old sessions are invalidated on password change
5. Test if sessions can be viewed/terminated by user
6. Check for session conflict during simultaneous actions

**Impact:** Account sharing, undetected compromise

### 3.6 Session Hijacking via Cross-Site Scripting (XSS)

**How it works:** XSS vulnerabilities allow theft of session tokens.

**Steps to Find:**
1. Identify XSS vectors in authenticated areas
2. Test cookie accessibility via JavaScript
3. Check if HttpOnly flag is missing
4. Verify if token in localStorage can be stolen
5. Test session fixation combined with XSS
6. Check if CSRF tokens can be stolen similarly

**Impact:** Complete account takeover

---

## 4. Multi-Factor Authentication (MFA) Bypasses

### 4.1 MFA Not Enforced for All Users

**How it works:** MFA is optional or can be disabled without re-authentication.

**Steps to Find:**
1. Enable MFA on account
2. Attempt to disable MFA
3. Check if password re-entry required
4. Test if admin can disable user MFA
5. Verify if MFA enforced for privileged roles
6. Check if MFA can be removed via API

**Impact:** MFA bypass, reduced security

### 4.2 MFA Code Brute Force

**How it works:** 4-6 digit MFA codes have small search space and no rate limiting.

**Steps to Find:**
1. Trigger MFA challenge
2. Attempt brute force of 000000-999999
3. Check for rate limiting
4. Test if codes expire quickly
5. Verify if multiple attempts lock account
6. Check if codes are time-based (TOTP) with window

**Impact:** MFA bypass, account takeover

### 4.3 MFA Code Reuse

**How it works:** MFA codes remain valid after use or for extended periods.

**Steps to Find:**
1. Request MFA code
2. Use it successfully
3. Attempt to reuse same code
4. Test with code after expiration time
5. Check if codes tied to specific sessions
6. Verify if backup codes can be reused

**Impact:** MFA bypass, session hijacking

### 4.4 MFA Bypass via "Remember This Device"

**How it works:** Device remember functionality bypasses MFA indefinitely.

**Steps to Find:**
1. Enable "Remember this device"
2. Analyze remember token
3. Test if token can be stolen/reused
4. Check if token expires
5. Verify if token tied to browser fingerprint
6. Test if MFA can be bypassed with stolen token

**Impact:** Persistent MFA bypass, account compromise

### 4.5 MFA Bypass via OAuth/SSO

**How it works:** SSO login bypasses MFA requirements.

**Steps to Find:**
1. Enable MFA on local account
2. Link Google/Facebook account
3. Login via SSO
4. Check if MFA is required
5. Test unlinking/re-linking scenarios
6. Verify MFA status after SSO login

**Impact:** Complete MFA bypass

### 4.6 2FA Disable via Password Reset

**How it works:** Password reset flow disables or bypasses MFA.

**Steps to Find:**
1. Enable MFA on account
2. Initiate password reset
3. Check if MFA required during reset
4. Verify MFA status after password change
5. Test if reset generates new MFA secret
6. Check if backup codes still work after reset

**Impact:** MFA bypass, account takeover

### 4.7 MFA Code Leakage

**How it works:** MFA codes exposed in URLs, responses, or logs.

**Steps to Find:**
1. Check if MFA code appears in URL
2. Inspect JSON responses for code
3. Test if code visible in SMS/email content
4. Verify if code sent with previous messages
5. Check if code stored in browser storage
6. Test for timing attacks on code validation

**Impact:** MFA code theft, account compromise

---

## 5. Credential Security Issues

### 5.1 Credentials in URL/Logs

**How it works:** Passwords transmitted in URL parameters or GET requests.

**Steps to Find:**
1. Monitor network traffic during login
2. Check if password appears in URL
3. Verify if login uses POST or GET
4. Test if password in referrer header
5. Check server logs for credential exposure
6. Verify browser autofill security

**Impact:** Credential theft via logs, history, referrers

### 5.2 Weak Password Policy

**How it works:** Password policy allows weak, common, or short passwords.

**Steps to Find:**
1. Attempt to set "password123"
2. Try single character password
3. Test with "admin" or "welcome"
4. Check for dictionary word blocking
5. Verify minimum length enforcement
6. Test Unicode normalization issues

**Impact:** Easy brute-force, credential stuffing

### 5.3 Credential Stuffing Vulnerability

**How it works:** No protection against automated credential stuffing attacks.

**Steps to Find:**
1. Test with breached credential lists
2. Check for rate limiting
3. Verify CAPTCHA implementation
4. Test IP-based blocking
5. Check for credential stuffing detection
6. Verify if breached password detection exists

**Impact:** Account takeover via leaked credentials

### 5.4 Password Equivalent Fields

**How it works:** Security questions or PINs act as password equivalents with weaker security.

**Steps to Find:**
1. Identify security question fields
2. Check if they can reset password
3. Test if answers are case-sensitive
4. Verify if answers have complexity requirements
5. Check if answers stored in plaintext
6. Test if answers can be brute-forced

**Impact:** Account takeover via weak secondary credentials

### 5.5 Password Change Without Current Password

**How it works:** Password change functionality doesn't verify current password.

**Steps to Find:**
1. Login to account
2. Navigate to password change
3. Attempt change without current password
4. Test with session token only
5. Check if CSRF protection exists
6. Verify if notification sent on change

**Impact:** Account takeover via CSRF, session hijacking

### 5.6 Password Hashing Weaknesses

**How it works:** Passwords stored with weak hashing algorithms.

**Steps to Find:**
1. Attempt to extract password hashes (if possible)
2. Identify hash type (MD5, SHA1, bcrypt)
3. Check if salts are used
4. Verify if same password = same hash
5. Test timing attacks on comparison
6. Check for default/backdoor passwords in code

**Impact:** Credential theft, offline brute-force

---

## 6. OAuth & SSO Vulnerabilities

### 6.1 OAuth Redirect URI Manipulation

**How it works:** Open redirect in OAuth flow allows authorization code theft.

**Steps to Find:**
1. Identify redirect_uri parameter
2. Attempt to change to attacker.com
3. Test with open redirect on whitelisted domain
4. Try path traversal in redirect
5. Use localhost for testing
6. Check if state parameter validated

**Impact:** Authorization code theft, account takeover

### 6.2 OAuth CSRF

**How it works:** No state parameter or weak validation allows CSRF in OAuth flow.

**Steps to Find:**
1. Initiate OAuth login
2. Check if state parameter is used
3. Test without state parameter
4. Attempt to reuse auth codes
5. Check if state is predictable
6. Verify state tied to session

**Impact:** Account linking to attacker's social account

### 6.3 SSO Signature Verification Bypass

**How it works:** SSO assertions (SAML, JWT) not properly verified.

**Steps to Find:**
1. Capture SSO assertion
2. Modify email/username
3. Check if signature verified
4. Test with algorithm none attack
5. Remove signature entirely
6. Try with expired/revoked assertions

**Impact:** Authentication as any user

### 6.4 Account Takeover via SSO

**How it works:** Email mismatch between SSO and local account allows hijacking.

**Steps to Find:**
1. Register account with email@test.com
2. Login with SSO using attacker@test.com
3. Check if accounts merge automatically
4. Test if email can be changed during SSO
5. Verify email verification before linking
6. Check if SSO email can be spoofed

**Impact:** Account takeover via SSO misconfiguration

### 6.5 OAuth Scope Escalation

**How it works:** Requested OAuth scopes can be escalated to access more data.

**Steps to Find:**
1. Intercept OAuth authorization request
2. Modify scope parameter (add admin scopes)
3. Check if excessive scopes granted
4. Test with previously authorized apps
5. Verify scope validation
6. Check if scopes can be downgraded

**Impact:** Unauthorized data access, privilege escalation

---

## 7. Logout & Session Termination Flaws

### 7.1 Incomplete Logout

**How it works:** Logout doesn't fully terminate all sessions or tokens.

**Steps to Find:**
1. Login and obtain session
2. Perform logout
3. Attempt to reuse old session cookie
4. Check if API tokens still work
5. Test with back button after logout
6. Verify if cached pages accessible

**Impact:** Session persistence, unauthorized access

### 7.2 Session Not Terminated on Password Change

**How it works:** Old sessions remain active after password change.

**Steps to Find:**
1. Login on Browser A and Browser B
2. Change password on Browser A
3. Continue using Browser B
4. Test privileged actions
5. Check if session invalidated globally
6. Verify mobile app sessions

**Impact:** Session hijacking persistence

### 7.3 Session Timeout Not Enforced

**How it works:** Session never expires or timeout is too long.

**Steps to Find:**
1. Login to application
2. Leave idle for extended period (24h, 7d)
3. Return and attempt action
4. Check if session still valid
5. Verify absolute vs idle timeout
6. Test sensitive actions after idle

**Impact:** Unauthorized access via abandoned sessions

### 7.4 Logout CSRF

**How it works:** Logout functionality lacks CSRF protection.

**Steps to Find:**
1. Identify logout endpoint
2. Check for CSRF tokens
3. Create malicious logout page
4. Test with victim logged in
5. Verify if confirmation required
6. Check GET vs POST logout

**Impact:** Denial of service, user annoyance

---

## 8. Rate Limiting & Brute Force Protections

### 8.1 No Rate Limiting on Auth Endpoints

**How it works:** Authentication endpoints allow unlimited requests.

**Steps to Find:**
1. Send 100 rapid login attempts
2. Test registration endpoint
3. Check password reset
4. Verify MFA validation
5. Test API authentication endpoints
6. Check if rate limiting IP-based or account-based

**Impact:** Brute-force attacks, credential stuffing

### 8.2 Rate Limiting Bypass via Headers

**How it works:** Rate limiting can be bypassed by manipulating headers.

**Steps to Find:**
1. Test with X-Forwarded-For spoofing
2. Rotate User-Agent
3. Try different Accept-Language
4. Use proxy rotation
5. Test with null origin
6. Check if rate limiting resets on certain actions

**Impact:** Continued brute-force despite protections

### 8.3 Account Lockout Policy Flaws

**How it works:** Account lockout creates DoS or can be bypassed.

**Steps to Find:**
1. Trigger lockout with failed attempts
2. Check if legitimate user notified
3. Test if lockout persists indefinitely
4. Verify lockout bypass via password reset
5. Check if lockout applies to all endpoints
6. Test if lockout creates enumeration vector

**Impact:** Denial of service, brute-force continuation

### 8.4 CAPTCHA Bypass

**How it works:** CAPTCHA can be bypassed, removed, or solved automatically.

**Steps to Find:**
1. Remove CAPTCHA parameter from request
2. Test with empty CAPTCHA value
3. Check if CAPTCHA reused
4. Test OCR bypass on simple CAPTCHAs
5. Verify CAPTCHA tied to session
6. Check if CAPTCHA appears after N attempts only

**Impact:** Automated attacks, brute-force

---

## 9. Authentication API Security

### 9.1 API Authentication Bypass

**How it works:** Mobile/API endpoints have weaker authentication than web.

**Steps to Find:**
1. Map all API endpoints
2. Test without authentication headers
3. Try with expired/invalid tokens
4. Check if API accepts alternative auth methods
5. Test versioned APIs (v1 vs v2)
6. Verify if API checks authorization

**Impact:** Unauthorized data access

### 9.2 JWT Vulnerabilities

**How it works:** JWT tokens misconfigured or weakly implemented.

**Steps to Find:**
1. Decode JWT, analyze structure
2. Check for "alg": "none" attack
3. Test RS256 to HS256 downgrade
4. Verify signature validation
5. Check for sensitive data in payload
6. Test token expiration enforcement

**Impact:** Authentication bypass, privilege escalation

### 9.3 Insecure Direct Object References (IDOR) in Auth

**How it works:** User IDs in requests allow accessing other accounts.

**Steps to Find:**
1. Identify endpoints with user_id parameter
2. Change to other user's ID
3. Test profile update endpoints
4. Check password change with user_id
5. Verify email update endpoints
6. Test if authorization checks performed

**Impact:** Account takeover, data exposure

### 9.4 Authentication Headers Exposed

**How it works:** API keys, tokens exposed in client-side code or requests.

**Steps to Find:**
1. Check JavaScript files for hardcoded keys
2. Monitor network for tokens in URLs
3. Test if tokens in WebSocket connections
4. Check browser storage for API keys
5. Verify if tokens logged in console
6. Test if tokens in source maps

**Impact:** Credential theft, API abuse

---

## 10. Testing Methodology

### Phase 1: Reconnaissance

1. **Map Authentication Surface:**
   - Registration, login, logout
   - Password reset/recovery
   - Profile/account settings
   - MFA setup/management
   - OAuth/SSO connections
   - API endpoints (mobile, web)

2. **Document Parameters:**
   - Usernames, emails, passwords
   - Tokens (session, reset, MFA)
   - User IDs and roles
   - Redirect URLs
   - State/nonce values

3. **Identify Technologies:**
   - Authentication framework
   - Session management method
   - Password hashing algorithm
   - MFA implementation (TOTP, SMS)
   - OAuth provider versions

### Phase 2: Functional Testing

1. **Happy Path Testing:**
   - Complete registration flow
   - Login with valid credentials
   - Logout functionality
   - Password change
   - Account recovery
   - MFA enrollment/usage

2. **Edge Cases:**
   - Invalid formats (email, password)
   - Boundary values (min/max length)
   - Special characters, Unicode
   - Concurrent operations
   - Network interruptions
   - Browser back/forward buttons

3. **State Testing:**
   - Session persistence
   - Multi-tab/browser behavior
   - Concurrent logins
   - Session after password change
   - Remember me functionality
   - Idle timeout

### Phase 3: Attack Patterns

1. **Input Validation:**
   - Injection attacks (SQL, NoSQL, LDAP)
   - Parameter manipulation
   - Mass assignment
   - Unicode normalization
   - Null byte injection
   - Format string bugs

2. **Authentication Bypass:**
   - Response manipulation
   - Cookie tampering
   - JWT attacks
   - OAuth redirects
   - Direct endpoint access
   - HTTP method tampering

3. **Privilege Escalation:**
   - Horizontal (other users)
   - Vertical (admin roles)
   - Parameter tampering
   - Role manipulation
   - Insecure direct object references

4. **Session Analysis:**
   - Token predictability
   - Session fixation
   - Cookie security flags
   - Session timeout
   - Concurrent session handling

5. **Rate Limiting Tests:**
   - Brute-force attempts
   - Credential stuffing
   - Header manipulation
   - Distributed attacks
   - CAPTCHA bypass
