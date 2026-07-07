

> **How does AWS know what I'm allowed to do?**

The answer is **IAM**.

---

# Identity + Permissions = Access

Imagine you enter an office.

There are two questions:

1. **Who are you?**
2. **What are you allowed to do?**

AWS asks the same questions.

```
Who are you?
        ↓
IAM User

What can you do?
        ↓
IAM Policy

Result
        ↓
Access
```

So:

**Identity + Permissions = Access**

---

# What is IAM?

IAM stands for:

> **Identity and Access Management**

Let's break that down.

* **Identity** = Who you are.
* **Access** = What you can do.
* **Management** = Controlling both.

So IAM is simply:

> The AWS service that controls **who can access what**.

---

# IAM User

Suppose a company has three employees.

* Alice
* Bob
* Charlie

Each gets their own username and password.

AWS calls each one an **IAM User**.

Example:

```
Alice
↓

IAM User

↓

Username
Password
Permissions
```

Every IAM User has its **own login** and **its own permissions**.

A common misconception is:

> "If I can log in, I can do everything."

No.

You can log in, but you can only do what your permissions allow.

---

# IAM Group

Now imagine your company has 100 developers.

Instead of giving permissions one by one, you create a group.

```
Developers

↓

Developer Group

↓

EC2 Access
S3 Access
```

Now every new developer joins the group.

No need to assign permissions individually.

That's why AWS recommends:

> Attach policies to **groups**, not directly to users.

---

# IAM Role

This is where beginners get confused.

Forget AWS for a minute.

Imagine you're a hotel guest.

You get a room key.

When you check out,

the key stops working.

You don't keep permanent access.

That's exactly how an IAM Role works.

It gives **temporary permissions**.

Examples:

* EC2 accessing S3
* Lambda accessing DynamoDB
* GitHub Actions deploying to AWS

For Week 1, learners only need to remember:

> **Roles give temporary access.**

---

# IAM Policy

A Policy is simply a **rule book**.

Example:

```
Can read S3

YES

Can delete S3

NO
```

Policies are written in JSON.

AWS reads the JSON and decides:

Allowed?

or

Denied?

---

# Types of Policies

Don't go too deep here.

Just remember:

### AWS Managed Policy

AWS created it.

Example:

AmazonS3ReadOnlyAccess

---

### Customer Managed Policy

You created it.

You can reuse it.

---

### Inline Policy

Created for one user, one role, or one group only.

Not reusable.

---

# Least Privilege

This is one of the most important AWS security principles.

Imagine you're a receptionist.

Should you have permission to delete production servers?

No.

You only need:

* View calendar
* Answer calls

Nothing else.

AWS says:

Give people **only the permissions they need**.

Nothing more.

---

# Permission Boundary

This is an advanced concept.

For Week 1,

don't over-explain it.

One sentence is enough:

> A Permission Boundary sets the **maximum permissions** a user or role can ever receive.

Think of it as a ceiling.

Even if someone later attaches an Admin policy,

the Permission Boundary can stop them from becoming an administrator.

---

# What should learners remember?

By the end of Day 2, they should know these five things:

| Term   | Easy Meaning                                                     |
| ------ | ---------------------------------------------------------------- |
| IAM    | Controls who can access what in AWS.                             |
| User   | One person or workload with its own login and permissions.       |
| Group  | A collection of users who share the same permissions.            |
| Role   | Temporary access for a trusted user or service.                  |
| Policy | A JSON document that defines what actions are allowed or denied. |

---




## Answers

### 1. What does IAM stand for?

IAM stands for **Identity and Access Management**. It is the AWS service that controls **who can access AWS resources and what actions they can perform**.

---

### 2. What is the difference between a User and a Group?

- An **IAM User** is a single identity (a person or application) with its own login credentials and permissions.
- An **IAM Group** is a collection of IAM users that share the same permissions.

**Example:**
- User: `alice`
- Group: `Developers`

Instead of giving permissions to Alice individually, you add her to the **Developers** group.

---

### 3. Why do we use Roles?

An **IAM Role** provides **temporary permissions** to trusted users, AWS services, or external applications.

**Examples:**
- An EC2 instance accessing an S3 bucket.
- A Lambda function reading from DynamoDB.
- GitHub Actions deploying resources to AWS using OIDC.

Remember:
> **Users are usually permanent identities. Roles provide temporary access.**

---

### 4. What is an IAM Policy?

An **IAM Policy** is a **JSON document** that defines what actions are **allowed** or **denied** for a user, group, or role.

For example, a policy can allow a user to:
- View EC2 instances 
- Read files from S3 
- Prevent deleting S3 buckets 

---

### 5. What does "Least Privilege" mean?

The **Principle of Least Privilege** means giving **only the permissions required to complete a task**—nothing more.

**Example:**
If a user only needs to view EC2 instances, give them **read-only access**, not full administrative access.
```

###  Quick Revision Table

| Question                 | Short Answer                                                  |
| ------------------------ | ------------------------------------------------------------- |
| What is IAM?             | AWS service that controls who can access what.                |
| What is a User?          | One person or application with its own login and permissions. |
| What is a Group?         | A collection of users with shared permissions.                |
| What is a Role?          | Temporary permissions for trusted users or AWS services.      |
| What is a Policy?        | A JSON document that defines allowed or denied actions.       |
| What is Least Privilege? | Give only the permissions needed to perform a task.           |



*  **User** = One person
*  **Group** = Many users
*  **Role** = Temporary access
*  **Policy** = Rules
*  **Least Privilege** = Only what's needed



