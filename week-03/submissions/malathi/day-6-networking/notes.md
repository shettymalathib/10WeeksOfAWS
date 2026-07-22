# Day 6 Story

Imagine AWS is a **big city**.

Inside this city, you built your own **housing society** called **VPC**.

Yesterday (Day 5), you built:

```
VPC
│
├── Public Subnet A
├── Public Subnet B
├── Private Subnet A
└── Private Subnet B

        |
        IGW
        |
     Internet
```

Now think...

Public people can go outside.

Private people cannot.

Today we learn:

> **How can private people safely go outside without anyone outside entering their house?**

That is today's lesson.

---

# Part 1 — NAT

First understand the problem.

Suppose there are two houses.

```
Public House
Private House
```

Public house has a road to the internet.

Private house has NO road.

Imagine this:

```
Internet
    |
    |
 IGW
    |
Public Subnet
    |
Private Subnet
```

Question:

Can the private subnet reach Google?

Answer:

❌ No.

Because there is no road.

---

Now imagine the private EC2 wants to install updates.

Like Ubuntu.

```
sudo apt update
```

Where does Ubuntu download packages from?

Internet.

But...

Private subnet cannot reach internet.

So...

It fails.

---

Now AWS says...

"I'll give you a helper."

That helper is called

# NAT

NAT = Network Address Translation

Don't worry about the long name.

Think of NAT as

> **Delivery Boy**

---

Imagine this.

You are inside your house.

You want pizza.

You cannot leave.

So...

You call the delivery boy.

```
You
 ↓
Delivery Boy
 ↓
Pizza Shop
 ↓
Delivery Boy
 ↓
You
```

Did pizza shop ever enter your house?

No.

Only the delivery boy went outside.

Exactly same thing.

---

Private EC2

↓

NAT Gateway

↓

Internet

↓

Response

↓

NAT Gateway

↓

Private EC2

---

Notice something.

Private EC2 never talks directly to internet.

NAT does.

---

# Real AWS Diagram

```
Internet
     |
     |
    IGW
     |
-------------------------
| Public Subnet         |
|                       |
| NAT Gateway           |
-------------------------
          |
          |
-------------------------
| Private Subnet        |
| EC2                   |
-------------------------
```

This is the most common AWS architecture.

---

# Why is NAT inside Public Subnet?

Very important interview question.

Let's think.

Can NAT reach internet?

Yes.

So NAT itself must have internet.

Only which subnet has internet?

Public subnet.

Therefore

NAT Gateway MUST be inside Public Subnet.

Never private.

---

# Why does NAT need Elastic IP?

Imagine your friend wants to call you.

Your phone number changes every minute.

Can they reach you?

No.

Elastic IP is a permanent public address.

NAT uses it to communicate with internet.

So

```
Internet

↓

Elastic IP

↓

NAT Gateway

↓

Private EC2
```

---

# Can Internet start talking to my Private EC2?

No.

This is the most important concept.

Internet cannot say

"Hello Private EC2."

Impossible.

Because

Private EC2 has no public IP.

Only NAT talks outside.

---

Think of apartment security.

Visitors cannot directly enter your flat.

Only security guard goes outside.

Guard returns.

Same idea.

---

# Traffic Flow

Suppose Private EC2 wants Google.

Step 1

Private EC2 sends request.

```
Private EC2

↓

Google
```

But...

There is no internet.

So route table says

```
Go to NAT.
```

Then

```
Private EC2

↓

NAT Gateway

↓

Internet

↓

Google
```

Google replies.

```
Google

↓

Internet

↓

NAT Gateway

↓

Private EC2
```

Finished.

---

# Very Important

Can Google start sending requests to Private EC2?

No.

Only responses to requests started by the private instance are returned.

That is why NAT is secure.

---

# NAT Gateway vs NAT Instance

AWS has two helpers.

Helper 1

NAT Gateway

Helper 2

NAT Instance

---

Imagine

NAT Gateway = Uber

NAT Instance = Your own car

---

Uber

AWS drives.

AWS repairs.

AWS services.

