# Week 1 Quiz Prep – Answers

## 1. What is the difference between a Region and an Availability Zone?

A **Region** is a geographic location where AWS provides cloud services.

An **Availability Zone (AZ)** is one or more isolated data centers within a Region.

A Region contains multiple Availability Zones to provide high availability and fault tolerance.

**Example:**

* Region: US West (Oregon) (`us-west-2`)
* Availability Zones:

  * `us-west-2a`
  * `us-west-2b`
  * `us-west-2c`

---

## 2. When should you use CloudFront and Edge Locations?

Use **Amazon CloudFront** when you want to deliver content quickly to users around the world.

CloudFront caches content at **Edge Locations**, which are AWS locations closer to users.

This reduces latency and improves website performance.

**Examples:**

* Websites
* Images
* Videos
* Static files
* APIs

---

## 3. What does AWS manage in the Shared Responsibility Model?

AWS is responsible for **Security of the Cloud**.

This includes:

* Physical data centers
* Servers
* Storage hardware
* Networking
* Power and cooling
* Hypervisor and virtualization layer

---

## 4. What does the customer manage in the Shared Responsibility Model?

Customers are responsible for **Security in the Cloud**.

This includes:

* IAM users and permissions
* Passwords and MFA
* Applications
* Operating systems (for EC2)
* Data
* Encryption settings
* Network configurations
* Security groups

---

## 5. Why should the root user not be used daily?

The root user has **unrestricted access** to the entire AWS account.

Using it daily increases the risk of accidental changes or account compromise.

Best practice:

* Enable MFA.
* Use the root account only for account-level tasks.
* Use IAM users or IAM roles for everyday work.

---

## 6. Why is MFA important for the root user?

MFA adds a second layer of authentication.

Even if someone knows your password, they cannot access the account without the second authentication factor.

This greatly improves account security.

---

## 7. What is the difference between an IAM user and an IAM role?

| IAM User                         | IAM Role                                   |
| -------------------------------- | ------------------------------------------ |
| Long-term identity               | Temporary identity                         |
| Has username and password        | No permanent login credentials             |
| Used by people or applications   | Assumed by trusted users or AWS services   |
| Permissions remain until changed | Temporary credentials expire automatically |

Examples:

* IAM User → Employee logging into AWS
* IAM Role → EC2 accessing S3

---

## 8. Why should permissions be attached to groups instead of individual users?

Managing permissions through groups is easier and more scalable.

Instead of assigning permissions to every user individually, assign permissions to a group and add users to that group.

Benefits:

* Easier management
* Consistent permissions
* Reduced configuration errors

---

## 9. What is Least Privilege?

Least Privilege means granting **only the permissions required to perform a specific task**.

Do not give unnecessary permissions.

Example:

A user who only needs to view EC2 instances should receive read-only access instead of administrator access.

---

## 10. What does an IAM policy contain?

An IAM Policy is a JSON document that defines permissions.

It specifies:

* Effect (Allow or Deny)
* Actions
* Resources
* Optional Conditions

