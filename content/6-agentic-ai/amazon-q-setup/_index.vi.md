---
title: "Thiết lập Amazon Q CLI"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b>8.1. </b>"
---

{{% notice info %}}
Kích hoạt sử dụng Foundation Models của Amazon Bedrock.

Sau đó, thiết lập Amazon Q CLI để sử dụng trong VS Code Server.
{{% /notice %}}

## Kích hoạt Sử dụng Bedrock Foundation Model

Truy cập trang [Amazon Bedrock Console](https://console.aws.amazon.com/bedrock).

Thay đổi Region thành **us-east-1** ở góc trên bên phải.

![Truy cập Amazon Bedrock Console](/images/workshop/go-to-bedrock-console.webp)

Chọn **Model access** trong menu Configure and learn bên trái.

![Thay đổi Region](/images/workshop/change-region.webp)

Nhấp nút **Enable specific models**.

![Kích hoạt truy cập model](/images/workshop/enable-model-access.png)

Chọn **Claude 3.7 Sonnet** và **Claude Sonnet 4**, sau đó nhấp nút **Next**. Nhấp nút **Submit** để hoàn tất.

![Chỉnh sửa truy cập model](/images/workshop/edit-model-access.png)

Bạn đã hoàn tất việc thiết lập truy cập đến Foundation Models của Amazon Bedrock.

## Thiết lập Sử dụng Amazon Q CLI

Truy cập Visual Code Server đã tạo trong môi trường workshop.

Bạn có thể tìm URL như hình bên dưới trên trang đầu tiên của workshop studio.

![URL VS code server](/images/workshop/vs-code-server-url.png)

Nhấp vào URL này sẽ cho phép bạn truy cập VS Code Server.

Trong Terminal, nhập lệnh sau để đăng nhập vào Amazon Q:

```bash
q login
```

Chọn **Use for Free with Builder ID**.

Bạn sẽ thấy thông báo như bên dưới, và nhấp URL để đăng nhập:

```text
✔ Select login method · Use for Free with Builder ID

Confirm the following code in the browser
Code: XXXX-YYYY

Open this URL: https://view.awsapps.com/start/#/device?user_code=XXXX-YYYY
▰▱▱▱▱▱▱ Logging in...
```

Hoàn tất đăng nhập trong trình duyệt.

![Xác nhận Amazon Q](/images/workshop/amazon-q-confirm.png)

![Cho phép truy cập Amazon Q](/images/workshop/amazon-q-allow-access.webp)

![Amazon Q đã phê duyệt](/images/workshop/amazon-q-approved.webp)

Khi đăng nhập hoàn tất trong trình duyệt, bạn sẽ thấy thông báo sau trong Terminal:

```text
...
Device authorized
Logged in successfully
```

Chạy Amazon Q CLI sử dụng lệnh sau:

```bash
q chat
```

Amazon Q CLI sẽ khởi động bình thường với thông báo như sau:

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

Việc thiết lập sử dụng Amazon Q CLI đã hoàn tất.

## Thiết lập AWS CLI

Chạy lệnh `aws configure` trong Terminal.

Nhập các thông tin sau:

- **AWS Access Key ID [None]:** -> Nhấn enter mà không nhập gì.
- **AWS Secret Access Key ID [None]:** -> Nhấn enter mà không nhập gì.
- **Default region name [None]:** -> Nhập giá trị Region nơi workshop studio hiện tại được tạo.
- **Default output format [None]:** -> Nhấn enter mà không nhập gì.

Việc cấu hình môi trường cho AWS CLI đã hoàn tất.
