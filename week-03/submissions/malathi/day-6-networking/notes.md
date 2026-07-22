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



# Part 4 – VPC Peering vs Transit Gateway

---

# First, what is a VPC?

Remember Day 5.

A VPC is like your own housing society.

```text
VPC A
```

Imagine another company builds another housing society.

```text
VPC B
```

Now we have

```text
🏠 Society A (VPC A)

🏠 Society B (VPC B)
```

---

## Can they talk?

No.

They are completely isolated.

Imagine two islands.

```text
Island A        Island B
```

Can people walk between them?

❌ No.

There is no bridge.

AWS is exactly the same.

Different VPCs cannot communicate unless **you create a connection**.

---

# AWS says

"I'll build a bridge."

That bridge is called

# VPC Peering

---

Imagine

```text
Village A ------------ Village B
```

Now people can travel.

That's VPC Peering.

---

## Real AWS

```text
VPC A

──────────────

VPC B
```

No communication.

After Peering

```text
VPC A
    │
    │
    │
VPC B
```

Now EC2 in VPC A can talk to EC2 in VPC B (assuming routing and security rules allow it).

---

# Example

Company has

Production VPC

and

Database VPC

```text
Production

↓

Peering

↓

Database
```

Application server talks to database.

Simple.

---

# Does Peering automatically work?

No.

Many beginners think

"I created peering."

Done.

Wrong.

You must also tell each VPC

where to send traffic.

Remember

# Route Tables

---

Imagine Google Maps.

If maps don't know the road,

can they drive?

No.

Same in AWS.

Route Tables need entries.

---

Example

VPC A

CIDR

```
10.0.0.0/16
```

VPC B

CIDR

```
10.1.0.0/16
```

Route Table in A

```text
Destination

10.1.0.0/16

↓

Peering Connection
```

Route Table in B

```text
Destination

10.0.0.0/16

↓

Peering Connection
```

Both sides need routes.

Think of it like two cities each putting up a road sign pointing to the other.

---

# Can overlapping CIDRs peer?

Suppose

VPC A

```
10.0.0.0/16
```

VPC B

```
10.0.0.0/16
```

Question

If packet says

"I need 10.0.5.10"

Which VPC owns it?

AWS cannot know.

So AWS says

❌ No Peering.

CIDRs must **not overlap**.

---

# Biggest Trap

This is asked in interviews.

Imagine

Three VPCs.

```text
VPC A

VPC B

VPC C
```

Create

A ↔ B

and

A ↔ C

Like this

```text
      B
      |
      |
      A
      |
      |
      C
```

Question

Can B talk to C?

Most beginners answer

Yes.

Wrong.

---

# Why?

Because

VPC Peering is

# NOT TRANSITIVE

Let's understand.

---

Imagine your friend.

You know Rahul.

Rahul knows Priya.

Does that automatically mean

you know Priya?

No.

Someone has to introduce you.

Exactly.

AWS says

A knows B.

A knows C.

But

B does NOT know C.

---

So

```text
B

↓

A

↓

C
```

Traffic stops.

AWS does not forward it.

---

This is called

# Non-Transitive Routing

Remember this forever.

---

# If B wants C?

You need another peering.

```text
A ↔ B

A ↔ C

B ↔ C
```

Now everyone has direct roads.

---

# Problem

Imagine

10 VPCs.

How many peerings?

Lots!

Example

```text
A-B

A-C

A-D

B-C

B-D

C-D
```

It becomes messy very quickly.

Lots of route tables.

Lots of maintenance.

---

AWS thought,

"There must be a better way."

So AWS invented

# Transit Gateway

---

# Think of an Airport

Imagine

Mumbai

Delhi

Chennai

Kolkata

Instead of every city having a direct flight to every other city,

they all connect to one big airport hub.

```text
Mumbai

      |

Delhi--Hub--Chennai

      |

Kolkata
```

Need Mumbai → Chennai?

Go through the hub.

Easy.

Transit Gateway works exactly like that.

---

# Real AWS

```text
VPC A

     |

VPC B

     |

Transit Gateway

     |

VPC C

     |

VPC D
```

Every VPC connects only once—to the Transit Gateway.

---

# Now Question

Can

A talk to D?

Yes.

Because the Transit Gateway acts as the central router (subject to its route tables and any security controls).

---

# Why is it called Transit?

Because traffic

**transits** (passes through)

the Gateway.

---

# Does TGW remove Route Tables?

No.

You still configure routes.

There are:

* VPC Route Tables (inside each VPC)
* Transit Gateway Route Tables (inside the TGW)

The TGW decides which attached VPCs can reach each other.

---

# Real Example

Imagine a company.

Production

Development

Testing

Shared Services

```text
Prod

Dev

Test

Shared
```

Instead of

connecting each pair,

everyone connects to

Transit Gateway.

Cleaner.

Easier.

---

# Peering vs Transit

Imagine roads.

### Peering

Private road between two houses.

```text
House A -------- House B
```

Only two houses.

---

### Transit Gateway

Big traffic circle.

