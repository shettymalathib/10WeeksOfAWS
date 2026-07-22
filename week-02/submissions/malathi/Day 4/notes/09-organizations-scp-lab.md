# 📌 Day 4 – AWS Organizations, Organizational Units (OU), Service Control Policies (SCP) & IAM Identity Center

# 🎯 Today's Goal

Learn how AWS helps companies:

* Manage multiple AWS accounts from one place.
* Separate environments like Development, Testing, and Production.
* Apply centralized security controls.
* Provide secure workforce access using IAM Identity Center.
* Manage billing for all AWS accounts from one management account.

---

# 🌍 The One-Account Problem

Imagine a company uses **only one AWS account**.

Inside that account, everyone works together:

* 👨‍💻 Development Team
* 🧪 Testing Team
* 🚀 Production Team
* 🔒 Security Team
* 💰 Finance Team

Everyone shares the same AWS account.

This creates several problems.

### Problem 1 – Shared Permissions

A developer might accidentally change or delete production resources.

Example:

A developer deletes an S3 bucket that the production application uses.

Production stops working.

---

### Problem 2 – Large Blast Radius

**Blast Radius** means:

> The amount of damage caused by one mistake.

If everyone uses one AWS account, one accidental mistake can affect the entire company.

Example:

Deleting an IAM role or VPC could impact every environment.

---

### Problem 3 – Mixed Billing

All AWS charges appear in one account.

It becomes difficult to answer questions like:

* Which team spent the money?
* Which application created these resources?
* Which environment costs the most?

---

### Problem 4 – Difficult Governance

Different teams require different permissions.

Example:

* Developers should manage Development resources.
* Testers should manage Testing resources.
* Only Operations should manage Production.

Managing all these permissions inside one AWS account becomes difficult.

---

# ✅ AWS Solution – Multiple AWS Accounts

Instead of using one AWS account, AWS recommends creating **multiple AWS accounts**.

Example:

```text
AWS Organization
│
├── Management Account
│
├── Development Account
│
├── Testing Account
│
├── Production Account
│
└── Security Account
```

Each account is isolated from the others.

If something goes wrong in one account, it usually does not affect the other accounts.

This reduces the blast radius and improves security.

---

# Why Use Multiple AWS Accounts?

Using multiple AWS accounts provides several benefits.

### 1. Environment Isolation

Keep Development, Testing, and Production separate.

Example:

```text
Development
      │
Testing
      │
Production
```

A mistake in Development will not directly affect Production.

---

### 2. Better Security

Each account has its own users, roles, permissions, and resources.

Compromising one account does not automatically give access to every account.

---

### 3. Smaller Blast Radius

Mistakes remain limited to one AWS account instead of affecting the whole organization.

---

### 4. Better Cost Tracking

Each AWS account has its own resources.

AWS Cost Explorer can show spending per account.

This makes it easy to understand where money is being spent.

---

### 5. Better Governance

Different environments can have different security policies.

Example:

Production may have stricter security controls than Development.

---

# 🏢 AWS Organizations

AWS Organizations is an AWS service that helps you centrally manage multiple AWS accounts.

Think of it as the **head office** of a company.

It allows you to:

* Create member accounts.
* Organize accounts.
* Apply security policies.
* Manage billing centrally.
* Manage access across multiple AWS accounts.

---

## Example

```text
AWS Organization
│
├── Management Account
├── Dev Account
├── Test Account
└── Production Account
```

Instead of managing every account separately, everything is managed from AWS Organizations.

---

# 🌳 Organization Root

The **Organization Root** is the top-level container inside AWS Organizations.

Example:

```text
AWS Organization
        │
        ▼
      Root
```

Every AWS account belongs somewhere under the Organization Root.

---

## ⚠️ Organization Root vs AWS Root User

Many beginners confuse these two concepts.

They are completely different.

| Organization Root                        | AWS Root User                                     |
| ---------------------------------------- | ------------------------------------------------- |
| Top-level container in AWS Organizations | First user created using the AWS account email    |
| Used to organize AWS accounts            | Has full administrative access to one AWS account |
| Not a login user                         | Can sign in using email and password              |

**Remember:**

**Organization Root ≠ AWS Root User**

---

# 👑 Management Account

Every AWS Organization has one **Management Account**.

It is the main account that manages the organization.

Responsibilities include:

* Creating member accounts.
* Creating Organizational Units (OUs).
* Creating and attaching SCPs.
* Managing IAM Identity Center.
* Managing organization settings.
* Paying the AWS bill for all member accounts.

---

## Best Practice

Do **not** run normal application workloads in the Management Account.

Instead, use it only for:

* Organization administration
* Security management
* Billing
* Identity Center administration

Applications should run inside Member Accounts.

> Important: Perform this practical only in a dedicated learning or sandbox AWS Organization. Do not test SCPs in a production organization because they can unintentionally block required operations.

---

# 👤 Member Account

A **Member Account** is an AWS account that belongs to an AWS Organization.

It is where applications and workloads normally run.

Examples of workloads:

* EC2
* S3
* RDS
* Lambda
* VPC

Many organizations create separate accounts for:

* Development
* Testing
* Production
* Security
* Shared Services

The exact structure depends on the organization's needs.

---

# 📂 Organizational Unit (OU)

An **Organizational Unit (OU)** is a logical group of AWS accounts.

Think of an OU like a folder.

Instead of applying policies individually to every AWS account, you apply policies to the OU.

Every account inside that OU inherits those policies.

---

## Example

```text
Root
│
├── Dev-Env
│      ├── Dev Account
│      └── Test Account
│
├── Production
│      └── Prod Account
│
└── Security
       └── Security Account
```

---

## Common Ways to Organize OUs

Organizations commonly group accounts by:

### Environment

```text
Development
Testing
Production
```

---

### Business Function

```text
Security
Shared Services
Applications
Networking
```

---

### Compliance Requirement

Example:

```text
PCI
HIPAA
General Workloads
```

---

### Business Unit

Example:

```text
Finance
Marketing
HR
Engineering
```

---

# Moving an Account Between OUs

An AWS account can be moved from one OU to another.

When it moves:

* The account starts inheriting policies from the new OU.
* Policies from the previous OU no longer apply.
* Parent OU policies continue to apply.

Example:

```text
Before

Root
│
├── CloudAdhar-Dev

SCP Not Applied

------------------------

After

Root
│
└── Dev-Env
       │
       └── CloudAdhar-Dev

SCP Applied
```

---

# 🛡️ Service Control Policy (SCP)

A **Service Control Policy (SCP)** is a policy used in AWS Organizations.

Its purpose is to define the **maximum permissions** available to users and roles in member accounts.

---

## Important

An SCP **does NOT grant permissions.**

It only limits permissions.

Think of it like a company rule.

Example:

Company Rule:

> Employees are not allowed to create new S3 buckets.

Even if an employee has Administrator permissions, the company rule can still block that action.

---

# Maximum Permission Boundary

Think of permissions like this:

```text
IAM Policy / Permission Set
        │
        ▼
Requests Permission

        │

SCP checks maximum allowed permission

        │

Allowed or Denied
```

---

# Permission Evaluation Formula

AWS evaluates permissions like this:

```text
Effective Permission =

Identity Policy OR Permission Set Allows

AND

Applicable SCP Allows

AND

No Explicit Deny
```

For an action to succeed:

* IAM Policy (or Permission Set) must allow it.
* Applicable SCP must also allow it.
* There must not be an Explicit Deny.

If any SCP explicitly denies the action, the final result is **Denied**.

---

# Example SCP

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyS3BucketCreation",
      "Effect": "Deny",
      "Action": "s3:CreateBucket",
      "Resource": "*"
    }
  ]
}
```

This SCP prevents:

```text
s3:CreateBucket
```

for all affected member accounts.

***


# 🔓 AWS Managed SCP – FullAWSAccess

When you create an AWS Organization, AWS automatically attaches an AWS-managed Service Control Policy called:

```text
FullAWSAccess
```

This policy allows all AWS services and actions.

Think of it as saying:

> "Do not restrict anything."

It is attached by default to the Organization Root and inherited by all accounts unless you change it.

---

## Why Keep FullAWSAccess?

Many beginners think they should delete it after creating their own SCP.

**Don't.**

AWS recommends keeping `FullAWSAccess` unless you have carefully designed and tested an **allow-list** strategy.

Example:

```text
FullAWSAccess
        +
Deny-S3-Bucket-Creation
        =
Everything is allowed
EXCEPT creating S3 buckets.
```

This means:

* Most AWS services continue working normally.
* Only the actions explicitly denied by your custom SCP are blocked.

---

# ⚠️ Important SCP Rules

## Rule 1 – SCPs Do NOT Grant Permissions

An SCP only defines the maximum permissions available.

It never gives permissions by itself.

Example:

```text
IAM Policy