Easy.

---

Your own car

You repair.

You buy fuel.

You maintain.

Your responsibility.

---

AWS version

NAT Gateway

AWS manages.

NAT Instance

You manage EC2.

---

# Comparison

NAT Gateway

✅ Managed by AWS

✅ Easier

✅ Automatic maintenance

✅ Recommended

---

NAT Instance

You launch EC2.

You patch it.

You scale it.

You fix crashes.

More work.

---

AWS recommends

Always choose

NAT Gateway

unless you need something special.

---

# Why NAT Costs Money?

Because AWS keeps it running.

Even if no traffic comes,

AWS keeps NAT alive.

So you pay hourly.

If data passes,

you pay for data too.

Think of it like

Taxi waiting charge

*

Distance charge

---

# Multiple NATs

Imagine two buildings.

```
Building A

Building B
```

Each has private subnet.

One NAT?

Possible.

But if Building A loses power,

Building B also loses internet.

So companies usually create

One NAT per AZ.

Better reliability.

---

# Quick Check

Can Private EC2 access internet directly?

❌ No

Can NAT access internet?

✅ Yes

Where is NAT placed?

✅ Public Subnet

Does NAT have Elastic IP?

✅ Yes

Can internet directly reach private EC2?

❌ Never

***



# Part 2 – Security Groups vs Network ACLs (NACL)

---

# First, imagine a school 🏫

Inside the school there are many classrooms.

```text
School (VPC)

├── Class A (Public Subnet)
├── Class B (Private Subnet)
├── Class C (Private Subnet)
```

Each classroom is a **Subnet**.

Inside every classroom are students.

The students are your **EC2 instances**.

```text
School (VPC)

Classroom (Subnet)

Student (EC2)
```

---

# Now someone wants to visit a student.

Question:

Should we stop them at the classroom door?

Or

Should we stop them at the student's desk?

AWS gives us both options.

---

# Option 1

Stop them before entering the classroom.

This is

# Network ACL (NACL)

Think of it as the **security guard at the classroom door.**

---

# Option 2

Even if they enter the classroom,

the student decides

"I'll talk to you."

or

"No."

This is

# Security Group

Think of it as the **student's personal permission list.**

---

So

```text
Visitor

↓

School

↓

Classroom Door
(NACL)

↓

Student
(Security Group)
```

Visitor must pass BOTH.

---

# Real AWS

```text
Internet

↓

IGW

↓

Subnet
(NACL)

↓

EC2
(Security Group)
```

Both check the traffic.

---

# Security Group

Think of Security Group as

**Personal bodyguard of the EC2.**

Every EC2 has its own bodyguard.

Example

```text
EC2

↓

Security Group
```

Someone wants SSH.

Security Group asks

> Which port?

22

Allowed?

Yes.

Come inside.

---

Someone wants RDP.

Port

3389

Security Group checks.

Not allowed.

Go away.

---

Security Group protects

**Resources**

Examples

* EC2
* RDS
* Lambda ENI
* Interface Endpoint
* etc.

Not subnet.

---

# NACL

NACL protects

Entire subnet.

Example

```text
Subnet

EC2 A

EC2 B

EC2 C
```

One NACL protects

ALL three.

---

Think

Apartment gate.

Everyone entering apartment passes one gate.

Exactly like NACL.

---

# Biggest Difference

Security Group protects

ONE resource.

NACL protects

WHOLE subnet.

---

# Example

Imagine

```text
Subnet

EC2-1

EC2-2

EC2-3
```

One NACL

↓

Protects all three.

---

Each EC2 has

its own Security Group.

```text
EC2-1 → SG-A

EC2-2 → SG-B

EC2-3 → SG-C
```

Different rules.

---

# Allow vs Deny

Security Group

Only knows one word.

"ALLOW"

That's all.

No deny.

Example

```text
Allow Port 80

Allow Port 22
```

Done.

---

Can you write

"Deny Port 22"

No.

Impossible.

---

NACL

Can do both.

```text
Allow

Deny
```

Much more powerful.

---

