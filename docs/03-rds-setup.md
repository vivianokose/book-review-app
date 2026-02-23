# 03 — RDS MySQL Setup

## Overview

A managed MySQL database was created using Amazon RDS inside the private DB subnets of `book-review-vpc`. A Read Replica was also provisioned in a second Availability Zone to support read scalability and demonstrate high availability best practices.

---

## Resources Created

| Resource | Name | Role |
|----------|------|------|
| DB Subnet Group | `book-review-db-subnet-group` | Defines which subnets RDS can use |
| Primary RDS Instance | `book-review-db` | Handles all read/write operations |
| Read Replica | `book-review-db-replica` | Handles read traffic, separate AZ |

---

## DB Subnet Group

- **Name:** `book-review-db-subnet-group`
- **VPC:** `book-review-vpc`
- **Subnets included:**

| Subnet | Availability Zone | CIDR |
|--------|-------------------|------|
| `db-subnet-1` | us-east-1a | 10.0.5.0/24 |
| `db-subnet-2` | us-east-1b | 10.0.6.0/24 |

> A DB subnet group tells RDS which subnets are eligible for database placement. Both subnets are private — no internet routing.

---

## Primary RDS Instance — `book-review-db`

| Setting | Value |
|---------|-------|
| Engine | MySQL 8.0 |
| Instance Class | db.t3.micro |
| Storage | 20 GB (gp2) |
| Storage Autoscaling | Disabled |
| Master Username | `admin` |
| Initial Database Name | `bookreview` |
| VPC | `book-review-vpc` |
| Subnet Group | `book-review-db-subnet-group` |
| Availability Zone | us-east-1a |
| Security Group | `db-sg` |
| Public Access | ❌ No |
| Authentication | Password authentication |

### Endpoint

```
book-review-db.xxxxxxxxxx.us-east-1.rds.amazonaws.com
```

*(Replace with your actual endpoint from the RDS console)*

---

## Read Replica — `book-review-db-replica`

| Setting | Value |
|---------|-------|
| Source Instance | `book-review-db` |
| Instance Class | db.t3.micro |
| VPC | `book-review-vpc` |
| Subnet Group | `book-review-db-subnet-group` |
| Availability Zone | us-east-1b |
| Security Group | `db-sg` |
| Public Access | ❌ No |

> The read replica is placed in `us-east-1b` — a different AZ from the primary — ensuring that a failure in one zone does not affect both database instances.

---

## Security & Network Design

```
App Tier EC2 (app-sg)
        │
        │  Port 3306 (MySQL)
        ▼
  [ db-sg inbound rule ]
        │
        ▼
  RDS Primary (us-east-1a)
  book-review-db-subnet-group
        │
        │  Replication
        ▼
  RDS Replica (us-east-1b)
```

- The RDS instances are in **private subnets only** — no route to the internet exists
- Only resources attached to `app-sg` can reach the database on port 3306
- The database is never directly accessible from a browser, SSH session, or public IP

---

## Database Details

| Property | Value |
|----------|-------|
| Database Name | `bookreview` |
| Port | `3306` |
| Username | `admin` |
| Password | *(stored securely — never committed to Git)* |

> ⚠️ Never hardcode your RDS password in your source code or commit it to GitHub. Always use environment variables (`.env` files) and add `.env` to your `.gitignore`.

---

## Why This Design?

- **Private subnets** — The database has no public IP and no internet route. It is only reachable from within the VPC by the app tier.
- **Read Replica in separate AZ** — Offloads read queries from the primary and provides a warm standby in case the primary AZ experiences issues.
- **Security group chaining** — `db-sg` only allows port 3306 from `app-sg`. Even if someone gains access to the VPC, they cannot reach the DB without also having the app-tier security group attached.
- **Named initial database** — Setting `bookreview` as the initial DB name during creation ensures the Sequelize ORM can connect and sync models immediately without manual DB creation.

---

## Commit Reference

```
chore: provision RDS MySQL primary instance and read replica in private subnets
```