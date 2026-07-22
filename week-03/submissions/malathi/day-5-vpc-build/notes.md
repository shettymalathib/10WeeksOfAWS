Absolutely. I'll teach this as if you're **8 years old**, one step at a time. Don't memorize. **Understand the story.** Once you understand the story, AWS networking becomes easy.

---

# Day 5 - VPC, CIDR, Subnets and Routing

## Imagine this...

Your father buys a **big piece of land**.

On this land, he builds:

* your house 🏠
* your uncle's house 🏠
* a garden 🌳
* a parking area 🚗

Everything is inside one compound.

That entire compound is called a...

# VPC

**VPC = Virtual Private Cloud**

Think:

> VPC = Your own private city inside AWS.

Nobody outside can enter unless you allow them.

---

# Real World

Suppose Amazon says

> "Malathi, here is your own private network."

That private network is your VPC.

Everything you create later—

* EC2
* RDS
* Load Balancer
* Kubernetes

—all live inside this VPC.

So remember:

```
AWS Region
    ↓
   VPC
      ↓
 EC2
 RDS
 ELB
 etc.
```

Everything starts with the VPC.

---

# What does "Regional" mean?

Imagine India.

India has many states.

Similarly AWS has Regions.

Example:

```
Mumbai
Singapore
Tokyo
London
Ohio
```

If you create a VPC in Mumbai,

it stays only in Mumbai.

It DOES NOT automatically exist in Singapore.

That's why AWS says

> VPC is Regional.

---

# What is CIDR?

This scares many beginners.

Actually it is very easy.

Suppose your father says

> "Our colony will have house numbers from 1 to 100."

Those numbers are the address space.

Similarly AWS needs IP addresses.

Instead of saying

```
1-100
```

AWS says

```
10.10.0.0/16
```

This is called

**CIDR Block**

It simply means

> "This VPC owns a big range of IP addresses."

Nothing more.

---

# What is this /16?

Ignore AWS for one minute.

Imagine pizza.

A pizza has

```
🍕🍕🍕🍕🍕🍕🍕🍕
```

Suppose you cut it into fewer pieces.

Large pieces.

Suppose you cut into many pieces.

Small pieces.

CIDR works similarly.

Smaller number after /

```
/16
```

means

Huge network.

Larger number

```
/24
```

means

Smaller network.

So

```
/16
```

is BIG.

```
/24
```

is smaller.

```
/28
```

is very tiny.

---

# Think of a school

School

↓

Many classrooms

The school is the VPC.

The classrooms are Subnets.

---

# What is a Subnet?

Suppose your land is huge.

Instead of everyone living randomly,

you divide it.

```
-------------------------

Family A

-------------------------

Family B

-------------------------

Garden

-------------------------

Parking

-------------------------
```

These smaller sections are

Subnets.

AWS also divides the VPC into smaller parts.

Example

```
VPC

10.10.0.0/16

↓

Public subnet

10.10.1.0/24

↓

Private subnet

10.10.11.0/24
```

The subnet must always fit inside the VPC.

Like

A bedroom must be inside the house.

It cannot exist outside.

---

# Why divide?

Suppose guests visit.

Will they enter your bedroom?

No.

They only enter the hall.

Similarly

Public things

↓

Public subnet

Private things

↓

Private subnet

---

Example

Public subnet

```
Web Server
Load Balancer
```

Private subnet

```
Database
Backend Server
```

Nobody from internet should directly reach the database.

---

# Availability Zone

Suppose Mumbai has

```
Building A

Building B

Building C
```

AWS calls these

Availability Zones.

Example

```
Mumbai

↓

AZ A

AZ B

AZ C
```

Now important.

One subnet belongs to only ONE AZ.

Example

```
Public-A

↓

AZ A
```

It cannot belong to

AZ A

and

AZ B

together.

Just like

One classroom cannot be inside two buildings.

---

# Internet Gateway (IGW)

Imagine your house.

There is one main gate.

Without the gate,

nobody can enter or leave.

Internet Gateway is that gate.

```
Internet

↓

IGW

↓

VPC
```

Without IGW,

your VPC is isolated.

No internet.

---

# Route Table

Imagine Google Maps.

You want to go somewhere.

Google Maps tells you

Take Road A

Turn Left

Take Highway

AWS also needs directions.

It asks

> "Where should I send this packet?"

The answer comes from

Route Table.

---

Example

```
Destination

↓

Internet

↓

Go to Internet Gateway
```

Another

```
Destination

↓

Inside VPC

↓

Stay inside
```

That's all a Route Table does.

It is simply

A list of directions.

---

Example Route Table

```
Destination

10.10.0.0/16

↓

local
```

Meaning

"If the destination is inside my VPC,

don't send it outside."

---

Another route

```
0.0.0.0/0

↓

Internet Gateway
```

Meaning