# Why Deny?

Imagine

One hacker IP

```text
100.20.30.40
```

You want

Block forever.

NACL

```text
DENY

100.20.30.40
```

Finished.

Security Group

Cannot write deny.

---

# Stateful vs Stateless

This is where many students get confused.

Let's go slowly.

---

Imagine

You call your friend.

```text
You

↓

Friend
```

Friend answers.

```text
Friend

↓

You
```

Do you need permission again?

No.

Because

You started the call.

---

Security Group works like this.

You send request.

Response automatically comes back.

No extra rule needed.

This is called

# Stateful

Meaning

"I remember who started the conversation."

---

Example

Your laptop

↓

EC2

SSH Port 22

Allowed.

After login,

EC2 sends data back.

Security Group says

"I remember."

Allowed automatically.

You never created another rule.

---

Now NACL.

NACL has poor memory.

Imagine security guard has Alzheimer's.

Every time

He asks

"Who are you?"

Again.

Again.

Again.

He forgets everything.

---

You enter.

Checked.

Now you're leaving.

He checks again.

No memory.

---

That's Stateless.

Every packet checked independently.

---

So

Request

↓

Needs rule.

Response

↓

Needs another rule.

---

# Example

Laptop wants website.

```text
Laptop

↓

EC2 Port 80
```

NACL checks.

Allowed.

Website sends response.

```text
EC2

↓

Laptop
```

NACL checks AGAIN.

If response isn't allowed,

Connection fails.

---

# Why Ephemeral Ports?

Let's understand first.

Your browser opens Google.

Did your browser use Port 80?

No.

It uses a random port.

Example

```text
52344
```

Tomorrow

```text
60122
```

Next time

```text
49210
```

Random.

These are called

Ephemeral Ports.

Think

Temporary ports.

---

Example

Laptop

```text
Port 53000
```

↓

Website

```text
Port 80
```

Website replies

```text
Port 80

↓

53000
```

Notice

The reply goes to

53000

NOT

80.

---

So NACL must allow

those random ports.

AWS training usually says

```text
1024-65535
```

because random ports are in that range.

---

# Why Security Group Doesn't Need This?

Because

Security Group is stateful.

It remembers

"This response belongs to an existing connection."

Automatically allowed.

---

# Rule Evaluation

Security Group

Looks at

ALL rules.

Example

```text
Allow 22

Allow 80

Allow 443
```

If any one matches

Allowed.

Order doesn't matter.

---

NACL

Order matters.

Very important.

Example

Rule 100

Allow HTTP

Rule 90

Deny Everything

Which runs first?

90

Traffic blocked.

AWS checks

Lowest number first.

---

Example

```text
90  DENY ALL

100 ALLOW 80
```

HTTP never reaches Rule 100.

Already denied.

---

# Default Behavior

Security Group

New Security Group

```text
Inbound

Nothing allowed.

Outbound

Everything allowed.
```

Meaning

Nobody can enter.

But EC2 can go outside.

---

Custom NACL

Starts

Everything denied

Until you create rules.

---

Default NACL

AWS already creates

Allow all.

---

# Easy Memory Trick

Imagine airport.

---

Security Group

Personal passport officer.

Checks

YOU.

---

NACL

Airport gate.

Checks

EVERYONE.

---

# Quick Comparison

| Feature               | Security Group     | NACL                  |
| --------------------- | ------------------ | --------------------- |
| Protects              | EC2/ENI (resource) | Entire Subnet         |
| Allow                 | ✅ Yes              | ✅ Yes                 |
| Deny                  | ❌ No               | ✅ Yes                 |
| Stateful              | ✅ Yes              | ❌ No                  |
| Checks Return Traffic | Automatically      | Must allow separately |
| Rule Order            | Doesn't matter     | Lowest number wins    |

---

# Mini Quiz 😊

**1. Which protects an EC2 instance?**

👉 **Security Group**

---

**2. Which protects an entire subnet?**

👉 **NACL**

---

**3. Which supports Deny rules?**

👉 **NACL**

---

**4. Which is stateful?**

