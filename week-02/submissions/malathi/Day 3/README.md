# Week 2 - Day 3: IAM Roles and AWS STS

## Name

Malathi Shetty

---

# Topics Practiced

* IAM Roles
* Trust Policy vs Permission Policy
* Amazon EC2 Instance Profiles
* AWS Security Token Service (STS)
* Temporary Security Credentials
* Amazon S3 Read Access
* AWS CLI
* Principle of Least Privilege

---

# What I Built

I built a secure AWS solution that enables an Amazon EC2 instance to access an Amazon S3 bucket using an IAM Role instead of permanent AWS Access Keys.

First, I created an IAM Role named **Week2Day3EC2S3ReadRole** with a Trust Policy allowing the Amazon EC2 service (`ec2.amazonaws.com`) to assume the role.

Next, I created an Inline IAM Permission Policy that granted only the minimum permissions required on my S3 bucket **malathi-day3-s3-demo-bucket**:

* `s3:ListBucket`
* `s3:GetObject`

The policy intentionally **did not** grant:

* `s3:PutObject`
* `s3:DeleteObject`

to follow the **Principle of Least Privilege**.

Finally, I attached the IAM Role to my Amazon EC2 instance through an Instance Profile. The EC2 instance automatically received temporary credentials from AWS STS, allowing it to securely access Amazon S3 without storing permanent AWS credentials.

---

# Allowed Test

I successfully verified that my Amazon EC2 instance could securely access my Amazon S3 bucket using the attached IAM Role.

## 1. Trust Policy Verification

Verified that the Trust Policy allows the Amazon EC2 service (`ec2.amazonaws.com`) to assume the IAM Role using `sts:AssumeRole`.

<img width="1912" height="740" alt="image" src="https://github.com/user-attachments/assets/cfdc0dcb-e824-48c1-a02c-1d7246267a22" />



---

## 2. Permission Policy Verification

Verified that the Inline IAM Policy grants only:

* `s3:ListBucket`
* `s3:GetObject`

for the bucket **malathi-day3-s3-demo-bucket**.

<img width="1920" height="1396" alt="image" src="https://github.com/user-attachments/assets/84dc6196-e673-4965-a22d-10d6c24f57ae" />


---

## 3. IAM Role Attached to EC2

Verified that the IAM Role **Week2Day3EC2S3ReadRole** was successfully attached to the Amazon EC2 instance through its Instance Profile.

<img width="1562" height="582" alt="image" src="https://github.com/user-attachments/assets/931c6389-af20-421b-9286-4662e882838a" />

---

## 4. Temporary Credentials Verification

Verified that AWS automatically provided temporary credentials to the EC2 instance.

Commands used:

```bash
aws configure list
```

```bash
aws sts get-caller-identity
```

The output confirmed:

* No manually configured AWS credentials.
* Credentials were obtained from the attached IAM Role.
* The caller identity showed an **assumed-role**, confirming that AWS STS issued temporary credentials.

**Screenshots:**

<img width="952" height="242" alt="image" src="https://github.com/user-attachments/assets/ed75c7f8-1d1c-4fec-9301-eb9089e751be" />


---

## 5. Amazon S3 Read Verification

Successfully listed the bucket contents:

```bash
aws s3 ls s3://malathi-day3-s3-demo-bucket
```

Successfully downloaded/read the object:

```bash
aws s3 cp s3://malathi-day3-s3-demo-bucket/day3-test.txt -
```

This confirmed that the IAM Role provided secure read-only access to Amazon S3.

<img width="852" height="120" alt="image" src="https://github.com/user-attachments/assets/387b3e5d-34e3-4cfa-996b-76da46930446" />


---

# Denied Test

To verify the Principle of Least Privilege, I attempted to upload a file to Amazon S3.

Commands used:

```bash
printf 'write test\n' > /tmp/write-test.txt
```

```bash
aws s3 cp /tmp/write-test.txt s3://malathi-day3-s3-demo-bucket/write-test.txt
```

The upload failed with:

```text
AccessDenied
```

This was the expected result because the IAM Policy did **not** include the `s3:PutObject` permission.

This confirmed that:

* The IAM Role allowed only explicitly permitted actions.
* Read operations succeeded.
* Write operations were denied.
* Least Privilege was correctly implemented.

<img width="1370" height="80" alt="image" src="https://github.com/user-attachments/assets/e8ec56f1-e68b-4675-91e8-7dc728fb9547" />


---

# What I Learned

This lab helped me understand how AWS securely provides temporary credentials to an Amazon EC2 instance using IAM Roles and AWS Security Token Service (STS).

### Trust Policy vs Permission Policy

A **Trust Policy** answers the question:

> **Who is allowed to assume this IAM Role?**

In this lab, the Trust Policy allowed the Amazon EC2 service (`ec2.amazonaws.com`) to assume the IAM Role using `sts:AssumeRole`.

