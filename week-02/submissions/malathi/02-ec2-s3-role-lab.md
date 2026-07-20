## 📚 AWS Week 2 - Day 3 Master Notes

### Part 1 – Introduction, Architecture & Core Concepts

---

# AWS Week 2 - Day 3

# EC2 Reads S3 Using an IAM Role

---

# Learning Objective

By the end of this lab, you will understand:

* What an IAM Role is.
* Why IAM Roles are more secure than Access Keys.
* How an EC2 instance gets temporary credentials.
* What STS (Security Token Service) does.
* What an Instance Profile is.
* How to give an EC2 instance read-only access to an S3 bucket.
* How the Principle of Least Privilege works.
* How to verify that everything is working correctly.

---

# Real-World Story

Imagine you work in a company.

There is a locked storage room containing important files.

There are four people involved:

```
Storage Room
↓

Employee

↓

Security Guard

↓

Company ID Card
```

Let's map this to AWS.

| Real World             | AWS                   |
| ---------------------- | --------------------- |
| Storage Room           | S3 Bucket             |
| Employee               | EC2 Instance          |
| Company ID Card        | IAM Role              |
| Security Guard         | STS                   |
| Temporary Visitor Pass | Temporary Credentials |

The employee does **not** carry the master key.

Instead:

1. Employee shows company ID.
2. Security Guard checks the ID.
3. Security Guard gives a temporary visitor pass.
4. Employee enters the storage room.
5. Visitor pass expires automatically.

AWS works exactly like this.

---

# The Biggest Lesson of Day 3

**Never store Access Keys on an EC2 instance.**

Instead:

```
EC2

↓

IAM Role

↓

STS

↓

Temporary Credentials

↓

S3
```

This is AWS Best Practice.

---

# What Problem Are We Solving?

Suppose your EC2 needs to read files from S3.

Many beginners do this:

```
EC2

↓

aws configure

↓

Access Key

↓

Secret Key
```

Problems:

* Keys never expire.
* Anyone who steals them can use your AWS account.
* Keys must be rotated manually.
* Not recommended by AWS.

Instead:

```
EC2

↓

IAM Role

↓

AWS automatically gives temporary credentials.
```

No passwords.

No secrets.

No manual configuration.

---

# Architecture

```
                    AWS Cloud

           +----------------------+

                  EC2 Instance
                (Virtual Computer)

                        │

                        ▼

              Instance Profile

                        │

                        ▼

                 IAM Role

                        │

                        ▼

        STS (Security Token Service)

                        │

     Temporary Credentials Created

                        │

                        ▼

                 Amazon S3 Bucket
```

This is exactly what happened in your lab.

---

# Understanding Every Component

## 1. EC2 Instance

Think of EC2 as a virtual computer.

Instead of buying a laptop, AWS gives you a computer in the cloud.

Example:

```
Your Laptop

↓

AWS Data Center

↓

EC2 Instance
```

Our EC2 wanted to read a file stored in S3.

---

## 2. Amazon S3

S3 is AWS's storage service.

Think of it as Google Drive or Dropbox.

It stores:

* Images
* Videos
* Documents
* PDFs
* Backups
* Log files

Everything inside an S3 bucket is called an **Object**.

Example:

```
Bucket

↓

day3-test.txt

↓

photo.png

↓

resume.pdf
```

---

## Bucket vs Object

Think of your computer.

```
Folder

↓

Files
```

AWS uses different words.

```
Bucket

↓

Objects
```

Example:

```
Bucket Name

malathi-day3-s3-demo-bucket

↓

Objects

day3-test.txt

photo.jpg

notes.pdf
```

---

# What is IAM?

IAM stands for:

**Identity and Access Management**

IAM answers two questions:

**Who are you?**

and

**What are you allowed to do?**

Example:

```
Can EC2 read S3?

YES

Can EC2 delete S3?

NO

Can EC2 launch EC2 instances?

NO
```

IAM decides.

---

# What is an IAM Role?

This is one of the most important AWS concepts.

A Role is **not a person**.

A Role is **a set of permissions**.

Think of a Role as a permission card.

Example:

```
Permission Card

↓

Read Bucket

YES

↓

Write Bucket

NO

↓

Delete Bucket

NO
```

When EC2 receives this permission card, it can perform only those actions.

---

# IAM User vs IAM Role

| IAM User                           | IAM Role                                       |
| ---------------------------------- | ---------------------------------------------- |
| Represents a person or application | Represents temporary permissions               |
| Uses Access Keys or Passwords      | Uses temporary credentials                     |
| Long-lived                         | Temporary                                      |
| Manual credential management       | AWS manages credentials automatically          |
| Good for humans                    | Good for AWS services (EC2, Lambda, ECS, etc.) |

