---
title: "Workshop tự thực hành"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>2.2. </b>"
---

## Tạo tài khoản AWS

{{% notice warning %}}
Nếu bạn đã có tài khoản AWS, bạn có thể tiến hành trực tiếp với hướng dẫn workshop này. Tuy nhiên, nếu bạn chưa có tài khoản, bạn cần tạo tài khoản AWS trước.
{{% /notice %}}

Nếu bạn chưa có tài khoản AWS với quyền quản trị viên, nhấp vào [đây](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) để tạo tài khoản.

## Người dùng IAM

Nếu bạn đã tạo hoặc đã có tài khoản AWS, hãy tạo một người dùng IAM có thể truy cập tài khoản AWS của bạn. Sau khi đăng nhập vào tài khoản, bạn có thể sử dụng bảng điều khiển IAM để tạo người dùng IAM. Thực hiện theo các bước dưới đây để tạo người dùng có quyền Administrator.

{{% notice info %}}
Nếu bạn đã có người dùng IAM với quyền quản trị viên, hãy bỏ qua các bước sau.
{{% /notice %}}

1. Trên trang đăng nhập, đăng nhập vào bảng điều khiển IAM với tư cách người dùng root của tài khoản AWS bằng địa chỉ email và mật khẩu tài khoản AWS của bạn.

2. Trong bảng điều khiển IAM, nhấp vào Users ở thanh bên trái, sau đó nhấp vào nút Add user.

3. Nhập Administrator cho User name.

4. Trong phần Access type, đánh dấu vào ô AWS Management Console access, chọn Custom password, sau đó nhập mật khẩu.

5. Nhấp Next: Permissions.

6. Chọn Attach existing policies directly, đánh dấu vào ô cho chính sách AdministratorAccess, sau đó nhấp Next: Tags.

![Tạo người dùng IAM](/images/workshop/iam_user_create.webp)

7. Ở bước Add tags (optional), nhấp Next: Review.

8. Xác minh rằng chính sách quản lý AdministratorAccess đã được thêm vào người dùng Administrator, và nhấp Create user.

![Thiết lập quyền](/images/workshop/iam_permission.webp)

9. Sau khi người dùng được thêm, sao chép URL đăng nhập. URL sẽ có định dạng sau: `https://<your_aws_account_id>.signin.aws.amazon.com/console`

![Sao chép thông tin đăng nhập](/images/workshop/copy_signin_details.webp)

Bây giờ hãy đăng xuất khỏi người dùng root, truy cập URL bạn vừa sao chép, và đăng nhập với tư cách người dùng Administrator mới tạo.
