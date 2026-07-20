#  Day 3 – IAM Roles, STS & Temporary Credentials

# Section 1 – Goal, Before You Start, IAM Role & Principal

---

#  Goal of Day 3

Understand **how an AWS service or application securely accesses another AWS service without storing permanent access keys**.

Instead of using permanent credentials (Access Keys), AWS uses:

* IAM Roles
* AWS STS (Security Token Service)
* Temporary Credentials

This is one of AWS's core security best practices.

---

#  Why is this needed?

Imagine an EC2 server needs to read files from an S3 bucket.

There are two ways to give it access.

### ❌ Option 1: Store Access Keys

```text
EC2
 │
 ├── Access Key
 └── Secret Key
```

Problems:

* Keys can be stolen.
* Keys can accidentally be uploaded to GitHub.
* Keys don't expire automatically.
* Keys must be rotated manually.
* Anyone with the keys can use them until they are revoked.

---

### ✅ Option 2: Use an IAM Role

```text
EC2
 │
 ▼
IAM Role
 │
 ▼
STS
 │
 ▼
Temporary Credentials
 │
 ▼
S3
```

Benefits:

* No permanent keys stored.
* Credentials expire automatically.
* AWS refreshes them automatically.
* Much more secure.

---

#  Before You Start (Answers)

### 1. Why should an application avoid storing access keys?

**Answer:**

Applications should avoid storing permanent access keys because they can be leaked, stolen, or accidentally shared. IAM Roles provide temporary credentials, making applications much more secure.

---

### 2. Who is allowed to assume a role?

**Answer:**

Only **trusted principals** defined in the **Trust Policy** can assume the role.

Think of AWS asking:

> "Do I trust you to use this role?"

If the answer is **Yes**, the role can be assumed.

---

### 3. What is the role allowed to do after it is assumed?

**Answer:**

The role can perform **only the actions allowed by its Permission Policy**. These permissions apply only after the role has been successfully assumed.

Example:

* Read S3 ✅
* Write DynamoDB ✅
* Delete EC2 ❌

The permissions depend entirely on the attached permission policies.

---

### 4. Why do temporary credentials expire?

**Answer:**

Temporary credentials expire to improve security.

Even if someone steals them, they stop working after their expiration time, reducing the risk of misuse.

---

#  What is an IAM Role?

An **IAM Role** is an AWS identity that contains permissions.

Unlike an IAM User, an IAM Role is **not permanently assigned** to one person or application.

Instead, it is **assumed temporarily** whenever it is needed.

Think of it like a visitor pass.

A visitor pass:

* gives temporary access
* works only for a limited time
* expires automatically

An IAM Role works in a similar way.

---

#  Real-Life Analogy

Imagine a company.

Employees have permanent ID cards.

Visitors receive temporary visitor badges.

The visitor badge:

* allows entry for a limited time
* grants only specific permissions
* expires automatically

An IAM Role is like that visitor badge.

AWS gives temporary access instead of permanent credentials.

---

# Why do we use IAM Roles?

Instead of placing permanent Access Keys inside an application,

AWS recommends:

```text
Application
      │
      ▼
IAM Role
      │
      ▼
Temporary Credentials
```

This follows AWS security best practices.

---

#  What is a Principal?

A **Principal** is any identity requesting access to AWS resources.

Simply remember:

> **Principal = Who is asking for permission?**

Examples of principals include:

* AWS Service (EC2, Lambda, ECS)
* IAM User
* IAM Role
* Another AWS Account
* Federated Identity (users authenticated through an external identity provider)

---

# Example

Suppose an EC2 instance wants to read files from S3.

Who is requesting access?

**EC2**

Therefore,

The **EC2 service is the trusted principal** in the Trust Policy. When an EC2 instance is launched with the role attached, applications on that instance receive credentials for the assumed role.

If a Lambda function requests access,

then **Lambda becomes the Principal.**

---

#  Quick Memory Trick

```text
Principal
=
Identity requesting permission
```

---

#  Key Takeaways

* IAM Roles provide temporary permissions instead of permanent credentials.
* Applications should not store Access Keys.
* A Principal is the identity requesting access.
* Only trusted principals can assume an IAM Role.
* After assuming the role, the principal can perform only the actions allowed by the Permission Policy.
* Temporary credentials expire automatically, making AWS more secure.

---

# ✅ Quick Revision

| Concept               | Remember This                                                     |
| --------------------- | ----------------------------------------------------------------- |
| IAM Role              | An AWS identity with permissions that is assumed temporarily.     |
| Principal             | The identity requesting access.                                   |
| Permanent Access Keys | Long-lived credentials that should not be stored in applications. |
| Temporary Credentials | Short-lived credentials generated by AWS for secure access.       |
| Goal of IAM Roles     | Secure AWS access without storing permanent credentials.          |

---

##  Why Didn't We See These in the AWS Console?

At this stage, **nothing is happening automatically yet** because we have only learned the concepts.

We haven't:

* launched an EC2 instance,
* attached the role to a running workload, or
* requested access to another AWS service.

So concepts like **STS**, **temporary credentials**, and **automatic credential refresh** remain **behind the scenes** for now. We'll understand and connect those pieces in the next sections.

---

***

# Section 2 – Trust Policy vs Permission Policy

---

#  Why do we need two policies?

Imagine someone comes to your house.

Before opening the door, you ask:

> **"Who are you?"**

If you trust them, you let them enter.

After they enter, they ask:

> **"Can I use your kitchen?"**

You say **Yes**.

Then they ask:

> **"Can I enter your bedroom?"**

You say **No**.

Notice something.

There are **two different decisions**.

### Decision 1

> **Who is allowed to enter?**

This is the **Trust Policy**.

---

### Decision 2

> **What are they allowed to do after entering?**

This is the **Permission Policy**.

AWS works exactly the same way.

---

# 🤝 Trust Policy

A **Trust Policy** answers one question:

> **WHO is allowed to assume (use) this IAM Role?**

It does **not** decide what the role can do.

It only decides **who is trusted**.

---

## Example Trust Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Don't worry about the brackets (`{}`, `[]`). Focus on the important parts.

---

# 📖 Line-by-Line Explanation

## 1. Version

```json
"Version": "2012-10-17"
```

This is the **policy language version**, **not today's date**.

AWS uses this version to understand how to interpret the policy.

You normally leave it unchanged.

---

## 2. Statement

```json
"Statement"
```

A **Statement** contains one or more permission rules.

Think of it as:

> "Here are the rules for this policy."

