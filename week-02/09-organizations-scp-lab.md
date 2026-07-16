# Day 4 Practical - Organizations, Identity Center, and SCPs

Goal: Prove that S3 bucket creation succeeds before an OU guardrail applies and
fails after the member account is moved into the OU.

## Prerequisites

- A dedicated learning AWS Organization with all features enabled
- Management account: `mkumardevops-Management`
- Member account: `mkumardevops-Dev`
- Empty OU: `Dev-Env`
- `Deny-S3-Bucket-Creation` SCP attached to `Dev-Env`
- IAM Identity Center organization instance enabled in `eu-central-1`
- Activated `manish-demo` user with MFA
- `CloudAdhar-Admin` permission set assigned to both accounts
- `CloudAdhar-Dev` directly under Root before testing
- No S3 test bucket remaining from an earlier run

Use a dedicated training organization. Do not perform this practical in a
production organization.

`CloudAdhar-Admin` uses broad access only to make the SCP demonstration clear.
Use groups and least-privilege permission sets in production.

## Part 1 - Review the Organization

1. Sign in to `mkumardevops` management account or root account.
2. Open AWS Organizations -> AWS accounts.
3. Identify Root, the management account, and the member account.
   ![](./submissions/Manish-Kumar/screenshots/01_Management_account_and_member_account.png)
4. Confirm `Dev-Env` is empty.
   ![](./submissions/Manish-Kumar/screenshots/02_Empty_organisation_unit.png)
5. Open the OU policies and confirm `FullAWSAccess` and
   `Deny-S3-Bucket-Creation` are applied.
   ![](./submissions/Manish-Kumar/screenshots/04_Attached_deny_s3_bucket_creation_policy_to_dev_env_ou.png)
6. Confirm `dev-account` is directly under Root, so the OU deny does not yet
   apply.
   ![](./submissions/Manish-Kumar/screenshots/05_dev-account_under_root.png)

Expected result: Root contains the management and Dev accounts; `Dev-Env` is
empty and has the custom SCP.

## Part 2 - Verify Identity Center Temporary Access

1. Open the AWS access portal in an incognito window.
2. Sign in as `manish-demo` and complete MFA.
3. Confirm `mkumardevops-Management` and `dev-account` are visible.
   ![](./submissions/Manish-Kumar/screenshots/P2_01_Both_Account_Visible.png)
4. Open `dev-account` with the `AministrationAccess` permission set.
5. Select Mumbai (`eu-central-1`) and open CloudShell.
6. Run:

```bash
aws sts get-caller-identity
```

![](./submissions/Manish-Kumar/screenshots/P2_02_get_caller_identity.png)

Expected result: the account is the Dev account and the ARN contains
`assumed-role/AWSReservedSSO_CloudAdhar-Admin` plus the user session name.

Always run `get-caller-identity` before making changes in a multi-account
environment. Confirming the account and role first prevents expensive mistakes.

Mask the account ID and full ARN in screenshots.

## Part 3 - Test Before the SCP Applies

```bash
export AWS_REGION=eu-central-1
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET="manish-before-scp-${ACCOUNT_ID}-$(date +%s)"
echo "$BUCKET"

aws s3api create-bucket \
  --bucket "$BUCKET" \
  --region "$AWS_REGION" \
  --create-bucket-configuration LocationConstraint="$AWS_REGION"

aws s3api head-bucket --bucket "$BUCKET"
```

![](./submissions/Manish-Kumar/screenshots/P3_01_Bucket_Creation_CLI.png)

![](./submissions/Manish-Kumar/screenshots/P3_02_Bucket_Existance_Console.png)

Expected result: the bucket is created. The permission set allows the action,
and the account is not yet inside the OU where the deny applies.

Delete the successful test bucket:

```bash
aws s3api delete-bucket --bucket "$BUCKET" --region "$AWS_REGION"
aws s3api head-bucket --bucket "$BUCKET"
```

Expected result: `HeadBucket` returns `404 Not Found` after deletion.

![](./submissions/Manish-Kumar/screenshots/P3_03_Delete_bucket.png)

## Part 4 - Move the Account into Dev-Env

1. Return to `mkumardevops-Management`.
2. Open AWS Organizations -> AWS accounts.
3. Select only `dev-account`.
4. Choose **Actions -> AWS account -> Move**.
5. Select `Dev-Env` and confirm the move.
6. Expand the OU and confirm the member account appears inside it.
   ![](./submissions/Manish-Kumar/screenshots/P4_01_Move_member_account_to_OU.png)

   Safety: choose **Move**, never **Close** or **Remove from organization**.

## Part 5 - Test After the SCP Applies

Return to the Dev account CloudShell session:

```bash
aws sts get-caller-identity
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET="manish-after-scp-${ACCOUNT_ID}-$(date +%s)"
echo "$BUCKET"

aws s3api create-bucket \
  --bucket "$BUCKET" \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1
```

Expected result: `AccessDenied` with an explicit deny in a service control
policy.

![](./submissions/Manish-Kumar/screenshots/P5_01_AccessDenied_To_Create_S3_Bucket.png)

Compare this identity result with the result from Part 2. The account, user
session, and Identity Center role are unchanged. The only change is that the
member account now inherits the SCP from `Dev-Env`.

```text
Identity Center permission set: Allow
Dev-Env SCP: Explicit deny
Final result: Deny
```

## Part 6 - Review Consolidated Billing

1. Return to `CloudAdhar-Management`.
2. Open Billing and Cost Management -> Bills.
3. Review the organization billing structure.
4. Open Cost Explorer and choose **Group by -> Linked account**.
5. Note that new account data may take time to appear.

![](./submissions/Manish-Kumar/screenshots/P6_01_Bills.png)

## Troubleshooting

- Bucket fails before the move: confirm the member account is under Root, the
  bucket name is unique, and the Mumbai `LocationConstraint` is present.
- Bucket succeeds after the move: confirm the SCP is attached to `Dev-Env` and
  the account is actually inside the OU.
- Portal shows no accounts: verify user, account, permission-set assignment,
  and provisioning status.
- Do not test the SCP in the management account; SCPs do not restrict its users
  or roles.
- Keep `FullAWSAccess` unless a tested allow-list strategy replaces it.
- `169.254.169.254` is the EC2 instance metadata endpoint. This lab does not
  query EC2 metadata; `get-caller-identity` proves the current STS session.

Complete [03-cleanup.md](./03-cleanup.md) and use the Day 4 structure in
[04-submission-format.md](./04-submission-format.md).