No Permission

+

SCP

Allows Everything

=

Final Result

No Permission
```

Since the IAM policy doesn't allow the action, the user still cannot perform it.

---

## Rule 2 – Explicit Deny Always Wins

If any applicable SCP explicitly denies an action, AWS blocks it immediately.

Example:

```text
Permission Set

Allow CreateBucket

+

SCP

Deny CreateBucket

=

Final Result

Denied
```

The Deny always overrides the Allow.

---

## Rule 3 – SCPs Do NOT Affect the Management Account

SCPs only apply to **Member Accounts**.

They do **not** restrict users or roles in the **Management Account**.

This allows administrators to continue managing the organization even if restrictive SCPs are attached to member accounts.

---

## Rule 4 – Don't Attach Test SCPs to the Organization Root

The Organization Root affects every member account in the organization.

If you attach a restrictive SCP directly to the Root, every member account inherits it.

Instead:

1. Create a separate OU (for example, `Dev-Env`).
2. Attach the SCP to that OU.
3. Test it safely before applying it more broadly.

---

# 👥 IAM Identity Center

**IAM Identity Center** (formerly AWS Single Sign-On) allows employees to access multiple AWS accounts and applications using one login.

Instead of creating IAM users in every AWS account, users sign in through a central access portal.

AWS then provides temporary credentials.

---

## Why Use IAM Identity Center?

Benefits include:

* Single sign-on (SSO)
* Centralized user management
* Multi-Factor Authentication (MFA)
* Temporary credentials
* No need for long-term access keys
* Easier access to multiple AWS accounts

---

## How It Works

```text
Employee
     │
     ▼
IAM Identity Center
     │
     ▼
Permission Set
     │
     ▼
Temporary IAM Role
     │
     ▼
AWS Account
```

The employee signs in once and can access the AWS accounts assigned to them.

---

# 👤 Users and Groups

IAM Identity Center allows you to create:

* Users
* Groups

### Best Practice

Instead of assigning permissions to individual users, assign them to groups.

Example:

```text
Developers Group
        │
        ▼
CloudAdhar-Admin Permission Set
```

When a new developer joins, simply add them to the group.

---

# 🔐 Multi-Factor Authentication (MFA)

AWS recommends enabling MFA for Identity Center users.

MFA requires:

* Password
* One-time verification code

This provides additional security if a password is compromised.

---

# 📄 Permission Set

A **Permission Set** is a template containing AWS permissions.

Example:

```text
CloudAdhar-Admin
```

A Permission Set is **not** an IAM Role.

It simply defines which permissions a user should receive.

---

## How Permission Sets Work

Example:

```text
cloudadhar-demo
        │
        ▼
CloudAdhar-Admin Permission Set
        │
        ▼
CloudAdhar-Dev Account
```

When the user selects the account from the AWS access portal, When a Permission Set is assigned to an AWS account, IAM Identity Center provisions an IAM role named AWSReservedSSO_... in that account. When the user signs in through the AWS access portal, they assume that role using temporary STS credentials.

---

# ⚠️ Permission Set vs IAM Role

Many beginners confuse these two concepts.

They are different.

| Permission Set                 | IAM Role                                    |
| ------------------------------ | ------------------------------------------- |
| Template of permissions        | AWS IAM Role created inside the AWS account |
| Created in IAM Identity Center | Automatically created by AWS                |
| Assigned to users or groups    | Assumed by users during sign-in             |

Example:

```text
Permission Set

CloudAdhar-Admin

        │

AWS automatically creates

        ▼

AWSReservedSSO_CloudAdhar-Admin_xxxxxxxxx
```

So:

* **CloudAdhar-Admin** = Permission Set
* **AWSReservedSSO_CloudAdhar-Admin_xxxxx** = IAM Role

---

# 🏷️ AWSReservedSSO Role

When a user signs in through IAM Identity Center, AWS automatically creates a role like:

```text
AWSReservedSSO_CloudAdhar-Admin_cb8ad60bf4027245
```

The exact suffix is generated automatically.

Users temporarily assume this role while working in the AWS account.

---

# 🔑 STS (Security Token Service)

AWS STS provides **temporary security credentials**.

Instead of permanent access keys, AWS generates credentials that expire automatically.

Temporary credentials are more secure because they:

* Expire automatically.
* Reduce the risk of leaked credentials.
* Do not require storing long-term access keys.

> STS returns temporary security credentials consisting of an Access Key ID, Secret Access Key, Session Token, and an Expiration time. AWS automatically refreshes these credentials for Identity Center sessions while the session is active.

---

# Temporary STS Session

When a user logs in through IAM Identity Center:

```text
cloudadhar-demo
        │
        ▼
