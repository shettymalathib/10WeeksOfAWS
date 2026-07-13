
Here we're only doing two things:

1. Secure your root account by enabling Multi-Factor Authentication (MFA).
2. Protect yourself from unexpected charges (Budget Alert)

---

# Lab 1 – Secure Root User

## Step 1: Sign in

Go to:

> [https://console.aws.amazon.com/](https://console.aws.amazon.com/)

Sign in using your **Root User**.

You'll enter:

* Email address
* Password

---

## Step 2: Open Security Settings

There are two ways to do this.

### Option 1 (Recommended)

* Click your account name (top-right).
* Select **Security credentials**.
* If prompted, choose **Go to Security Credentials**.

This is where you manage the **Root User's** security settings.

---

## Step 3: Enable MFA

Find the **Multi-factor authentication (MFA)** section.



Choose one of the supported MFA methods:

- Passkey (Recommended if supported on your device)
- Authenticator app (Google Authenticator, Microsoft Authenticator, Authy, etc.)
- Hardware security key (Advanced)

Any of these methods satisfies the lab requirement.

Click:

> **Assign MFA device**

AWS will ask you which type of MFA you want.

For beginners, use an authenticator app such as:

* Google Authenticator
* Microsoft Authenticator
* Authy

Install one on your phone if you don't already have one.

---

## Step 4: Scan the QR Code

AWS displays a QR code.

Open your authenticator app and:

1. Tap **Add Account**
2. Scan the QR code
3. The app will generate a 6-digit code.

AWS will ask for **two consecutive codes**.

Enter them and finish the setup.

---

## Step 5: Verify

You should now see something like:

```
Multi-factor authentication

Status

Enabled ✅
```

<img width="1920" height="1294" alt="image" src="https://github.com/user-attachments/assets/e08843a7-4a0b-45fd-bb63-33006b8f67d3" />

<img width="1902" height="800" alt="image" src="https://github.com/user-attachments/assets/ea520d9f-baf0-4098-9da7-3daee049ab52" />




**Do not include the QR code**—only the confirmation that MFA is enabled.

---



> The root user has full access to the AWS account. It should only be used for account-level tasks such as enabling MFA or managing billing. For everyday work, IAM users or roles are recommended.

> I secured my AWS root user by enabling a passkey. This adds a strong second factor of authentication and helps protect the AWS account from unauthorized access.

---

# Lab 2 – Create a Budget Alert

The goal is simple:

If your AWS spending reaches a limit (for example, **$5**), AWS sends you an email.

A billing budget does not stop AWS from charging you. It only monitors your costs and sends email alerts when your spending reaches the configured threshold.

---

## Step 1: Open Billing

In the AWS Console:

* Click your account name (top-right).
* Choose **Billing and Cost Management**.

---

## Step 2: Open Budgets

In the left menu:

```
Budgets
```

Click:

> **Create budget**


<img width="1892" height="800" alt="image" src="https://github.com/user-attachments/assets/d45bc344-6a7f-4fb0-9bcb-2433d96ab9f4" />


---



## Step 3: Choose Budget Type

Select:

```
Cost Budget
```

Then click **Next**.

---

## Step 4: Set Your Budget

Example:

* Budget Name: `Week1-Budget`
* Budget Amount: `$5`
* Time Period: Monthly

Click **Next**.

<img width="2560" height="2304" alt="image" src="https://github.com/user-attachments/assets/a1a7b306-c3db-47b5-b140-3aca19c32ae2" />


---

## Step 5: Configure Email Alerts

Enter your email address.

Set a notification threshold, for example:

* Alert at **80%** of your budget.

AWS will send an email when your spending reaches that amount.

Click **Create Budget**.


<img width="2557" height="922" alt="image" src="https://github.com/user-attachments/assets/e795ddc4-7ecb-4b3d-a123-29860094deb4" />
<img width="1722" height="1080" alt="image" src="https://github.com/user-attachments/assets/f5cd3426-9df2-4905-8f97-845f273f2dc0" />
<img width="2510" height="887" alt="image" src="https://github.com/user-attachments/assets/bf47b8b8-71ae-4b34-9bff-186a274ae052" />


---

## Step 6: Confirm

You should see your budget listed.

<img width="2557" height="785" alt="image" src="https://github.com/user-attachments/assets/c918f7aa-d6d5-4314-b61f-c48ef9189213" />


---



> A billing budget helps monitor AWS spending. It sends an email alert when costs approach the configured limit, helping avoid unexpected charges.

---



### Why should we enable MFA on the root user?

The root user has full access to the AWS account. Enabling MFA adds an extra layer of security and helps protect the account even if the password is compromised.

### Why create a billing budget?

A billing budget helps monitor AWS costs and sends alerts before spending exceeds the configured budget.