👉 **Security Group**

---

**5. Which is stateless?**

👉 **NACL**

---

**6. In a NACL, if request is allowed, is the response automatically allowed?**

👉 **No.** You must explicitly allow the return traffic (including the ephemeral port range).

***


# Part 3 – Gateway Endpoint vs Interface Endpoint

---

## First, remember our VPC

From Day 5, we have this.

```text
                    Internet
                        |
                       IGW
                        |
        ---------------------------------
        |          Public Subnet         |
        |      NAT Gateway               |
        ---------------------------------
                    |
        ------------------------------
        |       Private Subnet        |
        |        Private  EC2               |
        ------------------------------
```

Now suppose your Private EC2 wants to read a file from an S3 bucket.

Question:

How does it reach S3?

You may think:

> Through NAT.

That is one way.

But AWS says...

> "Wait! Why send traffic to the internet if you're only talking to another AWS service?"

That's why VPC Endpoints exist.

---

## Option 1 – Using NAT

The EC2 says,

> "I want to reach S3."

The request goes like this.

```text
Private EC2

↓

NAT Gateway

↓

Internet

↓

Amazon S3
```

Then S3 replies.

```text
Amazon S3

↓

Internet

↓

NAT Gateway

↓

Private EC2
```

Works?

✅ Yes.

But...

Did the traffic leave AWS's private network?

Yes.

It went through the Internet Gateway.

AWS says,

> "I have a better way."

---

# AWS creates a shortcut.

Imagine your city.

Normally you drive like this.

Your house is inside a gated society.

Outside the society is a shopping mall.

Normally, you leave the gate, go onto the main road, then reach the mall.

```text
House

↓

Society Gate

↓

Main Road

↓

Mall
```
This is like using a NAT Gateway.

---



But then the city builds a special road.

Now imagine the mall builds a private tunnel directly into your society.

You never use the main road.

```text
Home

↓

Secret Road / Private Tunnel

↓

Mall
```

Faster.

Safer.

No public road.

AWS did exactly this.

That special road is called a

# VPC Endpoint

A VPC Endpoint lets your VPC connect to AWS services **privately**, without sending traffic through the Internet Gateway or NAT Gateway.

---

# Why did AWS create VPC Endpoints?

Because companies wanted:

✅ More security

✅ Lower cost (for some services)

✅ Private communication

Instead of

```text
Private EC2

↓

Internet

↓

S3
```

AWS made this.

```text
Private EC2

↓

Private AWS Network

↓

S3
```

Notice

No Internet.

No NAT.

---

Why is this better?

Because

- No internet
- More secure
- Faster path
- Traffic stays inside the AWS network

---

# What exactly is a VPC Endpoint?

Think of it as

> A **private door** from your VPC directly into an AWS service.

No Internet Gateway.

No NAT.

No public IP.

Instead of walking outside the building,

you walk through a private hallway.

---

# AWS has TWO kinds of Endpoints.

```text
VPC Endpoint

├── Gateway Endpoint
└── Interface Endpoint
```

Today we'll understand both.

---

# Type 1 - Gateway Endpoint

Gateway Endpoint is very simple.

Gateway Endpoint works for ONLY two AWS services.

Can you guess?

It only works with

✅ Amazon S3

✅ DynamoDB

Nothing else.

Very important.

No other services.

---

Imagine your city has only two special buildings.

Your house.

```text
Home
```

Your school.

```text
School
```

The government builds a special road

ONLY

between

Home

and

School.

Not hospital.

Not mall.

Only school.

That's Gateway Endpoint.

AWS built a private road only for

S3

and

DynamoDB.

### Another Example:

```text
Society

↓

Private Road

↓

S3 Warehouse
```

OR

```text
Society

↓

Private Road

↓

DynamoDB Office
```

---

### Diagram

Without Gateway Endpoint

```text
Private EC2

↓

NAT

↓

Internet

↓

S3
```
Traffic leaves through the NAT.

---

With Gateway Endpoint

```text
Private EC2

↓

Gateway Endpoint

↓

Amazon S3
```

