---
title: "Thiết lập EMR Workspace"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b>3.3. </b>"
---

{{% notice info %}}
Phần này hướng dẫn thiết lập EMR Workspace sẽ được sử dụng để xử lý dữ liệu trong S3 Tables trong suốt workshop.
{{% /notice %}}

1. Điều hướng đến trang [EMR Console](https://console.aws.amazon.com/emr/home).
   - ![Bảng điều khiển EMR](/images/workshop/emr_console.webp)

2. Chọn 'EMR Serverless' từ menu bên trái.

3. Trong phần 'EMR Studio' ở bên phải, chọn 'workshop_emr_studio' và nhấp nút 'Manage applications'.

4. Nhấp nút này sẽ mở trang EMR Studio trong một cửa sổ mới.
   - Bạn sẽ thấy một EMR Serverless Application được tạo với tên 'workshop_emr_application'.
   - ![Ứng dụng EMR serverless](/images/workshop/emr_application.png)

5. Nhấp vào 'Workspaces' trong menu bên trái của EMR Studio.

6. Nhấp nút 'Create Workspace'.

7. Nhập 'workshop-workspace' cho 'Workshop name', sau đó nhấp nút 'Create Workspace'.
   - ![Tạo workspace](/images/workshop/create_workspace.png)

8. Sau khi tạo, một cửa sổ mới với trang JupyterLab sẽ mở. (Cho phép trình duyệt của bạn mở cửa sổ mới.)
   - ![JupyterLab](/images/workshop/jupyterlab_notebook.webp)

9. Nhấp biểu tượng 'EMR Compute' ở phía bên trái của JupyterLab để gắn EMR Serverless Application vào JupyterLab.
   - ![Gắn ứng dụng EMR](/images/workshop/attach_emr_application.png)
   - EMR Serverless application: Chọn `workshop_emr_application`
   - Interactive runtime role: Chọn `Workshop_Job_Role`

{{% notice warning %}}
Việc thiết lập EMR workspace đã hoàn tất.
{{% /notice %}}