Permission Set
        │
        ▼
AWSReservedSSO Role
        │
        ▼
STS Temporary Session
```

AWS automatically refreshes the temporary credentials as needed.

---

# Why Run `aws sts get-caller-identity`?

Before making changes in AWS, always verify your current identity.

Run:

```bash
aws sts get-caller-identity
```

This command tells you:

* Which AWS account you are using.
* Which IAM role you are using.
* Which identity AWS recognizes.

Example output:

```text
Account:
530310463208

Role:
AWSReservedSSO_CloudAdhar-Admin_xxxxx

User:
cloudadhar-demo
```

This is especially important in multi-account environments to avoid making changes in the wrong account.

---

# 🔄 Cross-Account Access

IAM Identity Center allows one user to access multiple AWS accounts.

Example:

```text
cloudadhar-demo
        │
        ▼
IAM Identity Center
        │
        ▼
CloudAdhar-Admin Permission Set
        │
        ├────────► Management Account
        │
        └────────► CloudAdhar-Dev
```

The user signs in once and selects the desired AWS account.

AWS provides a separate temporary session for that account.

---

# Two Cross-Account Role Patterns

You may see two common IAM roles in AWS Organizations.

### 1. AWSReservedSSO Role

Example:

```text
AWSReservedSSO_CloudAdhar-Admin_xxxxx
```

Purpose:

* Created automatically by IAM Identity Center.
* Used when users log in through the AWS access portal.

---

### 2. OrganizationAccountAccessRole

Purpose:

* Created when a member account is created through AWS Organizations.
* Allows administrators in the Management Account to access the Member Account.

Both roles use temporary STS credentials.

Neither requires permanent access keys.

---

# 💰 Consolidated Billing

AWS Organizations provides **Consolidated Billing**.

This means:

The **Management Account** receives one AWS bill for all member accounts.

Benefits include:

* Single monthly bill.
* Centralized payment.
* View spending by linked account.
* Easier cost tracking.
* Eligible usage-based pricing benefits may be shared across accounts.

---

## Cost Explorer

AWS Cost Explorer can group costs by:

```text
Linked Account
```

This allows you to see how much each member account is spending.

Note:

New accounts or newly created resources may take some time before they appear in Cost Explorer.

---

# Best Practices

AWS recommends the following:

* Use multiple AWS accounts instead of one shared account.
* Keep application workloads out of the Management Account.
* Use Organizational Units (OUs) to group similar accounts.
* Test new SCPs in a dedicated OU before applying them broadly.
* Keep `FullAWSAccess` unless you have designed a tested allow-list model.
* Use Groups instead of assigning permissions directly to individual users.
* Enable MFA for Identity Center users.
* Use least-privilege Permission Sets whenever possible.
* Use temporary STS credentials instead of permanent access keys.
* Always verify your identity with `aws sts get-caller-identity` before making changes.

***

Before starting the lab, ensure no previous test S3 bucket exists to avoid conflicts with the new bucket creation test.

# 🧪 Practical Goal

The goal of this practical is to prove that:

* An S3 bucket **can be created** before an SCP applies.
* The **same action fails** after the account moves into an OU where the SCP is attached.
* The Identity Center user and permission set remain the same. Only the inherited SCP changes.

---

# Lab Setup

Before starting the practical, the following resources were created.

### AWS Organization

```text
Organization
│
├── Management Account
│
└── CloudAdhar-Dev (Member Account)
```

---

### Organizational Unit (OU)

```text
Dev-Env
```

Initially, this OU is empty.

---

### Service Control Policy (SCP)

Created a customer-managed SCP:

```text
Deny-S3-Bucket-Creation
```

Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyS3BucketCreation",
      "Effect": "Deny",
      "Action": "s3:CreateBucket",
      "Resource": "*"
    }
  ]
}
```

This SCP is attached to the **Dev-Env OU**.

---

### IAM Identity Center

Created:

```text
User

cloudadhar-demo
```

