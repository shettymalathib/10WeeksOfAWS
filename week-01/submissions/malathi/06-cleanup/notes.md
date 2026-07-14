# Week 1 Cleanup

> **Important:** Take all required screenshots before deleting any resources. Once deleted, they cannot be recovered for your submission.

---

## Resources to Delete

Delete these resources if they were created only for Week 1 practice:

### IAM Users

```
learner-s3
learner-ec2
learner-billing
```

### IAM Groups

```
S3ReadOnlyGroup
EC2ReadOnlyGroup
BillingViewGroup
```

### Customer Managed Policy

```
CustomS3ReadOnlyTrainingPolicy
```

---

# Step 1 – Delete IAM Users

Go to:

```text
AWS Console
→ IAM
→ Users
```

For each practice user:

1. Select the user.
2. Click **Delete**.
3. Confirm the deletion.

Delete:

* `learner-s3`
* `learner-ec2`
* `learner-billing`

---

# Step 2 – Delete IAM Groups

Go to:

```text
IAM
→ User groups
```

Delete:

* `S3ReadOnlyGroup`
* `EC2ReadOnlyGroup`
* `BillingViewGroup`

For each group:

1. Open the group.
2. Verify it contains **no users**.
3. Click **Delete**.
4. Confirm.

> If AWS reports that the group still contains users, delete or remove the users first.

---

# Step 3 – Delete the Custom Policy

Go to:

```text
IAM
→ Policies
```

Search for:

```text
CustomS3ReadOnlyTrainingPolicy
```

Then:

1. Select the policy.
2. Click **Actions** → **Delete**.
3. Confirm.

> If the **Delete** option is unavailable, the policy is still attached to a user, group, or role. Detach it first, then delete the policy.

---

# Final Verification

Verify that the following resources no longer exist:

### IAM → Users

```
❌ learner-s3
❌ learner-ec2
❌ learner-billing
```

### IAM → User groups

```
❌ S3ReadOnlyGroup
❌ EC2ReadOnlyGroup
❌ BillingViewGroup
```

### IAM → Policies

```
❌ CustomS3ReadOnlyTrainingPolicy
```

---

# Keep These Resources

Do **not** delete the following:

* ✅ Root MFA
* ✅ Billing Budget / Budget Alerts
* ✅ Your main IAM admin user (if created for daily use)

### Optional

The following resources do not incur charges and can be kept for future GitHub Actions and CI/CD practice:

* ✅ `github-oidc-challenge-role`
* ✅ `token.actions.githubusercontent.com` (OIDC Identity Provider)

---

## Summary

| Resource                       | Action                           |
| ------------------------------ | -------------------------------- |
| Root MFA                       | ✅ Keep                           |
| Billing Budget                 | ✅ Keep                           |
| Main IAM Admin User            | ✅ Keep                           |
| learner-s3                     | ❌ Delete                         |
| learner-ec2                    | ❌ Delete                         |
| learner-billing                | ❌ Delete                         |
| S3ReadOnlyGroup                | ❌ Delete                         |
| EC2ReadOnlyGroup               | ❌ Delete                         |
| BillingViewGroup               | ❌ Delete                         |
| CustomS3ReadOnlyTrainingPolicy | ❌ Delete                         |
| github-oidc-challenge-role     | ✅ Optional (Recommended to Keep) |
| OIDC Identity Provider         | ✅ Optional (Recommended to Keep) |