---

# Why Not Use Access Keys?

Imagine writing your ATM PIN on a sticky note and taping it to your laptop.

That's similar to storing AWS Access Keys on an EC2 instance.

Problems:

* Someone steals the server.
* They steal the keys.
* They access your AWS account.
* Keys never expire unless you rotate them.

AWS recommends avoiding this.

---

# The Better Way: IAM Roles

Instead of storing credentials:

```
EC2

↓

IAM Role

↓

AWS automatically provides temporary credentials.
```

Advantages:

* No stored secrets.
* Credentials rotate automatically.
* Credentials expire.
* More secure.
* AWS Best Practice.

---

# New Concept: Instance Profile

Many beginners think:

```
EC2

↓

IAM Role
```

This is **not exactly true**.

AWS actually uses an **Instance Profile**.

```
EC2

↓

Instance Profile

↓

IAM Role
```

Think of the Instance Profile as an adapter.

EC2 cannot attach directly to a Role.

The Instance Profile acts as the bridge.

You selected the IAM Role while launching the EC2 instance, and AWS automatically created/used the associated Instance Profile behind the scenes.

---

# New Concept: STS (Security Token Service)

STS stands for:

**Security Token Service**

STS creates **temporary credentials**.

Think of STS as a security guard.

Flow:

```
EC2

↓

"I'm using this IAM Role."

↓

STS checks the role.

↓

STS creates temporary credentials.

↓

EC2 receives temporary credentials.

↓

EC2 accesses S3.
```

Without STS, the EC2 instance would have no credentials.

---

# What Are Temporary Credentials?

Temporary credentials include:

* Temporary Access Key
* Temporary Secret Key
* Session Token

These credentials:

* Are created automatically.
* Expire after a period of time.
* Rotate automatically.
* Are never manually configured on your EC2 instance.

---

# How Does EC2 Get These Credentials?

EC2 talks to a special AWS service called the **Instance Metadata Service (IMDS)**.

```
EC2

↓

IMDS

↓

STS

↓

Temporary Credentials

↓

EC2
```

That's why your command:

```bash
aws configure list
```

showed:

```
TYPE : iam-role
```

instead of credentials you manually configured.

***


# Part 2 – Hands-on Lab (Step 1 to Step 3)

---

# What Are We Building?

Today's architecture:

```text
                 AWS Cloud

          +----------------------+

             Amazon S3 Bucket
        (Stores our training file)

                   ▲
                   │
               Read Only

                   │

      Temporary Credentials (STS)

                   ▲
                   │

               IAM Role
 Week2Day3EC2S3ReadRole

                   ▲
                   │

            Instance Profile

                   ▲
                   │

              EC2 Instance
```

Goal:

✅ EC2 can list files.

✅ EC2 can read files.

❌ EC2 cannot upload files.

❌ EC2 cannot delete files.

---

# Before Starting

Let's understand **what AWS resources we'll create**.

| AWS Resource  | Purpose                      |
| ------------- | ---------------------------- |
| S3 Bucket     | Stores our file              |
| IAM Role      | Gives permissions            |
| Inline Policy | Defines what the role can do |
| EC2 Instance  | Runs our commands            |

---

# Step 1 – Create an S3 Bucket

## What is a Bucket?

A bucket is like a folder.

Example:

Your laptop

```text
Documents
```

Inside:

```text
resume.pdf

photo.jpg

notes.txt
```

AWS:

```text
Bucket

↓

Objects
```

Everything inside S3 is called an **Object**.

---

# Open AWS Console

Go to

```text
AWS Console

↓

Services

↓

S3
```

Click

```text
Create Bucket
```

---

# Bucket Name

AWS asks:

```text
Bucket Name
```

Example

```text
malathi-day3-s3-demo-bucket
```

### Why must it be unique?

Unlike folders on your laptop, bucket names are **globally unique**.

Imagine millions of AWS customers.

Only one person in the world can own:

```text
mybucket
```

So AWS forces unique names.

---

# Region

Example

```text
US West (Oregon)

us-west-2
```

or

```text
Asia Pacific (Mumbai)

ap-south-1
```

Use the same Region where your EC2 instance is running whenever possible.

---

# Block Public Access

AWS shows

```text
Block Public Access

☑ Enabled
```

Leave it ON.

---

## Why?

Imagine your bucket contains

```text
Salary.xlsx
```

If Public Access is enabled,

everyone on the Internet might be able to access it (depending on your policies).

AWS protects you by blocking public access by default.

---

# Create Bucket

Click

```text
Create Bucket
```

Done.

Congratulations.

You now own an S3 bucket.

---

# Upload a File

Open your bucket.

Click

```text
Upload
```

Create

