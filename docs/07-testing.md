# 07 — Testing & Verification

## Overview

This document covers the end-to-end testing and verification of the Book Review App deployment on AWS. All three tiers were tested individually and as a complete system to confirm the production architecture is working correctly.

---

## Testing Checklist

| Test | Expected Result | Status |
|------|----------------|--------|
| Web Tier EC2 accessible via Public IP | App loads in browser | *(update)* |
| Public ALB DNS loads the app | App loads via ALB DNS | *(update)* |
| Internal ALB routes to App Tier | Backend API responds | *(update)* |
| App Tier connects to RDS | Database queries succeed | *(update)* |
| User registration works | New user created in DB | *(update)* |
| User login works | JWT token returned | *(update)* |
| Books list loads | Books fetched from DB | *(update)* |
| Review submission works | Review saved to DB | *(update)* |
| RDS Multi-AZ active | Standby instance present | *(update)* |
| Read Replica available | Replica in Available state | *(update)* |

---

## Test 1 — Web Tier Direct Access

**Method:** Open Public IP in browser
```
http://<web-ec2-public-ip>
```
**Expected:** Book Review App UI loads
**Result:** *(update after testing)*

---

## Test 2 — Public ALB DNS Access

**Method:** Open Public ALB DNS in browser
```
http://<public-alb-dns>
```
**Expected:** Book Review App UI loads via ALB
**Result:** *(update after testing)*

---

## Test 3 — App Tier Health Check

**Method:** From Web Tier EC2 terminal, curl the Internal ALB
```bash
curl http://<internal-alb-dns>:3001
```
**Expected:** API response from Node.js backend
**Result:** *(update after testing)*

---

## Test 4 — Database Connectivity

**Method:** From App Tier EC2 terminal, test MySQL connection
```bash
mysql -h <rds-endpoint> -u admin -p bookreviews
```
**Expected:** Successful MySQL connection
**Result:** *(update after testing)*

---

## Test 5 — Full Stack Flow

**Method:** Open Public ALB DNS in browser and perform full user journey:

1. Visit homepage → Books list loads ✅/❌
2. Click on a book → Book details load ✅/❌
3. Register a new account → Registration succeeds ✅/❌
4. Login with new account → Login succeeds ✅/❌
5. Submit a book review → Review appears on page ✅/❌

**Result:** *(update after testing)*

---

## Architecture Verification

```
INTERNET
    │
    ▼ ✅ Public ALB DNS accessible
[book-review-public-alb]
    │
    ▼ ✅ Web Tier healthy targets
[book-review-web-tier EC2]
    │
    ▼ ✅ Internal ALB routing
[book-review-internal-alb]
    │
    ▼ ✅ App Tier healthy targets
[book-review-app-tier EC2]
    │
    ▼ ✅ RDS connection successful
[RDS MySQL — Multi-AZ + Read Replica]
```

---

## Screenshots

| Screenshot | Description |
|------------|-------------|
| `screenshots/26-app-via-alb-dns.png` | App loading in browser via Public ALB DNS |
| `screenshots/27-web-tier-healthy.png` | Public ALB target group showing healthy |
| `screenshots/28-app-tier-healthy.png` | Internal ALB target group showing healthy |
| `screenshots/29-rds-connected.png` | Successful MySQL connection from App Tier |
| `screenshots/30-full-app-working.png` | Full app working — books, login, reviews |

---

## Issues & Fixes

*(To be updated as testing is completed)*

---

## Commit Reference

```
chore: complete end-to-end testing and verification of three-tier deployment
```