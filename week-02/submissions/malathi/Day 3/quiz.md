Absolutely! Here's the same answer key explained in **simple, beginner-friendly language**.

---

# 1. Which policy defines who may assume an IAM role?

**Answer:** The role's **trust policy**.

### Simple explanation

Think of a role as a building.

* The **trust policy** is the **security guard at the entrance**.
* It decides **who is allowed to enter** (assume the role).

It answers:

> "Who can use this role?"

---

# 2. Which policy defines what the assumed role may do?

**Answer:** The role's **permission policy**.

### Simple explanation

Once someone gets inside the building, what are they allowed to do?

The **permission policy** answers that.

For example:

* ✅ Read an S3 bucket
* ✅ Start an EC2 instance
* ❌ Delete a database

Think:

* **Trust policy = Who gets in**
* **Permission policy = What they can do**

---

# 3. What API action is central to assuming a role?

**Answer:** `sts:AssumeRole`

### Simple explanation

AWS uses a service called **STS (Security Token Service)**.

When someone wants to use a role, they ask STS:

> "Can I use this role?"

The API command for that request is:

```
sts:AssumeRole
```

It gives temporary login credentials if allowed.

---

# 4. What four values describe an STS temporary credential set?

**Answer**

* Access Key ID
* Secret Access Key
* Session Token
* Expiration Time

### Simple explanation

When STS says "Yes," it gives a temporary identity.

It includes:

| Item              | Meaning                           |
| ----------------- | --------------------------------- |
| Access Key ID     | Username for AWS API              |
| Secret Access Key | Password                          |
| Session Token     | Extra security code               |
| Expiration Time   | When the credentials stop working |

Unlike IAM user keys, these **expire automatically**.

---

# 5. What connects an IAM role to an EC2 instance?

**Answer:** An **Instance Profile**

### Simple explanation

Imagine:

```
IAM Role
      ↓
Instance Profile
      ↓
EC2 Instance
```

An EC2 instance cannot use a role directly.

The **Instance Profile** acts like the connector between them.

---

# 6. Why should an application on EC2 use the AWS SDK credential provider instead of storing keys?

**Answer:** Because the SDK automatically gets and refreshes temporary credentials.

### Simple explanation

❌ Bad way:

Store this in a file:

```
Access Key
Secret Key
```

If someone steals them, they can access your AWS account.

---

✅ Better way:

Attach an IAM role to EC2.

The AWS SDK automatically gets temporary credentials.

Benefits:

* No hardcoded passwords
* Credentials refresh automatically
* More secure

---

# 7. An EC2 role has `s3:GetObject` but not `s3:PutObject`. Which operation should fail?

**Answer:** Uploading an object (`s3:PutObject`)

### Simple explanation

Permissions:

```
GetObject ✅
PutObject ❌
```

That means:

Read a file?

✅ Allowed

Upload a file?

❌ Not allowed

---

# 8. If a Lambda function needs to assume a service role, which service principal belongs in the trust policy?

**Answer**

```
lambda.amazonaws.com
```

### Simple explanation

AWS needs to know **who is allowed to use the role**.

For Lambda, the trust policy says:

```
lambda.amazonaws.com
```

Meaning:

> "AWS Lambda is allowed to assume this role."

---

# 9. Does a role's trust policy alone grant access to an S3 bucket?

**Answer:** No.

### Simple explanation

Remember:

Trust policy:

> Can you use this role?

Permission policy:

> What can you do after using it?

Even if the trust policy lets EC2 use the role...

...the role still needs permission like:

```
s3:GetObject
```

Otherwise you'll get:

```
AccessDenied
```

---

# 10. What is the security benefit of short-lived credentials?

**Answer:** They expire automatically.

### Simple explanation

Suppose someone steals your credentials.

Permanent IAM keys:

❌ They work until someone manually deletes them.

Temporary STS credentials:

✅ They stop working automatically after a short time (for example, one hour).

That greatly reduces the risk if credentials are exposed.

---

# Scenario 1

**Question**

An EC2 application stores an IAM user's access key in a config file.

What should you recommend?

### Simple explanation

Don't save passwords or access keys in code or files.

Instead:

* Create an IAM role
* Attach it to EC2 using an Instance Profile
* Let the AWS SDK automatically get temporary credentials

This is more secure and easier to manage.

---

# Scenario 2

**Question**

EC2 assumes the role successfully, but S3 says:

```
AccessDenied
```

What should you check?

### Check 1

Does the role actually allow:

```
s3:GetObject
```

If not, access is denied.

---

### Check 2

Is the resource (ARN) correct?

For example:

```
arn:aws:s3:::my-bucket/*
```

If the ARN points to the wrong bucket or doesn't include `/*` for objects, access won't work.

---

### Also check

* Bucket policy
* Any explicit `Deny` statements (these override `Allow`)
* If the object is encrypted with AWS KMS, ensure the role has permission to use the KMS key.

---

# Scenario 3

**Question**

Uploading works even though the role doesn't allow:

```
s3:PutObject
```

Why?

### Simple explanation

The command might not actually be using the EC2 role.

Instead, it could be using:

* Another IAM user
* Environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
* A local AWS CLI profile

Or the role might have another attached or inline policy that grants `s3:PutObject`.

---

## Quick Memory Trick

| Question               | Easy Way to Remember                                     |
| ---------------------- | -------------------------------------------------------- |
| Trust policy           | **Who can use the role?**                                |
| Permission policy      | **What can the role do?**                                |
| `sts:AssumeRole`       | **Request temporary credentials**                        |
| STS credentials        | **Access Key + Secret Key + Session Token + Expiration** |
| Instance Profile       | **Connects EC2 to a role**                               |
| SDK                    | **Automatically gets temporary credentials**             |
| `GetObject`            | **Download/Read**                                        |
| `PutObject`            | **Upload/Write**                                         |
| `lambda.amazonaws.com` | **Lambda service identity**                              |
| Temporary credentials  | **Expire automatically = safer**                         |

A simple analogy that ties it all together:

* **Trust policy** = The guest list at the entrance ("Who may enter?")
* **Permission policy** = The list of rooms and actions allowed inside ("What may they do?")
* **STS** = The reception desk that issues a temporary visitor badge.
* **Instance Profile** = The badge holder that lets an EC2 instance carry and use that temporary badge automatically.