```text
day3-test.txt
```

Contents

```text
Hello AWS!
This is my Day 3 test.
```

Upload.

Your bucket now looks like

```text
Bucket

↓

day3-test.txt
```

---

# What Happened Behind the Scenes?

When you clicked Upload,

AWS stored an **Object**.

Example

```text
Bucket

malathi-day3-s3-demo-bucket

↓

Object

day3-test.txt
```

Important:

An object has

* Data
* Metadata
* Permissions
* Size

---

# Our Lab Mistake (Very Important)

In our lab,

when we downloaded

```bash
aws s3 cp s3://bucket/day3-test.txt -
```

we saw

```text
day3-test.txt
```

instead of

```text
Hello AWS!
This is my Day 3 test.
```

Why?

Because the object itself contained

```text
day3-test.txt
```

instead of the intended message.

We confirmed this using:

```bash
cat /tmp/day3-test.txt
```

and

```bash
aws s3api head-object
```

which showed:

```text
ContentLength : 13
```

That told us the file stored in S3 was only 13 bytes, so it couldn't contain the longer message.

**Lesson:** If the downloaded content is unexpected, first verify the object itself before assuming there's an IAM problem.

---

# Step 2 – Create an IAM Role

Now we create the permission card.

Go to

```text
AWS Console

↓

IAM

↓

Roles

↓

Create Role
```

---

# What is a Role?

Think of it like

```text
Employee ID Card
```

The card itself does nothing.

It only tells people

what you're allowed to do.

Example

```text
Read Files

YES

Write Files

NO

Delete Files

NO
```

Exactly the same idea.

---

# Trusted Entity

AWS asks

```text
Who will use this role?
```

Choose

```text
AWS Service
```

Then

```text
EC2
```

---

# Why EC2?

Because this permission card belongs to the EC2 instance.

Not

* You
* Another user
* Lambda
* ECS

Only EC2.

---

# Don't Attach AmazonS3FullAccess

AWS shows many policies.

One of them is

```text
AmazonS3FullAccess
```

Do NOT choose it.

---

## Why?

Because it allows

```text
Read

Write

Delete

Everything
```

Today's lesson is

**Least Privilege**

Give only what is needed.

Nothing more.

---

# Name the Role

Example

```text
Week2Day3EC2S3ReadRole
```

Create Role.

Done.

---

# What Did We Create?

Right now

the role has almost no useful permissions.

Think of a blank ID card.

```text
Employee ID

↓

No permissions yet
```

Now we must define what it can do.

---

# Understanding Trust Policy

This is one of the most confusing AWS concepts.

There are **two different policies**.

## 1. Trust Policy

Answers:

> **Who is allowed to use this role?**

Example

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "ec2.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
}
```

Meaning:

Only EC2 may assume this role.

---

## Where to Verify It

Go to

```text
IAM

↓

Roles

↓

Week2Day3EC2S3ReadRole

↓

Trust relationships

↓

View trust policy
```

You should see

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

---

# Why Is It Called a Trust Policy?

Imagine you own a house.

Someone knocks on the door.

You ask:

> "Who are you?"

If they're a delivery person you trust,

you open the door.

If they're a stranger,

you don't.

AWS does exactly the same thing.

EC2 says:

```text
Can I use this IAM Role?
```

AWS checks the Trust Policy.

If EC2 is trusted,

AWS allows it to assume the role.

Otherwise,

AWS says no.

---

# Proof That Ours Worked

We already proved it.

We ran

```bash
aws sts get-caller-identity
```

Output

```text
assumed-role/Week2Day3EC2S3ReadRole
```

That means

EC2

↓

Asked to use the role

↓

Trust Policy allowed it

↓

STS created temporary credentials

↓

Success

---

# Step 3 – Create the Inline Policy

Now we answer a different question.

The Trust Policy answered:

> Who can use the role?

Now we answer:

> What can the role do?

This is the **Permission Policy**.

Go to

```text
IAM

↓

Roles

↓

Week2Day3EC2S3ReadRole

↓

Permissions

↓

Add Permissions

↓

Create Inline Policy

↓

