---
title: "Amazon Q CLI Setup"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b>8.1. </b>"
---

{{% notice info %}}
Enable the use of Amazon Bedrock's Foundation Models.

Then, set up Amazon Q CLI for use in VS Code Server.
{{% /notice %}}

## Enable Bedrock Foundation Model Usage

Go to the [Amazon Bedrock Console](https://console.aws.amazon.com/bedrock) page.

Change the Region to **us-east-1** in the upper right corner.

![Goto Amazon Bedrock Console](/images/workshop/go-to-bedrock-console.webp)

Select **Model access** under the Configure and learn menu on the left.

![Change Region](/images/workshop/change-region.webp)

Click the **Enable specific models** button.

![Enable model access](/images/workshop/enable-model-access.png)

Select **Claude 3.7 Sonnet** and **Claude Sonnet 4**, then click the **Next** button. Click the **Submit** button to complete.

![Edit model access](/images/workshop/edit-model-access.png)

You have completed setting up access to Amazon Bedrock's Foundation Models.

## Set Up Amazon Q CLI Usage

Access the Visual Code Server created in the workshop environment.

You can find the URL as shown below on the first page of the workshop studio.

![VS code server URL](/images/workshop/vs-code-server-url.png)

Clicking on this URL will allow you to access the VS Code Server.

In the Terminal, enter the following command to log in to Amazon Q:

```bash
q login
```

Select **Use for Free with Builder ID**.

You will see a message like the one below, and click the URL for login:

```text
✔ Select login method · Use for Free with Builder ID

Confirm the following code in the browser
Code: XXXX-YYYY

Open this URL: https://view.awsapps.com/start/#/device?user_code=XXXX-YYYY
▰▱▱▱▱▱▱ Logging in...
```

Complete the login in your browser.

![Amazon Q confirm](/images/workshop/amazon-q-confirm.png)

![Amazon Q allow access](/images/workshop/amazon-q-allow-access.webp)

![Amazon Q approved](/images/workshop/amazon-q-approved.webp)

Once the login is completed in the browser, you can see the following message in the Terminal:

```text
...
Device authorized
Logged in successfully
```

Run Amazon Q CLI using the following command:

```bash
q chat
```

Amazon Q CLI will start normally with a message like this:

```text
To learn more about MCP safety, see https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-mcp-security.html

╭─────────────────────────────── Did you know? ────────────────────────────────╮
│                                                                              │
│      Set a default model by running q settings chat.defaultModel MODEL.      │
│                          Run /model to learn more.                           │
│                                                                              │
╰──────────────────────────────────────────────────────────────────────────────╯

/help all commands  •  ctrl + j new lines  •  ctrl + s fuzzy search
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🤖 You are chatting with claude-4-sonnet

>
```

The setup for using Amazon Q CLI has been completed.

## AWS CLI Setup

Run the `aws configure` command in the Terminal.

Enter the following:

- **AWS Access Key ID [None]:** -> Press enter without any input.
- **AWS Secret Access Key ID [None]:** -> Press enter without any input.
- **Default region name [None]:** -> Enter the Region value where the current workshop studio was created.
- **Default output format [None]:** -> Press enter without any input.

The environment configuration for AWS CLI has been completed.