```text
      A

      |

B -- Roundabout -- C

      |

      D
```

Everyone enters the roundabout.

Everyone can reach where they're allowed to go.

---

# Which one is cheaper?

For **2 VPCs**

Peering is usually simpler.

For **50 VPCs**

Transit Gateway is much easier to manage, though it has its own pricing.

---

# Easy Memory Trick

**VPC Peering**

= One bridge between two islands.

**Transit Gateway**

= One central airport or bus station connecting many cities.

---

# Quick Comparison

| VPC Peering                | Transit Gateway                             |
| -------------------------- | ------------------------------------------- |
| Connects two VPCs directly | Central hub connecting many VPCs            |
| One-to-one                 | Hub-and-spoke                               |
| Needs routes in both VPCs  | Uses VPC and TGW route tables               |
| No transitive routing      | Supports transitive routing through the hub |
| CIDRs must not overlap     | Careful CIDR planning is still required     |

---

# Mini Quiz 😊

### 1. Can two VPCs communicate automatically?

👉 **No.**

---

### 2. What creates a direct connection between two VPCs?

👉 **VPC Peering**

---

### 3. Is VPC Peering transitive?

👉 **No.**

If A ↔ B and A ↔ C, **B cannot reach C** through A.

---

### 4. What AWS service acts like a central networking hub?

👉 **Transit Gateway**

---

### 5. Which is better for 100 VPCs?

👉 **Transit Gateway**, because managing direct peerings between every pair of VPCs becomes complex.

***


# Part 5 – VPC Flow Logs

---

# Imagine your school

Yesterday we learned

* Security Guard (NACL)
* Bodyguard (Security Group)

Now imagine something happens.

A student says,

> "Someone came to my classroom."

Teacher asks,

> "Who?"

Student says,

> "I don't know."

Teacher asks,

> "What time?"

Student says,

> "I don't know."

Teacher asks,

> "Which classroom?"

Student says,

> "I don't know."

Can anyone solve the problem?

❌ No.

There is no evidence.

---

Now imagine the school has CCTV.

Every person entering is recorded.

Teacher can now see

```text
Who came?

When?

From where?

Which room?

Allowed?

Blocked?
```

Now we have evidence.

AWS does the same thing.

---

# What is VPC Flow Log?

A **VPC Flow Log** is like a **CCTV camera for your network traffic**.

It records

* Who talked
* Who received
* Which port
* Which protocol
* Whether AWS allowed or rejected the traffic

It **does not** record the actual data inside the packets.

---

Imagine your house.

Someone knocks.

CCTV records

```text
Person came

3 PM

Front Door

Allowed
```

Did CCTV record the conversation?

No.

It only records

* Time
* Person
* Entry

Flow Logs are exactly like that.

---

# Very Important

Flow Logs do NOT record

❌ Passwords

❌ Website contents

❌ Packet contents

❌ HTTP body

❌ Images

❌ Files

They only record

**Network metadata**.

---

# What is Metadata?

Think of a courier parcel.

Outside the box

```text
From

To

Date

Weight
```

Inside the box

Birthday gift.

Courier company doesn't know what's inside.

It only knows the label.

Flow Logs are exactly the same.

---

# Example

Suppose

Your laptop

opens a website.

```text
Laptop

↓

EC2
```

Flow Log records something like

```text
Source IP

Destination IP

Port

Protocol

Action
```

Example

```text
203.0.113.10

↓

10.0.1.20

Port 80

TCP

ACCEPT
```

Notice

It doesn't record

```text
GET /index.html
```

or

```text
Username = John
```

Only connection details.

---

# Where can Flow Logs be enabled?

AWS allows you to enable Flow Logs at three levels.

## 1. Entire VPC

Everything inside the VPC is monitored.

```text
VPC

↓

Flow Logs
```

---

## 2. One Subnet

Only one subnet is monitored.

```text
Subnet

↓

Flow Logs
```

---

## 3. One ENI (Network Interface)

Only one EC2/network interface is monitored.

```text
EC2

↓

ENI

↓

Flow Logs
```

---

# Where are Flow Logs stored?

AWS doesn't keep them forever automatically.

You choose where to send them.

Usually

### Option 1

CloudWatch Logs

Good for searching.

---

### Option 2

Amazon S3

Good for long-term storage.

---

### Option 3

Amazon Data Firehose

Good for streaming logs to analytics systems.

---

# What information is recorded?

Imagine this traffic.

```text
Laptop

↓

EC2
```

Flow Log may record

```text
Source IP

Destination IP

Source Port

Destination Port

Protocol

Packets

Bytes

Action
```

Let's understand each one.

---

## Source IP

Who started the conversation?

Example

```text
203.0.113.10
```

This is the client's IP.

---

## Destination IP

Who received it?

Example

```text
10.0.1.15
```

Your EC2.

---

## Source Port

Temporary port.

Example

```text
53000
```

Remember ephemeral ports?

Exactly.

---

## Destination Port

Service port.

Example

```text
80
```

Means HTTP.

---

## Protocol

How is data sent?

Usually

```text
TCP

UDP

ICMP
```