JSON
```

Paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListOneBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME"
    },
    {
      "Sid": "ReadObjectsInOneBucket",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

Replace

```text
YOUR-BUCKET-NAME
```

with your actual bucket name.

Example:

```text
malathi-day3-s3-demo-bucket
```

---

# Understanding the JSON (Line by Line)

```json
"Effect": "Allow"
```

Means:

> "Permit this action."

---

```json
"Action": "s3:ListBucket"
```

Means:

> "Allow viewing the list of objects in the bucket."

Example:

```bash
aws s3 ls s3://malathi-day3-s3-demo-bucket
```

---

```json
"Resource": "arn:aws:s3:::malathi-day3-s3-demo-bucket"
```

This permission applies **only to that bucket**.

Not to every bucket in your AWS account.

---

```json
"Action": "s3:GetObject"
```

Means:

> "Allow reading objects."

Example:

```bash
aws s3 cp s3://malathi-day3-s3-demo-bucket/day3-test.txt -
```

---

```json
"Resource": "arn:aws:s3:::malathi-day3-s3-demo-bucket/*"
```

The `*` means:

> "Any object inside this bucket."

Example:

* `day3-test.txt`
* `photo.jpg`
* `notes.pdf`

---

# Why Didn't We Add `PutObject`?

Because today's lesson is **Least Privilege**.

If we added:

```json
"Action": "s3:PutObject"
```

the EC2 instance could upload files.

We intentionally left it out so that the upload test would fail with `AccessDenied`. That failure proves our policy is correctly restricting access.

***


# Part 3 – Launch EC2, Test IAM Role & Understand Every Command

---

# Step 4 – Launch the EC2 Instance

Now that we have:

* ✅ S3 Bucket
* ✅ IAM Role
* ✅ Permission Policy

we need a computer that will use them.

That computer is the **EC2 instance**.

---

# Launch EC2

Go to:

```text
AWS Console
    ↓
EC2
    ↓
Launch Instance
```

---

## Choose an AMI (Amazon Machine Image)

Think of an AMI like the operating system installer for a new computer.

Choose:

```text
Amazon Linux 2023
```

Why?

* Lightweight
* AWS CLI is already installed
* Recommended for AWS labs
* Free Tier eligible (where available)

---

## Choose Instance Type

Choose:

```text
t2.micro
```

or

```text
t3.micro
```

These are small, inexpensive instances suitable for practice.

---

## The Most Important Step

Scroll to:

```text
Advanced Details
```

Find:

```text
IAM Instance Profile
```

Select:

```text
Week2Day3EC2S3ReadRole
```

---

## What Actually Happens?

Many people think:

```text
EC2
    ↓
IAM Role
```

Internally AWS does:

```text
EC2
    ↓
Instance Profile
    ↓
IAM Role
```

The Instance Profile acts as the bridge between the EC2 instance and the IAM Role.

---

# Launch the Instance

Wait until:

```text
Running
```

and both **Status Checks** show:

```text
2/2 checks passed
```

Only then connect to the instance.

---

# Step 5 – Connect to EC2

You can use:

* EC2 Instance Connect
* Session Manager
* SSH

You'll get a prompt similar to:

```bash
ec2-user@ip-172-31-29-191 ~]$
```

Now we're inside the Linux machine.

---

# First Command

```bash
aws configure list
```

---

# Why Do We Run This?

We are asking AWS:

> **"Where are my AWS credentials coming from?"**

---

# Your Actual Output

```text
NAME       : VALUE                    : TYPE             : LOCATION
profile    : <not set>                : None             : None
access_key : ****************HACF     : iam-role
secret_key : ****************/cIP     : iam-role
region     : us-west-2                : imds
```

Let's understand every line.

---

## profile

```text
profile : <not set>
```

AWS CLI supports named profiles such as:

```text
Personal

Work

Production
```

You are not using any saved profile.

That is completely fine.

---

## access_key

```text
access_key : ************HACF
TYPE : iam-role
```

This is extremely important.

Notice:

```text
TYPE : iam-role
```

This means:

> The AWS CLI obtained temporary credentials from the IAM Role.

You never ran:

```bash
aws configure
```

to manually store an Access Key.

AWS automatically provided temporary credentials.

---

## secret_key

```text
secret_key : ************/cIP
TYPE : iam-role
```

This is the matching temporary Secret Access Key.

Again:

```text
TYPE : iam-role
```

confirms the credentials came from the IAM Role, not from a manually configured user.

---

## region

```text
region : us-west-2
TYPE : imds
```

The region was discovered using the **Instance Metadata Service (IMDS)**.

---

# What is IMDS?

IMDS is a special AWS service that only the EC2 instance can access.

It provides information such as:

* Instance ID
* Region
* IAM Role credentials
* Availability Zone

Flow:

```text
EC2
    ↓
IMDS
    ↓