Created Permission Set:

```text
CloudAdhar-Admin
```

Assigned the permission set to:

* Management Account
* CloudAdhar-Dev

Enabled MFA for the Identity Center user.

---

# Part 1 – Review the Organization

Open:

```text
AWS Organizations
```

Confirm:

```text
Root
│
├── Management Account
│
├── CloudAdhar-Dev
│
└── Dev-Env (Empty)
```

Verify:

* The Management Account is under Root.
* CloudAdhar-Dev is directly under Root.
* Dev-Env exists but is empty.
* `FullAWSAccess` is attached.
* `Deny-S3-Bucket-Creation` is attached to Dev-Env.

### Why?

Because the SCP only applies to accounts inside the OU.

Since CloudAdhar-Dev is still under Root, the SCP does not affect it yet.

---

# Part 2 – Verify Identity Center Session

Sign in through the AWS Access Portal using:

```text
cloudadhar-demo
```

Select:

```text
CloudAdhar-Dev
```

using the Permission Set:

```text
CloudAdhar-Admin
```

Open CloudShell and run:

```bash
aws sts get-caller-identity
```

Example:

```json
{
  "Account": "530310463208",
  "Arn": "arn:aws:sts::530310463208:assumed-role/AWSReservedSSO_CloudAdhar-Admin_xxxxx/cloudadhar-demo"
}
```

### Why run this command?

It confirms:

* Which AWS account you are using.
* Which IAM role is active.
* Which Identity Center user is signed in.

Always verify your identity before making changes in a multi-account environment.

> For screenshots, use an Incognito window and mask the AWS Account ID and the full ARN before submitting.

---

# Part 3 – Test Before the SCP Applies

Set the AWS Region:

```bash
export AWS_REGION=us-west-2
```

Get the Account ID:

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
```

Create a unique bucket name:

```bash
BUCKET="cloudadhar-before-scp-${ACCOUNT_ID}-$(date +%s)"
```

Display the bucket name:

```bash
echo "$BUCKET"
```

Create the bucket:

```bash
aws s3api create-bucket \
  --bucket "$BUCKET" \
  --region "$AWS_REGION" \
  --create-bucket-configuration LocationConstraint="$AWS_REGION"
```

Verify the bucket exists:

```bash
aws s3api head-bucket --bucket "$BUCKET"
```

### Expected Result

Bucket creation succeeds.

### Why?

Because:

* The Permission Set allows the action.
* The account is still under Root.
* The SCP has not been inherited.

Permission evaluation:

```text
Permission Set → Allow

SCP → Not Applied

Final Result → Allow
```

Delete the bucket:

```bash
aws s3api delete-bucket --bucket "$BUCKET" --region "$AWS_REGION"
```

Verify deletion:

```bash
aws s3api head-bucket --bucket "$BUCKET"
```

Expected result:

```text
404 Not Found
```

This confirms the bucket was successfully deleted.

---

# Part 4 – Move the Account into Dev-Env

Return to the Management Account.

Open:

```text
AWS Organizations
```

Select:

```text
CloudAdhar-Dev
```

Choose:

```text
Actions

↓

Move
```

Move it into:

```text
Dev-Env
```

Now the organization looks like:

```text
Root
│
├── Management Account
│
└── Dev-Env
      │
      └── CloudAdhar-Dev
```

### Why?

Once the account is inside Dev-Env, it inherits all SCPs attached to that OU.

> Important: Choose Move only. Never choose Close Account or Remove from Organization, as those actions have completely different purposes.

---

# Part 5 – Test After the SCP Applies

Return to the CloudAdhar-Dev session.

Run:

```bash
aws sts get-caller-identity
```

The identity should remain the same.

Attempt to create another bucket.

Example:

```bash
aws s3api create-bucket \
  --bucket cloudadhar-after-scp-20260721 \
  --region us-west-2 \
  --create-bucket-configuration LocationConstraint=us-west-2
```

### Expected Result

AWS returns:

```text
AccessDenied

with an explicit deny in a service control policy
```

### Why?

The user did not change.

The Permission Set did not change.

The IAM Role did not change.

The only change was that the account now inherits the SCP from Dev-Env.

Permission evaluation:

```text
Permission Set

Allow CreateBucket

+

SCP

Explicit Deny CreateBucket

=

Final Result