---

## Packets

How many packets were exchanged.

---

## Bytes

How much data was transferred.

---

## Action

This is very important.

Two possibilities.

```text
ACCEPT

REJECT
```

---

# ACCEPT

Imagine security guard.

Visitor comes.

Security guard says

> Come inside.

Flow Log records

```text
ACCEPT
```

---

# REJECT

Visitor comes.

Security guard says

> No entry.

Flow Log records

```text
REJECT
```

---

# Very Important Interview Question

If Flow Log says

```text
ACCEPT
```

Does it mean the application worked?

Most beginners say

Yes.

Wrong.

---

Example

Your EC2

accepted network traffic.

But

Apache isn't running.

Browser shows

```text
Connection Failed
```

Flow Log still says

```text
ACCEPT
```

because

The **network** allowed the traffic.

The **application** failed.

This is why the notes say:

> **An `ACCEPT` record proves the network controls accepted the flow; it does not prove the application listener or process worked.**

---

# Log Status

Besides ACCEPT/REJECT, you'll also see a status.

Three common values.

```text
OK

NODATA

SKIPDATA
```

Let's understand.

---

# OK

Everything normal.

AWS successfully recorded the traffic.

Think

Teacher checked attendance.

Everything saved.

---

# NODATA

No traffic happened.

Imagine CCTV watched the classroom all day.

Nobody entered.

CCTV still worked.

There was simply nothing to record.

That's

```text
NODATA
```

---

# SKIPDATA

AWS wanted to record traffic,

but some records couldn't be captured.

Usually because of internal service limitations (for example, under very high logging load).

Think

CCTV recorded most visitors,

but missed a few because it couldn't keep up.

That's

```text
SKIPDATA
```

---

# Troubleshooting Example

Suppose

You cannot SSH into EC2.

Question

Where do you look?

Flow Logs.

You find

```text
REJECT
```

That means

Something in the network blocked it.

Maybe

* NACL
* Security Group
* Route problem

Now suppose

Flow Logs show

```text
ACCEPT
```

But SSH still doesn't work.

Now you know

The network allowed it.

The problem may be

* SSH service not running
* Wrong username
* Wrong key pair
* OS firewall
* Application issue

Flow Logs helped narrow down the problem.

---

# CloudWatch Logs Insights

AWS lets you search logs using queries.

Example

```text
fields @timestamp, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, action
| filter action = "REJECT"
| stats count(*) as rejects by dstAddr, dstPort
| sort rejects desc
| limit 20
```

Don't worry about memorizing this.

It simply asks:

> "Show me the destinations and ports with the most rejected connections."

Think of it like searching CCTV footage for only people who were denied entry.

---

# Easy Memory Trick

Imagine

Flow Logs = CCTV Camera

Security Group = Personal Bodyguard

NACL = Apartment Gate Security

NAT = Delivery Boy

Gateway Endpoint = Private Road to S3/DynamoDB

Interface Endpoint = Private AWS Office (ENI)

Transit Gateway = Airport Hub

These analogies fit together and make the networking concepts much easier to remember.

---

# Quick Revision of Day 6

## NAT Gateway

Private EC2 → NAT → Internet

✔ Outbound only

✔ Lives in Public Subnet

✔ Has Elastic IP

---

## Security Group

Protects EC2/ENI

✔ Stateful

✔ Allow only

---

## NACL

Protects Subnet

✔ Stateless

✔ Allow + Deny

✔ Return traffic must also be allowed

---

## Gateway Endpoint

✔ S3

✔ DynamoDB

✔ Route Table entry

✔ No ENI

---

## Interface Endpoint

✔ Many AWS services

✔ Creates ENI

✔ Has Private IP

✔ Can have Security Group

---

## VPC Peering

✔ Connects two VPCs directly

✔ Not transitive

---

## Transit Gateway

✔ Connects many VPCs

✔ Central hub

✔ Supports transitive routing through the hub

---

## VPC Flow Logs

✔ Network metadata only

✔ No packet contents

✔ Shows ACCEPT or REJECT

✔ Can be sent to CloudWatch Logs, S3, or Data Firehose

✔ `ACCEPT` means the **network** allowed the traffic—not necessarily that the application worked.

---

# One Final Story (Remember the Whole Day)

Imagine your company:

🏢 **VPC** = Office Building

🚪 **Security Group** = Employee's Cabin Door

🚧 **NACL** = Main Gate Security

📦 **NAT Gateway** = Office Delivery Person who goes outside and brings things back

🛣️ **Gateway Endpoint** = Private Road to the Warehouse (S3/DynamoDB)

🏬 **Interface Endpoint** = Private Service Desk inside the office for many AWS services

🌉 **VPC Peering** = Bridge connecting two office buildings

🚌 **Transit Gateway** = Central Bus Terminal connecting many office buildings

📹 **VPC Flow Logs** = CCTV system recording who entered, where they went, and whether they were allowed or blocked (but not what they talked about).

If you remember this story, you'll be able to reconstruct most of the Day 6 networking concepts even if you forget the exact wording.

