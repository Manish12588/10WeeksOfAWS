# Week 2 Cleanup

Clean up after collecting your proof so the practice environment stays secure
and does not create unnecessary cost.

## Cleanup Order

1. Terminate the test EC2 instance.
   ![](./submissions/Manish-Kumar/screenshots/EC2_Terminated.png)

2. Confirm that any attached EBS volume set to persist was deleted if no longer
   needed.
   ![](./submissions/Manish-Kumar/screenshots/EBS_deleted.png)

3. Delete the objects from the test S3 bucket.
   ![](./submissions/Manish-Kumar/screenshots/Deleted_object_from_s3_bucket.png)

4. Delete the empty test bucket.
   ![](./submissions/Manish-Kumar/screenshots/Deleted_s3_bucket.png)

5. Delete the inline policy or delete the test IAM role.
   ![](./submissions/Manish-Kumar/screenshots/Deleted_Policy.png)

6. Confirm that the related instance profile was removed with the role. If you
   created it manually, remove it separately.
   ![](./submissions/Manish-Kumar/screenshots/Deleted_Role.png)

## Final Check

- No test EC2 instance is running.
- No test EBS volume or Elastic IP remains.
- The test bucket and objects are deleted.
- The training role and instance profile are deleted if not needed again.
- No access keys were created during the lab.

Add a cleanup note or safe screenshot to your submission.

## Day 4 - Organizations and SCP Cleanup

1. Confirm the successful pre-SCP test bucket was deleted.
2. The denied post-SCP command should not have created a bucket. Verify before
   continuing.
3. Move `CloudAdhar-Dev` back under the organization Root if you need to repeat
   the practical.
4. Keep `Dev-Env` and `Deny-S3-Bucket-Creation` for another practice unless the
   trainer asks you to remove them.
5. Sign out of the AWS access portal and close the incognito session.

Do not close or remove the member account as cleanup. The correct reversible
operation for this lab is **Move AWS account**.
