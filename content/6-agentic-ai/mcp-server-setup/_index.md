---
title: "MCP Server Setup"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>8.2. </b>"
---

{{% notice info %}}
The Amazon S3 Tables MCP Server is managed in the GitHub repository below.

https://github.com/awslabs/mcp/tree/main/src/s3-tables-mcp-server
{{% /notice %}}

## Amazon S3 Tables MCP Server Setup

Create an `mcp.json` file as shown below to set up the MCP Server configuration values.

```bash
vi ~/.aws/amazonq/mcp.json
```

```json
{
  "mcpServers": {
    "awslabs.s3-tables-mcp-server": {
      "command": "uvx",
      "args": [
        "awslabs.s3-tables-mcp-server@latest",
        "--allow-write"
      ],
      "env": {
        "AWS_PROFILE": "default",
        "AWS_REGION": "us-east-1"
      }
    }
  }
}
```

Run the Amazon Q CLI using the following command:

```bash
q chat
```

Check if the MCP server has loaded properly using the command below:

```text
/tools
```

Verify that the MCP Server has been initialized as shown below:

![MCP Server Loading](/images/workshop/mcp-loading.webp)

The Amazon S3 Tables MCP Server has been successfully added.

## Adding Amazon S3 Tables Context

{{% notice info %}}
To help Amazon Q understand S3 Tables better, we provide information through a CONTEXT.md file.

Adding additional context allows Amazon Q to provide better responses and improved code suggestions.
{{% /notice %}}

Enter `/quit` to exit the Amazon Q CLI.

Check out the CONTEXT.md file using the following commands:

```bash
cd ~
git clone --filter=blob:none --no-checkout https://github.com/awslabs/mcp.git
cd mcp
git sparse-checkout init --cone
git sparse-checkout set src/s3-tables-mcp-server/CONTEXT.md
git checkout
```

Review the CONTEXT.md file using the command below:

You can see that it contains an overall description of Amazon S3 Tables and instructions.

```bash
more ~/mcp/src/s3-tables-mcp-server/CONTEXT.md
```

Run `q chat` to start the Amazon Q CLI again.

Add the context using the following command:

```text
/context add ~/mcp/src/s3-tables-mcp-server/CONTEXT.md
```

The context settings for Amazon S3 Tables have been added.

This allows the Foundation Model to better understand S3 Tables.