---

## 3. Effect

```json
"Effect": "Allow"
```

Meaning:

AWS is allowing something.

The other possible value is:

```json
"Effect": "Deny"
```

which explicitly blocks access.

---

## 4. Principal

```json
"Principal"
```

This answers:

> **Who is requesting access?**

A Principal can be:

* EC2
* Lambda
* IAM User
* IAM Role
* Another AWS Account
* Federated Identity

In this example:

```json
"Principal": {
    "Service": "ec2.amazonaws.com"
}
```

AWS trusts **only the EC2 service**.

---

## 5. Service

```json
"Service": "ec2.amazonaws.com"
```

Meaning:

Only EC2 is allowed to assume this role.

If Lambda needed the role, it would be:

```json
"Service": "lambda.amazonaws.com"
```

---

## 6. Action

```json
"Action": "sts:AssumeRole"
```

This is one of the most misunderstood lines.

It **does NOT mean**

> Access S3

It means

> **Allow the trusted principal to assume (use) this IAM Role.**

Think of it like:

```text
sts:AssumeRole
=
Permission to wear/use this role
```

---

#  Common Beginner Mistake

Many people think:

> EC2 is trusted, so it can automatically read S3.

❌ Wrong.

The Trust Policy **does not grant S3 permissions.**

It only answers:

> **Who can use this role?**

---

#  Permission Policy

After AWS decides **who** can use the role,

it asks another question:

> **What is the role allowed to do?**

This is answered by the **Permission Policy**.

---

## Example Permission Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

---

# 📖 Line-by-Line Explanation

## Version

Same meaning as before.

It tells AWS which policy language version is being used.

---

## Effect

```json
"Effect":"Allow"
```

AWS allows the actions listed below.

---

## Action

### 1.

```json
"s3:ListBucket"
```

Allows the role to:

* View the bucket
* List the objects inside it

---

### 2.

```json
"s3:GetObject"
```

Allows the role to:

* Read objects
* Download objects

It **does not** allow:

* Upload objects
* Delete objects

---

## Resource

```json
"arn:aws:s3:::YOUR-BUCKET-NAME"
```

This refers to the **bucket itself**.

That's why it's used with:

```text
s3:ListBucket
```

because you're listing the bucket.

---

Next:

```json
"arn:aws:s3:::YOUR-BUCKET-NAME/*"
```

The `/*` means:

> **Every object inside the bucket.**

That's why it's used with:

```text
s3:GetObject
```

because you're reading individual objects.

---

#  Why are there two Resources?

Without `/*`

```text
Bucket
```

With `/*`

```text
Bucket
 ├── photo.jpg
 ├── report.pdf
 ├── video.mp4
```

The `/*` represents **all objects inside the bucket**.

---

#  Trust Policy vs Permission Policy

| Trust Policy                    | Permission Policy                            |
| ------------------------------- | -------------------------------------------- |
| Answers **WHO**                 | Answers **WHAT**                             |
| Defines who can assume the role | Defines what the role can do                 |
| Contains `Principal`            | Contains `Action` and `Resource`             |
| Uses `sts:AssumeRole`           | Uses AWS service actions like `s3:GetObject` |

---

#  What We Did in the AWS Console

When we clicked:

**IAM → Roles → Create Role**

AWS asked:

> **Who should use this role?**

We selected:

✅ AWS Service

Then:

✅ EC2

This automatically created the **Trust Policy**.

---

Next AWS asked:

> **What permissions should this role have?**

We selected:

```text
AmazonS3ReadOnlyAccess
```

This attached a **Permission Policy** that allows read-only access to S3.

---

# Why Didn't We Write JSON Manually?

Because the AWS Console generated it for us.

Behind the scenes, AWS created:

* A Trust Policy
* A Permission Policy

If you use AWS CLI, CloudFormation, Terraform, or the IAM JSON editor, you'll often work directly with these JSON policies.

---

#  Complete Flow So Far

```text
EC2
 │
 │ Wants to use the role
 ▼
Trust Policy
 │
 │ "Do I trust EC2?"
 ▼
Yes ✅
 │
 ▼
Permission Policy
 │
 │ "What is EC2 allowed to do?"
 ▼
Read S3 only ✅
```

At this point, AWS knows:

* **WHO** can use the role.
* **WHAT** the role can do.

The next step (covered in Section 3) is where **AWS STS creates temporary credentials**.

---

#  Common Beginner Mistakes

❌ Thinking Trust Policy gives S3 access.

✅ It only decides **WHO** can assume the role.

---

❌ Thinking `sts:AssumeRole` means "access S3."

✅ It only means:

> "Allow this trusted principal to use the role."

---

❌ Thinking `s3:GetObject` allows uploads.

✅ It only allows **reading/downloading** objects.

---

❌ Thinking `arn:aws:s3:::bucket-name` and `arn:aws:s3:::bucket-name/*` are the same.

✅ They are different:

* `bucket-name` → The bucket itself.
* `bucket-name/*` → Every object inside the bucket.

---

#  Quick Revision

| Concept           | Remember This                                  |
| ----------------- | ---------------------------------------------- |
| Trust Policy      | **WHO can assume the role?**                   |
| Permission Policy | **WHAT can the role do?**                      |
| Principal         | The identity requesting access.                |
| `sts:AssumeRole`  | Allows a trusted principal to assume the role. |
| `s3:ListBucket`   | List/view the bucket.                          |
| `s3:GetObject`    | Read/download objects.                         |
| `bucket-name`     | Refers to the bucket itself.                   |
| `bucket-name/*`   | Refers to all objects in the bucket.           |

***

# Section 3 – AWS STS & Temporary Credentials

---

#  What is AWS STS?

**STS** stands for **AWS Security Token Service**.

It is an AWS service that creates **temporary security credentials**.

Instead of giving an application permanent Access Keys, STS generates credentials that:

* Work for a limited time.
* Expire automatically.
* Can be refreshed when needed.

This is much safer than storing permanent credentials.

---

#  Why does AWS use STS?

Imagine your friend needs to borrow your laptop for one hour.

Would you give them your laptop forever?

❌ No.

Instead, you let them use it for one hour.

After one hour, they must return it.

AWS follows the same idea.

Instead of permanent credentials, AWS gives **temporary credentials**.

---

#  Complete STS AssumeRole Flow

Let's connect everything we've learned.

### Step 1 – A Principal Needs Access

Example:

An EC2 instance wants to read files from an S3 bucket.

```text
EC2
 │
 │ "I need to read files from S3."
 ▼
IAM Role
```