Example:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "*"
}
```

---

## 11. What is the difference between a managed policy and an inline policy?

### AWS Managed Policy

* Created and maintained by AWS
* Reusable
* Updated by AWS

Example:

* AmazonS3ReadOnlyAccess

---

### Customer Managed Policy

* Created by you
* Reusable
* Managed by you

---

### Inline Policy

* Attached directly to one user, group, or role
* Not reusable
* Deleted when the identity is deleted

---

## 12. What does a permission boundary do?

A Permission Boundary defines the **maximum permissions** that an IAM user or role can receive.

It does **not** grant permissions.

Think of it as a ceiling.

Even if someone attaches an Administrator policy, the permission boundary can prevent the user from receiving permissions beyond the defined limit.

---

## 13. Why are temporary credentials safer than long-lived access keys?

Temporary credentials:

* Expire automatically
* Reduce the risk of leaked credentials
* Are issued only when needed
* Do not require storing permanent AWS access keys

Long-lived access keys remain valid until manually rotated or deleted.

---

## 14. What problem does GitHub OIDC solve?

Traditionally, GitHub Actions required storing AWS Access Keys as GitHub Secrets.

If those keys were leaked, attackers could access AWS.

GitHub OIDC allows GitHub Actions to authenticate using an identity token.

AWS verifies the token and issues temporary credentials through STS.

Benefits:

* No stored AWS access keys
* Temporary credentials
* Improved security
* Easier credential management

---

## 15. What does AWS STS provide?

AWS Security Token Service (STS) issues **temporary security credentials**.

These credentials allow users, applications, or AWS services to access AWS resources for a limited time.

STS is commonly used with:

* IAM Roles
* Cross-account access
* GitHub OIDC
* AWS Identity Center (SSO)
* Federated users

---

# Quick Revision

| Service/Concept       | Remember This                                                        |
| --------------------- | -------------------------------------------------------------------- |
| Region                | Geographic AWS location                                              |
| Availability Zone     | Isolated data center within a Region                                 |
| Edge Location         | Delivers cached content closer to users                              |
| CloudFront            | CDN that uses Edge Locations                                         |
| IAM User              | Long-term identity                                                   |
| IAM Group             | Collection of users with shared permissions                          |
| IAM Role              | Temporary identity with temporary credentials                        |
| IAM Policy            | JSON document defining permissions                                   |
| Least Privilege       | Grant only the permissions required                                  |
| Permission Boundary   | Maximum permissions allowed                                          |
| STS                   | Issues temporary security credentials                                |
| OIDC                  | Secure authentication without storing AWS access keys                |
| Root User             | Use only for account-level tasks                                     |
| MFA                   | Adds a second authentication factor                                  |
| Shared Responsibility | AWS secures the cloud; customers secure what they build in the cloud |

---

# GitHub OIDC – Common Interview Questions & Answers

## 1. What is OIDC?

**Answer:**

OIDC (OpenID Connect) is an authentication protocol built on top of OAuth 2.0. It allows GitHub Actions to securely authenticate with AWS without storing long-lived AWS Access Keys.

GitHub generates an OIDC token, AWS verifies the token, and AWS STS issues temporary credentials to access AWS resources.

**One-line answer:**

> OIDC is a secure authentication protocol that enables GitHub Actions to access AWS using temporary credentials instead of permanent AWS Access Keys.

---

## 2. Why is OIDC preferred over AWS Access Keys?

**Answer:**

Traditionally, GitHub Actions used AWS Access Keys stored as GitHub Secrets. These keys are long-lived and can be leaked or misused.

OIDC is preferred because:

* No AWS Access Keys are stored in GitHub.
* AWS issues temporary credentials.
* Credentials expire automatically.
* Reduces the risk of credential leakage.
* Follows AWS security best practices.

**One-line answer:**

> OIDC is preferred because it replaces long-lived AWS Access Keys with temporary credentials, making CI/CD pipelines more secure.

---

## 3. What is an IAM Role?

**Answer:**

An IAM Role is an AWS identity that provides temporary permissions to trusted users, applications, or services.

Unlike an IAM User, an IAM Role does not have permanent credentials. It is assumed when needed and AWS STS provides temporary credentials based on the permissions attached to the role.

In GitHub OIDC, GitHub Actions assumes the IAM Role after AWS verifies the OIDC token.

**One-line answer:**

> An IAM Role is a temporary AWS identity that grants permissions without using permanent credentials.

---

## 4. What is a Trust Policy?

**Answer:**

A Trust Policy is a special IAM policy attached to an IAM Role that defines **who is allowed to assume the role**.

For GitHub OIDC, the Trust Policy usually checks:

* The GitHub repository
* The GitHub organization or owner
* The branch
* The OIDC provider
* The audience (`sts.amazonaws.com`)

If these conditions match, AWS allows GitHub Actions to assume the IAM Role.

**One-line answer:**

> A Trust Policy specifies which trusted identities are allowed to assume an IAM Role.

---

## 5. What is AWS STS?

**Answer:**

AWS STS (Security Token Service) is an AWS service that generates temporary security credentials.

When GitHub successfully authenticates using OIDC, AWS STS issues:

* Temporary Access Key ID
* Temporary Secret Access Key
* Temporary Session Token

These credentials are used to access AWS resources and expire automatically.

**One-line answer:**

> AWS STS issues temporary security credentials after successful authentication.

---

## 6. Why are temporary credentials more secure?

**Answer:**

Temporary credentials improve security because:

* They automatically expire after a short period.
* They are generated only when needed.
* They reduce the impact of credential leaks.
* They eliminate the need to store permanent AWS Access Keys.

Even if temporary credentials are compromised, they become useless after they expire.

**One-line answer:**

> Temporary credentials are more secure because they expire automatically and reduce the risk of long-term unauthorized access.

---

## 7. What is the purpose of an OIDC Provider?

**Answer:**

An OIDC Provider establishes a trust relationship between AWS and an external identity provider, such as GitHub.

It allows AWS to:

* Verify the OIDC token.
* Confirm that the token was issued by GitHub.
* Validate the token before allowing an IAM Role to be assumed.

Without an OIDC Provider, AWS cannot trust or verify GitHub's identity.

**One-line answer:**

> An OIDC Provider allows AWS to trust and verify GitHub's identity before granting temporary access.

---

# Bonus Interview Question

## 8. Explain the GitHub OIDC authentication flow.

**Answer:**

1. GitHub Actions starts a workflow.
2. GitHub generates an OIDC token.
3. AWS IAM OIDC Provider verifies the token.
4. AWS checks the IAM Role's Trust Policy.
5. AWS STS issues temporary credentials.
6. GitHub Actions uses those credentials to access AWS resources.
7. The credentials expire automatically after the workflow completes.

**One-line answer:**

> GitHub authenticates using an OIDC token, AWS verifies it, STS issues temporary credentials, and GitHub securely accesses AWS resources.

---

# Quick Revision Table

| Question                   | One-Line Answer                                                                                                  |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| What is OIDC?              | A secure authentication protocol that allows GitHub Actions to access AWS using temporary credentials.           |
| Why OIDC over Access Keys? | It eliminates long-lived AWS Access Keys and uses temporary credentials.                                         |
| What is an IAM Role?       | A temporary AWS identity that grants permissions without permanent credentials.                                  |
| What is a Trust Policy?    | A policy that defines who can assume an IAM Role.                                                                |
| What is AWS STS?           | A service that issues temporary AWS security credentials.                                                        |
| Why temporary credentials? | They expire automatically, reducing security risks.                                                              |
| What is an OIDC Provider?  | A trust relationship that allows AWS to verify GitHub's identity.                                                |
| GitHub OIDC Flow?          | GitHub → OIDC Token → OIDC Provider → Trust Policy → IAM Role → AWS STS → Temporary Credentials → AWS Resources. |
