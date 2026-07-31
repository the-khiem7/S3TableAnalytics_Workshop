---
title: "Thiết lập MCP Server"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>8.2. </b>"
---

{{% notice info %}}
Amazon S3 Tables MCP Server được quản lý trong repository GitHub bên dưới.

https://github.com/awslabs/mcp/tree/main/src/s3-tables-mcp-server
{{% /notice %}}

## Thiết lập Amazon S3 Tables MCP Server

Tạo file `mcp.json` như bên dưới để thiết lập các giá trị cấu hình MCP Server.

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

Chạy Amazon Q CLI sử dụng lệnh sau:

```bash
q chat
```

Kiểm tra xem MCP server đã tải đúng chưa bằng lệnh bên dưới:

```text
/tools
```

Xác minh rằng MCP Server đã được khởi tạo như hình bên dưới:

![Tải MCP Server](/images/workshop/mcp-loading.webp)

Amazon S3 Tables MCP Server đã được thêm thành công.

## Thêm Context Amazon S3 Tables

{{% notice info %}}
Để giúp Amazon Q hiểu S3 Tables tốt hơn, chúng ta cung cấp thông tin thông qua file CONTEXT.md.

Thêm context bổ sung cho phép Amazon Q cung cấp phản hồi tốt hơn và gợi ý mã cải thiện.
{{% /notice %}}

Nhập `/quit` để thoát Amazon Q CLI.

Kiểm tra file CONTEXT.md bằng các lệnh sau:

```bash
cd ~
git clone --filter=blob:none --no-checkout https://github.com/awslabs/mcp.git
cd mcp
git sparse-checkout init --cone
git sparse-checkout set src/s3-tables-mcp-server/CONTEXT.md
git checkout
```

Xem file CONTEXT.md bằng lệnh bên dưới:

Bạn có thể thấy nó chứa mô tả tổng quan về Amazon S3 Tables và các hướng dẫn.

```bash
more ~/mcp/src/s3-tables-mcp-server/CONTEXT.md
```

Chạy `q chat` để khởi động lại Amazon Q CLI.

Thêm context bằng lệnh sau:

```text
/context add ~/mcp/src/s3-tables-mcp-server/CONTEXT.md
```

Cài đặt context cho Amazon S3 Tables đã được thêm.

Điều này cho phép Foundation Model hiểu S3 Tables tốt hơn.
