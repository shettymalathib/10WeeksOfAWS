# Week 2 - Day 4: AWS Organizations, IAM Identity Center & SCPs

## Name

Malathi Shetty

---

## Tasks Completed

- [x] Watched/read the weekly content
- [x] Completed the hands-on lab
- [x] Added screenshots and proof
- [x] Posted on LinkedIn
- [x] Performed cleanup (remaining cleanup pending AWS mandatory waiting period)

---

# Architecture Diagram

### One Shared AWS Account

<img width="1472" height="455" alt="image" src="https://github.com/user-attachments/assets/ef4984ea-9e98-4a76-bc3f-359eb99264a2" />


### AWS Organizations Architecture

<img width="1497" height="1055" alt="image" src="https://github.com/user-attachments/assets/ed9829bc-c0bf-466f-bdef-3923ccc9729f" />


### Permission Flow

<img width="1765" height="1076" alt="image" src="https://github.com/user-attachments/assets/a3c1c5b5-32c1-44ea-a7b5-b27a40fc5576" />


---

## Topics Practiced

- AWS Organizations
- Organization Root
- Management Account
- Member Account
- Organizational Units (OU)
- Service Control Policies (SCP)
- IAM Identity Center
- Permission Sets
- AWS STS Temporary Credentials
- Consolidated Billing

---

# What I Built

I created an AWS Organization with a Management Account and a Member Account to understand how AWS centrally manages multiple AWS accounts.

During this lab, I:

- Created a Dev-Env Organizational Unit (OU).
- Created a `Deny-S3-Bucket-Creation` Service Control Policy (SCP).
- Attached the SCP to the Dev-Env OU.
- Enabled IAM Identity Center.
- Created the `cloudadhar-demo` user.
- Created the `CloudAdhar-Admin` Permission Set.
- Assigned the Permission Set to both AWS accounts.
- Verified the active IAM Identity Center temporary session.
- Successfully created an Amazon S3 bucket before the SCP applied.
- Moved the Member Account into the Dev-Env OU.
- Verified that Amazon S3 bucket creation failed because of the SCP.
- Reviewed AWS Organizations consolidated billing.

---

# Part 1 – Review AWS Organization

## AWS Organization

AWS Organizations lets us centrally manage multiple AWS accounts from one place.

## Organization Root

The Organization Root is the top-level container in AWS Organizations.

It is **not** the same as the AWS Root User.

## Management Account

The Management Account manages the AWS Organization.

Responsibilities include:

- Managing AWS Organizations
- Creating Organizational Units (OUs)
- Managing IAM Identity Center
- Applying Service Control Policies (SCPs)
- Managing consolidated billing

## Member Account

A Member Account is where applications and workloads run.

It inherits policies attached through Organizational Units.

## Organizational Unit (OU)

An Organizational Unit groups AWS accounts together.

Accounts inside an OU inherit the policies attached to that OU.

## AWS Organization Structure

<img width="2147" height="686" alt="image" src="https://github.com/user-attachments/assets/30600bfd-80d7-4a16-9125-4bb4d68bda1f" />


## Dev-Env SCP Attached

<img width="2560" height="1481" alt="image" src="https://github.com/user-attachments/assets/661c712e-c6e6-46c5-aa33-2c5cd3b0dd7c" />


**Result**

- Verified the Management Account and Member Account.
- Confirmed the Dev-Env Organizational Unit exists.
- Verified that the `Deny-S3-Bucket-Creation` SCP is attached to the Dev-Env OU.

---

# Part 2 – Verify IAM Identity Center Access

## Why IAM Identity Center?

IAM Identity Center provides:

- Single Sign-On (SSO)
- Temporary credentials
- Centralized user management
- MFA support
- No long-term access keys
- No duplicate IAM users across multiple AWS accounts

## Permission Set vs SCP

| Permission Set | SCP |
|---------------|-----|
| Grants permissions | Does not grant permissions |
| Used by IAM Identity Center | Used by AWS Organizations |
| Assigned to users or groups | Attached to the Organization Root or an OU |
| Allows actions | Limits the maximum permissions |

## STS Identity

<img width="2552" height="555" alt="image" src="https://github.com/user-attachments/assets/80b4f2dd-b0c5-427f-9002-a78d08c0892d" />


> **Note**
>
> The CloudShell environment for the Member Account could not be created because the new AWS account was still undergoing AWS verification.
>
> Therefore, I verified the active IAM Identity Center session using the AWS Access Portal and AWS Console instead.
>
> The SCP behavior was still successfully demonstrated by confirming the **AccessDenied** error after moving the Member Account into the **Dev-Env** OU.

