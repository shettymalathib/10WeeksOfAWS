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
<img width="2557" height="507" alt="image" src="https://github.com/user-attachments/assets/2bbc7dfd-bb49-47a9-a7c5-4e848a2ef51d" />


### IAM Groups

```
S3ReadOnlyGroup
EC2ReadOnlyGroup
BillingViewGroup
```

<img width="2547" height="576" alt="image" src="https://github.com/user-attachments/assets/11ce8e16-1362-4f84-bac4-2904206cee10" />


### Customer Managed Policy

```
CustomS3ReadOnlyTrainingPolicy
```

<img width="2541" height="432" alt="image" src="https://github.com/user-attachments/assets/b69db290-14a0-4da7-95d5-620eb01a82e0" />


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

<img width="2552" height="532" alt="image" src="https://github.com/user-attachments/assets/6481ed43-d6ed-4d40-8592-2937a1f5bb30" />


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

<img width="2542" height="547" alt="image" src="https://github.com/user-attachments/assets/7b9ea4d9-eaad-413b-b0ab-e8872ec348d8" />


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

<img width="2527" height="486" alt="image" src="https://github.com/user-attachments/assets/f9a04256-211f-4647-9275-ba3a60dbfb05" />


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

