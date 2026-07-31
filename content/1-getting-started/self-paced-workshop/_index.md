---
title: "Self-Paced Workshop"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>2.2. </b>"
---

## Create AWS Account

{{% notice warning %}}
If you already have an AWS account, you can proceed directly with this workshop guide. However, if you don't have an account, you need to create an AWS account first.
{{% /notice %}}

If you don't have an AWS account with administrator privileges yet, click [here](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) to create an account.

## IAM User

If you have created or already have an AWS account, create an IAM user that can access your AWS account. After logging into your account, you can use the IAM console to create an IAM user. Follow the steps below to create a user with Administrator privileges.

{{% notice info %}}
If you already have an IAM user with administrator privileges, skip the following steps.
{{% /notice %}}

1. On the login page, sign in to the IAM console as the root user of your AWS account using your AWS account email address and password.

2. In the IAM console, click Users in the left sidebar, then click the Add user button.

3. Enter Administrator for the User name.

4. Under Access type, select the checkbox for AWS Management Console access, choose Custom password, and then enter a password.

5. Click Next: Permissions.

6. Select Attach existing policies directly, check the box for the AdministratorAccess policy, and then click Next: Tags.

![IAM user create](/images/workshop/iam_user_create.webp)

7. On the Add tags (optional) step, click Next: Review.

8. Verify that the AdministratorAccess managed policy has been added to the Administrator user, and click Create user.

![Setup permission](/images/workshop/iam_permission.webp)

9. Once the user is added, copy the login URL. The URL will have the following format: `https://<your_aws_account_id>.signin.aws.amazon.com/console`

![Copy signin details](/images/workshop/copy_signin_details.webp)

Now log out of the root user, access the URL you just copied, and log in as the newly created Administrator user.