**Result**

- Successfully signed in through IAM Identity Center.
- Verified access to both AWS accounts.
- Confirmed the active IAM Identity Center session.

---

# Part 3 – Test Before SCP Applies

Before moving the Member Account into the Dev-Env OU:

- The Member Account was directly under the Organization Root.
- The SCP was not applied.
- The Permission Set allowed `s3:CreateBucket`.

## S3 Bucket Created Before SCP

<img width="2532" height="721" alt="image" src="https://github.com/user-attachments/assets/e04292f6-1e9d-489f-96c5-5f2bb72c6610" />


**Result**

✅ Amazon S3 bucket creation succeeded because the Member Account had not yet inherited the SCP.

---

# Part 4 – Move Member Account into Dev-Env

The Member Account was moved from the Organization Root into the Dev-Env Organizational Unit.

Once moved, it inherited the attached `Deny-S3-Bucket-Creation` Service Control Policy.

## Member Account Moved into Dev-Env

<img width="2113" height="811" alt="image" src="https://github.com/user-attachments/assets/01908a4d-4fb5-4e79-b108-d925f011a49d" />

<img width="2202" height="1005" alt="image" src="https://github.com/user-attachments/assets/9a9cf722-f2ed-4157-9702-faa8285e5f8e" />


**Result**

The Member Account now inherits the SCP attached to the Dev-Env OU.

---

# Part 5 – Test After SCP Applies

## Why is an SCP a Guardrail?

A Service Control Policy (SCP) **does not grant permissions**.

Instead, it defines the **maximum permissions** available to IAM users and IAM roles inside member accounts.

Even if IAM allows an action, an Explicit Deny in an SCP overrides it.

## Before vs After SCP

| Before | After |
|---------|-------|
| Member Account under Root | Member Account inside Dev-Env OU |
| SCP not applied | SCP applied |
| Permission Set allowed bucket creation | Permission Set still allowed bucket creation |
| Bucket creation succeeded | Bucket creation failed |

## SCP Access Denied

<img width="2555" height="1256" alt="image" src="https://github.com/user-attachments/assets/87c17507-2619-46f1-a462-01c1ca359cba" />


**Result**

The bucket creation request failed with:

- **AccessDenied**
- Explicit deny in a Service Control Policy
- `s3:CreateBucket` operation denied

This demonstrates that an Explicit Deny in an SCP overrides permissions granted through the `CloudAdhar-Admin` Permission Set.

---

# Part 6 – Review Consolidated Billing

The Management Account pays the AWS bill for all member accounts.

AWS Cost Explorer can group costs by linked account, making it easier to monitor spending across the organization.

## Consolidated Billing

<img width="1896" height="891" alt="image" src="https://github.com/user-attachments/assets/daad5b7c-a5d1-4bbd-9136-3aa9254c3341" />


---

# What I Learned

- AWS Organizations helps manage multiple AWS accounts from one place.
- Organization Root is different from the AWS Root User.
- Organizational Units (OUs) help group accounts and apply common security policies.
- Service Control Policies (SCPs) define the maximum permissions available but do not grant permissions.
- An Explicit Deny in an SCP always overrides permissions granted by IAM policies or Permission Sets.
- IAM Identity Center provides secure access using temporary STS credentials instead of long-term access keys.
- A Permission Set is different from an IAM Role. AWS automatically creates an `AWSReservedSSO_...` IAM Role when a Permission Set is assigned.
- Consolidated Billing allows the Management Account to pay for all member accounts while still tracking costs separately.

---

# Where I Got Stuck

CloudShell could not be created in the Member Account because the new AWS account was still undergoing AWS verification.

To continue the lab, I verified the IAM Identity Center session through the AWS Access Portal and AWS Console and successfully confirmed the SCP behavior by reproducing the **AccessDenied** error after moving the Member Account into the Dev-Env OU.

---

# Cleanup

The following cleanup steps were completed:

- Deleted the temporary Amazon S3 bucket created during testing.
- Completed customer verification for the Member Account.
- Deleted the IAM Identity Center user (`cloudadhar-demo`).
- Deleted the `CloudAdhar-Admin` Permission Set.
- Deleted the `Deny-S3-Bucket-Creation` Service Control Policy (SCP).
- Deleted the `Dev-Env` Organizational Unit (OU).

The following cleanup is pending because AWS enforces a mandatory waiting period before a newly verified member account can leave an organization:

- Remove the Member Account from the AWS Organization.
- Delete the AWS Organization.

## Cleanup Screenshots

### Bucket Deleted