A **Permission Policy** answers the question:

> **What actions can the IAM Role perform after it has been assumed?**

In this lab, the Permission Policy allowed only:

* `s3:ListBucket`
* `s3:GetObject`

while denying upload and delete operations.

### Key Learnings

* IAM Roles eliminate the need to store permanent AWS Access Keys.
* AWS STS automatically generates temporary security credentials.
* Instance Profiles connect IAM Roles to Amazon EC2 instances.
* The AWS CLI automatically retrieves credentials from the EC2 Instance Metadata Service (IMDS).
* Following the Principle of Least Privilege improves security by granting only the permissions required.

### Key Concepts

* **IAM Role** → Temporary AWS permissions.
* **Trust Policy** → Defines who can assume the role.
* **Permission Policy** → Defines what the role can do.
* **AWS STS** → Issues temporary credentials.
* **Instance Profile** → Attaches the IAM Role to EC2.
* **Least Privilege** → Grant only the minimum permissions necessary.

---

# Where I Got Stuck

During testing, I expected the contents of `day3-test.txt` to be displayed when I ran:

```bash
aws s3 cp s3://malathi-day3-s3-demo-bucket/day3-test.txt -
```

Instead, the output did not match the expected file contents.

While troubleshooting, I first tried:

```bash
aws s3api head-object
```

AWS returned an error indicating that the required `--bucket` and `--key` parameters were missing. After correcting the command to include the bucket name and object key, I successfully retrieved the object's metadata.

I then downloaded the object locally and inspected its contents. This helped me determine that the IAM Role and permissions were working correctly. The issue was with the content of the uploaded file rather than the IAM configuration.

This experience taught me the importance of reading AWS CLI error messages carefully and verifying both IAM permissions and the actual contents of Amazon S3 objects when troubleshooting.

<img width="777" height="415" alt="image" src="https://github.com/user-attachments/assets/342ad059-a9d4-4976-a947-76443dbec1c6" />


---

# Cleanup

After completing the lab, I deleted all AWS resources to avoid unnecessary charges.

### Amazon EC2

* Terminated the Amazon EC2 instance.

<img width="1887" height="565" alt="image" src="https://github.com/user-attachments/assets/29a50468-7a6d-42de-8efd-57e15f5f370f" />


---

### Amazon S3

* Deleted all objects from the bucket.
* Deleted the S3 bucket **malathi-day3-s3-demo-bucket**.

<img width="1892" height="712" alt="image" src="https://github.com/user-attachments/assets/404551e6-4a96-411e-a4ac-81c6dc580de5" />
<img width="1266" height="647" alt="image" src="https://github.com/user-attachments/assets/1966bd74-d0ae-4b74-85e9-efb8a025b60f" />



---

### IAM

* Deleted the IAM Role **Week2Day3EC2S3ReadRole**.
* Deleted the Inline IAM Permission Policy.

<img width="1906" height="587" alt="image" src="https://github.com/user-attachments/assets/c0a89ac7-c30a-42e1-bbcd-e3b8b1b276e4" />


All resources created during the lab were successfully removed.

---

# Repository Structure

```text
week-02/
└── submissions/
    └── malathi-shetty/
        └── day-3/
            ├── README.md
            ├── policies/
            │   ├── ec2-trust-policy.json
            │   └── s3-read-permission-policy.json
            └── screenshots/
                ├── architecture.png
                ├── role-trust-policy.png
                ├── s3-read-permission-policy.png
                ├── role-attached-to-ec2.png
                ├── aws-configure-list.png
                ├── assumed-role-identity.png
                ├── s3-read-allowed.png
                ├── s3-write-denied.png
                ├── ec2-cleanup.png
                ├── s3-cleanup.png
                └── iam-role-cleanup.png
```

---

## Tasks Completed

* [x] Studied the Day 3 concepts
* [x] Completed the hands-on lab
* [x] Created an IAM Role for Amazon EC2
* [x] Created a Trust Policy and an Inline Permission Policy
* [x] Attached the IAM Role to an EC2 instance
* [x] Verified temporary credentials using AWS STS
* [x] Verified read-only access to Amazon S3
* [x] Verified write access was denied (Least Privilege)
* [x] Captured screenshots
* [x] Cleaned up AWS resources
* [x] Shared my learning on LinkedIn

---

# Architecture Diagram

This lab demonstrates how an Amazon EC2 instance securely accesses an Amazon S3 bucket without storing long-term AWS Access Keys.

Instead of using permanent credentials, the EC2 instance assumes an IAM Role through an Instance Profile. AWS STS automatically generates temporary credentials, allowing the EC2 instance to securely access Amazon S3 while allowing read access and denying write access according to the attached IAM policy.



<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/7c739232-c450-454c-af4f-9d51244ec772" />





---

# LinkedIn Post

LinkedIn Post:

**Paste your LinkedIn post URL here.**
