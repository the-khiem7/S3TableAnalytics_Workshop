---
title: "Environment Setup"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>6.2. </b>"
---

## 1. Upload CSV file and Apache Iceberg jar in S3

We will upload Apache Iceberg Runtime Jar file, and Glow Jar file required for this workshop to the S3 bucket.

1. Download the built Glow jar file, `glow-spark3-assembly-2.0.0.jar`.

2. Download the Apache Iceberg runtime jar file for Amazon S3 Tables, `s3-tables-catalog-for-iceberg-runtime-0.1.5.jar`.

3. Access the [S3 Console](https://console.aws.amazon.com/s3/) page.

4. Select 'General purpose buckets' from the left menu.

5. Click on the 'workshop-bucket-xxxxxx' bucket.

6. Click 'Create folder' and create a folder named 'lib'.

7. Click 'Upload', then use the 'Add files' button to upload the downloaded files to their respective paths.

{{% notice info %}}
Upload two Jar files (s3-tables-catalog-for-iceberg-runtime-0.1.5.jar & glow-spark3-assembly-2.0.0.jar) to the 's3://${bucket}/lib/' path.
{{% /notice %}}

![Upload jar file](/images/workshop/upload-jar.webp)

You have completed uploading the files necessary for the workshop to S3.

## 2. Packaging Python 3.10.12 and pypi packages

This section covers packaging Python 3.10.12 version with the Glow pypi package required for VCF format data processing into a gz file for use as a Python virtual environment in EMR Serverless.

If you do not plan to process VCF format data, you can skip this part.

https://docs.aws.amazon.com/emr/latest/EMR-Serverless-UserGuide/using-python.html

Glow 2.0 version requires Python 3.10.12 version.

EMR provides options to use custom Python environments in addition to the Python environment installed on EMR.

This section covers packaging Python 3.10.12 version and the glow.py package into a gz file format for use in EMR Serverless.

1. Navigate to the [EC2 Console](https://console.aws.amazon.com/ec2/) page (EC2 > Instances).

2. Click on the Instance ID of the created EC2 instance.

3. Click the 'Connect' button.

![Session Manager](/images/workshop/ec2-connect.webp)

4. Select 'Session Manager' and click 'Connect' to start SSH session.

![Session Manager](/images/workshop/ec2-ssm.webp)

5. Change the user to root with `sudo su`.

![sudo su CLI](/images/workshop/ec2-sudo.webp)

Packaging a Python virtual environment using the following shell script usually takes 5 to 10 minutes.

After changing to root user with `sudo su`, run the following shell script:

Replace `${bucket}` with the S3 bucket name mentioned above.

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

This shell script will ultimately upload the `pyspark_venv_python_3.10.12.tar.gz` file to the S3 bucket.

This gz file includes Python 3.10.12 with glow.py, matplotlib, and pandas packages installed.

This file will be used as a configuration value for Spark in EMR Serverless in the future.

You have completed uploading the files necessary for the workshop to S3.
