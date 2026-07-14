# Day 5 – GitHub OIDC Challenge

---

# Lab Overview

In traditional CI/CD pipelines, AWS credentials are often stored as GitHub Secrets:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

Although GitHub encrypts these secrets, they are still **long-lived credentials**. If they are accidentally exposed, an attacker may be able to access your AWS account until those credentials are rotated or deleted.

AWS and GitHub recommend using **OpenID Connect (OIDC)** instead.

With OIDC, GitHub does not store AWS credentials. Instead, it requests a temporary identity token whenever a workflow runs. AWS validates this token and issues short-lived credentials through **AWS Security Token Service (AWS STS)**.

This approach significantly reduces the risk associated with credential leakage.

---

# What is OpenID Connect (OIDC)?

**OpenID Connect (OIDC)** is an identity layer built on top of OAuth 2.0.

It enables applications to verify the identity of a user or service without exchanging passwords or long-lived credentials.

In this lab:

* GitHub acts as the **Identity Provider (IdP)**.
* AWS IAM trusts GitHub's identity.
* AWS STS issues temporary credentials after successful verification.

Because the credentials are temporary, they automatically expire after a short period, making them much more secure than permanent access keys.

---

# Why Use OIDC Instead of AWS Access Keys?

## Traditional Method

```
GitHub Repository
        │
        ▼
GitHub Secrets
        │
        ▼
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
        │
        ▼
AWS
```

### Drawbacks

* Permanent credentials are stored in GitHub.
* Keys must be rotated regularly.
* If leaked, they remain valid until revoked.
* Increased operational overhead.

---

## Modern OIDC Method

```
GitHub Actions
        │
        ▼
OIDC Token
        │
        ▼
AWS IAM OIDC Provider
        │
        ▼
AWS STS
        │
        ▼
Temporary Credentials
        │
        ▼
AWS Resources
```

### Benefits

* No AWS access keys stored in GitHub.
* Credentials are generated only when needed.
* Credentials expire automatically.
* Follows AWS security best practices.
* Reduces the risk of credential compromise.

---

# Repository Information

For this lab, the following repository information is used:

| Item            | Value                        |
| --------------- | ---------------------------- |
| GitHub Username | `shettymalathib`             |
| Repository      | `10WeeksOfAWS`               |
| Default Branch  | `main`                       |
| AWS Region      | `us-west-2`                  |
| IAM Role        | `github-oidc-challenge-role` |

---

# Architecture Overview

The authentication flow for this lab is:

```
GitHub Actions
        │
        ▼
OIDC Token
        │
        ▼
AWS IAM OIDC Provider
        │
        ▼
AWS STS
        │
        ▼
Temporary AWS Credentials
        │
        ▼
Access AWS Resources
```

---

## 💡 Why Are We Doing This?

Modern DevOps pipelines should never rely on permanent cloud credentials.

By using OIDC:

* GitHub proves its identity to AWS.
* AWS verifies the request.
* AWS issues temporary credentials.
* The workflow completes its tasks.
* The credentials expire automatically.

This provides a secure and scalable authentication mechanism for CI/CD workflows.

---

## 🔐 Security Tip

Never commit the following to GitHub:

* AWS Access Key ID
* AWS Secret Access Key
* Session Tokens
* `.pem` files
* IAM user passwords
* MFA QR codes

Using OIDC eliminates the need to store AWS access keys in your repository.


---



# Lab 1 – Create the GitHub OIDC Identity Provider

---

# Lab Objective

In this lab, you will establish a trust relationship between **GitHub** and **AWS IAM** using **OpenID Connect (OIDC)**.

Instead of storing AWS Access Keys inside GitHub, GitHub Actions will authenticate using an OIDC token. AWS verifies this token and issues temporary credentials through AWS Security Token Service (AWS STS).

This is the first step toward building a secure CI/CD pipeline.

---

# Prerequisites

Before starting this lab, ensure you have:

* An AWS account
* Access to the AWS Management Console
* A GitHub repository
* Your repository details:

| Item            | Value            |
| --------------- | ---------------- |
| GitHub Username | `shettymalathib` |
| Repository      | `10WeeksOfAWS`   |
| Branch          | `main`           |

---

# What is an Identity Provider?

An **Identity Provider (IdP)** is a trusted service that authenticates users or applications and provides identity information to another system.

In this lab:

* **GitHub** acts as the Identity Provider.
* **AWS IAM** trusts GitHub after verification.
* **AWS STS** issues temporary credentials once GitHub's identity is confirmed.

Without an Identity Provider, AWS would have no way of knowing whether a GitHub workflow is genuine.

---

# Understanding the Authentication Flow

When a GitHub Actions workflow starts, the following sequence occurs:

```text
GitHub Actions Workflow Starts
            │
            ▼
GitHub Creates an OIDC Token
            │
            ▼
AWS IAM Validates the Token
            │
            ▼
AWS STS Issues Temporary Credentials
            │
            ▼
Workflow Accesses AWS Resources
```

Notice that no AWS Access Keys are involved.

---

# Step 1 – Open IAM

1. Sign in to the AWS Management Console.
2. In the search bar, type:

```text
IAM
```

3. Open the **IAM** service.

IAM (Identity and Access Management) is the AWS service used to manage authentication, authorization, users, roles, policies, and identity providers.

