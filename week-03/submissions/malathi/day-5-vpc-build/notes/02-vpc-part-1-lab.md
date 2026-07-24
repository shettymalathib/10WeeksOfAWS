 We will use the **AWS Console** (GUI), **not CLI**, because first you should understand what you're building.

---

# Today's Mission

Imagine you're building a small city.

By the end of today, your city will look like this:

```text
                    AWS Mumbai Region
                           │
               ┌────────────────────────┐
               │       Your VPC         │
               │   10.10.0.0/16         │
               │                        │
               │  Public-A   Private-A │
AZ-1           │ 10.10.1.0   10.10.11.0│
               │                        │
               │  Public-B   Private-B │
AZ-2           │10.10.2.0   10.10.12.0 │
               └─────────┬──────────────┘
                         │
                 Internet Gateway
                         │
                      Internet
```

Today we're **NOT creating EC2**.

We're only building the roads.

---

# Step 0 - Login

Login to AWS Console.

Change Region to

```
Mumbai
ap-south-1
```

Why?

Because the lab says

> Use Mumbai Region.

---

# Step 1 - Open VPC Service

Search

```
VPC
```

Click it.

You will see something like

```
Virtual Private Cloud

Your VPCs
Subnets
Route Tables
Internet Gateways
...
```

Everything today happens here.

---

# Step 2 - Create VPC

Click

```
Your VPCs
```

Then

```
Create VPC
```

You will see two options.

```
○ VPC and more

○ VPC only
```

Choose

✅ **VPC only**

Why?

Because we want to build everything ourselves.

---

Now fill the details.

### Name

```
cloudadhar-day5-vpc
```

This is just a label.

---

### IPv4 CIDR

```
10.10.0.0/16
```

Remember?

This is your city's land.

---

### IPv6

Choose

```
No IPv6 CIDR block
```

Because today's lab uses only IPv4.

---

### Tenancy

Keep

```
Default
```

No need to change.

---

Click

```
Create VPC
```

🎉 Congratulations!

You own your own AWS network.

---

# Step 3 - Enable DNS

AWS usually enables this automatically, but let's verify.

Click your VPC.

Look for

```
Actions
```

↓

```
Edit VPC Settings
```

Ensure both are checked:

✅ Enable DNS Resolution

✅ Enable DNS Hostnames

Save.

---

## Why?

Imagine opening Google.

Instead of remembering

```
142.250.xxx.xxx
```

you type

```
google.com
```

DNS converts names into IP addresses.

EC2 instances need this.

---

# Step 4 - Create Subnets

Search

```
Subnets
```

Click

```
Create subnet
```

---

Choose your VPC

```
cloudadhar-day5-vpc
```

Now AWS asks

```
How many subnets?
```

We'll make four.

---

# Subnet 1

Name

```
cloudadhar-public-a
```

Availability Zone

Choose the **first AZ**.

Example

```
ap-south-1a
```

(Your account might show a different letter. That's okay.)

CIDR

```
10.10.1.0/24
```

Click

```
Add new subnet
```

---

# Subnet 2

Name

```
cloudadhar-private-a
```

Same AZ

```
First AZ
```

CIDR

```
10.10.11.0/24
```

Click

```
Add new subnet
```

---

# Subnet 3

Name

```
cloudadhar-public-b
```

Choose

Second AZ

Example

```
ap-south-1b
```

CIDR

```
10.10.2.0/24
```

Add another.

---

# Subnet 4

Name

```
cloudadhar-private-b
```

Same second AZ.

CIDR

```
10.10.12.0/24
```

Click

```
Create Subnets
```

Done!

---

# What did we just build?

Our city now has four neighborhoods.

```
VPC

├── Public-A
├── Private-A
├── Public-B
└── Private-B
```

No roads yet.

Only neighborhoods.

---

# Step 5 - Enable Public IP only for Public Subnets

Very important.

Go back to

```
Subnets
```

Click

```
cloudadhar-public-a
```

Choose

```
Actions

↓

Edit subnet settings
```

Find

```
Auto-assign public IPv4
```

Enable it.

Save.

---

Repeat for

```
cloudadhar-public-b
```

Enable.

---

Do NOT enable it for

```
cloudadhar-private-a

cloudadhar-private-b
```

Leave them disabled.

---

## Why?

Imagine houses.

Public houses have an address the courier can reach.

Private houses don't.

Exactly the same idea.

---

# Step 6 - Create Internet Gateway

Search

```
Internet Gateways
```

Click

```
Create Internet Gateway
```

Name

```
cloudadhar-day5-igw
```

Click

```
Create
```

Done.

But...

It isn't connected yet.

Think of buying a gate.

The gate is lying on the ground.

It isn't attached to your compound wall.

---

# Step 7 - Attach IGW

Select your IGW.

Click

```
Actions

↓

Attach to VPC
```

Choose

```
cloudadhar-day5-vpc
```

Attach.

Done.

Now your city has a gate.

---

# Step 8 - Route Tables

Search

```
Route Tables
```

You'll already see one.

Why?

Because every VPC automatically gets one.

This is the

```
Main Route Table
```

---

Select it.

Edit Name.

Rename it

```
cloudadhar-main-rt-local-only
```

Don't touch anything else.

It should contain only

```
10.10.0.0/16

↓

local
```

Leave it.

---

# Step 9 - Create Public Route Table

Click

```
Create Route Table
```

Name

```
cloudadhar-public-rt
```

Choose

```
cloudadhar-day5-vpc
```

Create.

---

Open it.

Go to

```
Routes

↓

Edit Routes

↓

Add Route
```

Destination

```
0.0.0.0/0
```

Target

Choose

```
Internet Gateway

↓

cloudadhar-day5-igw
```

Save.

Excellent!

Now this route table knows how to reach the internet.

---

# Step 10 - Associate Public Subnets

Inside

```
cloudadhar-public-rt
```

Go to

```
Subnet Associations
```

↓

```
Edit
```

Tick

✅ cloudadhar-public-a

✅ cloudadhar-public-b

Save.

Now both public subnets use the internet route.

---

# Step 11 - Create Private Route Table

Create another Route Table.

Name

```
cloudadhar-private-rt
```

Choose

```
cloudadhar-day5-vpc
```

Create.

Do NOT add any routes.

Leave only

```
local
```

---

Associate

```
cloudadhar-private-a

cloudadhar-private-b
```

Done.

---



Your network is complete.

```
                    Internet
                        │
              Internet Gateway
                        │
         cloudadhar-public-rt
        0.0.0.0/0 → IGW
             │           │
      Public-A      Public-B
         │              │

-----------------VPC-------------------

      Private-A     Private-B
            │            │
    cloudadhar-private-rt
          local only

----------------------------

Main Route Table
local only
(Not associated intentionally)
```

---

# Before moving to Day 6, verify these points:

✅ VPC name: `cloudadhar-day5-vpc`
✅ CIDR: `10.10.0.0/16`
✅ 4 subnets created
✅ Public subnets have **Auto-assign Public IPv4 = Enabled**
✅ Private subnets have **Auto-assign Public IPv4 = Disabled**
✅ Internet Gateway is attached to the VPC
✅ Public route table contains `0.0.0.0/0 → Internet Gateway`
✅ Public-A and Public-B are associated with the public route table
✅ Private-A and Private-B are associated with the private route table
✅ Main route table contains only the local route

This VPC will be reused in **Day 6**, so **don't delete it** yet.