Region
IAM Role Credentials
Instance Information
```

---

# Important Note

Some older tutorials expect:

```text
access_key : <not set>
secret_key : <not set>
```

On modern Amazon Linux with an attached IAM Role, seeing:

```text
TYPE : iam-role
```

is the correct and expected behavior.

---

# Second Command

```bash
aws sts get-caller-identity
```

---

# Why?

We are asking AWS:

> **"Who am I?"**

---

# Your Actual Output

```json
{
    "UserId": "ABCD",
    "Account": "1235",
    "Arn": "arn:aws:sts::1235:assumed-role/Week2Day3EC2S3ReadRole/i-079f90f5c3855c45c"
}
```

---

## UserId

This is an internal AWS identifier for the temporary identity.

You usually don't need it for day-to-day work.

---

## Account

This is your AWS Account ID.

For screenshots or reports, hide part or all of it.

---

## ARN

The most important part is:

```text
assumed-role
```

This proves:

* EC2 contacted AWS.
* AWS checked the Trust Policy.
* STS issued temporary credentials.
* The EC2 instance successfully assumed the IAM Role.

If the Trust Policy were wrong, this command would fail.

---

# Behind the Scenes

```text
EC2
    ↓
Uses Instance Profile
    ↓
IAM Role
    ↓
STS
    ↓
Temporary Credentials
    ↓
aws sts get-caller-identity
```

---

# Third Command

```bash
aws s3 ls s3://malathi-day3-s3-demo-bucket
```

---

# Why?

We are asking S3:

> **"Show me the objects in this bucket."**

---

# Your Output

```text
2026-07-20 12:37:19         13 day3-test.txt
```

Let's decode it.

```text
2026-07-20
```

Upload date.

---

```text
12:37:19
```

Upload time.

---

```text
13
```

Object size in bytes.

This turned out to be an important clue.

---

```text
day3-test.txt
```

Object name.

The command succeeded, proving:

✅ The IAM Role allows `s3:ListBucket`.

---

# Fourth Command

```bash
aws s3 cp s3://malathi-day3-s3-demo-bucket/day3-test.txt -
```

---

# Why the Dash (`-`)?

Normally:

```bash
aws s3 cp s3://bucket/file.txt .
```

downloads the file to the current directory.

Using:

```bash
aws s3 cp s3://bucket/file.txt -
```

tells AWS CLI:

> Print the file contents directly to the terminal.

---

# Expected Output

```text
Hello AWS!
This is my Day 3 test.
```

---

# What You Actually Saw

```text
day3-test.txt
```

At first glance, it looked like an IAM problem.

It wasn't.

---

# How We Investigated

First:

```bash
aws s3api head-object \
  --bucket malathi-day3-s3-demo-bucket \
  --key day3-test.txt
```

Output included:

```text
ContentLength : 13
```

This told us:

The object stored in S3 is only 13 bytes long.

It cannot contain:

```text
Hello AWS!
This is my Day 3 test.
```

because that text is much longer.

---

# Second Investigation

Download the object:

```bash
aws s3 cp s3://malathi-day3-s3-demo-bucket/day3-test.txt /tmp/day3-test.txt
```

Then read it:

```bash
cat /tmp/day3-test.txt
```

Output:

```text
day3-test.txt
```

This proved:

The object itself contained:

```text
day3-test.txt
```

The IAM Role worked correctly.

The file content was simply different from what we expected.

---

# Lesson Learned

When troubleshooting S3:

Do **not** immediately blame IAM.

Check:

1. Object exists.
2. Correct bucket.
3. Correct object key.
4. Object content.

Many S3 issues are caused by uploading the wrong file, not by permissions.

---

# Understanding the Two S3 Permissions

Our policy contained:

```json
"s3:ListBucket"
```

Allows:

```bash
aws s3 ls s3://bucket
```

---

And:

```json
"s3:GetObject"
```

Allows:

```bash
aws s3 cp s3://bucket/file.txt -
```

These are different permissions.

You can have one without the other.

---

# Step 6 – Test Least Privilege (Verify Write is Denied)

## Why are we doing this?

So far, we proved that our EC2 instance can:

* ✅ List the bucket
* ✅ Read objects

Now we want to prove that it **cannot write** to the bucket.

This is called **testing Least Privilege**.

Remember our IAM policy only allows:

```text
s3:ListBucket

s3:GetObject
```

It does **NOT** allow:

```text
s3:PutObject

s3:DeleteObject
```

So the upload should fail.

---

# Create a Test File

Run:

```bash
printf 'write test\n' > /tmp/write-test.txt
```

---

## What does this command do?

Let's break it down.

```bash
printf
```

Prints text.

---

```text
'write test\n'
```

This is the text we want inside the file.

`\n` means:

```text
New Line
```

---

```text
>
```

Means:

Save the output into a file.

---

```text
/tmp/write-test.txt
```

The file location.

---

After running the command:

```text
/tmp/write-test.txt
```

will contain:

```text
write test
```

---

# Upload the File

Run:

```bash
aws s3 cp /tmp/write-test.txt s3://malathi-day3-s3-demo-bucket/write-test.txt
```

---

# What does this command mean?

```text
aws
```

Run the AWS CLI.

---

```text
s3
```

Use the S3 service.

---

```text
cp
```

Copy a file.

---

```text
/tmp/write-test.txt
```

Source (local file).

---

```text
s3://malathi-day3-s3-demo-bucket/write-test.txt
```

Destination (S3 bucket).

---

# Expected Output

You should see something similar to:

```text
upload failed:
/tmp/write-test.txt to s3://malathi-day3-s3-demo-bucket/write-test.txt