Here:

* **Principal = EC2**
* **Requested Role = IAM Role**

---

### Step 2 – AWS Checks the Trust Policy

AWS asks:

> "Is EC2 allowed to use this role?"

It checks the Trust Policy.

Example:

```json
"Principal": {
   "Service": "ec2.amazonaws.com"
}
```

If EC2 is trusted,

✅ AWS continues.

If not,

❌ Access is denied.

---

### Step 3 – AWS Checks the Permission Policy

Now AWS asks:

> "If EC2 gets this role, what is it allowed to do?"

Example:

```json
"s3:GetObject"
```

Meaning:

EC2 can read S3 objects.

It cannot upload or delete unless those permissions are also granted.

---

### Step 4 – STS Creates Temporary Credentials

Now AWS Security Token Service (STS) creates a temporary session.

These credentials belong to the **role**, not to an IAM User.

STS returns:

* Access Key ID
* Secret Access Key
* Session Token
* Expiration Time

These four values together are called **temporary credentials**.

---

### Step 5 – EC2 Uses the Temporary Credentials

EC2 now uses those credentials to call AWS APIs.

Example:

```text
EC2
 │
 ▼
Temporary Credentials
 │
 ▼
Amazon S3
 │
 ▼
Read Object
```

Notice:

EC2 never stores permanent Access Keys.

---

### Step 6 – Credentials Expire

After some time,

the temporary credentials expire automatically.

This limits the damage if they are ever stolen.

---

### Step 7 – New Credentials Are Obtained

If the application is using the AWS SDK or AWS CLI correctly,

The AWS SDK or CLI automatically retrieves and refreshes temporary credentials when required.

The application keeps working without storing permanent keys.

---

#  Complete Flow Diagram

```text
EC2
 │
 ▼
Requests AssumeRole 
 │ Wants S3 access
 ▼
IAM checks Trust Policy
 │
 ▼
Trust Policy
 │
 │ Is EC2 trusted?
 ▼
Yes
 │
 ▼
Permission Policy
 │
 │ What can EC2 do?
 ▼
Read S3
 │
 ▼
AWS STS issues credentials
 │
 ▼
Creates Temporary Credentials
 │
 ▼
Application calls S3
 │
 ▼
Application (EC2) Uses Credentials

 │
 ▼
IAM evaluates Permission Policy

 │
 ▼
S3 responds
 │
 ▼
 
Reads S3
 │
 ▼
Credentials Expire
 │
 ▼
AWS Automatically Refreshes Them
```

---

#  What Does STS Return?

When STS creates temporary credentials, it returns four things.

### 1. Access Key ID

Identifies the temporary credentials.

Think of it like a username.

---

### 2. Secret Access Key

Works like a password.

It proves that the request is authentic.

---

### 3. Session Token

This exists **only for temporary credentials**.

It tells AWS:

> "These credentials came from STS and belong to a temporary session."

Without the session token, temporary credentials won't work.

---

### 4. Expiration Time

Every temporary credential has an expiry time.

Example:

```text
Valid Until

4:30 PM
```

After that time,

AWS stops accepting those credentials.

---

#  Permanent Credentials vs Temporary Credentials

| Permanent IAM User Credentials | Temporary STS Credentials |
| ------------------------------ | ------------------------- |
| Access Key ID                  | Access Key ID             |
| Secret Access Key              | Secret Access Key         |
| ❌ No Session Token             | ✅ Session Token           |
| ❌ No Automatic Expiration      | ✅ Expiration Time         |
| Long-lived                     | Short-lived               |

---

# Why Didn't We See These Credentials?

Many beginners ask:

> "If STS creates credentials, where are they?"

You didn't see them because:

* We created the IAM Role only.
* We never launched an EC2 instance.
* Nothing actually assumed the role.

Without a running workload, STS has nothing to generate.

---

# Why Didn't We Click an STS Button?

Because there isn't one.

STS works automatically.

AWS calls STS behind the scenes whenever a trusted principal assumes a role.

Most AWS users never interact directly with STS.

---

#  Why Do Temporary Credentials Expire?

Imagine someone steals your credentials.

If they are permanent,

the attacker can keep using them until you manually revoke them.

If they are temporary,

they stop working automatically after they expire.

This greatly reduces the security risk.

---

#  Why Temporary Credentials Are Safer

Instead of this:

```text
Application
 │
 └── Permanent Access Keys ❌
```

AWS recommends:

```text
Application
 │
 ▼
IAM Role
 │
 ▼
STS
 │
 ▼
Temporary Credentials ✅
```

---

#  Real-Life Example

Imagine an employee badge.

A permanent employee badge works every day.

A visitor badge works only until the visitor leaves.

IAM User credentials are like a permanent badge.

STS credentials are like a visitor badge.

---

#  Common Beginner Mistakes

❌ Thinking STS is another type of IAM Role.

✅ STS is a **service** that creates temporary credentials.

---

❌ Thinking IAM Roles contain Access Keys.

✅ IAM Roles contain **permissions**, not credentials.

STS creates the credentials **when the role is assumed**.

---

❌ Thinking temporary credentials must be created manually.

✅ AWS creates them automatically when a trusted principal assumes the role.

---

❌ Thinking expired credentials mean the application stops.

✅ If you're using the AWS SDK or CLI, new temporary credentials are automatically obtained.

---

#  Quick Revision

| Concept               | Remember This                                                        |
| --------------------- | -------------------------------------------------------------------- |
| AWS STS               | Creates temporary security credentials.                              |
| Temporary Credentials | Access Key ID + Secret Access Key + Session Token + Expiration Time. |
| Session Token         | Makes temporary credentials valid.                                   |
| Expiration Time       | Automatically limits how long credentials can be used.               |
| Trust Policy          | Decides **WHO** can assume the role.                                 |
| Permission Policy     | Decides **WHAT** the role can do.                                    |
| STS                   | STS creates credentials **after** the trust policy allows the role to be assumed. Permission policies are evaluated later when AWS API requests are made.         |

---

#  What We Learned in This Section

After a trusted principal (like EC2) requests to use an IAM Role:

1. AWS checks the **Trust Policy**.
2. AWS checks the **Permission Policy**.
3. AWS STS creates **temporary credentials**.
4. The workload uses those credentials to call AWS services.
5. The credentials expire automatically.
6. The AWS SDK or CLI can automatically obtain fresh credentials when needed.

***

# Section 4 – Instance Profiles, EC2 Roles & Credential Management

---