Notice

No NAT.

No Internet.

---

# How does Gateway Endpoint work?
OR
# Does Gateway Endpoint create an EC2?

This is important.

It does NOT create a new machine.

It does NOT create an IP address.

It simply adds a new route.
It simply updates the Route Table.

Think of Google Maps.

Yesterday your route was
```text
S3

↓

NAT

↓

Internet
```
Today AWS changes the map.
```text
S3

↓

Gateway Endpoint
```
The traffic follows the new road.


Remember Route Tables?

Earlier we had

```text
0.0.0.0/0

↓

NAT Gateway
```

Now AWS adds another route.

```text
S3 Prefix

↓

Gateway Endpoint
```

So when EC2 says

"I want S3"

Route Table says

"No need for NAT."

"Go here."

Very smart.

---

Very Important

Gateway Endpoint works by modifying the

✅ Route Table

Not Security Group.

Not EC2.

Not IGW.

Only Route Table.

---

Which services?

Only remember
```text
Gateway Endpoint

↓

S3

DynamoDB
```
Nothing else.




---

# Does Gateway Endpoint have a Security Group?

No.

Why?

Because nothing is created.

No ENI.

No EC2.

No network interface.

Only Route Table changes.

---

# Easy Memory Trick

Gateway Endpoint

↓

Route Table

↓

S3/DynamoDB

That's it.

---

# Type 2 - Interface Endpoint

Now suppose EC2 wants to access

- Secrets Manager
- CloudWatch
- Systems Manager (SSM)
- KMS
- ECR API
- Many other AWS services

Can Gateway Endpoint help?

No.

Gateway Endpoint only understands

S3

and

DynamoDB.

AWS needed another solution.

AWS says

We'll create a tiny network card inside your subnet.

That network card is called an ENI (Elastic Network Interface).

---

Imagine AWS builds

A small office

inside your neighborhood.

Instead of travelling outside,

you visit that office.

That office contacts AWS.

That's Interface Endpoint.

---

Think of it like this

Your house gets a private door.
```text
House

↓

Private Door

↓

AWS Service
```
This private door has its own private IP address.

That's the Interface Endpoint.

---

Technically,

AWS creates

an

# ENI

Elastic Network Interface.

Think of ENI as

A virtual network card.

It gets a

Private IP.

---

Diagram

```text
Private EC2

↓

Private IP

↓

Interface Endpoint (ENI)

↓

AWS Service
```

No Internet.

Everything stays private.

Notice

The Interface Endpoint actually lives inside your subnet.

---

# Does it have an IP?

Yes.

Private IP.

---

# Does it have a Security Group?

Yes.

Very important.

Unlike Gateway Endpoint.

---

Gateway Endpoint

❌ No Security Group

---

# What is ENI?

Remember EC2.

Every EC2 has

a network card.

AWS calls it

Elastic Network Interface.

Interface Endpoint creates one too.

Example

```text
Subnet

EC2

192.168.1.20

↓

Interface Endpoint

192.168.1.100
```

Both have private IPs.

They communicate privately.

---

# Why Security Group?

Since Interface Endpoint creates an ENI,


Gateway Endpoint cannot, because it creates no ENI.


Interface Endpoint  Has Security Group Exactly like EC2 Because it's an ENI.

Remember from Part 2:

Security Groups are attached to network interfaces (ENIs).



---

# Private DNS



Let's simplify.

Normally

When you type

```text
s3.amazonaws.com
```

DNS finds

Public IP.

With Interface Endpoint,

AWS can change the answer.

Instead of

Public IP,

DNS returns

Private IP.

Example

Without Endpoint

```text
CloudWatch

↓

52.x.x.x
```

With Interface Endpoint

```text
CloudWatch

↓

10.0.2.50
```

Same name.

Different IP.

Everything stays inside AWS.

---


# PrivateLink

You will often hear:

> AWS PrivateLink

Don't panic.

PrivateLink is simply the technology used by Interface Endpoints.

Think:

```text
Interface Endpoint

=

PrivateLink
```

