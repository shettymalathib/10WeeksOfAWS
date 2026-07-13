How to give **only the permissions a user needs** (Least Privilege).

Today  create:

* IAM Groups
* IAM Users
* Attach Policies
* Test what users **can** and **cannot** do

---



 **Use your Root User only to create IAM resources.**

After creating the IAM users,  **sign in as those IAM users** to test their permissions.

---

# Lab 1 – S3 Read-Only Access

## Step 1 – Open IAM

1. Sign in to AWS (Root User).
2. Search for **IAM**.
3. Open **IAM Dashboard**.

<img width="1902" height="817" alt="image" src="https://github.com/user-attachments/assets/11808ba9-64ab-42ce-8c1c-726ce321a8e4" />


---

## Step 2 – Create an IAM Group

Go to:

```
IAM
→ User groups
→ Create group
```

Group Name:

```
S3ReadOnlyGroup
```

Click **Next**.


<img width="1917" height="540" alt="image" src="https://github.com/user-attachments/assets/9e6ad509-808e-4be8-8104-13c68858fcc8" />

<img width="1915" height="437" alt="image" src="https://github.com/user-attachments/assets/7abc1356-deee-4751-a33f-3e7bb4872162" />


---

## Step 3 – Attach Policy

Search for:

```
AmazonS3ReadOnlyAccess
```

Select it.

Click:

```
Create group
```

 Now your group has permission to **view S3 only**.

<img width="2546" height="1061" alt="image" src="https://github.com/user-attachments/assets/ba3efea4-bfe6-44a6-8451-abbf4a813ceb" />


---

## Step 4 – Create IAM User

Go to:

```
IAM
→ Users
→ Create user
```

User name:

```
learner-s3
```

Check:

```
Provide user access to the AWS Management Console
```

Choose:

```
I want to create an IAM user
```

Set a password.

Click **Next**.

---

## Step 5 – Add User to Group

Select:

```
S3ReadOnlyGroup
```

Click:

```
Create User
```


<img width="1917" height="632" alt="image" src="https://github.com/user-attachments/assets/340ce97e-7e91-480a-99b1-b719a016ccd2" />


---

## Step 6 – Save Login Details

AWS will show:

* Console Sign-in URL
* Username
* Password

 **Save these.**

 use them to log in as `learner-s3`.

---

## Step 7 – Test Permissions

Open an **Incognito/Private Window**.

Sign in as:

```
learner-s3
```

Go to:

```
S3
```

I should be able to:

- Access the S3 Console

- View the bucket list (even if it is empty)

If a bucket exists, I should also be able to open it.

<img width="2402" height="917" alt="image" src="https://github.com/user-attachments/assets/ec65c204-77cb-4798-962a-5b8eca268f3b" />

<img width="2535" height="822" alt="image" src="https://github.com/user-attachments/assets/3caa7496-c114-45d0-a00c-2f6b55b39465" />

<img width="2422" height="952" alt="image" src="https://github.com/user-attachments/assets/bd72869c-1902-49ef-97d5-135f1f4af884" />

<img width="2427" height="800" alt="image" src="https://github.com/user-attachments/assets/6f9b651b-1ad7-4679-b082-d5680d344a19" />


---

## Step 8 – Test Denied Access

Try:

```
Create Bucket
```

or

```
Delete Bucket
```

AWS should display:

```
Access Denied 
```

> This confirms that the IAM user has read-only permissions and cannot perform write operations.

<img width="2560" height="3426" alt="image" src="https://github.com/user-attachments/assets/2e35db48-3c91-4800-9da5-d73f92ab8da2" />





---

# Lab 2 – EC2 Read-Only Access

Repeat the same process.

Group:

```
EC2ReadOnlyGroup
```

Policy:

```
AmazonEC2ReadOnlyAccess
```

User:

```
learner-ec2
```
<img width="2527" height="1022" alt="image" src="https://github.com/user-attachments/assets/44be483d-bfdb-4b0a-b325-9deed2e30282" />

<img width="2182" height="447" alt="image" src="https://github.com/user-attachments/assets/b4fcbbc3-4eea-4fba-87d8-5bb3e8a50dd2" />