#  What is an Instance Profile?

An **Instance Profile** is a container that makes an **IAM Role available to an EC2 instance**.

Think of it like a delivery box.

* **IAM Role** = The permission card.
* **Instance Profile** = The box that delivers the permission card to EC2.

Without the Instance Profile, EC2 cannot use the IAM Role.

---

#  Simple Analogy

Imagine a company.

* **Employee ID Card** = IAM Role
* **Envelope carrying the ID Card** = Instance Profile
* **Employee** = EC2

The employee receives the ID card through the envelope.

Similarly, EC2 receives the IAM Role through the Instance Profile.

---

#  Important Difference

| IAM Role                             | Instance Profile               |
| ------------------------------------ | ------------------------------ |
| Contains Trust Policy                | Does not contain policies      |
| IAM Roles have permission policies attached to them.          | Does not contain permissions   |
| Defines what the role can do         | a container that associates an IAM Role with an EC2 instance.       |
| Can be assumed by trusted principals | Is attached to an EC2 instance |

---

#  How It Works

```text
IAM Role
   │
   ▼
Instance Profile
   │
   ▼
EC2 Instance
```

EC2 receives the IAM Role through the Instance Profile. Applications running on the instance obtain temporary credentials through the Instance Metadata Service (IMDS).

The Instance Profile acts as the bridge.

---

# Why Didn't We Create an Instance Profile?

Because the AWS Console creates it automatically.

When we selected:

```text
IAM
↓
Roles
↓
Create Role
↓
AWS Service
↓
EC2
```

AWS automatically created and managed the Instance Profile for us.

That's why you never saw a separate screen asking you to create one.

---

#  What Happens When EC2 Starts?

Suppose an EC2 instance has an IAM Role attached.

When it starts:

1. EC2 receives the IAM Role through the Instance Profile.
2. When the application needs AWS access, it asks for credentials.
3. AWS STS generates temporary security credentials for the assumed role because It emphasizes that the credentials belong to the role session, not to the principal itself.
4. The application uses them to access AWS services.

Everything happens automatically.

---

#  AWS Credential Provider Chain

This sounds complicated, but the idea is simple.

When an application needs AWS credentials, it looks in several places in order.

This search order is called the **Credential Provider Chain**.

For an EC2 instance with an IAM Role, the important part is:

### what the application sees

```text
Application on EC2
        │
        ▼
AWS SDK / AWS CLI
        │
        ▼
Instance Metadata Service (IMDS)
        │
        ▼
Receives Temporary Credentials
        │
        ▼
Application Calls AWS Service
        │
        ▼
Amazon S3 / DynamoDB / etc.
```

### Behind the scenes - what AWS does internally
```text
IAM Role
      │
      ▼
Instance Profile
      │
      ▼
EC2 Instance
      │
      ▼
AWS STS
      │
Issues Temporary Credentials
      │
      ▼
IMDS
      │
      ▼
AWS SDK / AWS CLI
```


You don't have to remember every step today—just know that the AWS SDK and CLI know where to look for credentials.
> Behind the scenes, IMDS obtains these temporary credentials for the IAM Role attached to the EC2 instance (through the Instance Profile) using AWS STS.

---

#  What is the Instance Metadata Service (IMDS)?

The **Instance Metadata Service (IMDS)** is a special service that runs on every EC2 instance.

It provides information about the instance, including temporary credentials for the attached IAM Role.

- Applications don't usually call AWS STS directly.
- Instead, they request credentials from the Instance Metadata Service (IMDS).
- IMDS provides temporary credentials for the IAM Role attached to the EC2 instance through its Instance Profile.

---

#  Automatic Credential Refresh

Temporary credentials expire.

Does the application stop working?

❌ No.

The AWS SDK and AWS CLI automatically:

1. Check whether the credentials are about to expire.
2. Request fresh temporary credentials.
3. Continue working without interruption.

This happens behind the scenes.

---

# Why Didn't We See Automatic Refresh?

Because we never launched an EC2 instance.

We also didn't:

* Install the AWS CLI.
* Write an application using the AWS SDK.

Without a running workload, there was nothing to refresh.

---

#  Never Store Permanent Access Keys

Your original notes listed places where permanent keys should **never** be stored.

Let's understand why.

### ❌ User Data

User Data is a startup script that runs when an EC2 instance launches.

If you place Access Keys here:

* Anyone with permission to view User Data might see them.
* They become part of the instance configuration.

---

### ❌ Environment Files

Example:

```text
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
```

Problems:

* Easy to accidentally expose.
* May be included in backups or logs.

---

### ❌ Application Source Code

Never write credentials like this:

```python
ACCESS_KEY = "AKIA..."
SECRET_KEY = "abcd1234..."
```

If the code is shared or pushed to GitHub, the credentials are exposed.

---

### ❌ AMIs (Amazon Machine Images)

If Access Keys are saved inside an AMI, every EC2 instance created from that AMI will contain the same credentials.

This is a serious security risk.

---

### ❌ Shell History

Commands entered in a terminal may be stored in shell history.

If you type credentials directly into commands, they could be visible later.

---

# ✅ Best Practice

Instead of storing keys anywhere,

attach an IAM Role to the EC2 instance.

AWS will automatically provide temporary credentials whenever they are needed.

---

#  Real-Life Example

Imagine a hotel.

Instead of giving every guest a master key,

the hotel gives each guest a room key card.

The key card:

* Opens only the assigned room.
* Stops working after checkout.

IAM Roles work the same way.

Applications receive temporary access instead of permanent keys.

---

#  Common Beginner Mistakes

❌ Thinking an Instance Profile is another type of IAM Role.

✅ It is simply a container that delivers the role to EC2.

---

❌ Thinking applications should manually store credentials.

✅ Applications should use IAM Roles and let the AWS SDK or CLI obtain credentials automatically.

---

❌ Thinking expired credentials require manual replacement.

✅ The AWS SDK and CLI refresh them automatically.

---

❌ Thinking IMDS is the same as STS.

✅ They are different:

* **STS** creates temporary credentials.
* **IMDS** makes those credentials available to applications running on EC2.

---

#  Quick Revision

| Concept           | Remember This                                                                           |
| ----------------- | --------------------------------------------------------------------------------------- |
| Instance Profile  | Connects an IAM Role to an EC2 instance.                                                |
| EC2 Role          | An IAM Role attached to an EC2 instance (through an Instance Profile).                  |
| AWS SDK / CLI     | Automatically retrieves and refreshes temporary credentials.                            |
| IMDS              | Provides temporary credentials and instance information to applications running on EC2. |
| Automatic Refresh | New temporary credentials are obtained before the old ones expire.                      |
| Best Practice     | Never store permanent Access Keys inside applications or EC2 instances.                 |

