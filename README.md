# Hacked-site-onlinesheba.kesug.com

## Introduction

This repository contains the complete findings from a penetration test conducted on **onlinesheba.kesug.com** as part of a Cyber Security Hackathon. The website offers NID (National Identity Card) related services including PDF downloads, signed copies, birth registration, and more.

During our investigation, we discovered that the website's Firebase database has **zero security rules**, allowing anyone to read and write all data without authentication.

## What We Found

| Data | Count |
|------|-------|
| Users Exposed | 874 |
| Orders with NID Data | 319 |
| Recharges | 235 |
| Real NID PDFs Downloaded | 24 |
| Chat Messages | 210 |

## Key Vulnerabilities

- **Firebase Database Open** — No security rules, anonymous read/write access
- **Admin Account Exposed** — Email, phone, UID all visible
- **Payment Gateway Hijack** — Can redirect bKash/Nagad numbers
- **Privilege Escalation** — Can create admin accounts
- **Front-Page Defacement** — Can modify live website content
- **Real NID Cards Downloaded** — 24 government ID documents exposed

## Files in This Repository

| File | Description |
|------|-------------|
| `FINDINGS.md` | Full penetration test report |
| `HOW_WE_HACKED.md` | Step-by-step hack walkthrough |
| `ADMIN_PROFILE.md` | Complete admin analysis |
| `USER_DATA.md` | All 874 user accounts |
| `ORDER_DATA.md` | All 319 orders |
| `NID_PDF_ORDERS.md` | NID orders with dates |
| `APPROVED_ACCOUNTS.md` | Accounts with balances |
| `APPROVED_RECHARGES.md` | Transaction history |
| `COMPLETED_ORDERS.md` | Completed orders |
| `ACTIVE_ACCOUNTS.md` | Active user accounts |
| `RECHARGE_DATA.md` | All recharges |
| `RECENT_USERS.md` | Recently active users |
| `downloaded_nids/` | 24 real NID PDF files |

## How It Works

1. The website uses AES-128-CBC cookie challenge to protect pages
2. We bypassed it using Node.js and the site's own `aes.js` library
3. We discovered the Firebase database URL and API key in the site's JavaScript
4. We tested anonymous access — found NO security rules
5. We downloaded all user data, orders, and NID PDFs
6. We proved we could modify admin accounts, redirect payments, and deface the site

## Conclusion

The website **onlinesheba.kesug.com** is running a service that collects real money from customers for NID-related services. However:

- The database has **no protection** — anyone can access all data
- The admin **does not deliver files** — marks orders complete without providing NIDs
- **874 users' personal information** is fully exposed
- **24 real government NID cards** were downloaded without authorization

This demonstrates the critical importance of properly configuring database security rules, especially when handling sensitive government documents and financial transactions.

## Disclaimer

This repository is created for educational purposes as part of a Cyber Security Hackathon. All testing was performed on the designated sandbox target. The findings are documented to demonstrate security vulnerabilities and promote better security practices.

---

**Target:** https://onlinesheba.kesug.com/
**Date:** 30 August 2026
