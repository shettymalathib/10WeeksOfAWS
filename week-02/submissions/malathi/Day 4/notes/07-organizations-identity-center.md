# Day 4 — AWS Organizations, SCPs & IAM Identity Center 

---

# Step 1: First understand why companies need multiple AWS accounts

Imagine you start a company.

You have only **one AWS account**.

Inside that account, everyone works.

There are:

* 5 developers
* 2 testers
* Security team
* Finance team
* Production servers

Everything is inside ONE account.

```
AWS Account

├── EC2
├── S3
├── RDS
├── Lambda
├── IAM
├── Billing
└── Everyone uses this account
```

Seems simple.

But problems start.

---

## Problem 1 — Developer accidentally deletes Production

Suppose Rahul is testing something.

He accidentally runs:

```
Delete Production Database
```

Production goes down.

Millions of users are affected.

---

## Problem 2 — Testing increases production bill

A tester launches:

```
50 EC2 Instances
```

Now the AWS bill becomes huge.

Finance asks:

> Who created these?

Nobody knows.

---

## Problem 3 — Security issue

Suppose one developer's credentials get hacked.

Because everyone uses the same account,

The hacker now has access to:

* Production
* Development
* Database
* Billing
* IAM

Everything.

---

## Problem 4 — Permissions become messy

Suppose:

Developer should only access:

```
EC2
```

Tester needs

```
S3
```

Finance needs

```
Billing
```

Security needs

```
CloudTrail
```

Managing permissions inside one account becomes difficult.

---

# AWS Solution

Instead of one account...

Create MANY AWS Accounts.

Example:

```
Company

├── Development Account
├── Testing Account
├── Production Account
├── Security Account
├── Finance Account
```

Each account is isolated.

Think of every AWS account as its own **house**.

If a fire starts in one house,

It doesn't burn all the houses.

This is called **Isolation**.

---

# Step 2: What is AWS Organizations?

AWS Organizations is simply a **service that manages multiple AWS accounts from one place**.

Imagine you own many apartments.

Instead of visiting each apartment separately,

You have one office that manages them all.

That's AWS Organizations.

```
AWS Organization

                Company

                    │
        --------------------------
        │      │      │      │
      Dev   Test   Prod   Finance
```

One dashboard.

Many AWS accounts.

---

# Step 3: Organization Structure

Every AWS Organization has a hierarchy.

```
Organization

        Root
         │
   ----------------
   │              │
Development     Production
```

The **Root** is only a container.

It is **NOT** the AWS root user.

This is one of the biggest exam tricks.

---

## Root User vs Organization Root

These are completely different things.

### AWS Root User

This is:

```
your_email@gmail.com
```

The very first user created when an AWS account is opened.

It has full power **within that single AWS account**.

---

### Organization Root

This is simply the **top folder** inside AWS Organizations.

Example:

```
Organization

Root
│
├── Dev OU
├── Prod OU
└── Finance OU
```

It is **not a login**, **not a user**, and **cannot sign in**.

### Easy memory trick

* AWS **Root User** = Super Admin **person/account credentials**
* Organization **Root** = Top **folder/container** in the organization hierarchy

---

# Step 4: Management Account

One account becomes the **management account**.

It controls the organization.

```
Management Account

│
├── Development
├── Testing
├── Production
└── Security
```

Responsibilities:

* Create accounts
* Invite accounts
* Manage Organization
* Pay bills
* Attach SCPs

**Best practice:** Keep this account for administration only. Don't run normal applications here.

---

# Step 5: Member Accounts

All other accounts are called **member accounts**.

Example:

```
Management

│
├── Dev
├── Test
├── Prod
└── Security
```

Each one has its own:

* EC2
* S3
* IAM
* VPC
* RDS
* CloudWatch

They are isolated from each other by default.

---

# Step 6: Why Multiple Accounts?

Imagine a school.

Instead of keeping:

* Nursery
* Grade 1
* Grade 12

inside one classroom,

you create separate classrooms.

Same idea.

Benefits:

* Better security
* Better billing
* Better permissions
* Easy auditing
* Smaller blast radius
* Easier compliance

---

# Step 7: Organizational Units (OU)

Suppose your company has 100 AWS accounts.

Managing them one by one is painful.

Instead, group similar accounts.

Example:

```
Root

│

├── Development OU
│      ├── Dev1
│      ├── Dev2
│      └── Dev3
│
├── Production OU
│      ├── Prod1
│      ├── Prod2
│
└── Security OU
       ├── Audit
       └── Logging
```

OU = **folder of accounts**.

You attach policies to the folder instead of every account.

---

# Step 8: What is an SCP?

SCP = **Service Control Policy**

This is one of the most misunderstood AWS concepts.

Think of an SCP as a **company rulebook**.

Imagine your father says:

> Nobody in this house can drive after 10 PM.

Even if your brother has a driving license,

House rules say NO.

Company rule wins.

That's exactly how an SCP works.

It **doesn't grant permissions**. It only defines the **maximum permissions** allowed.

---

# Step 9: IAM vs SCP

Suppose your IAM policy says:

```
Allow Delete EC2
```

But the SCP says:

```
Deny Delete EC2
```

What happens?

**Result: Denied.**

Why?

Because an explicit **Deny** always wins.

---

## Permission Formula

An action is allowed only if **all** of these are true:

* IAM policy or Identity Center permission set allows it.
* The applicable SCP also allows it (or at least does not deny it).
* There is no explicit deny from any applicable policy.

A simple way to remember it:

```
Allowed =
IAM Allow
AND
SCP allows
AND
No Explicit Deny
```

---

# Step 10: Example SCP

Suppose the company never wants developers creating S3 buckets.

SCP:

```json
{
  "Effect": "Deny",
  "Action": "s3:CreateBucket",
  "Resource": "*"
}
```

Now imagine the developer has:

```
AdministratorAccess
```

Can they create a bucket?

**No.**

The SCP blocks it.

This is why SCPs are called **guardrails**.

---

# Step 11: Does an SCP Give Permission?

No.

Suppose:

IAM:

```
No EC2 permission
```

SCP:

```
Allow EC2
```

Can the user launch EC2?

**No.**

Why?

Because the SCP never grants permissions. It only limits the maximum permissions available.

Think of it this way:

* **IAM policy** = Gives someone the keys.
* **SCP** = Company rules about which doors those keys may ever open.

Without the keys (IAM), the rules (SCP) alone don't help.

---

# Step 12: IAM Identity Center

Earlier, companies created many IAM users.

Example:

```
Developer

Account 1 user

Account 2 user

Account 3 user

Account 4 user
```

One person.

Four IAM users.

Very hard to manage.

---

AWS introduced **IAM Identity Center**.

Now the developer has **one identity**.

```
Developer

↓

IAM Identity Center

↓

Development
Testing
Production
Security
```

One login.

Many accounts.

This is similar to logging into your company's portal once and then accessing different internal systems without signing in again.

---

# Step 13: Permission Sets

Permission Sets are **templates** that define what access a user gets in an AWS account.

Examples:

```
ReadOnly
```

```
Administrator
```

```
EC2Admin
```

```
S3ReadOnly
```

When assigned, Identity Center creates a special IAM role in the target account (its name starts with `AWSReservedSSO_...`).

---

# Step 14: Temporary Credentials (STS)

Imagine you check into a hotel.

You receive a room card.

It works only:

* today
* for your room
* until checkout

After that,

The card stops working.

AWS STS works the same way.

Instead of long-term passwords or access keys,

AWS gives you **temporary credentials**.

Safer.

More secure.

Less risk if stolen.

---

# Step 15: What is a Cross-Account Role?

Suppose:

Development account needs to access Production.

Instead of sharing passwords,

Production creates a **role**.

```
Production

Role

↓

Trust Development Account
```

Development temporarily assumes that role using STS.

No permanent credentials are shared.

This is called a **cross-account role**.

---

# Step 16: Two Important Roles

### 1. `AWSReservedSSO_...`

Created automatically by IAM Identity Center.

Example:

```
AWSReservedSSO_AdministratorAccess_xxxxx
```

Purpose:

Identity Center users log in and assume this role in the target account.

---

### 2. `OrganizationAccountAccessRole`

Created when AWS Organizations creates a new member account (unless configured otherwise).

Purpose:

Allows administrators in the **management account** to assume a role into the member account for administration.

---

# Step 17: Consolidated Billing

Imagine:

```
Dev
$50

Test
$100

Prod
$700

Security
$80
```

Without Organizations,

Everyone pays separately.

With Organizations,

```
Management Account

↓

Total Bill

$930
```

Benefits:

* One invoice
* View costs by linked account
* Easier budgeting and reporting
* Some eligible usage-based pricing benefits can be shared across the organization

---

# Step 18: Best Practices

* Use multiple AWS accounts instead of one large account.
* Keep the management account only for organization administration.
* Group accounts into OUs.
* Test new SCPs on a test OU before applying broadly.
* Don't attach risky deny SCPs directly to the organization Root.
* Keep the default `FullAWSAccess` SCP unless you're intentionally building and testing a strict allow-list model.
* Use IAM Identity Center with groups instead of creating duplicate IAM users.
* Require MFA.
* Prefer temporary STS credentials over long-lived access keys.
* Follow the principle of least privilege.

---

# Complete Flow (Big Picture)

```text
Company
        │
        ▼
AWS Organization
        │
        ▼
Organization Root (top container)
        │
        ├──────────────┐
        ▼              ▼
Development OU     Production OU
        │              │
   Dev Account     Prod Account
        │              │
        └───────┬──────┘
                ▼
        SCP Guardrails
                │
                ▼
IAM Identity Center
                │
         Permission Set
                │
                ▼
AWSReservedSSO Role
                │
                ▼
STS Temporary Session
                │
                ▼
Access AWS Resources

Management Account
        │
        ▼
Consolidated Billing
```

# Exam Tips (High Value)

1. **Organization Root ≠ AWS Root User**. The Organization Root is just the top-level container.
2. **SCPs never grant permissions**; they only limit the maximum permissions.
3. **Explicit Deny in an SCP always overrides an Allow** in IAM policies or permission sets.
4. **SCPs do not apply to the management account**.
5. **IAM Identity Center** is the recommended way to provide workforce access across multiple AWS accounts.
6. **STS provides temporary credentials** by assuming roles.
7. **`AWSReservedSSO_...`** indicates an Identity Center-created role.
8. **`OrganizationAccountAccessRole`** is commonly used by the management account to access member accounts.
9. **Consolidated billing** lets the management account pay for all member accounts while still tracking costs by account.

# Answers to "Check Your Understanding"

1. **Why is the Organization Root different from the AWS root user?**
   The Organization Root is the top container in AWS Organizations. The AWS root user is the highest-privilege user account for a single AWS account.

2. **Does attaching an SCP allow a user to call an AWS API?**
   No. SCPs never grant permissions. IAM policies or Identity Center permission sets must allow the action first.

3. **What happens when a permission set allows an action but an applicable SCP explicitly denies it?**
   The request is denied because an explicit deny in an SCP overrides the allow.

4. **Why should normal workloads stay out of the management account?**
   To reduce risk and keep the account dedicated to organization administration, governance, and billing.

5. **Which role name indicates an Identity Center session?**
   A role whose name starts with **`AWSReservedSSO_`**.