<img width="2382" height="737" alt="image" src="https://github.com/user-attachments/assets/39ce6178-ce6b-41cb-a356-80593b3fc8a1" />


### IAM Identity Center User Deleted

<img width="2552" height="851" alt="image" src="https://github.com/user-attachments/assets/295cb560-300c-407f-b063-f4c137e79a39" />


### Permission Set Deleted

<img width="2551" height="921" alt="image" src="https://github.com/user-attachments/assets/af941d36-0d8a-40d2-8587-85bc7dd18204" />


### SCP Deleted

<img width="2356" height="1040" alt="image" src="https://github.com/user-attachments/assets/b0589b94-c00c-4090-b5b7-5ceadd67fba4" />


### Organizational Unit Deleted

<img width="2415" height="857" alt="image" src="https://github.com/user-attachments/assets/c806c556-5d2d-4097-ba8d-84583cd3ea99" />


### Waiting Period Before Leaving the Organization

<img width="2432" height="1102" alt="image" src="https://github.com/user-attachments/assets/722b7723-e4ed-40ab-b65d-7c85a37676fb" />


---



# 1. Organization, Root, Management Account, Member Account & OU

```markdown
# Part 1 – Review AWS Organization

## AWS Organization

AWS Organizations is an AWS service that lets us centrally manage multiple AWS accounts from one place. It helps improve security, governance, and billing management.

## Organization Root

The Organization Root is the top-level container in AWS Organizations. It organizes AWS accounts and Organizational Units (OUs).

**Note:** The Organization Root is different from the AWS Root User.

## Management Account

The Management Account is the primary account in an AWS Organization. It creates member accounts, manages Organizational Units (OUs), attaches Service Control Policies (SCPs), manages IAM Identity Center, and pays the consolidated AWS bill.

## Member Account

A Member Account is where AWS resources such as EC2, S3, Lambda, and RDS are deployed. It inherits policies attached through its Organizational Unit.

## Organizational Unit (OU)

An Organizational Unit (OU) is used to group AWS accounts together. Policies attached to an OU automatically apply to every member account inside that OU.
```

---

# 2. Why an SCP is a Guardrail Rather Than a Permission Grant

```markdown
## Why is an SCP a Guardrail?

A Service Control Policy (SCP) does not grant permissions.

Instead, it defines the maximum permissions available to IAM users and IAM roles inside member accounts.

Even if an IAM policy or Permission Set allows an action, an Explicit Deny in an SCP overrides it.

This is why an SCP is called a **guardrail**—it limits what accounts can do but never grants permissions by itself.
```

---

# 3. Why IAM Identity Center is Preferred Over Duplicate IAM Users

```markdown
## Why IAM Identity Center?

IAM Identity Center provides secure access to multiple AWS accounts using a single sign-on experience.

Instead of creating separate IAM users in every AWS account, users sign in once and receive temporary AWS credentials.

Benefits include:

- Single Sign-On (SSO)
- Centralized user management
- Temporary STS credentials
- MFA support
- No long-term access keys
- No duplicate IAM users across multiple AWS accounts
```

---

# 4. Difference Between a Permission Set and an SCP

```markdown
## Permission Set vs SCP

| Permission Set | SCP |
|---------------|-----|
| Grants permissions | Does not grant permissions |
| Used by IAM Identity Center | Used by AWS Organizations |
| Assigned to users or groups | Attached to the Organization Root or an OU |
| Defines what a user can do | Defines the maximum permissions allowed |
```

---

# 5. Why S3 Creation Worked Before the Account Move and Failed Afterwards

```markdown
## Before vs After SCP

### Before Moving the Member Account

- The Member Account was directly under the Organization Root.
- The SCP was not applied.
- The Permission Set allowed `s3:CreateBucket`.

**Result**

✅ Amazon S3 bucket creation succeeded.

### After Moving the Member Account

- The Member Account was moved into the Dev-Env Organizational Unit.
- The SCP became applicable.
- The Permission Set still allowed `s3:CreateBucket`.

**Result**

❌ Amazon S3 bucket creation failed with **AccessDenied** because the Service Control Policy explicitly denied the action.

This demonstrates that an Explicit Deny in an SCP overrides permissions granted through IAM Identity Center Permission Sets.
```

---

# 6. What You Learned About Consolidated Billing

```markdown
## Consolidated Billing

The Management Account pays the AWS bill for all member accounts in the organization.

AWS Cost Explorer can group costs by linked account, making it easier to monitor and manage spending for each account while receiving a single consolidated bill.
```





---

## LinkedIn Post

Add your LinkedIn post link here.