---

# DNS Magic

This is another important concept.

Normally, when your EC2 accesses S3 or another AWS service, it uses a hostname like:

```text
service.amazonaws.com
```

With an Interface Endpoint and **Private DNS** enabled, AWS performs a trick.

Instead of resolving that hostname to a public IP,

it resolves it to the **private IP of the Interface Endpoint**.

So your application doesn't need to change its code.

It still uses the same hostname, but the traffic stays private.

---

# Comparison Story

Imagine you order food.

---

## NAT

```text
You

↓

Public Road

↓

Restaurant
```

Uses the public road.

---

## Gateway Endpoint

```text
You

↓

Private Tunnel

↓

S3 Warehouse
```

Only for S3 and DynamoDB.

---

## Interface Endpoint

```text
You

↓

Private Door

↓

Any Supported AWS Office
```

For many AWS services.

---


# Does Interface Endpoint replace IAM?

No.



Suppose

Network says

"Yes."

Suppose your EC2 reaches S3 through a Gateway Endpoint.

Can it automatically read every bucket?

❌ No.

Network is only one layer.

AWS still checks permissions.

Think of it this way:

The Endpoint only opens the **road**.

It does **not** give you permission to enter the building.

AWS still checks things like:

* IAM policies
* Endpoint policy (if configured)
* S3 bucket policy
* KMS key policy (if the object is encrypted)
* SCPs (if you're using AWS Organizations)

The road and the permissions are two different things.


Example

```text
IAM Policy

↓

Endpoint Policy

↓

Bucket Policy

↓

KMS Key Policy (if encrypted)

↓

SCP (if Organization)

↓

Permission Boundary (if used)
```

Only if all required permissions allow the action will it succeed.

Think of it like this:

The road lets you reach the library, but you still need a valid library card to borrow a book.

---

# Which one should I choose?

# Easy Decision Rule

### Want S3 or DynamoDB privately?

👉 Use a **Gateway Endpoint**.

---

### Want CloudWatch, SSM, Secrets Manager, KMS, ECR API, etc. privately?

👉 Use an **Interface Endpoint**.

---

### Want to browse Google or download Ubuntu updates?

👉 Use a **NAT Gateway**.

Endpoints are only for AWS services, not the public internet.

---

# Quick Comparison

| Feature           | Gateway Endpoint   | Interface Endpoint                                                                     |
| ----------------- | ------------------ | -------------------------------------------------------------------------------------- |
| Works for          | S3 & DynamoDB only | Many AWS services                                                                      |
| Lives as         | Route Table entry         | ENI with private IP              |
| Creates ENI       | ❌ No               | ✅ Yes                                                                                  |
| Private IP        | ❌ No               | ✅ Yes                                                                                  |
| Security Group    | ❌ No               | ✅ Yes (on the endpoint ENI)                                                            |
| Uses Route Table  | ✅ Yes              | Not as the main mechanism; traffic goes to the endpoint ENI via its private IP and DNS |
| Uses PrivateLink  | ❌ No               | ✅ Yes                                                                                  |
| Internet Required | ❌ No               | ❌ No                                                                                   |
| Cost             | No endpoint hourly charge | Hourly + data processing charges |

---

# Easy Memory Trick

Imagine your school.

**Gateway Endpoint** = **Special road** that only goes to **S3 and DynamoDB**.

**Interface Endpoint** = **Small AWS office inside your neighborhood** with its own **private address (ENI + Private IP)** that can connect you to many AWS services.


Think of this sentence:

> **"S3 takes the Gateway. Everything else takes the Interface."**


---

# Mini Quiz 😊

**1. Which endpoint is only for S3 and DynamoDB?**

👉 **Gateway Endpoint**

---

**2. Which endpoint creates an ENI with a private IP?**

👉 **Interface Endpoint**

---

**3. Which endpoint can have a Security Group?**

👉 **Interface Endpoint**

---

**4. Does a VPC Endpoint remove the need for IAM permissions?**

👉 **No.** It provides a private network path, but IAM and other authorization policies are still checked.

***