---

#  Complete EC2 Credential Flow

```text
Application on EC2
        │
        ▼
AWS SDK / AWS CLI
        │
        ▼
Instance Metadata Service (IMDS)
        │
        ▼
Instance Profile
        │
        ▼
IAM Role
        │
        ▼
AWS STS
        │
Creates Temporary Credentials
        │
        ▼
Application Uses AWS Services (S3, DynamoDB, etc.)
        │
        ▼
Credentials Expire
        │
        ▼
AWS SDK Automatically Refreshes Them
```

Note:
**One small clarification:** your original notes mentioned the **"standard AWS credential provider chain"** but didn't mention **IMDS** by name. I included IMDS because it's the mechanism EC2 uses to provide role credentials to applications. It's useful to know, but for a Day 3 beginner, you mainly need to remember: **AWS SDK/CLI automatically gets and refreshes credentials from the IAM Role attached to EC2.**

***

# Section 5 – Cross-Service Role Assumption, Hands-on Recap & Final Revision

---

#  What is Cross-Service Role Assumption?

Cross-Service Role Assumption means:

> **One AWS service uses an IAM Role to securely access another AWS service.**

AWS Service assuming a role to access another AWS service. Instead of storing Access Keys, AWS services assume an IAM Role and use temporary credentials provided by STS.

---

## Simple Example

Imagine an EC2 instance needs to read a file from an S3 bucket.

```text
EC2
 │
 ▼
Assumes IAM Role
 │
 ▼
AWS STS
 │
 ▼
Temporary Credentials
 │
 ▼
S3
```

Here:

* EC2 is the **calling service**.
* S3 is the **target service**.
* IAM Role provides permissions.
* STS provides temporary credentials.

---

# More Examples

### Example 1 – EC2 → S3

An application running on EC2 needs to download files from an S3 bucket.

Trust Policy:

```text
Trust EC2
```

Permission Policy:

```text
Allow s3:GetObject
Allow s3:ListBucket
```

Result:

EC2 can read files from S3.

---

### Example 2 – Lambda → CloudWatch Logs

A Lambda function automatically writes logs.

Trust Policy:

```text
Trust Lambda
```

Permission Policy:

```text
Allow CloudWatch Logs actions
```

Result:

Lambda writes logs to CloudWatch.

---

### Example 3 – Step Functions → Lambda

A Step Functions workflow needs to invoke a Lambda function.

Trust Policy:

```text
Trust Step Functions
```

Permission Policy:

```text
Allow lambda:InvokeFunction
```

Result:

Step Functions can invoke Lambda.

---

### Example 4 – ECS → Secrets Manager

An ECS task needs to read a database password.

Trust Policy:

```text
Trust ECS
```

Permission Policy:

```text
Allow secretsmanager:GetSecretValue
```

Result:

The application securely reads the secret.

---

# What Changes Between Examples?

Only **two things** usually change:

### 1. Trust Policy (WHO)

Example:

```text
EC2
```

or

```text
Lambda
```

or

```text
ECS
```

The trusted service changes depending on who needs the role.

---

### 2. Permission Policy (WHAT)

Example:

Instead of

```text
Read S3
```

you might allow

```text
Write CloudWatch Logs
```

or

```text
Read Secrets Manager
```

The permissions change depending on what the workload needs to do.

---

#  The Pattern Never Changes

No matter which AWS services are involved, AWS always follows the same process.

```text
Service
      │
      ▼
Trust Policy
(WHO?)
      │
      ▼
Permission Policy
(WHAT?)
      │
      ▼
STS
Creates Temporary Credentials
      │
      ▼
Target AWS Service
```

This pattern is used across almost all AWS services.

---

# Why Didn't We Build These Examples?

Today's hands-on focused only on creating an IAM Role.

We did **not** create:

* EC2 instance
* Lambda function
* ECS task
* Step Functions workflow

Because each of these services has its own lesson.

Today's goal was to understand **how IAM Roles work**, not how every AWS service uses them.

---

#  Hands-on Recap

Let's review exactly what we did in the AWS Console.

---

## Step 1

Opened:

```text
IAM
↓
Roles
↓
Create Role
```

Purpose:

Create a new IAM Role.

---

## Step 2

Selected:

```text
Trusted Entity
↓
AWS Service
```

Why?

Because the role will be used by an AWS service, not by another AWS account or an IAM user.

---

## Step 3

Selected:

```text
EC2
```

Why?

Because we wanted an EC2 instance to use this role.

AWS automatically created a Trust Policy similar to:

```json
{
  "Principal": {
    "Service": "ec2.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
}
```

Meaning:

> EC2 is trusted to assume this role.
> This action appears in the Trust Policy because assuming a role is an STS operation.
> That helps connect IAM and STS.

---

## Step 4

Attached Permission:

```text
AmazonS3ReadOnlyAccess
```

Why?

Because we wanted EC2 to read objects from S3.

This permission allows:

* List buckets
* Read objects

It does **not** allow:

* Upload objects
* Delete objects

---

## Step 5

Created the Role.

Behind the scenes AWS also created an **Instance Profile** for EC2.

You didn't have to create it manually.

---

#  What Happened Behind the Scenes?

Even though you only clicked a few buttons, AWS performed several actions.

```text
You Created IAM Role
        │
        ▼
AWS Created Trust Policy
        │
        ▼
AWS Attached Permission Policy
        │
        ▼
AWS Created Instance Profile
```

Later, when this role is attached to an EC2 instance:

```text
EC2 Starts
      │
      ▼
Assumes IAM Role
      │
      ▼
AWS STS Generates Temporary Credentials
      │
      ▼
EC2 Uses Those Credentials
      │
      ▼
Reads S3
```

This is why you didn't see STS or temporary credentials during the role creation process.

---

#  Key Line (Most Important)

Memorize this:

> **Trust Policy answers WHO. Permission Policy answers WHAT. STS supplies short-lived credentials. Instance Profile associates an IAM Role with an EC2 instance.**

If you remember this sentence, you'll remember the entire lesson.

---

#  Check Your Understanding (Answers)

### 1. Does an EC2 Trust Policy grant permission to read an S3 object?

**Answer:**

❌ No.

The Trust Policy only allows EC2 to assume the role.

The Permission Policy grants S3 access.