<img width="2502" height="897" alt="image" src="https://github.com/user-attachments/assets/3a871074-97f9-4a60-b4a1-362e7a0575ca" />

<img width="2421" height="986" alt="image" src="https://github.com/user-attachments/assets/683c4e6a-f955-41ae-a01f-ee6a2146168c" />


---

Test:

Can:

- View EC2 Dashboard

Cannot:

-  Launch Instance

-  Stop Instance

-  Terminate Instance

> This confirms that the attached policy allows viewing EC2 resources but prevents creating, modifying, or deleting them.

<img width="2560" height="1229" alt="image" src="https://github.com/user-attachments/assets/0ab1dde9-b744-481c-abd4-a82ba498aefa" />

<img width="2560" height="1229" alt="image" src="https://github.com/user-attachments/assets/3a5de674-b86a-4e79-a477-539a1e366a24" />


---

# Lab 3 – Billing Read-Only

Again, same process.

Group:

```
BillingViewGroup
```

Policy:

```
AWSBillingReadOnlyAccess
```

User:

```
learner-billing
```

<img width="2560" height="1364" alt="image" src="https://github.com/user-attachments/assets/8dbe5589-0a1f-41ca-b287-690a1731b9e9" />


---

Test:

Can:

 Open Billing Dashboard

<img width="2560" height="1229" alt="image" src="https://github.com/user-attachments/assets/76dfad3b-dba0-4c05-bc47-99e625f6c7bf" />


Cannot:

-  Create EC2

-  Delete S3 Bucket

> The Billing user can only view billing information. Access to other AWS services is restricted.

<img width="2557" height="1166" alt="image" src="https://github.com/user-attachments/assets/9cff0802-6ec7-434f-8cf5-f1ecd00c8048" />


---

# Lab 4 – Custom Policy

Now  create **your own policy** instead of using one provided by AWS.

Go to:

```
IAM
→ Policies
→ Create Policy
```
<img width="2547" height="627" alt="image" src="https://github.com/user-attachments/assets/d6d0c9f2-93a2-45b3-b0d2-a77120b564eb" />

Choose:

```
JSON
```

Paste the JSON from the lab.

```
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["s3:ListAllMyBuckets"], "Resource": "*" },
    { "Effect": "Allow", "Action": ["s3:ListBucket"], "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME" },
    { "Effect": "Allow", "Action": ["s3:GetObject"], "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*" }
  ]
}
```

Replace:

```
YOUR-BUCKET-NAME
```

with your actual bucket name.

Example:

```text
my-training-bucket
```

```
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["s3:ListAllMyBuckets"], "Resource": "*" },
    { "Effect": "Allow", "Action": ["s3:ListBucket"], "Resource": "arn:aws:s3:::my-training-bucket" },
    { "Effect": "Allow", "Action": ["s3:GetObject"], "Resource": "arn:aws:s3:::my-training-bucket/*" }
  ]
}
```

Policy name:

```
CustomS3ReadOnlyTrainingPolicy
```

Create the policy.

<img width="2537" height="1097" alt="image" src="https://github.com/user-attachments/assets/1c7af3dc-eb42-4c21-976a-8e21337f23e2" />


Saved the JSON as:

```
custom-s3-readonly-policy.json
```

inside:

```
week-01/submissions/malathi/04-iam-hands-on-lab/code-or-policies/custom-s3-readonly-policy.json
```

Note:

This policy grants read-only access to a specific S3 bucket.

It allows:

- Listing all buckets
- Listing objects inside the specified bucket
- Reading objects

It does NOT allow uploading or deleting objects.

<img width="2557" height="566" alt="image" src="https://github.com/user-attachments/assets/41d4b36e-f3bc-4b0c-9fa8-e5b6764e9a7b" />


## Key Takeaways

- IAM Groups make permission management easier.
- Attach permissions to Groups instead of individual Users.
- AWS Managed Policies are created by AWS.
- Customer Managed Policies are created by you.
- Least Privilege means giving only the permissions required.
- Always test both Allowed and Denied actions.