---

# Step 2 – Open Identity Providers

From the navigation pane on the left:

```text
IAM
│
├── Users
├── User Groups
├── Roles
├── Policies
└── Identity Providers
```

Click:

```text
Identity Providers
```

Then select:

```text
Add Provider
```

<img width="2540" height="772" alt="image" src="https://github.com/user-attachments/assets/d60b5c6a-6b4c-4360-bf3b-a5968fecedd7" />


---

# Step 3 – Configure the Identity Provider

Choose the following settings.

## Provider Type

Select:

```text
OpenID Connect
```

### Explanation

AWS supports several identity providers.

Examples include:

* SAML
* OpenID Connect (OIDC)

GitHub uses **OpenID Connect**, so this is the correct option.

---

## Provider URL

Enter:

```text
https://token.actions.githubusercontent.com
```

### Important

The URL **must** be:

* lowercase
* exactly as shown
* without additional spaces

AWS treats this URL as **case-sensitive**.

If the URL is incorrect, GitHub authentication will fail.

---

## Audience

Enter:

```text
sts.amazonaws.com
```

### What is the Audience?

The audience identifies which AWS service is allowed to accept GitHub's token.

Here:

```text
sts.amazonaws.com
```

means that the token can be used only with **AWS Security Token Service (STS)**.

Since STS issues temporary credentials, this is exactly what we need.

---

# Configuration Summary

Your screen should now display something similar to:

```text
Provider Type
OpenID Connect

Provider URL
https://token.actions.githubusercontent.com

Audience
sts.amazonaws.com
```

Verify every value carefully.

Then click:

```text
Add Provider
```

<img width="2535" height="1005" alt="image" src="https://github.com/user-attachments/assets/6d0640fb-97e1-4715-b2db-ab091bba3677" />


---

# Step 4 – Verify the Provider

After creation, you should see an Identity Provider similar to:

```text
token.actions.githubusercontent.com
```

The provider status should indicate that it has been successfully created.

---

# Screenshot

The newly created Identity Provider - oidc-provider.png

<img width="2532" height="777" alt="image" src="https://github.com/user-attachments/assets/75ab8ae1-06e1-41f9-86f8-64d00cd1e6a0" />


---

# Why Is This Step Important?

Without an Identity Provider:

* GitHub cannot prove its identity.
* AWS cannot verify GitHub.
* AWS STS will refuse to issue temporary credentials.

Creating the Identity Provider establishes the trust relationship required for secure authentication.

---

# What Happens Behind the Scenes?

When you add the Identity Provider:

1. AWS records GitHub as a trusted Identity Provider.
2. AWS stores GitHub's OIDC endpoint.
3. AWS is now capable of validating OIDC tokens issued by GitHub.
4. No permissions are granted yet.
5. No AWS resources are accessible yet.

At this stage, AWS only **trusts GitHub's identity**.

Authorization will be configured later using an IAM Role.

---

# Understanding the Difference

Many beginners confuse **authentication** and **authorization**.

| Authentication               | Authorization                           |
| ---------------------------- | --------------------------------------- |
| Verifies who you are         | Determines what you can do              |
| Handled by the OIDC Provider | Handled by the IAM Role                 |
| "Are you GitHub?"            | "What AWS resources can GitHub access?" |

This lab separates these two responsibilities clearly.

---

# Notes – What is an OIDC Provider?

An **OIDC (OpenID Connect) Provider** establishes trust between AWS and GitHub.

Instead of storing long-lived AWS Access Keys inside GitHub, GitHub Actions authenticates using an OIDC token.

AWS validates the token and then issues temporary credentials through AWS STS.

This approach is significantly more secure because permanent AWS credentials are never stored in the GitHub repository.

---

# Security Tip

Using OIDC provides several security benefits:

* No AWS Access Keys stored in GitHub.
* Temporary credentials expire automatically.
* Reduced attack surface.
* Easier credential management.
* Recommended by AWS for CI/CD authentication.

---

# Common Mistakes

❌ Typing the Provider URL incorrectly.

❌ Using uppercase letters in the Provider URL.

❌ Entering the wrong Audience value.

❌ Forgetting to create the Identity Provider before creating the IAM Role.

---

# Verification Checklist

Before continuing, verify that:

* ✅ IAM service opened successfully.
* ✅ Identity Provider created.
* ✅ Provider Type is **OpenID Connect**.
* ✅ Provider URL is correct.
* ✅ Audience is `sts.amazonaws.com`.
* ✅ Screenshot captured.
* ✅ Screenshot saved in the correct folder.

---

# Summary

At this point, GitHub can be **trusted** by AWS, but it still has **no permissions**.
---

# Key Takeaways

* OIDC eliminates the need for long-lived AWS Access Keys.
* GitHub authenticates using an OIDC token.
* AWS validates the token using the Identity Provider.
* AWS STS issues temporary credentials.
* Authentication happens before authorization.
* The IAM Role created in the next lab will define the permissions granted to GitHub.

---

> Note:
> The screenshots and instructions in many tutorials use the older AWS IAM console. AWS has updated the IAM Role creation wizard.
> In the new wizard, instead of manually writing the Trust Policy during role creation, AWS asks for your GitHub Organization, Repository, and Branch, and automatically generates the Trust Policy.
> This approach is easier and follows AWS security best practices.