---

### 2. What must change if Lambda, rather than EC2, needs to assume the role?

**Answer:**

The Trust Policy must trust Lambda instead of EC2.

Example:

```text
ec2.amazonaws.com
```

becomes

```text
lambda.amazonaws.com
```

The Permission Policy may stay the same if Lambda needs the same S3 permissions.

---

### 3. Why is an Instance Role safer than storing IAM User Access Keys on EC2?

**Answer:**

Because IAM Roles use temporary credentials that:

* expire automatically,
* are generated only when needed,
* don't require permanent Access Keys to be stored on the EC2 instance.

---

### 4. What happens when temporary credentials expire?

**Answer:**

The AWS SDK or CLI automatically obtains new temporary credentials through the IAM Role, so the application can continue working without manual intervention.

---

#  Common Beginner Mistakes

❌ Trust Policy gives S3 access.

✅ No. It only answers **WHO** can assume the role.

---

❌ STS is an IAM Role.

✅ No. STS is an AWS service that creates temporary credentials.

---

❌ Instance Profile contains permissions.

✅ No. The IAM Role contains permissions. The Instance Profile simply makes the role available to EC2.

---

❌ Temporary credentials are permanent.

✅ No. They expire automatically and are refreshed when needed.

---

#  Cheat Sheet

| Question                                | Answer                                                                           |
| --------------------------------------- | -------------------------------------------------------------------------------- |
| What is an IAM Role?                    | An AWS identity with permissions that is assumed temporarily.                    |
| What is a Principal?                    | The identity requesting access.                                                  |
| What does the Trust Policy answer?      | WHO can assume the role?                                                         |
| What does the Permission Policy answer? | WHAT can the role do?                                                            |
| What does STS do?                       | Creates temporary credentials.                                                   |
| What are temporary credentials?         | Access Key ID, Secret Access Key, Session Token, and Expiration Time.            |
| What is an Instance Profile?            | A container that makes an IAM Role available to an EC2 instance.                 |
| Why are IAM Roles safer?                | No permanent credentials are stored; temporary credentials expire automatically. |

---


##  Final Trick

Whenever you're confused, remember this simple sequence:

```text
Application
      │
      ▼
Needs AWS Access
      │
      ▼
IAM Role
      │
      ├── Trust Policy → WHO can use the role?
      │
      └── Permission Policy → WHAT can the role do?
      │
      ▼
AWS STS
      │
Creates Temporary Credentials
      │
      ▼
Instance Profile (for EC2)
      │
      ▼
Application accesses AWS service
      │
      ▼
Credentials expire and AWS SDK/CLI refreshes them automatically
```

***

# Section 6 – Final Revision &  Notes

---

#  Complete Story (Understand the Big Picture)

Imagine you have an EC2 instance that needs to read a file from an S3 bucket.

Instead of storing permanent Access Keys inside EC2, AWS uses an IAM Role.

The complete process looks like this:

```text
EC2 Instance
      │
      ▼
Application needs S3 access
      │
      ▼
AWS SDK / AWS CLI
      │
      ▼
Requests temporary credentials
      │
      ▼
IAM checks Trust Policy
      │
      │ Is EC2 trusted to assume the role?
      ▼
Yes
      │
      ▼
AWS STS issues temporary credentials
      │
      ▼
Access Key ID
Secret Access Key
Session Token
Expiration Time
      │
      ▼
Application calls S3 API
      │
      ▼
IAM evaluates the Permission Policy
      │
      │ Is s3:GetObject allowed?
      ▼
Yes
      │
      ▼
Amazon S3 returns the object
      │
      ▼
Credentials expire
      │
      ▼
AWS SDK / CLI automatically refreshes them
```

STS doesn't ask:

> "Can this role read S3?"

STS only cares about **issuing temporary credentials after the role can be assumed** (based on the trust relationship).

The **Permission Policy** is evaluated **later**, when the application actually calls an AWS API such as `GetObject`.

Think of it like this:

```text
Phase 1
Can I wear this badge?
        ↓
Trust Policy
        ↓
STS gives badge

-----------------------

Phase 2
Now I'm wearing the badge.

Can I enter Room 101?
        ↓
Permission Policy
        ↓
Access granted/denied
```

This summarizes how IAM Roles, Trust Policies, Permission Policies, and AWS STS work together to provide secure access to AWS resources.

---

#  The Four Questions You Should Always Ask

Whenever you see an IAM Role, ask these four questions.

### Question 1

**Who wants access?**

Answer:

The **Principal**.

Example:

* EC2
* Lambda
* ECS
* IAM User
* Another AWS Account

---

### Question 2

**Who is allowed to use the role?**

Answer:

The **Trust Policy**.

Remember:

> Trust Policy = WHO

---

### Question 3

**What can the role do?**

Answer:

The **Permission Policy**.

Remember:

> Permission Policy = WHAT

---

### Question 4

**How does AWS securely provide credentials?**

Answer:

AWS STS creates temporary credentials.

---

#  Everything Connected

```text
EC2 Instance
      │
      ▼
Instance Profile
      │
      ▼
IAM Role
      │
      ▼
Trust Policy
      │
      ▼
AWS STS
      │
      ▼
Temporary Credentials
      │
      ▼
Application
      │
      ▼
Calls Amazon S3
      │
      ▼
Permission Policy evaluated
```

---

# 📚 Quick Definitions

## IAM Role

An AWS identity with permissions that is assumed temporarily by a trusted principal.

---

## Principal

The identity requesting access to AWS resources.

Examples:

* EC2
* Lambda
* IAM User
* Another AWS Account
* Federated User

---

## Trust Policy

Defines **who** can assume the role.

Contains:

* Principal
* `sts:AssumeRole`

---

## Permission Policy

Defines **what** the role can do after it is assumed.

Contains:

* Actions
* Resources

---

## AWS STS

AWS Security Token Service.

Creates temporary credentials whenever a trusted principal assumes a role.

---

## Temporary Credentials

Contain:

* Access Key ID
* Secret Access Key
* Session Token
* Expiration Time

Unlike IAM User credentials, they expire automatically.

---

## Instance Profile

A container that makes an IAM Role available to an EC2 instance.

AWS automatically creates it when you create an EC2 role through the console.

---

## AWS SDK / CLI

Automatically:

* Retrieves temporary credentials.
* Refreshes them before they expire.

---

#  Why IAM Roles Are Better Than IAM User Access Keys

