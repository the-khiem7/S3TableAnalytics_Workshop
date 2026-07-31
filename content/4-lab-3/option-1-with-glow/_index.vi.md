---
title: "Tùy chọn 1: với Glow"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b>6.3. </b>"
---

## Truy cập EMR workspace và mở notebook

{{% notice info %}}
Phần này hướng dẫn truy cập EMR Workspace đã thiết lập trong Điều kiện tiên quyết > Thiết lập EMR Workspace.
{{% /notice %}}

1. Truy cập trang [EMR Console](https://console.aws.amazon.com/emr/).

2. Chọn 'EMR Serverless' từ menu bên trái.

3. Trong phần 'EMR Studio' bên phải, chọn 'workshop_emr_studio' và nhấp nút 'Manage applications'.

4. Nhấp nút này sẽ mở trang EMR Studio trong cửa sổ mới.

5. Nhấp 'Workspaces' trong menu bên trái của EMR Studio.

6. Nhấp vào 'workshop-workspace' đã tạo trước đó.

7. Một cửa sổ mới với trang JupyterLab sẽ mở ra.

8. Trong menu bên trái của JupyterLab, nhấp để mở notebook 'workshop-workspace.ipynb' đã được tạo.

9. Chọn 'Pyspark' trong tùy chọn 'Select Kernel'.

![Mở notebook](/images/workshop/emr-open-notebook.webp)

Bạn đã sẵn sàng tạo Namespace và Table trong S3 Tables dựa trên EMR Serverless.

## Thiết lập Cấu hình Spark cho Glow

{{% notice info %}}
Phần này thiết lập cấu hình Spark để sử dụng môi trường ảo Python 3.10.12 đã đóng gói với Amazon S3 Tables, AWS Glue, Apache Iceberg và Glow.
{{% /notice %}}

Nếu bạn gặp lỗi liên quan đến bộ nhớ hoặc được yêu cầu khởi động lại kernel khi chạy mã Spark sau khi thực thi mã configuration, khuyến nghị khởi động lại EMR Serverless Application như sau:

1. Chọn 'Serverless' > 'Applications' từ menu bên trái trên màn hình EMR Studio.
2. Chọn 'workshop_emr_application' và nhấp nút 'Stop application' để dừng.
3. Sau đó nhấp nút 'Start application' để khởi động EMR Serverless Application.
4. Xác nhận rằng cluster đã được gắn đúng trong notebook JupyterLab trước khi tiếp tục công việc.

Truy cập EMR Workspace (JupyterLab) đã tạo trước đó.

1. Mở 'workshop_workspace.ipynb'.

2. Thêm mã configuration sau vào cell đầu tiên của Notebook mới:

```python
%%configure -f
{
    "conf": {
        "spark.jars": "/usr/share/aws/iceberg/lib/iceberg-spark3-runtime.jar,s3://${bucket}/lib/s3-tables-catalog-for-iceberg-runtime-0.1.5.jar,s3://${bucket}/lib/glow-spark3-assembly-2.0.0.jar",
        "spark.sql.catalog.s3table": "org.apache.iceberg.spark.SparkCatalog",
        "spark.sql.catalog.s3table.catalog-impl": "software.amazon.s3tables.iceberg.S3TablesCatalog",
        "spark.sql.catalog.s3table.warehouse": "${table_bucket_arn}",
        "spark.sql.extensions": "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
        "spark.hadoop.io.compression.codecs": "io.projectglow.sql.util.BGZFCodec",
        "spark.archives": "s3://${bucket}/lib/pyspark_venv_python_3.10.12.tar.gz#environment",
        "spark.executorEnv.PYSPARK_PYTHON": "./environment/bin/python",
        "spark.emr-serverless.driverEnv.PYSPARK_DRIVER_PYTHON": "./environment/bin/python",
        "spark.emr-serverless.driverEnv.PYSPARK_PYTHON": "./environment/bin/python"
    }
}
```

Mã này cấu hình Spark để sử dụng các thành phần sau:
- Cài đặt Catalog để sử dụng AWS Glue và Amazon S3 Tables
- Thêm thư viện để sử dụng Apache Iceberg
- Cài đặt môi trường ảo Python 3.10.12 để sử dụng Glow kết hợp với hai thành phần trên

Các giá trị cấu hình sau tương ứng với điều này:
- `spark.archives`
- `spark.executorEnv.PYSPARK_PYTHON`
- `spark.emr-serverless.driverEnv.PYSPARK_DRIVER_PYTHON`
- `spark.emr-serverless.driverEnv.PYSPARK_PYTHON`

Giá trị `${bucket}` và `${table_bucket_arn}` cần được thay đổi phù hợp với môi trường của bạn. Đặc biệt, thay đổi giá trị `${table_bucket_arn}` thành ARN của Table bucket bạn đã sao chép trước đó.

3. Nhấp nút Run để thực thi cell notebook.

![Cấu hình Spark](/images/workshop/lab3-configure.webp)

Chuẩn bị đã hoàn tất để sử dụng mã Pyspark tham chiếu đến Amazon S3 Tables, Glue, Apache Iceberg, cũng như thư viện Glow.

## Tải Dữ liệu VCF

{{% notice info %}}
Phần này tải dữ liệu VCF sử dụng Glow và Spark.
{{% /notice %}}

1. Nhấp nút '+' để thêm cell.

2. Thêm mã sau vào cell mới và chạy để khởi động Spark App:

```python
import glow
spark = glow.register(spark)
```

Mã này đăng ký phiên Spark với Glow. Điều này cho phép đọc dữ liệu, biến đổi và các hàm liên quan dựa trên Glow trong Spark.

3. Đọc dữ liệu VCF từ 1000genomes:

```python
vcf_file = "s3://1000genomes/phase1/analysis_results/integrated_call_sets/ALL.chr17.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.vcf.gz"

df = spark.read.format("vcf").load(vcf_file)
df.printSchema()
```

`spark.read.format("vcf").load(vcf_file)` — Như bạn thấy từ mã này, Spark tải file VCF ở định dạng 'vcf'.

Nhấp nút 'Run' để thực thi cell này. Lệnh gọi hàm `printSchema()` sẽ xuất schema của dữ liệu VCF.

![Tải file VCF](/images/workshop/lab3-load-vcf-data.webp)

4. Nhấp nút '+' để thêm cell.

5. Sử dụng mã sau để xác định số lượng dòng của file VCF:

```python
df.count()
```

Kết quả sẽ xuất **1046733**. Đây là số lượng dòng của file VCF.

Chúng ta đã tải file VCF từ S3 vào Spark Dataframe và xuất số lượng dòng trong Dataframe đó.

## Tạo Bảng Dữ liệu VCF

{{% notice info %}}
Phần này tạo bảng trong S3 Tables từ dữ liệu VCF đã tải.
{{% /notice %}}

1. Nhấp nút '+' để thêm cell.

2. Thêm mã sau vào cell mới:

```python
catalog = "s3table"
namespace = "workshop_namespace"
table = "vcf_delta"

df = df.toDF(*[col.lower() for col in df.columns])

df \
    .write \
    .saveAsTable(f"{catalog}.{namespace}.{table}")
```

Mã này gọi hàm `saveAsTable()`. Nó tạo bảng `vcf_delta` từ dữ liệu VCF đã tải trước đó.

`df = df.toDF(*[col.lower() for col in df.columns])` — Nếu tên cột trong df là chữ hoa, chúng có thể không được nhận diện đúng trong Athena. Do đó, tất cả tên cột trong df được chuyển sang chữ thường.

3. Nhấp nút 'Run' để tạo bảng với dữ liệu VCF này.

![Tạo bảng VCF](/images/workshop/lab3-create-vcf-table.webp)

4. Để kiểm tra dữ liệu trong bảng đã tạo, nhấp nút '+' để thêm cell.

5. Thêm mã sau vào cell mới:

```python
vcf_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

vcf_df.drop('genotypes').show(3, False)
```

Mã này khai báo một Dataframe đọc bảng `vcf_delta`. Nó gọi hàm `show()` để hiển thị 3 dòng dữ liệu trên màn hình.

Chạy cell này và kiểm tra đầu ra để xác nhận bảng đã được tạo chính xác.

6. Nhấp nút '+' để thêm cell.

7. Thêm mã sau vào cell mới và chạy:

```python
vcf_df.count()
```

Mã này gọi hàm `count()` để kiểm tra số lượng dòng dữ liệu. Bạn có thể xác nhận rằng nó xuất **1046733** dòng, giống như số lượng dòng của file gốc.

Chúng ta đã hoàn thành việc tạo bảng trong S3 Tables sử dụng Spark Dataframe đã tải từ file VCF.

## Merge Dữ liệu VCF khác

{{% notice info %}}
Phần này đọc dữ liệu VCF khác và merge vào bảng trong S3 Tables.
{{% /notice %}}

1. Nhấp nút '+' để thêm cell.

2. Thêm mã sau vào cell mới và chạy:

```python
other_vcf_file = "s3://1000genomes/phase1/analysis_results/integrated_call_sets/ALL.chr18.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.vcf.gz"

other_df = spark.read.format("vcf").load(other_vcf_file)
other_df.count()
```

Mã này đọc file VCF khác. Nó gọi hàm `count()` để kiểm tra số lượng dòng của dữ liệu VCF này. Kết quả xuất **1088820** dòng.

3. Nhấp nút '+' để thêm cell.

4. Thêm mã sau vào cell mới và chạy:

```python
other_df = other_df.toDF(*[col.lower() for col in other_df.columns])
other_df.createOrReplaceTempView("other_vcf_table")
```

Mã này chuyển tất cả tên cột trong Dataframe sang chữ thường, như đã làm trước đó. Điều này đăng ký Dataframe đã đọc dữ liệu file VCF khác dưới dạng Temp View. Sau khi đăng ký, nó có thể được tham chiếu như một bảng trong các câu lệnh Spark SQL.

5. Nhấp nút '+' để thêm cell.

6. Thêm mã sau vào cell mới và chạy:

```python
spark.sql(f"""
    MERGE INTO {catalog}.{namespace}.{table} t
    USING other_vcf_table s
    ON (s.contigname=t.contigname and s.start=t.start and s.end=t.end and s.names=t.names and 
        s.referenceallele=t.referenceallele and s.alternatealleles=t.alternatealleles)
    WHEN MATCHED THEN
    UPDATE SET
        t.contigname = s.contigname,
        t.start = s.start,
        t.end = s.end,
        t.names = s.names,
        t.referenceallele = s.referenceallele,
        t.alternatealleles = s.alternatealleles,
        t.qual = s.qual,
        t.filters = s.filters,
        t.splitfrommultiallelic = s.splitfrommultiallelic,
        t.info_avgpost = s.info_avgpost,
        t.info_ac = s.info_ac,
        t.info_ciend = s.info_ciend,
        t.info_ldaf = s.info_ldaf,
        t.info_afr_af = s.info_afr_af,
        t.info_vt = s.info_vt,
        t.info_snpsource = s.info_snpsource,
        t.info_an = s.info_an,
        t.info_theta = s.info_theta,
        t.info_cipos = s.info_cipos,
        t.info_aa = s.info_aa,
        t.info_af = s.info_af,
        t.info_amr_af = s.info_amr_af,
        t.info_asn_af = s.info_asn_af,
        t.info_svlen = s.info_svlen,
        t.info_erate = s.info_erate,
        t.info_homseq = s.info_homseq,
        t.info_rsq = s.info_rsq,
        t.info_end = s.info_end,
        t.info_eur_af = s.info_eur_af,
        t.info_homlen = s.info_homlen,
        t.info_svtype = s.info_svtype
    WHEN NOT MATCHED THEN
    INSERT (contigname, start, end, names, referenceallele, alternatealleles, 
            qual, filters, splitfrommultiallelic, info_avgpost, 
            info_ac, info_ciend, info_ldaf, info_afr_af, info_vt, info_snpsource, info_an, info_theta, 
            info_cipos, info_aa, info_af, info_amr_af, info_asn_af, info_svlen, info_erate, 
            info_homseq, info_rsq, info_end, info_eur_af, info_homlen, info_svtype)
    VALUES (s.contigname, s.start, s.end, s.names, s.referenceallele, s.alternatealleles, 
            s.qual, s.filters, s.splitfrommultiallelic, s.info_avgpost, 
            s.info_ac, s.info_ciend, s.info_ldaf, s.info_afr_af, s.info_vt, s.info_snpsource, s.info_an, s.info_theta, 
            s.info_cipos, s.info_aa, s.info_af, s.info_amr_af, s.info_asn_af, s.info_svlen, s.info_erate, 
            s.info_homseq, s.info_rsq, s.info_end, s.info_eur_af, s.info_homlen, s.info_svtype)
""")
```

Mã này thực thi truy vấn Merge sử dụng `spark.sql()`.

Xem xét truy vấn Merge:
- `{catalog}.{namespace}.{table}` là bảng Target để merge dữ liệu vào.
- `other_vcf_table` là bảng Source đã đăng ký làm Temp View trước đó.
- Mệnh đề `ON` đặt điều kiện khớp.
- `WHEN MATCHED THEN` theo sau bởi câu lệnh UPDATE thực thi cho dữ liệu khớp.
- `WHEN NOT MATCHED THEN` theo sau bởi câu lệnh INSERT thực thi cho dữ liệu không khớp.
- Nói cách khác, nó UPDATE bảng target cho dữ liệu hiện có và INSERT vào bảng target cho dữ liệu mới.

![Merge dữ liệu VCF](/images/workshop/lab3-merge-vcf-data.webp)

7. Nhấp nút '+' để thêm cell.

8. Thêm mã sau vào cell mới và chạy:

```python
vcf_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

vcf_df.count()
```

Mã này tải lại bảng. Nó kiểm tra số lượng dòng của bảng. Vì không có dòng khớp giữa hai Dataframe đã merge, kết quả xuất **2135553**, là tổng của 1046733 dòng ban đầu khi tạo bảng và 1088820 dòng nhập lần này. Điều này xác nhận merge đã thành công.

Chúng ta đã hoàn thành việc merge dữ liệu VCF khác vào bảng đã tạo trước đó.

## Cập nhật / Xóa Dữ liệu VCF

{{% notice info %}}
Phần này cập nhật hoặc xóa dữ liệu trong bảng S3 Tables.
{{% /notice %}}

### Cập nhật Dòng

1. Nhấp nút '+' để thêm cell.

2. Thêm mã sau vào cell mới và chạy:

```python
vcf_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

vcf_df.filter("qual < 100").count()
```

Gọi `filter("qual < 100")` để truy vấn số lượng dòng có chất lượng nhỏ hơn 100. Bạn có thể xác nhận có **10382** bản ghi.

3. Nhấp nút '+' để thêm Cell.

4. Thêm mã sau vào Cell mới và nhấp 'Run'.

```python
vcf_df \
    .filter("qual < 100") \
    .drop('genotypes') \
    .show(5, False)
```

Hiển thị 5 bản ghi có chất lượng nhỏ hơn 100. Sẽ cập nhật hàng loạt các giá trị qual của dữ liệu này thành 0.

5. Nhấp nút '+' để thêm Cell.

6. Thêm mã sau vào Cell mới và nhấp 'Run'.

```python
spark.sql(f"""
UPDATE {catalog}.{namespace}.{table} SET qual = 0 WHERE qual < 100
""")
```

Cập nhật hàng loạt giá trị qual thành 0 cho dữ liệu có giá trị cột qual nhỏ hơn 100.

![Cập nhật bảng VCF](/images/workshop/lab3-update-vcf-table.webp)

7. Nhấp nút '+' để thêm Cell.

8. Thêm mã sau vào Cell mới và nhấp 'Run'.

```python
vcf_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

vcf_df.filter("qual = 0").count()
```

Tải lại bảng. Kiểm tra số lượng dòng có giá trị cột qual là 0. Xác nhận rằng cùng giá trị **10382** bản ghi như trước khi cập nhật được hiển thị.

Cập nhật đã hoàn thành thành công.

### Xóa Dòng

1. Nhấp nút '+' để thêm Cell.

2. Thêm mã sau vào Cell mới và 'Run':

```python
spark.sql(f"""
DELETE FROM {catalog}.{namespace}.{table} WHERE qual = 0
""")
```

Mã này thực hiện truy vấn DELETE cho dữ liệu có giá trị cột qual là 0.

![Xóa bảng VCF](/images/workshop/lab3-delete-vcf-table.webp)

3. Nhấp nút '+' để thêm Cell.

4. Thêm mã sau vào Cell mới và 'Run':

```python
vcf_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

vcf_df.filter("qual = 0").count()
```

Tải lại bảng. Kiểm tra số lượng dòng có giá trị cột qual là 0. Xác nhận rằng đầu ra là 0 vì tất cả đã bị xóa.

Việc xóa đã hoàn thành thành công.

## Time Travel

### Lấy Lịch sử Bảng (Snapshots)

```python
spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}.history
""").show(5, False)
```

```text
+-----------------------+-------------------+-------------------+-------------------+
|made_current_at        |snapshot_id        |parent_id          |is_current_ancestor|
+-----------------------+-------------------+-------------------+-------------------+
|2025-04-02 02:26:45.356|1996392549628528971|NULL               |true               |
|2025-04-02 02:27:16.224|2842948084683911314|1996392549628528971|true               |
|2025-04-02 02:27:37.664|4907549559143542142|2842948084683911314|true               |
|2025-04-02 02:28:08.795|5291307565494698844|4907549559143542142|true               |
+-----------------------+-------------------+-------------------+-------------------+
```

Mã này xuất lịch sử Snapshot của bảng. Nếu bạn thực hiện theo thứ tự của workshop:
- Dòng trên cùng là Snapshot khi bảng được tạo/Insert
- Dòng thứ hai là Snapshot khi thực hiện truy vấn Merge
- Dòng thứ ba là Snapshot khi thực hiện truy vấn Update
- Dòng thứ tư là Snapshot khi thực hiện truy vấn Delete

### Lấy Thông tin Snapshot

```python
snapshot_id = "<snapshot_id>"

spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}.snapshots
WHERE snapshot_id = {snapshot_id} 
""").show(5, False)
```

Mã này xuất vị trí file metadata và dữ liệu liên quan đến file của Snapshot.

### Truy vấn Dòng với Update Snapshot

```python
update_snapshot_id = "<update_snapshot_id>"

spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table} 
FOR VERSION AS OF {update_snapshot_id}
WHERE qual = 0
""").show(5, False)
```

Đặt giá trị `<snapshot_id>` thành ID Snapshot trước khi cập nhật. Bạn có thể xác nhận rằng dữ liệu đã được cập nhật được xuất với giá trị trước khi cập nhật.

### Truy vấn Dòng trước Thời điểm Xóa

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}
FOR TIMESTAMP AS OF TIMESTAMP '{deletion_time}'
WHERE qual = 0
""").show(5, False)
```

Đặt `deletion_time` thành thời điểm trước khi xóa dữ liệu. Bạn có thể kiểm tra các Dòng đã bị xóa.

### Khôi phục sau Xóa

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
CALL {catalog}.system.rollback_to_timestamp('{namespace}.{table}', TIMESTAMP '{deletion_time}')
""")

spark.sql(f"""
SELECT COUNT(*) FROM {catalog}.{namespace}.{table}
WHERE qual = 0
""").show()
```

Đặt `deletion_time` thành thời điểm trước khi xóa dữ liệu. Bạn có thể xác nhận rằng dữ liệu đã xóa trước đó đã được khôi phục.

Chúng ta đã cùng tìm hiểu các truy vấn Time Travel của Apache Iceberg.