# Lab 2 – Create the IAM Role for GitHub Actions

---

# Lab Objective

In the previous lab, AWS learned to **trust GitHub's identity** by creating an OpenID Connect (OIDC) Identity Provider.

In this lab, you'll create an **IAM Role** that GitHub Actions can assume after authentication.

This role defines **what GitHub is allowed to do** in your AWS account.

---

# Understanding IAM Roles

An **IAM Role** is an AWS identity that provides **temporary permissions** to trusted users, applications, or AWS services.

Unlike an IAM User:

* An IAM User has permanent credentials.
* An IAM Role does **not** have permanent credentials.
* Temporary credentials are issued only when the role is assumed.

GitHub Actions will authenticate using OIDC, assume this IAM Role, and receive temporary AWS credentials through AWS STS.

---

# Authentication vs Authorization

The previous lab handled **authentication**.

This lab handles **authorization**.

| Authentication             | Authorization          |
| -------------------------- | ---------------------- |
| OIDC Identity Provider     | IAM Role               |
| Verifies GitHub's identity | Grants AWS permissions |
| "Who are you?"             | "What can you do?"     |

Both are required before GitHub can access AWS resources.

---

# Authentication Flow So Far

```text
GitHub Actions
        │
        ▼
OIDC Token
        │
        ▼
AWS IAM OIDC Provider
        │
        ▼
Authentication Successful
```

After this lab, the flow becomes:

```text
GitHub Actions
        │
        ▼
OIDC Token
        │
        ▼
OIDC Provider
        │
        ▼
IAM Role
        │
        ▼
Temporary AWS Credentials
```

---

# Step 1 – Open IAM Roles

Navigate to:

```text
IAM
│
└── Roles
```

Click:

```text
Create role
```

<img width="2557" height="762" alt="image" src="https://github.com/user-attachments/assets/7e4e4fc4-15a4-4bda-a3d1-1d0539c27d34" />


---

# Step 2 – Select Trusted Entity

Choose:

```text
Trusted entity type

✔ Web identity
```

### Why Web Identity?

GitHub authenticates using an OIDC token.

OIDC is considered a **Web Identity**, so this is the correct trusted entity type.

---

# Identity Provider

Choose:

```text
token.actions.githubusercontent.com
```

This is the Identity Provider created in Lab 1.

---

# Audience

Select:

```text
sts.amazonaws.com
```

AWS STS will validate GitHub's OIDC token and issue temporary credentials.

---

# Using the New AWS IAM Wizard

AWS has updated the IAM Role creation wizard.

Instead of manually writing a Trust Policy, AWS now asks for GitHub-specific information.

This makes the setup easier and more secure.

---

# GitHub Organization

Enter:

```text
shettymalathib
```

Because the repository is under a personal GitHub account, the username also serves as the organization.

---

# GitHub Repository

Enter:

```text
10WeeksOfAWS
```

This restricts the IAM Role so that only this repository can assume it.

---

# GitHub Branch

Enter:

```text
main
```

This further restricts the role.

Only workflows running from the **main** branch will be allowed to authenticate.

---

# Summary of Configuration

| Field               | Value                               |
| ------------------- | ----------------------------------- |
| Trusted Entity      | Web Identity                        |
| Identity Provider   | token.actions.githubusercontent.com |
| Audience            | sts.amazonaws.com                   |
| GitHub Organization | shettymalathib                      |
| GitHub Repository   | 10WeeksOfAWS                        |
| GitHub Branch       | main                                |

Click:

```text
Next
```

<img width="2560" height="1382" alt="image" src="https://github.com/user-attachments/assets/6ef202e9-bf80-4d49-9235-68e39a5e1ce4" />


---

# Why Restrict the Repository?

Suppose another GitHub repository attempts to use your IAM Role.

For example:

```text
someoneelse/10WeeksOfAWS
```

AWS compares the OIDC token against the Trust Policy.

Since the repository does not match:

```text
shettymalathib/10WeeksOfAWS
```

AWS rejects the authentication request.

This prevents unauthorized repositories from accessing your AWS account.

---

# Why Restrict the Branch?

Suppose someone creates a branch named:

```text
testing
```

If your Trust Policy allows only:

```text
main
```

then workflows running from:

```text
testing
```

cannot assume the IAM Role.

This adds another layer of security.

---

# Step 3 – Attach Permissions

Search for the AWS managed policy:

```text
AmazonS3ReadOnlyAccess
```

Select it.

Click:

```text
Next
```

<img width="2537" height="721" alt="image" src="https://github.com/user-attachments/assets/36633fdc-a397-40f5-890f-17fa8d9183a4" />


---

# Why AmazonS3ReadOnlyAccess?

This managed policy allows:

* Viewing S3 buckets
* Listing bucket contents
* Reading objects (where permitted)

It **does not** allow:

* Creating buckets
* Uploading files
* Deleting files
* Modifying bucket configurations

Using a read-only policy follows the **Principle of Least Privilege**, which grants only the permissions required for the task.

---

# Step 4 – Name the Role

Enter the following role name:

```text
github-oidc-challenge-role
```

Click:

```text
Create role
```

<img width="2560" height="1861" alt="image" src="https://github.com/user-attachments/assets/cefc878a-c50d-4da2-8534-6f5cef2fece3" />