"If destination is anywhere else,

go to internet."

---

# What is Local Route?

AWS automatically creates this.

```
10.10.0.0/16

↓

local
```

Meaning

Every server inside the VPC

can talk to another server.

No internet needed.

Example

Web Server

↓

Database

Both inside VPC.

They communicate using Local Route.

---

# What makes a subnet Public?

Very important interview question.

Many people answer wrongly.

People think

> "Because it has a public IP."

Wrong.

The REAL answer

A subnet is Public

ONLY IF

its Route Table contains

```
0.0.0.0/0

↓

Internet Gateway
```

That is the definition.

---

# Then why Public IP?

Suppose the road exists.

But your house has no address.

Can Amazon deliver?

No.

Similarly

Even if the subnet has IGW,

the EC2 also needs

Public IP

(or Elastic IP)

to communicate over the internet.

Think of it this way:

* **Route table** = There is a road.
* **Public IP** = Your house has an address on that road.

You need **both**.

---

# Private Subnet

Private subnet has

```
Local Route
```

Only.

No internet route.

So internet cannot directly reach it.

Perfect for

* Database
* Redis
* Internal servers

---

# Main Route Table

Every VPC automatically gets one Route Table.

This is called

Main Route Table.

If you forget to attach another Route Table,

AWS uses the Main one.

Think of it as

Default Route Table.

---

# Elastic IP

Normally

Public IP changes

after stopping and starting an EC2 instance.

Elastic IP

never changes.

Imagine

Your phone number.

Instead of changing every day,

it stays the same.

That's Elastic IP.

---

# Why avoid overlapping CIDRs?

Suppose your home address is

```
House No. 12
```

Your neighbor also says

```
House No. 12
```

The courier gets confused.

Similarly,

if two networks use the same IP range,

AWS doesn't know where traffic should go.

Example (bad):

```
Office

10.10.0.0/16

Home

10.10.0.0/16
```

Now AWS cannot distinguish them.

That's why CIDRs must not overlap.

---

# AWS reserves 5 IP addresses

Suppose your subnet is

```
10.10.1.0/24
```

There are **256 total IP addresses**.

AWS reserves **5** of them for networking purposes.

| Address     | Purpose                                                                       |
| ----------- | ----------------------------------------------------------------------------- |
| 10.10.1.0   | Network address                                                               |
| 10.10.1.1   | VPC router                                                                    |
| 10.10.1.2   | Amazon DNS                                                                    |
| 10.10.1.3   | Reserved for future use                                                       |
| 10.10.1.255 | Network address end (broadcast address, though AWS doesn't support broadcast) |

So:

```
256 total
-5 reserved
---------
251 usable
```

That's why a **/24 subnet has 251 usable IP addresses**.

---

# Longest Prefix Match

Suppose there are two directions.

One says

```
Go anywhere.
```

Another says

```
Go specifically to your house.
```

Which one is more accurate?

The second one.

AWS always chooses the **most specific route**.

Example:

```
10.10.0.0/16 → local

0.0.0.0/0 → IGW
```

If the destination is **10.10.1.50**, AWS chooses:

```
10.10.0.0/16 → local
```

because it's more specific than "go anywhere."

---

# Security Groups vs Route Tables

This is another common interview question.

Imagine a shopping mall.

* **Roads** help you reach the mall.
* **Security guards** decide whether you may enter.

Similarly:

* **Route Table** = Decides **where traffic goes**.
* **Security Group / NACL** = Decides **whether traffic is allowed**.

A road to the mall doesn't guarantee you'll be allowed inside.

---

# Final Story (Remember This)

```
AWS Region
        │
        ▼
      VPC (your private city)
        │
 ┌──────┴──────┐
 ▼             ▼
Public      Private
Subnet       Subnet
 │             │
EC2         Database
 │
Route Table
 │
0.0.0.0/0
 │
Internet Gateway
 │
Internet
```

---

# Check Your Understanding (Answers)

1. **What makes a subnet public?**
   **Answer:** A subnet is public when its **route table has a route `0.0.0.0/0 → Internet Gateway (IGW)`**. A public IP alone does **not** make it public.

2. **How many usable IPv4 addresses does an AWS `/24` subnet provide?**
   **Answer:** **251 usable IP addresses** (256 total minus 5 reserved by AWS).

3. **Why must application VPC ranges not overlap with on-premises ranges?**
   **Answer:** Because overlapping IP ranges cause routing confusion. AWS won't know whether traffic should go to the VPC or the on-premises network.

4. **Does enabling auto-assign public IPv4 add an IGW route?**
   **Answer:** **No.** Auto-assigning a public IP only gives instances a public address. It does **not** create or modify route table entries.

5. **Which route wins: `/16` or `/0` for an address matching both?**
   **Answer:** **`/16` wins**, because AWS always uses the **longest (most specific) prefix match**.
