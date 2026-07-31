---
title: "Thiết lập Môi trường"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>6.2. </b>"
---

## 1. Tải lên file CSV và Apache Iceberg jar vào S3

Chúng ta sẽ tải lên file Apache Iceberg Runtime Jar và file Glow Jar cần thiết cho workshop này lên S3 bucket.

1. Tải xuống file Glow jar đã build, `glow-spark3-assembly-2.0.0.jar`.

2. Tải xuống file Apache Iceberg runtime jar cho Amazon S3 Tables, `s3-tables-catalog-for-iceberg-runtime-0.1.5.jar`.

3. Truy cập trang [S3 Console](https://console.aws.amazon.com/s3/).

4. Chọn 'General purpose buckets' từ menu bên trái.

5. Nhấp vào bucket 'workshop-bucket-xxxxxx'.

6. Nhấp 'Create folder' và tạo thư mục có tên 'lib'.

7. Nhấp 'Upload', sau đó sử dụng nút 'Add files' để tải lên các file đã tải xuống vào đường dẫn tương ứng.

{{% notice info %}}
Tải lên hai file Jar (s3-tables-catalog-for-iceberg-runtime-0.1.5.jar & glow-spark3-assembly-2.0.0.jar) vào đường dẫn 's3://${bucket}/lib/'.
{{% /notice %}}

![Tải lên file jar](/images/workshop/upload-jar.png)

Bạn đã hoàn thành việc tải lên các file cần thiết cho workshop lên S3.

## 2. Đóng gói Python 3.10.12 và các gói pypi

Phần này hướng dẫn đóng gói Python phiên bản 3.10.12 với gói Glow pypi cần thiết cho xử lý dữ liệu định dạng VCF thành file gz để sử dụng làm môi trường ảo Python trong EMR Serverless.

Nếu bạn không có kế hoạch xử lý dữ liệu định dạng VCF, bạn có thể bỏ qua phần này.

https://docs.aws.amazon.com/emr/latest/EMR-Serverless-UserGuide/using-python.html

Glow phiên bản 2.0 yêu cầu Python phiên bản 3.10.12.

EMR cung cấp các tùy chọn để sử dụng môi trường Python tùy chỉnh ngoài môi trường Python đã cài đặt trên EMR.

Phần này hướng dẫn đóng gói Python phiên bản 3.10.12 và gói glow.py thành định dạng file gz để sử dụng trong EMR Serverless.

1. Điều hướng đến trang [EC2 Console](https://console.aws.amazon.com/ec2/) (EC2 > Instances).

2. Nhấp vào Instance ID của EC2 instance đã tạo.

3. Nhấp nút 'Connect'.

![Session Manager](/images/workshop/ec2-connect.webp)

4. Chọn 'Session Manager' và nhấp 'Connect' để bắt đầu phiên SSH.

![Session Manager](/images/workshop/ec2-ssm.webp)

5. Chuyển sang user root bằng `sudo su`.

![Lệnh sudo su](/images/workshop/ec2-sudo.webp)

Việc đóng gói môi trường ảo Python sử dụng script shell sau thường mất từ 5 đến 10 phút.

Sau khi chuyển sang user root bằng `sudo su`, chạy script shell sau:

Thay thế `${bucket}` bằng tên S3 bucket đã đề cập ở trên.

```bash
# install Python 3.10.12 and activate the venv
yum install -y gcc openssl-devel bzip2-devel libffi-devel tar gzip wget make zlib-devel
wget https://www.python.org/ftp/python/3.10.12/Python-3.10.12.tgz && \
tar xzf Python-3.10.12.tgz && cd Python-3.10.12 && \
./configure --enable-optimizations && \
make altinstall

# create python venv with Python 3.10.12
python3.10 -m venv pyspark_venv_python_3.10.12 --copies
source pyspark_venv_python_3.10.12/bin/activate

# copy system python3 libraries to venv
cp -r -u /usr/local/lib/python3.10/* ./pyspark_venv_python_3.10.12/lib/python3.10/

# package venv to archive.
# **Note** that you have to supply --python-prefix option
# to make sure python starts with the path where your
# copied libraries are present.
# Copying the python binary to the "environment" directory.
pip3 install venv-pack glow.py matplotlib pandas
venv-pack -f -o pyspark_venv_python_3.10.12.tar.gz --python-prefix /home/hadoop/environment

# stage the archive in S3
#s3://workshop-bucket-125397608849/lib/
aws s3 cp pyspark_venv_python_3.10.12.tar.gz s3://workshop-bucket-<Account-ID>/lib/

# optionally, remove the virtual environment directory
# rm -fr pyspark_venv_python_3.10.12
```

Script shell này cuối cùng sẽ tải lên file `pyspark_venv_python_3.10.12.tar.gz` lên S3 bucket.

File gz này bao gồm Python 3.10.12 với các gói glow.py, matplotlib và pandas đã cài đặt.

File này sẽ được sử dụng làm giá trị cấu hình cho Spark trong EMR Serverless trong tương lai.

Bạn đã hoàn thành việc tải lên các file cần thiết cho workshop lên S3.