---

# Notes – What is an IAM Role?

An IAM Role is an AWS identity that provides temporary permissions to trusted users or services.

In this lab:

* GitHub Actions assumes the IAM Role.
* AWS validates GitHub's OIDC token.
* AWS STS issues temporary credentials.
* The workflow uses those credentials to access AWS.

This eliminates the need to store AWS Access Keys in GitHub.

---

# Architecture After Creating the Role

```text
GitHub Actions
       │
       ▼
OIDC Provider
       │
       ▼
IAM Role
       │
       ▼
Temporary Credentials
```

The IAM Role defines **what GitHub can do** after it successfully authenticates.

---

# Important Note

Do **not** edit the Trust Policy yet.

The next lab will verify and, if necessary, update the Trust Policy so that only:

```text
repo:shettymalathib/10WeeksOfAWS
```

can assume this IAM Role.

This is one of the most important security configurations in the entire setup.

---

# Common Mistakes

❌ Choosing **AWS Service** instead of **Web Identity**.

❌ Selecting the wrong Identity Provider.

❌ Typing the GitHub username incorrectly.

❌ Forgetting to specify the repository.

❌ Attaching an overly permissive policy instead of `AmazonS3ReadOnlyAccess`.

❌ Naming the role incorrectly.

---

# Security Tip

Always grant the minimum permissions required.

For this lab, the workflow only needs to list S3 buckets.

Using **AmazonS3ReadOnlyAccess** is much safer than granting full administrative access.

Following the **Principle of Least Privilege** reduces the potential impact of compromised credentials.

---

# Verification Checklist

Before moving to the next lab, confirm that:

* ✅ IAM Role created successfully.
* ✅ Trusted Entity is **Web Identity**.
* ✅ Identity Provider is `token.actions.githubusercontent.com`.
* ✅ Audience is `sts.amazonaws.com`.
* ✅ GitHub Organization is `shettymalathib`.
* ✅ Repository is `10WeeksOfAWS`.
* ✅ Branch is `main`.
* ✅ `AmazonS3ReadOnlyAccess` policy attached.
* ✅ Role name is `github-oidc-challenge-role`.

---

# Summary

In this lab you learned:

* What an IAM Role is.
* Why GitHub Actions use IAM Roles instead of IAM Users.
* How temporary credentials improve security.
* How to restrict access to a specific GitHub repository.
* How to restrict access to a specific branch.
* Why the Principle of Least Privilege is important.

At this stage:

* AWS trusts GitHub's identity.
* GitHub has an IAM Role available.
* Permissions have been assigned.

The only remaining security configuration is the **Trust Policy**, which determines exactly **who is allowed to assume the IAM Role**.

---

# Key Takeaways

* IAM Roles provide temporary AWS credentials.
* GitHub Actions assume IAM Roles using OIDC.
* Repository and branch restrictions improve security.
* Read-only permissions follow AWS security best practices.
* The Trust Policy is the final layer of protection before GitHub can access AWS resources.


---



# Lab 3 – Verify and Understand the Trust Policy

---

# Lab Objective

In the previous lab, you created an IAM Role for GitHub Actions.

However, simply creating the role is **not enough**.

AWS must also know **who is allowed to assume that role**.

This is controlled by the **Trust Policy**.

The Trust Policy is one of the most important security configurations in AWS IAM because it defines **who can use the IAM Role**.

---

# What is a Trust Policy?

A **Trust Policy** is a special IAM policy attached to an IAM Role.

Unlike permission policies, which define **what actions are allowed**, a Trust Policy defines **who is allowed to assume the role**.

Think of it like this:

* **Permission Policy** → What can you do?
* **Trust Policy** → Who is allowed to become this role?

Both are required.

Without a valid Trust Policy, GitHub cannot assume the IAM Role, even if the role has the correct permissions.

---

# Why is the Trust Policy Important?

Suppose your IAM Role has permission to access Amazon S3.

Without a Trust Policy:

* Anyone who knows the Role ARN could attempt to assume the role.
* AWS would have no way of determining whether the request came from your GitHub repository.

The Trust Policy prevents unauthorized access by verifying:

* The Identity Provider
* The audience (`aud`)
* The repository
* The branch

---

# Step 1 – Open the IAM Role

Navigate to:

```text
IAM
│
└── Roles
```

Open:

```text
github-oidc-challenge-role
```

<img width="2522" height="815" alt="image" src="https://github.com/user-attachments/assets/480a4067-82e6-4322-8399-125580c935cd" />


---

# Step 2 – Open Trust Relationships

Select:

```text
Trust relationships
```

Click:

```text
View trust policy
```

You should now see the JSON Trust Policy.

---

# Expected Trust Policy

The policy should look similar to this:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::138852020826:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:shettymalathib/10WeeksOfAWS:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

---

# Understanding Every Line

Let's understand each section of the policy.

---

## Version

```json
"Version": "2012-10-17"
```

This specifies the IAM policy language version.

It does **not** represent the creation date of the policy.

AWS recommends using this version for IAM policies.

---

## Statement

```json
"Statement": [
```

A policy can contain one or more statements.

Each statement defines a rule.

In this lab, only one statement is required.

---

## Effect

```json
"Effect": "Allow"
```

