# Week 1 - IAM Challenge

## Name

Malathi Shetty

---

## Topics Practiced

* AWS Global Infrastructure (Regions, Availability Zones, Edge Locations)
* Shared Responsibility Model
* Root User Security (MFA)
* AWS Budgets and Billing Alerts
* IAM Users
* IAM Groups
* IAM Roles
* IAM Policies
* Principle of Least Privilege
* Customer Managed IAM Policy
* GitHub OIDC Authentication (Optional Challenge)

---

## What I Built

* Enabled MFA for the AWS Root User.
* Created a monthly AWS Budget with email alerts.
* Created IAM Users and Groups with read-only permissions for S3, EC2, and Billing.
* Created a Customer Managed IAM Policy for S3 read-only access.
* Configured GitHub OIDC authentication with AWS IAM Role.
* Successfully authenticated GitHub Actions using temporary AWS credentials without storing access keys.

---

## What I Learned

This week helped me understand the fundamentals of AWS security.

Some of the key concepts I learned are:

* AWS Regions contain multiple Availability Zones.
* AWS follows the Shared Responsibility Model.
* IAM controls who can access AWS resources and what actions they can perform.
* Groups simplify permission management by assigning permissions to multiple users.
* Roles provide temporary access to trusted identities.
* Policies define permissions using JSON.
* Following the Principle of Least Privilege improves security.
* GitHub OIDC allows GitHub Actions to securely access AWS using temporary credentials instead of long-lived access keys.

---

## Where I Got Stuck

While testing the IAM users, I initially expected to see S3 buckets. Since no buckets had been created, I learned that read-only access only allows viewing existing resources. I also spent some time understanding the GitHub OIDC trust policy and role configuration before successfully completing the challenge.

---

## Screenshots Added

### Root MFA Enabled
<img width="1902" height="800" alt="image" src="https://github.com/user-attachments/assets/46a8624c-9e0b-4a0e-a2c0-dc286baebcc2" />


### Billing Budget Alert
<img width="2557" height="785" alt="image" src="https://github.com/user-attachments/assets/963c91a5-d14b-4384-8c19-8334f382f7c1" />

### IAM User Groups
<img width="2547" height="576" alt="image" src="https://github.com/user-attachments/assets/3b252fe7-f97b-4180-b2d3-83bcd7854ae7" />


### IAM Users

<img width="2557" height="507" alt="image" src="https://github.com/user-attachments/assets/c17698a3-b586-43fd-a1cd-4b186e11884c" />

### S3 Read-Only Access
### EC2 Read-Only Access
<img width="2502" height="897" alt="image" src="https://github.com/user-attachments/assets/9bed8b81-6863-4bb0-a74e-962adb1c8dc0" />

### Billing Read-Only Access
<img width="2560" height="1364" alt="image" src="https://github.com/user-attachments/assets/d883897e-c90c-4d62-a4dc-fbe3079a2da0" />
<img width="2560" height="1229" alt="image" src="https://github.com/user-attachments/assets/ceb77c0a-a82c-40b1-8b16-840a54716350" />

### Access Denied Examples
<img width="2560" height="1229" alt="image" src="https://github.com/user-attachments/assets/309c6283-fa7c-468b-b194-61c7a1ef0657" />

<img width="2560" height="1229" alt="image" src="https://github.com/user-attachments/assets/6b34809d-281a-4b38-8303-425676ee9272" />
<img width="2557" height="1166" alt="image" src="https://github.com/user-attachments/assets/c7a30487-f71f-40de-908d-25b96fae4b54" />

### Customer Managed IAM Policy
<img width="2541" height="432" alt="image" src="https://github.com/user-attachments/assets/ef2d1afd-e386-4774-b074-ce672180ceeb" />

### OIDC Identity Provider
<img width="2532" height="777" alt="image" src="https://github.com/user-attachments/assets/a9f649ca-d4f1-422c-a134-2b69e5248ab4" />

### GitHub OIDC IAM Role
<img width="2097" height="942" alt="image" src="https://github.com/user-attachments/assets/dd20a506-63b5-47e3-9a2b-d363c011f853" />

<img width="1487" height="412" alt="image" src="https://github.com/user-attachments/assets/d87c2c0e-bda3-4607-b55b-a93f884ef729" />

### Successful GitHub Actions Workflow
<img width="2557" height="921" alt="image" src="https://github.com/user-attachments/assets/d87980a7-dd9f-4183-97bf-f6f56d91757c" />
<img width="2560" height="1377" alt="image" src="https://github.com/user-attachments/assets/99e6f322-044a-4810-865b-728570666ebc" />



---



## Key Takeaway

The biggest lesson from Week 1 is:

**Identity + Permissions = Access**

Security is the foundation of AWS. Protecting the root user, following the Principle of Least Privilege, and using temporary credentials through IAM Roles and OIDC are essential best practices for building secure cloud environments.