| IAM User Access Keys | IAM Role            |
| -------------------- | ------------------- |
| Permanent            | Temporary           |
| Must be stored       | No storage needed   |
| Manual rotation      | Automatic refresh   |
| Higher security risk | Lower security risk |
| Can leak             | Much safer          |

---

#  Never Store Permanent Access Keys In

❌ User Data

❌ Environment Files

❌ Application Source Code

❌ AMIs

❌ Shell History

Instead, attach an IAM Role to the workload.

---

#  Cross-Service Role Assumption

One AWS service assumes an IAM Role to securely access another AWS service.

Examples:

| Calling Service | Target Service  |
| --------------- | --------------- |
| EC2             | S3              |
| Lambda          | CloudWatch Logs |
| Step Functions  | Lambda          |
| ECS             | Secrets Manager |

The pattern is always the same:

1. Trust Policy checks **WHO**.
2. Permission Policy checks **WHAT**.
3. STS creates temporary credentials.
4. The calling service accesses the target service.

---

#  Hands-on Recap

In the AWS Console, we:

### Step 1

Created a new IAM Role.

---

### Step 2

Selected:

```
Trusted Entity
↓

AWS Service
```

Why?

Because an AWS service (EC2) would use the role.

---

### Step 3

Selected:

```
EC2
```

Why?

So EC2 could assume the role.

AWS created the Trust Policy automatically.

---

### Step 4

Attached:

```
AmazonS3ReadOnlyAccess
```

Why?

To allow EC2 to:

* List buckets (where permitted) and read S3 objects.
* Read S3 objects.

Not upload or delete them.

> Note: AmazonS3ReadOnlyAccess also includes read-only actions beyond just GetObject

---

### Step 5

Created the role.

AWS automatically created the Instance Profile in the background.

---

# 👀 What Happened Behind the Scenes?

Although we only clicked a few buttons, AWS also:

* Created the Trust Policy.
* Attached the Permission Policy.
* Created the Instance Profile.

Later, when an EC2 instance uses this role:

* STS creates temporary credentials.
* The AWS SDK/CLI retrieves them.
* The SDK/CLI refreshes them automatically when they expire.

---

#  Common Beginner Mistakes

❌ Trust Policy gives permissions.

✅ It only decides **WHO** can assume the role.

---

❌ Permission Policy decides who can use the role.

✅ It only decides **WHAT** the role can do.

---

❌ IAM Role stores Access Keys.

✅ IAM Roles store permissions, not credentials.

---

❌ STS is a type of IAM Role.

✅ STS is a service that creates temporary credentials.

---

❌ Instance Profile contains permissions.

✅ The IAM Role contains permissions. The Instance Profile simply makes the role available to EC2.

---

#   Questions

### What is an IAM Role?

An IAM Role is an AWS identity that contains permissions but does not have long-term credentials. 
Trusted entities such as AWS services (EC2, Lambda), IAM users, IAM roles, or federated identities can temporarily assume the role to securely access AWS resources using temporary credentials.

---

### What is a Principal?

The identity requesting access to AWS resources.

---

### What is the difference between a Trust Policy and a Permission Policy?

* Trust Policy → WHO can assume the role.
* Permission Policy → WHAT the role can do.

---

### What does `sts:AssumeRole` do?

It allows a trusted principal to assume (use) the IAM Role.

It does **not** grant permissions to access AWS resources.

---

### What does AWS STS do?

Creates temporary security credentials.

---

### Why are temporary credentials safer?

Because they expire automatically and reduce the risk of credential misuse.

---

### What is an Instance Profile?

A container that makes an IAM Role available to an EC2 instance.

---

### Why should you use IAM Roles instead of storing Access Keys?

Because IAM Roles provide temporary credentials automatically, eliminating the need to store permanent Access Keys.

---

# 🧾 One-Page Revision

| Concept                       | Remember                                                        |
| ----------------------------- | --------------------------------------------------------------- |
| IAM Role                      | An IAM Role is an AWS identity that can be assumed temporarily and contains permissions.                       |
| Principal                     | Who is requesting access?                                       |
| Trust Policy                  | WHO can assume the role?                                        |
| Permission Policy             | WHAT can the role do?                                           |
| `sts:AssumeRole`              | Allows a trusted principal to use the role.                     |
| AWS STS                       | Creates temporary credentials.                                  |
| Temporary Credentials         | Access Key ID + Secret Access Key + Session Token + Expiration. |
| Instance Profile              | Connects an IAM Role to EC2.                                    |
| AWS SDK/CLI                   | Automatically retrieves and refreshes credentials.              |
| Cross-Service Role Assumption | One AWS service securely accesses another using an IAM Role.    |

> Note:
> An IAM Role is an AWS identity that can be assumed temporarily and contains permissions.
> Why?
> The role itself isn't temporary—it always exists until deleted. The session created when it's assumed is temporary.

---

#  Final Takeaway

If you remember just one flow from Day 3, make it this:

```text
Application (EC2, Lambda, ECS, etc.)
        │
        ▼
Needs AWS Access
        │
        ▼
IAM Role
        │
        ├── Trust Policy → WHO can assume the role?
        │
        └── Permission Policy → WHAT can the role do?
        │
        ▼
AWS STS
        │
Creates Temporary Credentials
        │
        ▼
Application accesses AWS service
        │
        ▼
Credentials expire
        │
        ▼
AWS SDK/CLI automatically refreshes them
```

***

# Section 7 –  Tips & Real-World Scenarios

---

#  Real-World Scenario 1: EC2 Accessing S3

Imagine you launch an EC2 instance that needs to download files from an S3 bucket.

❌ Bad Approach:

```text
Store Access Key and Secret Key inside the EC2 instance.
```

Problems:

* Keys can be stolen.
* Keys don't expire.
* Keys must be rotated manually.

✅ Best Practice:

```text
Attach an IAM Role to the EC2 instance.
```

AWS will:

1. Check the Trust Policy.
2. Check the Permission Policy.
3. Use STS to create temporary credentials.
4. Allow EC2 to access S3.

---

#  Real-World Scenario 2: Lambda Writing Logs

A Lambda function automatically writes logs to CloudWatch.

How?

```text
Lambda
   │
   ▼
IAM Role
   │
   ▼
CloudWatch Logs
```

The Trust Policy trusts **Lambda**.

The Permission Policy allows **CloudWatch Logs** actions.

---

#  Real-World Scenario 3: ECS Reading Secrets

An ECS application needs a database password stored in Secrets Manager.

Instead of saving the password inside the application:

```text
ECS Task
     │
     ▼
IAM Role
     │
     ▼
Secrets Manager
```

