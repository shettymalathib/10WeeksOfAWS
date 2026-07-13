
You're only doing two things:

1. Secure your account (MFA)
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

Take a screenshot.

**Do not include the QR code**—only the confirmation that MFA is enabled.

---

## Step 6: Write a Short Note

Example:

> The root user has full access to the AWS account. It should only be used for account-level tasks such as enabling MFA or managing billing. For everyday work, IAM users or roles are recommended.

---

# Lab 2 – Create a Budget Alert

The goal is simple:

If your AWS spending reaches a limit (for example, **$5**), AWS sends you an email.

This helps prevent surprise bills.

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

---

## Step 5: Configure Email Alerts

Enter your email address.

Set a notification threshold, for example:

* Alert at **80%** of your budget.

AWS will send an email when your spending reaches that amount.

Click **Create Budget**.

---

## Step 6: Confirm

You should see your budget listed.

Take a screenshot.

---



> A billing budget helps monitor AWS spending. It sends an email alert when costs approach the configured limit, helping avoid unexpected charges.

---



### Why should we enable MFA on the root user?

The root user has full access to the AWS account. Enabling MFA adds an extra layer of security and helps protect the account even if the password is compromised.

### Why create a billing budget?

A billing budget helps monitor AWS costs and sends alerts before spending exceeds the configured budget.