This tells AWS that the matching request is allowed.

Possible values are:

* Allow
* Deny

Here, we explicitly allow GitHub to assume the role when all conditions are satisfied.

---

## Principal

```json
"Principal": {
  "Federated": "arn:aws:iam::138852020826:oidc-provider/token.actions.githubusercontent.com"
}
```

### What is the Principal?

The **Principal** identifies **who** is allowed to request access.

In this case, the Principal is the GitHub OIDC Identity Provider.

Notice the ARN contains:

* AWS Account ID
* OIDC Provider

Example:

```text
arn:aws:iam::138852020826:oidc-provider/token.actions.githubusercontent.com
```

This tells AWS:

> "Only tokens issued by GitHub's OIDC provider are trusted."

---

# Why Must the AWS Account ID Be Present?

A common mistake is writing:

```text
arn:aws:iam:::oidc-provider/token.actions.githubusercontent.com
```

Notice the missing account ID.

This ARN is invalid.

The correct ARN is:

```text
arn:aws:iam::138852020826:oidc-provider/token.actions.githubusercontent.com
```

Without the account ID, AWS cannot locate the Identity Provider.

---

## Action

```json
"Action": "sts:AssumeRoleWithWebIdentity"
```

This tells AWS which operation GitHub is allowed to perform.

GitHub is **not** allowed to:

* Create users
* Delete resources
* Modify IAM

GitHub is only allowed to request temporary credentials using:

```text
sts:AssumeRoleWithWebIdentity
```

This operation is handled by **AWS Security Token Service (STS).**

---

# Condition

The Condition section contains the security restrictions.

```json
"Condition": {
```

Without these conditions, any trusted OIDC token could attempt to assume the role.

The conditions ensure that only your repository can use it.

---

# StringEquals

```json
"StringEquals": {
  "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
}
```

### What does this mean?

AWS checks the **Audience (aud)** claim inside GitHub's OIDC token.

The value must be:

```text
sts.amazonaws.com
```

If the audience does not match, AWS rejects the request.

---

# StringLike

```json
"StringLike": {
  "token.actions.githubusercontent.com:sub":
  "repo:shettymalathib/10WeeksOfAWS:ref:refs/heads/main"
}
```

This is the most important line in the entire policy.

AWS checks the **Subject (sub)** claim inside GitHub's OIDC token.

The token must identify:

* Repository:

```text
shettymalathib/10WeeksOfAWS
```

* Branch:

```text
main
```

If either value does not match, AWS denies access.

---

# Understanding the Subject Claim

The Subject follows this format:

```text
repo:<owner>/<repository>:ref:refs/heads/<branch>
```

For this lab:

```text
repo:shettymalathib/10WeeksOfAWS:ref:refs/heads/main
```



Breaking it down:

| Part           | Meaning                      |
| -------------- | ---------------------------- |
| repo           | GitHub repository identifier |
| shettymalathib | Repository owner             |
| 10WeeksOfAWS   | Repository name              |
| refs/heads     | Git reference                |
| main           | Branch name                  |

---

# Why Restrict the Branch?

Suppose someone pushes malicious code to:

```text
feature/testing
```

If the Trust Policy only allows:

```text
main
```

then workflows from the feature branch cannot obtain AWS credentials.

This protects production resources.

---

# Common Trust Policy Mistakes

## Missing Account ID

Incorrect:

```text
arn:aws:iam:::oidc-provider/token.actions.githubusercontent.com
```

Correct:

```text
arn:aws:iam::138852020826:oidc-provider/token.actions.githubusercontent.com
```

---

## Duplicate Subject Entries

Incorrect:

```json
"repo:shettymalathib/10WeeksOfAWS:ref:refs/heads/main",
"repo:shettymalathib/10WeeksOfAWS:ref:refs/heads/main"
```

Only one Subject value is required.

---

## Wrong Repository

Incorrect:

```text
repo:someoneelse/10WeeksOfAWS
```

Correct:

```text
repo:shettymalathib/10WeeksOfAWS
```

---

## Wrong Branch

Incorrect:

```text
refs/heads/master
```

Correct (for this lab):

```text
refs/heads/main
```

Always use your repository's default branch.

---

# Save the Trust Policy

Save the JSON as:

```text
oidc/trust-policy.json
```

This file becomes part of your Week 1 submission.

---

<img width="2097" height="942" alt="image" src="https://github.com/user-attachments/assets/6381be06-35f1-446e-9177-bbfcec8dbf7a" />
<img width="1487" height="412" alt="image" src="https://github.com/user-attachments/assets/a3929e28-8e5c-4f1c-871f-0679c31b0bac" />

The screenshots demonstrate that:

* The Identity Provider exists.
* The IAM Role exists.
* The Trust Policy correctly restricts access.

---

# Authentication Flow After the Trust Policy

```text
GitHub Actions
        │
        ▼
OIDC Token
        │
        ▼
AWS IAM OIDC Provider
        │
        ▼
Trust Policy Validation
        │
        ▼
AWS STS
        │
        ▼
Temporary AWS Credentials
        │
        ▼
Access AWS Resources
```

Notice that the Trust Policy acts like a security checkpoint before AWS issues credentials.

---

# Security Tip

The Trust Policy should always be as restrictive as possible.

Instead of allowing:

* Every GitHub repository

restrict access to:

* A specific repository
* A specific branch

This follows the **Principle of Least Privilege**.

---

# Verification Checklist

Before moving to the next lab, confirm:

* ✅ Trust Policy opened.
* ✅ Principal references the correct OIDC Provider.
* ✅ AWS Account ID is correct.
* ✅ Audience is `sts.amazonaws.com`.
* ✅ Repository is `shettymalathib/10WeeksOfAWS`.
* ✅ Branch is `main`.
* ✅ Trust Policy saved as `trust-policy.json`.
* ✅ Screenshots captured.

---

# Summary

In this lab you learned:

* What a Trust Policy is.
* Why it is different from a Permission Policy.
* How AWS verifies GitHub's identity.
* Why the `aud` claim is required.
* Why the `sub` claim is the most important security restriction.
* Common Trust Policy mistakes and how to avoid them.

At this point:

* GitHub is trusted.
* The IAM Role exists.
* The Trust Policy restricts access to your repository.

The final step is connecting GitHub Actions to AWS by creating a workflow that assumes the IAM Role and requests temporary credentials.

---

# Key Takeaways

* A Trust Policy controls **who** can assume an IAM Role.
* Permission Policies control **what** the role can do.
* The `Principal` identifies the trusted Identity Provider.
* The `aud` claim ensures the token is intended for AWS STS.
* The `sub` claim restricts access to a specific repository and branch.
* Proper Trust Policies are essential for secure CI/CD authentication.

---



# Lab 4 – Create and Run the GitHub Actions Workflow

---

# Lab Objective

Now that the AWS configuration is complete, the next step is to connect GitHub to AWS.

In this lab, you will:

* Create a GitHub Actions workflow.
* Configure GitHub to assume the IAM Role using OIDC.
* Authenticate with AWS without storing access keys.
* Verify authentication using AWS STS.
* List Amazon S3 buckets using temporary credentials.

This lab completes the GitHub OIDC authentication flow.

---

# Authentication Flow

At this point, the authentication process looks like this:

```text
GitHub Actions
        │
        ▼
OIDC Token
        │
        ▼
AWS IAM OIDC Provider
        │
        ▼
Trust Policy
        │
        ▼
AWS STS
        │
        ▼
Temporary AWS Credentials
        │
        ▼
AWS Resources
```

---

# What is GitHub Actions?

**GitHub Actions** is GitHub's built-in automation platform.

It allows you to automate tasks such as:

* Running tests
* Deploying applications
* Building software
* Executing AWS commands
* Automating CI/CD pipelines

Workflows are written using **YAML** and stored inside the repository.

---

# Step 1 – Create the Workflow Folder

Inside your repository, create the following folder structure:

```text
10WeeksOfAWS
└── .github
    └── workflows
        └── aws-oidc-challenge.yml
```

If you are using Visual Studio Code:

1. Open the cloned repository.
2. Create a folder named:

```text
.github
```

> **Important:** The folder name begins with a period (`.`).

3. Inside `.github`, create another folder:

```text
workflows
```

4. Inside `workflows`, create the file:

```text
aws-oidc-challenge.yml
```

---

# Why Must It Be Inside `.github/workflows`?

GitHub automatically scans the following directory:

```text
.github/workflows/
```

Every YAML file inside this directory is treated as a GitHub Actions workflow.

If the file is placed elsewhere, GitHub will not recognize it.

---

# Step 2 – Create the Workflow

Paste the following workflow into the file.

```yaml
name: AWS OIDC Challenge

on:
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  test-aws-oidc:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::138852020826:role/github-oidc-challenge-role
          aws-region: us-west-2

      - name: Verify Identity
        run: aws sts get-caller-identity

      - name: List S3 Buckets
        run: aws s3 ls
```

---

# Understanding the Workflow

Let's examine each section.

---

## Workflow Name

```yaml
name: AWS OIDC Challenge
```

This is the name displayed in the **Actions** tab.

---

## Trigger

```yaml
on:
  workflow_dispatch:
```

This means the workflow runs **manually**.

It will execute only when you click:

```text
Run workflow
```

inside GitHub Actions.

---

## Permissions

```yaml
permissions:
  id-token: write
  contents: read
```

### Why `id-token: write`?

This permission allows GitHub to generate an OIDC token.

Without it:

* GitHub cannot request an OIDC token.
* AWS authentication will fail.

---

### Why `contents: read`?

This allows the workflow to read the contents of the repository.

It is required by the checkout action.

---

# Job

```yaml
jobs:
```

A workflow can contain multiple jobs.

This workflow contains a single job:

```yaml
test-aws-oidc
```

---

# Runner

```yaml
runs-on: ubuntu-latest
```

GitHub creates a temporary Ubuntu virtual machine.

Every step runs on this virtual machine.

After the workflow finishes, the virtual machine is destroyed automatically.

---

# Step 1 – Checkout Repository

```yaml
- name: Checkout Repository
  uses: actions/checkout@v4
```

This downloads the repository onto the GitHub runner.

Without this step, the workflow would not have access to your project files.

---

# Step 2 – Configure AWS Credentials

```yaml
uses: aws-actions/configure-aws-credentials@v4
```

This official GitHub Action performs the OIDC authentication.

It:

1. Requests an OIDC token from GitHub.
2. Sends the token to AWS.
3. AWS validates the token.
4. AWS STS issues temporary credentials.
5. The credentials become available for the remaining workflow steps.

---

# Role to Assume

```yaml
role-to-assume:
arn:aws:iam::138852020826:role/github-oidc-challenge-role
```

This tells AWS which IAM Role should be assumed.

This ARN is **not a secret**.

It is safe to store in a public repository.

---

# AWS Region

```yaml
aws-region: us-west-2
```

This specifies the AWS Region where commands will run.

In this lab:

* Region = US West (Oregon)
* Region code = `us-west-2`

---

# Why Not `ap-south-1`?

The original lab used:

```yaml
ap-south-1
```

because it assumed the Mumbai Region.

Since this lab uses Oregon, the correct Region is:

```yaml
us-west-2
```

---

# Step 3 – Verify Identity

```yaml
aws sts get-caller-identity
```

This command asks AWS:

> "Who am I currently authenticated as?"

If authentication succeeds, AWS returns:

* User ID
* AWS Account ID
* IAM Role ARN

---

# Step 4 – List S3 Buckets

```yaml
aws s3 ls
```

This command lists every Amazon S3 bucket the IAM Role has permission to view.

If there are no buckets, AWS displays:

```text
No buckets found.
```

This is still considered a successful result.

---

# Step 3 – Commit the Workflow

Open a terminal inside the repository.

Run:

```bash
git add .
git commit -m "Add GitHub OIDC challenge workflow"
git push origin main
```

Alternatively, you can use the Source Control panel in Visual Studio Code.

---

# Step 4 – Run the Workflow

Open GitHub.

Navigate to:

```text
Repository
    │
    ▼
Actions
```

You should see:

```text
AWS OIDC Challenge
```

Click it.

Then click:

```text
Run workflow
```

Click the green **Run workflow** button.

Wait approximately **30–60 seconds**.

https://github.com/shettymalathib/10WeeksOfAWS/actions/workflows/aws-oidc-challenge.yml

<img width="2557" height="921" alt="image" src="https://github.com/user-attachments/assets/0b37d6c9-3b3c-4dec-97c0-06da9a78202d" />


---

# Expected Workflow

The workflow should execute the following sequence:

```text
Checkout Repository
        │
        ▼
Configure AWS Credentials
        │
        ▼
Verify Identity
        │
        ▼
List S3 Buckets
```

If every step completes successfully, the workflow displays a **green check mark**.

---

# If the Workflow Fails

If the workflow fails:

1. Open the failed workflow.
2. Expand the failed step.
3. Read the error message carefully.

Common causes include:

* Incorrect Trust Policy
* Wrong AWS Account ID
* Wrong IAM Role ARN
* Incorrect branch restriction
* Missing `id-token: write` permission

Most authentication failures occur because one of these settings is incorrect.

---



The screenshots of:

* Successful workflow execution
* Verify Identity output
* List S3 Buckets output

<img width="2560" height="1377" alt="image" src="https://github.com/user-attachments/assets/8d39685d-9cc2-4d4a-b0e2-bc0f41061a31" />


These screenshots demonstrate that GitHub authenticated successfully using OIDC.

---

# Verification Checklist

Before continuing, verify:

* ✅ Workflow file created.
* ✅ Workflow stored inside `.github/workflows`.
* ✅ IAM Role ARN is correct.
* ✅ AWS Region is correct.
* ✅ Workflow committed.
* ✅ Workflow pushed.
* ✅ Workflow executed.
* ✅ Workflow completed successfully.

---

# Summary

In this lab you learned:

* How GitHub Actions workflows are structured.
* Why workflows are stored in `.github/workflows`.
* How GitHub authenticates to AWS using OIDC.
* How temporary AWS credentials are generated.
* How to verify the authenticated identity.
* How to execute AWS CLI commands securely from GitHub Actions.

At this stage, the complete OIDC authentication pipeline is operational.

The final lab explains the workflow output, common troubleshooting techniques, security best practices, and how to prepare your Week 1 submission.

---

# Key Takeaways

* GitHub Actions automate CI/CD workflows.
* OIDC eliminates the need for AWS Access Keys.
* `id-token: write` is required for OIDC authentication.
* The IAM Role ARN is safe to publish.
* AWS STS provides temporary credentials.
* Successful execution confirms that GitHub authenticated securely with AWS.

---



#  Understanding the Output for above github actions yml file, Security Best Practices

---

# Lab Objective


You have successfully completed the GitHub OIDC setup.

In this final lab, you will learn:

* How to understand the workflow output.
* Why the authentication was successful.
* Why the IAM Role ARN is safe to publish.
* What information should never be public.
* Security best practices for production environments.
* How to prepare your Week 1 submission.

---

# Understanding the Workflow Output

Your workflow executed two AWS CLI commands:

```yaml
- run: aws sts get-caller-identity

- run: aws s3 ls
```

These commands verify that GitHub successfully authenticated with AWS using OpenID Connect (OIDC).

---

# Command 1 – Verify Identity

The first command executed:

```bash
aws sts get-caller-identity
```

This command asks AWS:

> **"Who am I currently authenticated as?"**

AWS responds with information about the temporary identity that is currently using AWS credentials.

---

# Example Output

A successful response looks similar to:

```json
{
  "UserId": "AROAXXXXXXXXXXXXX:GitHubActions",
  "Account": "138852020826",
  "Arn": "arn:aws:sts::138852020826:assumed-role/github-oidc-challenge-role/GitHubActions"
}
```

---

# Understanding the Output

## UserId

```text
AROAXXXXXXXXXXXXX:GitHubActions
```

This is the temporary identity created by AWS Security Token Service (STS).

Unlike an IAM User, this identity exists only for the duration of the workflow.

---

## Account

```text
138852020826
```

This is your AWS Account ID.

It confirms that GitHub authenticated against the correct AWS account.

---

## ARN

```text
arn:aws:sts::138852020826:assumed-role/github-oidc-challenge-role/GitHubActions
```

This confirms:

* GitHub successfully assumed the IAM Role.
* The role name is:

```text
github-oidc-challenge-role
```

This is the strongest proof that OIDC authentication worked successfully.

---

# Why Is This Command Important?

Without successful authentication:

```bash
aws sts get-caller-identity
```

would return an error.

A successful response proves:

* GitHub created an OIDC token.
* AWS validated the token.
* AWS STS issued temporary credentials.
* The IAM Role was assumed successfully.

---

# Command 2 – List Amazon S3 Buckets

The second command executed:

```bash
aws s3 ls
```

This command lists every Amazon S3 bucket that the IAM Role has permission to view.

---

# Example Output (No Buckets)

```text
No buckets found.
```

This is completely normal.

It simply means no S3 buckets currently exist in the AWS account.

The authentication was still successful.

---

# Example Output (Existing Buckets)

```text
2026-07-14 18:20:11 my-training-bucket
```

If buckets exist, AWS lists:

* Creation date
* Bucket name

---

# Why Did This Command Work?

The IAM Role has the policy:

```text
AmazonS3ReadOnlyAccess
```

This policy allows:

* Listing S3 buckets
* Viewing bucket information
* Reading objects (where permitted)

The workflow successfully used the temporary credentials issued by AWS STS.

---

# Complete Authentication Flow

The following sequence occurred behind the scenes:

```text
GitHub Actions Starts
        │
        ▼
GitHub Creates an OIDC Token
        │
        ▼
AWS IAM OIDC Provider Validates the Token
        │
        ▼
Trust Policy Verification
        │
        ▼
AWS STS Issues Temporary Credentials
        │
        ▼
GitHub Assumes the IAM Role
        │
        ▼
AWS CLI Commands Execute
        │
        ▼
Workflow Completes Successfully
```

Notice something important:

* No AWS Access Keys were used.
* No AWS Secret Access Keys were stored.
* Temporary credentials were generated only while the workflow was running.

This is why OIDC is considered an AWS security best practice.

---

# Why Is the IAM Role ARN Safe to Publish?

The workflow contains:

```yaml
role-to-assume:
arn:aws:iam::138852020826:role/github-oidc-challenge-role
```

Many beginners assume this is sensitive information.

It is **not**.

An IAM Role ARN is similar to a **house address**.

Knowing the address does not allow someone to enter the house.

AWS still verifies:

* The OIDC token
* The Identity Provider
* The repository
* The branch
* The Trust Policy

Only if all conditions match does AWS issue temporary credentials.

Therefore, publishing the IAM Role ARN in GitHub is considered safe.

---

# What Protects the IAM Role?

Your Trust Policy contains:

```text
repo:shettymalathib/10WeeksOfAWS:ref:refs/heads/main
```

This means only:

* Repository:

```text
shettymalathib/10WeeksOfAWS
```

* Branch:

```text
main
```

can assume the IAM Role.

If another repository attempts to use the same Role ARN, AWS rejects the request because the OIDC token does not satisfy the Trust Policy.

---

# What Should Never Be Public?

The following items must always remain private:

* AWS Access Key ID
* AWS Secret Access Key
* Temporary Session Tokens
* Root account password
* IAM User passwords
* MFA recovery codes
* MFA QR codes
* Private SSH keys
* `.pem` files
* Database passwords
* API keys
* Encryption keys

Publishing any of these can compromise your AWS account.

---

# What Is Safe to Publish?

The following information is generally safe to include in a public repository:

* IAM Role ARN
* IAM Role name
* IAM Policy JSON
* Trust Policy JSON
* OIDC Provider URL
* GitHub Actions workflow
* AWS Region
* AWS Account ID (while not secret, avoid sharing it unnecessarily)

Even though the AWS Account ID is not considered a secret, it is a good practice to share only what is necessary.

---

# Production Security Improvements

This lab restricts access to:

* One repository
* One branch


In production environments, you can further restrict access by allowing only:

* Specific workflow files
* Protected GitHub environments
* Signed commits
* Release tags
* Approved branches

These additional restrictions reduce the attack surface and improve overall security.



---

# Key Takeaways

* OIDC removes the need for long-lived AWS credentials.
* GitHub Actions authenticate using OIDC tokens.
* AWS IAM verifies the token using the configured Identity Provider.
* Trust Policies determine who can assume an IAM Role.
* AWS STS issues temporary credentials.
* Temporary credentials automatically expire.
* Following the Principle of Least Privilege improves security.
* OIDC is the recommended authentication method for GitHub Actions interacting with AWS.