The application securely retrieves the secret when needed.

---

#  Real-World Scenario 4: Cross-Account Access

Suppose:

* Account A owns an S3 bucket.
* Account B needs access.

Instead of creating IAM users in both accounts:

```text
Account B
     │
Assumes IAM Role
     │
     ▼
Account A
     │
     ▼
S3 Bucket
```

The Trust Policy trusts Account B.

The Permission Policy allows access to the S3 bucket.

This is called **Cross-Account Role Assumption**.
> The Trust Policy specifies the IAM role or AWS account that is allowed to assume the role.

---

#  Most Common  Questions

### Q1. Why use IAM Roles instead of IAM Users?

Answer:

IAM Roles provide temporary credentials that are automatically generated and refreshed. IAM Users use long-term credentials that must be stored and managed.

---

### Q2. What is the difference between IAM Users and IAM Roles?

| IAM User                | IAM Role                                                 |
| ----------------------- | -------------------------------------------------------- |
| Permanent identity      | Temporary identity                                       |
| Long-term credentials   | Temporary credentials                                    |
| Used by people          | Used by AWS services, applications, or users when needed |
| Requires key management | No permanent keys required                               |

---

### Q3. What is the difference between Trust Policy and Permission Policy?

**Trust Policy**

Answers:

> Who can assume the role?

**Permission Policy**

Answers:

> What can the role do?

---

### Q4. What is STS?

AWS Security Token Service.

It generates temporary security credentials for IAM Roles.

---

### Q5. What is `sts:AssumeRole`?

It allows a trusted principal to assume (use) an IAM Role.

It **does not** grant permissions like S3 access.

---

### Q6. What happens when temporary credentials expire?

The AWS SDK or CLI automatically requests new temporary credentials through the IAM Role.

---

### Q7. What is an Instance Profile?

A container that makes an IAM Role available to an EC2 instance.


### Q8. IAM User vs IAM Role vs Resource Policy
Difference between IAM Role and Resource Policy?

Example
```text  
IAM Role

↓

Identity-based policy

↓

Attached to identity
```
vs
```text
Bucket Policy

↓

Resource-based policy

↓

Attached to S3 bucket
```


### Q9.  How does an EC2 instance get temporary credentials?

Answer

- EC2 has an attached IAM Role (via an Instance Profile).
- The application uses the AWS SDK.
- The SDK requests credentials from the EC2 Instance Metadata Service (IMDS).
- IMDS returns temporary credentials for the role.
- The SDK uses them automatically.


---

#  Easy Memory Tricks

### Trust Policy

Think:

```text
WHO can assume the role?
```

---

### Permission Policy

Think:

```text
WHAT can the role do?
```

---

### STS

Think:

```text
Generates temporary credentials
```

---

### Instance Profile

Think:

```text
Role → EC2 
```

---

### IAM Role

Think:

```text
Permissions
```

---

### Principal

Think:

```text
Who is requesting access?
```

---

#  30-Second Revision

If someone asks, "Explain IAM Roles in AWS," you can say:

> "IAM Role is an AWS identity that can be assumed temporarily with permissions. A trusted principal, such as EC2 or Lambda, assumes the role. The Trust Policy decides who can assume it, and the Permission Policy decides what actions it can perform. AWS STS generates temporary credentials, which are automatically retrieved and refreshed by the AWS SDK or CLI. For EC2, the role is attached through an Instance Profile. This approach is more secure because permanent Access Keys are never stored."

---

#  Day 3 Complete Summary

```text
EC2 / Lambda
       │
       ▼
Needs AWS Access
       │
       ▼
IAM Role
       │
       ├── Trust Policy
       │     (WHO can assume the role?)
       │
       └── Permission Policy
             (WHAT can the role do?)
       │
       ▼
Instance Profile (EC2 only)
       │
       ▼
AWS STS
Creates Temporary Credentials
       │
       ▼
Application Calls AWS Service
       │
       ▼
Access Allowed / Denied
       │
       ▼
Credentials Expire
       │
       ▼
AWS SDK / CLI Automatically Refreshes Credentials
```

> Note: For EC2, an Instance Profile attaches the IAM Role to the EC2 instance.


***



#  Day 3 Hands-on Lab (AWS Console)

##  Objective

Create an IAM Role for an EC2 instance that has **read-only access to Amazon S3**.

---

## Step 1: Open IAM

Logged into the AWS Management Console.

Navigated to:

```text
AWS Console
→ IAM
→ Roles
→ Create Role
```

---

## Step 2: Select Trusted Entity

Selected:

```text
Trusted Entity Type:
AWS Service
```

### Why?

Because the role will be used by an **AWS service**, not by:

* Another AWS Account
* An IAM User
* A Federated User

---

## Step 3: Choose the Service

Selected:

```text
EC2
```

### Why?

Because we wanted an **EC2 instance** to assume this role.

AWS automatically created a Trust Policy similar to:

```json
{
  "Principal": {
    "Service": "ec2.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
}
```

Meaning:

> Only the EC2 service is trusted to assume this role.

---

## Step 4: Attach Permissions

Selected the AWS managed policy:

```text
AmazonS3ReadOnlyAccess
```

### What does it allow?

✅ List S3 buckets

✅ Read (download) S3 objects

### What does it NOT allow?

❌ Upload objects

❌ Delete objects

❌ Modify bucket settings

This demonstrates the **Principle of Least Privilege**—grant only the permissions required.

---

## Step 5: Name the Role

Example:

```text
EC2-S3-ReadOnly-Role
```

(Your role name may be different.)

---

## Step 6: Review and Create

Verified:

* Trusted Entity = EC2
* Permission Policy = AmazonS3ReadOnlyAccess

Clicked:

```text
Create Role
```

The IAM Role was created successfully.

---

## Step 7: What AWS Did Automatically

Behind the scenes, AWS:

* Created the Trust Policy.
* Attached the Permission Policy.
* Created an Instance Profile for EC2.

These steps happen automatically, so they are not shown during role creation.

---

#  What I Learned from the Hands-on

* IAM Roles are attached to workloads instead of storing Access Keys.
* Trust Policy answers **WHO** can assume the role.
* Permission Policy answers **WHAT** the role can do.
* EC2 was selected because it is the trusted AWS service using the role.
* `AmazonS3ReadOnlyAccess` follows the Principle of Least Privilege.
* AWS automatically creates the Instance Profile.
* STS and temporary credentials work behind the scenes and are not visible during role creation.