Denied
```

This demonstrates that an Explicit Deny in an SCP overrides even an Administrator permission set.

---

# Part 6 – Review Consolidated Billing

Return to the Management Account.

Open:

```text
Billing and Cost Management
```

Review:

* Bills
* Cost Explorer

Group costs by:

```text
Linked Account
```

This allows you to view spending separately for each member account, even though the Management Account receives the combined bill.

Note:

New accounts or recent activity may take some time to appear in Cost Explorer.

> Note: 169.254.169.254 is the EC2 Instance Metadata Service (IMDS) endpoint. This lab does not use EC2 metadata. Instead, aws sts get-caller-identity is used to verify the current STS session.

---

# Troubleshooting

## Bucket creation fails before moving the account

Check:

* CloudAdhar-Dev is still under Root.
* The bucket name is globally unique.
* The correct Region and `LocationConstraint` are used.
* The Permission Set includes the required permissions.

---

## Bucket creation succeeds after moving the account

Check:

* The SCP is attached to Dev-Env.
* CloudAdhar-Dev is actually inside Dev-Env.
* The SCP has propagated (this can take a short time).

---

## Identity Center shows no AWS accounts

Verify:

* The user exists.
* The Permission Set is assigned.
* Account assignments are provisioned.
* The user is signing in to the correct AWS access portal.

---

## CloudShell cannot be created

New AWS accounts may display:

```text
Your account verification is in progress.
```

CloudShell may not be available until AWS completes account verification.

In this case, you can still:

* Verify the AWS Organization.
* Test the SCP through the AWS Console.
* Capture the required screenshots.
* Retry CloudShell later after verification is complete.

---

# Cleanup

After completing the practical:

* Delete any test S3 buckets created during the lab.
* Remove temporary resources if they are no longer needed.
* Keep the AWS Organization if you will continue with future labs.
* Do not remove `FullAWSAccess` unless you are intentionally implementing a tested allow-list strategy.

---

# Interview Questions

### What is AWS Organizations?

AWS Organizations is a service that centrally manages multiple AWS accounts, allowing administrators to organize accounts, apply policies, manage access, and consolidate billing.

---

### What is an Organizational Unit (OU)?

An Organizational Unit is a logical container that groups AWS accounts so they can inherit the same Service Control Policies and administrative controls.

---

### What is a Service Control Policy (SCP)?

An SCP defines the maximum permissions available to IAM users and IAM roles in member accounts. It does not grant permissions; it only limits them.

---

### Does an SCP grant permissions?

No.

Permissions must first be allowed by an IAM policy or Permission Set. An SCP only limits those permissions.

---

### What happens if an IAM policy allows an action but an SCP explicitly denies it?

The request is denied because an Explicit Deny always overrides an Allow.

---

### Do SCPs affect the Management Account?

No.

SCPs apply only to Member Accounts.

---

### What is IAM Identity Center?

IAM Identity Center provides centralized workforce access to multiple AWS accounts and applications using single sign-on (SSO) and temporary credentials.

---

### What is a Permission Set?

A Permission Set is a template of permissions created in IAM Identity Center. When assigned to a user or group, AWS automatically creates an `AWSReservedSSO_...` IAM role in the target AWS account.

---

### Why use `aws sts get-caller-identity`?

It verifies the current AWS account, IAM role, and identity before making changes, helping prevent mistakes in multi-account environments.

---

# What I Learned Today

* AWS Organizations centrally manages multiple AWS accounts.
* The Organization Root is a hierarchy container and is different from the AWS Root User.
* The Management Account manages the organization and pays the consolidated bill.
* Member Accounts are used to run workloads.
* Organizational Units (OUs) group accounts with common policies.
* Service Control Policies (SCPs) define the maximum permissions available to member accounts but do not grant permissions.
* An Explicit Deny in an SCP overrides permissions granted by IAM policies or Permission Sets.
* `FullAWSAccess` is attached by default and should generally be kept unless implementing a tested allow-list strategy.
* IAM Identity Center provides secure single sign-on using temporary STS credentials.
* A Permission Set is different from an IAM Role; AWS automatically creates an `AWSReservedSSO_...` role when a Permission Set is assigned.
* Running `aws sts get-caller-identity` before making changes confirms the current account and role.
* Consolidated Billing allows the Management Account to pay for all member accounts while still tracking costs by linked account.
* Moving an account into an OU causes it to inherit the OU's SCPs, which can immediately change what actions are allowed or denied.

***