An error occurred (AccessDenied) when calling the PutObject operation:
Access Denied
```

---

# Is This an Error?

Yes.

But it is the **correct** error.

Think of it like this.

You gave someone permission to:

* Enter your house.
* Read a book.

But you did **not** give permission to:

* Paint your walls.
* Throw away your furniture.

If they try, you stop them.

AWS is doing the same thing.

---

# Behind the Scenes

```text
EC2

↓

IAM Role

↓

Permission Policy

↓

Request:
s3:PutObject

↓

Permission Found?

NO

↓

AWS returns

AccessDenied
```

---

# What Did We Prove?

We proved our IAM policy follows the **Principle of Least Privilege**.

The EC2 instance can only do what we explicitly allowed.

---

# Step 7 – Collect Safe Proof (Screenshots)

If you're submitting this lab or saving it for your portfolio, take screenshots of:

### 1. Trust Policy

IAM

↓

Roles

↓

Week2Day3EC2S3ReadRole

↓

Trust Relationships

Verify it contains:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "ec2.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
}
```

---

### 2. Permission Policy

Take a screenshot of the inline policy showing:

```text
s3:ListBucket

s3:GetObject
```

---

### 3. EC2 Instance

Show that the EC2 instance has:

```text
IAM Role

Week2Day3EC2S3ReadRole
```

attached.

---

### 4. AWS CLI Output

Capture:

```bash
aws configure list
```

---

### 5. Caller Identity

Capture:

```bash
aws sts get-caller-identity
```

Hide your AWS Account ID if sharing publicly.

---

### 6. Successful Read

Capture:

```bash
aws s3 ls s3://malathi-day3-s3-demo-bucket
```

and

```bash
aws s3 cp s3://malathi-day3-s3-demo-bucket/day3-test.txt -
```

---

### 7. Denied Upload

Capture the `AccessDenied` output.

This is proof that your Least Privilege policy is working.

---

# Step 8 – Cleanup

## Why Cleanup?

AWS resources can cost money if left running.

Even if you're using the Free Tier, it's a good habit to clean up resources you no longer need.

---

## Cleanup Order

Always clean up in this order:

```text
EC2 Instance

↓

S3 Objects

↓

S3 Bucket

↓

IAM Role
```

---

## Step 8.1 – Terminate the EC2 Instance

Go to:

```text
AWS Console

↓

EC2

↓

Instances
```

Select your instance.

Click:

```text
Instance State

↓

Terminate Instance
```

Confirm.

Wait until the instance shows:

```text
Terminated
```

---

## Step 8.2 – Delete the S3 Object

Go to:

```text
AWS Console

↓

S3

↓

malathi-day3-s3-demo-bucket
```

Select:

```text
day3-test.txt
```

Click:

```text
Delete
```

AWS will ask you to type:

```text
permanently delete
```

Type it and confirm.

---

## Step 8.3 – Delete the Bucket

After the bucket is empty:

Go back to the bucket list.

Select:

```text
malathi-day3-s3-demo-bucket
```

Click:

```text
Delete
```

Type the bucket name when AWS asks for confirmation.

Click:

```text
Delete Bucket
```

---

## Step 8.4 – Delete the IAM Role

Go to:

```text
IAM

↓

Roles

↓

Week2Day3EC2S3ReadRole
```

First delete the inline policy:

```text
Permissions

↓

ReadOneTrainingBucket

↓

Delete
```

Then return to the Roles list.

Select:

```text
Week2Day3EC2S3ReadRole
```

Click:

```text
Delete
```

Type the role name if prompted and confirm.

---



Do **not** include in screenshot:

* Access Key IDs
* Secret Access Keys
* Session Tokens
* Private SSH keys

---

# Common Troubleshooting

## `Unable to locate credentials`

Check:

* Is the IAM Role attached to the EC2 instance?
* Wait a minute after attaching the role.
* Ensure no environment variables or CLI profiles are overriding the role.

---

## `AccessDenied` while reading

Check:

* Bucket name.
* Object key.
* `s3:ListBucket` permission.
* `s3:GetObject` permission.
* Correct bucket ARN and object ARN.

---

## Wrong caller identity

Run:

```bash
aws configure list
```

Ensure credentials are coming from:

```text
TYPE : iam-role
```

not from a local profile or environment variables.

---

## Upload unexpectedly succeeds

This means the role has broader permissions than intended.

Inspect all policies attached to the role and remove any permissions that include `s3:PutObject` or broader S3 access.

---

# Key Takeaways

* EC2 accessed S3 **without storing long-term Access Keys**.
* The IAM Role was assumed successfully through the Trust Policy.
* STS issued temporary credentials.
* IMDS delivered those credentials to the EC2 instance.
* `s3:ListBucket` allowed listing objects.
* `s3:GetObject` allowed reading objects.
* The unexpected file output was due to the object's content, not an IAM issue.
* `AccessDenied` on upload is the expected result and proves **Least Privilege**.

***


# Part 4 – Revision, Interview Questions, MCQs & Cheat Sheet

---

# 1. Complete Mind Map

```text
                    DAY 3

        EC2 Reads S3 Using IAM Role

                     │

     ┌───────────────┼─────────────────┐
     │               │                 │

   EC2            IAM Role             S3
(Virtual PC)   (Permission Card)    (Storage)

     │               │                 │

     └───────────────┼─────────────────┘
                     │

             Instance Profile

                     │

                     ▼

                  AWS STS

                     │

      Temporary Credentials

                     │

                     ▼

          List Bucket & Read Object

                     │

             Upload = AccessDenied
```

---

# 2. Complete Request Flow

This is what actually happens when your EC2 accesses S3.

```text
Step 1

EC2 starts.

↓

Step 2

Instance Profile is attached.

↓

Step 3

IAM Role is attached.

↓

Step 4

EC2 contacts IMDS.

↓

Step 5

IMDS requests credentials from STS.

↓

Step 6

STS creates temporary credentials.

↓

Step 7

EC2 receives temporary credentials.

↓

Step 8

EC2 calls S3.

↓

Step 9

IAM Permission Policy is checked.

↓

Step 10

Allowed or Denied.
```

---

# 3. Interview Questions

## Q1. What is IAM?

Answer:

IAM stands for **Identity and Access Management**.

It controls:

* Who can access AWS.
* What actions they can perform.
* Which resources they can access.

---

## Q2. What is an IAM Role?

Answer:

An IAM Role is a collection of permissions that can be temporarily assumed by AWS services, users, or applications.

Unlike an IAM User, it does not have permanent credentials.

---

## Q3. Why use IAM Roles instead of Access Keys?

Answer:

Because IAM Roles:

* Do not require storing credentials.
* Use temporary credentials.
* Rotate automatically.
* Are more secure.
* Follow AWS Best Practices.

---

## Q4. What is STS?

Answer:

STS stands for **Security Token Service**.

It creates temporary credentials.

---

## Q5. What are Temporary Credentials?

Temporary credentials include:

* Access Key
* Secret Access Key
* Session Token

They expire automatically.

---

## Q6. What is IMDS?

Answer:

IMDS stands for **Instance Metadata Service**.

It provides information to the EC2 instance such as:

* Instance ID
* Region
* IAM Role credentials
* Availability Zone

---

## Q7. What is an Instance Profile?

Answer:

An Instance Profile is a container that allows an IAM Role to be attached to an EC2 instance.

Relationship:

```text
EC2

↓

Instance Profile

↓

IAM Role
```

---

## Q8. Difference Between IAM User and IAM Role

| IAM User                     | IAM Role                              |
| ---------------------------- | ------------------------------------- |
| Permanent identity           | Temporary identity                    |
| Has Access Keys              | Gets temporary credentials            |
| Used by humans               | Used by AWS services and applications |
| Credentials managed manually | Credentials managed automatically     |

---

## Q9. Difference Between Trust Policy and Permission Policy

| Trust Policy          | Permission Policy                        |
| --------------------- | ---------------------------------------- |
| Who can use the role? | What can the role do?                    |
| Contains `Principal`  | Contains `Action` and `Resource`         |
| Uses `sts:AssumeRole` | Uses service actions like `s3:GetObject` |

---

## Q10. Why did we grant `ListBucket` and `GetObject` separately?

Because they control different actions:

* `s3:ListBucket` → Lists object names.
* `s3:GetObject` → Reads the object contents.

Having one permission does not automatically grant the other.

---

## Q11. Why didn't we grant `PutObject`?

To follow the **Principle of Least Privilege**.

Only the minimum required permissions should be granted.

---

## Q12. Why did `AccessDenied` mean success?

Because the lab's goal was to allow **read-only** access.

The upload failing proved that the policy correctly blocked write access.

---

# 4. Common Interview Scenarios

## Scenario 1

"My EC2 cannot access S3."

What do you check?

Answer:

1. Is the IAM Role attached?
2. Is the Trust Policy correct?
3. Does the Permission Policy allow the action?
4. Is the bucket name correct?
5. Is the object key correct?
6. Is another policy (bucket policy, SCP, permission boundary) denying access?

---

## Scenario 2

"`aws sts get-caller-identity` shows an IAM User instead of an assumed role."

Possible causes:

* Someone configured `aws configure`.
* Environment variables override the IAM Role.
* Another AWS CLI profile is being used.

---

## Scenario 3

"`aws s3 ls` works, but `aws s3 cp` fails."

Possible reason:

Missing:

```json
"s3:GetObject"
```

---

## Scenario 4

"`aws s3 cp` works, but `aws s3 ls` fails."

Possible reason:

Missing:

```json
"s3:ListBucket"
```

---

## Scenario 5

"Upload unexpectedly succeeds."

Possible reason:

The role has broader permissions than intended, such as:

```json
"s3:PutObject"
```

or

```json
"s3:*"
```

---

# 5. MCQs

### Q1

Which AWS service provides temporary credentials?

A. IAM

B. STS

C. EC2

D. S3

✅ Answer:

B

---

### Q2

Which AWS service stores files?

A. EC2

B. IAM

C. S3

D. STS

✅ Answer:

C

---

### Q3

Which permission allows reading an object?

A.

```text
s3:ListBucket
```

B.

```text
s3:GetObject
```

C.

```text
s3:PutObject
```

D.

```text
s3:DeleteObject
```

✅ Answer:

B

---

### Q4

Which permission allows listing object names?

A.

```text
s3:GetObject
```

B.

```text
s3:ListBucket
```

C.

```text
s3:PutObject
```

D.

```text
s3:DeleteBucket
```

✅ Answer:

B

---

### Q5

What does `sts:AssumeRole` do?

A.

Deletes roles.

B.

Allows a trusted principal to use a role.

C.

Creates S3 buckets.

D.

Launches EC2 instances.

✅ Answer:

B

---

# 6. Commands Cheat Sheet

## Check credentials source

```bash
aws configure list
```

---

## Check identity

```bash
aws sts get-caller-identity
```

---

## List objects

```bash
aws s3 ls s3://bucket-name
```

---

## Read object

```bash
aws s3 cp s3://bucket-name/file.txt -
```

---

## Download object

```bash
aws s3 cp s3://bucket-name/file.txt .
```

---

## View object metadata

```bash
aws s3api head-object \
  --bucket bucket-name \
  --key file.txt
```

---

## Upload object

```bash
aws s3 cp file.txt s3://bucket-name
```

---

# 7. Common Mistakes

### Mistake 1

Attaching the IAM Role after launching EC2 and testing immediately.

**Fix:** Wait a minute for the role credentials to become available.

---

### Mistake 2

Using the wrong bucket name.

**Fix:** Verify the exact bucket name.

---

### Mistake 3

Using the wrong object key.

Remember:

```text
day3-test.txt
```

is different from

```text
Day3-Test.txt
```

S3 object keys are case-sensitive.

---

### Mistake 4

Confusing the Trust Policy with the Permission Policy.

Remember:

* Trust Policy = Who can assume the role?
* Permission Policy = What can the role do?

---

### Mistake 5

Assuming every S3 problem is an IAM problem.

Your lab showed otherwise.

The object contained unexpected content, and IAM was working correctly.

---

# 8. One-Page Revision Sheet

## Remember This Flow

```text
EC2
↓

Instance Profile
↓

IAM Role
↓

STS
↓

Temporary Credentials
↓

S3
```

---

## Trust Policy

Answers:

> Who can assume the role?

---

## Permission Policy

Answers:

> What actions are allowed?

---

## Lab Permissions

Allowed:

```text
s3:ListBucket

s3:GetObject
```

Denied:

```text
s3:PutObject

s3:DeleteObject
```

---

## Why IAM Roles?

* No stored Access Keys.
* Temporary credentials.
* Automatic rotation.
* AWS Best Practice.

---

## Commands to Remember

```bash
aws configure list
```

Where do my credentials come from?

---

```bash
aws sts get-caller-identity
```

Who am I?

---

```bash
aws s3 ls
```

List bucket contents.

---

```bash
aws s3 cp
```

Read or upload objects.

---

# 9. Day 3 Summary

By completing this lab, you learned how AWS securely allows an EC2 instance to access an S3 bucket without storing long-term credentials. You created an IAM Role, attached it to an EC2 instance through an Instance Profile, granted only `s3:ListBucket` and `s3:GetObject`, and verified that AWS issued temporary credentials using STS. You also confirmed that read operations succeeded while write operations were denied, demonstrating the Principle of Least Privilege.

---


