---
title: "Về Bộ Dữ liệu"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b>6.1. </b>"
---

## 1. VCF là gì?

Định dạng file để lưu trữ các biến thể di truyền được tìm thấy trong bộ gen. Chủ yếu được sử dụng để ghi lại SNPs, indels, v.v.

Định dạng file này được phát triển bởi Dự án 1000 Genomes, và hiện tại phiên bản VCF 4.5 được sử dụng phổ biến.

File VCF dựa trên văn bản và được sử dụng để lưu trữ dữ liệu gen cho nhiều loài khác nhau, không chỉ bộ gen người.

**Ứng dụng**

- Phân tích gen
  - Khám phá các biến thể trong gen cụ thể
  - Phân tích tần số alen
- Nghiên cứu bệnh
  - Phân tích xem các biến thể cụ thể có liên quan đến bệnh không
  - Nghiên cứu GWAS (Genome-Wide Association Study)
- Trực quan hóa
  - Dữ liệu VCF có thể được xem trong IGV (Integrative Genomics Viewer)

## 2. Cấu trúc File VCF

File VCF bao gồm hai phần chính:

- **Header**
  - Chứa metadata của file, bắt đầu bằng ký tự `#`.
  - Bao gồm định dạng file, phiên bản bộ gen tham chiếu, mô tả trường, v.v.
- **Body**
  - Chứa thông tin biến thể thực tế ở dạng bảng.
  - Bao gồm các trường như CHROM, POS, ID, REF, ALT, v.v.

### 2.1. Header VCF

Header file VCF bao gồm metadata bắt đầu bằng `##` và tiêu đề cột bắt đầu bằng `#CHROM`.

```text
##fileformat=VCFv4.2
##source=GATK
##reference=hg19
##INFO=<ID=DP,Number=1,Type=Integer,Description="Total Depth">
##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
#CHROM  POS     ID      REF     ALT     QUAL    FILTER  INFO    FORMAT  SAMPLE1 SAMPLE2
```

- `##fileformat`: Phiên bản file VCF
- `##source`: Thông tin về phần mềm tạo file
- `##reference`: Phiên bản bộ gen tham chiếu
- `##INFO`: Định nghĩa dữ liệu sẽ được bao gồm trong trường INFO
- `##FORMAT`: Mô tả các trường FORMAT cho từng mẫu
- `#CHROM`: Tên cột dữ liệu

### 2.2. Body VCF

Các trường thông tin biến thể. Được lưu trữ ở định dạng phân tách bằng tab, chứa thông tin biến thể.

```text
#CHROM  POS     ID      REF  ALT  QUAL  FILTER  INFO                 FORMAT    SAMPLE1  SAMPLE2
chr1    12345   rs123   G    A    99.0  PASS    DP=100;AF=0.5        GT:DP     0/1:50   1/1:40
chr2    67890   .       T    C    60.0  PASS    DP=80;AF=0.3         GT:DP     0/0:20   0/1:60
```

- **CHROM**: Nhiễm sắc thể nơi biến thể xảy ra (ví dụ: chr1, chr2, v.v.)
- **POS**: Vị trí của biến thể trên nhiễm sắc thể
- **ID**: ID duy nhất của biến thể (ID cơ sở dữ liệu biến thể)
- **REF**: Trình tự base của bộ gen tham chiếu
- **ALT**: Trình tự base thay thế nơi biến thể xảy ra
- **QUAL**: Điểm chất lượng của biến thể (điểm càng cao thì độ tin cậy càng lớn)
- **FILTER**: Cờ chỉ ra bộ lọc nào đã được vượt qua
- **INFO**: Thông tin biến thể bổ sung (ví dụ: DP=100, AF=0.5)
  - `DP=100`: Read Depth (số lần biến thể được đọc)
  - `AF=0.5`: Allele Frequency (tần số của base thay thế)
  - `MQ=60`: Mapping Quality
  - ...
- **FORMAT**: Danh sách mở rộng các trường mô tả mẫu

| Tên Tag | Mô tả |
|---------|-------|
| GT | Kiểu gen của mẫu tại vị trí này. ví dụ: 0/0 – nghĩa là đồng hợp tử tham chiếu; ví dụ: 0/1 – nghĩa là dị hợp tử, có alen REF/ALT; ví dụ: 1/1 – nghĩa là đồng hợp tử thay thế |
| AD | Unfiltered Allele Depth (phân tách bằng dấu phẩy) |
| DP | Filtered Depth |
| PL | Normalized Phred-scaled likelihoods cho kiểu gen dự đoán |
| GQ | Genotype Quality, độ tin cậy Phred-scaled chỉ ra xác suất GT đúng |
| MQ | RMSMappingQuality |

## 3. Glow là gì?

Glow là bộ công cụ mã nguồn mở để làm việc với dữ liệu gen ở quy mô biobank và hơn thế nữa.

Bộ công cụ được xây dựng trực tiếp trên Apache Spark, công cụ hợp nhất hàng đầu cho xử lý dữ liệu lớn và machine learning, cho phép các quy trình gen mở rộng đến quy mô dân số.

### 3.1. Giới thiệu

- Dữ liệu gen đang phát triển đến quy mô dữ liệu lớn, tăng gấp đôi mỗi 7 tháng trên toàn cầu.
- Tuy nhiên, hầu hết các công cụ dữ liệu gen chạy trên một node đơn lẻ và không thể mở rộng.
- Glow ra đời để giải quyết vấn đề này.
- Được xây dựng trên Apache Spark và Delta Lake, cho phép tính toán phân tán và lưu trữ dữ liệu di truyền.

### 3.2. Tính năng

- **Nhập dữ liệu**: Có thể đọc các định dạng VCF, BGEN, Plink vào Apache Spark DataFrames.
- **Chức năng tích hợp sẵn**: Các tác vụ phổ biến như tính toán thống kê kiểm soát chất lượng, chạy kiểm tra hồi quy, và thực hiện các phép biến đổi đơn giản được cung cấp dưới dạng hàm Spark có thể gọi từ Python, SQL, Scala, hoặc R.
- **Biến đổi dữ liệu**: Cung cấp các tính năng biến đổi tập dữ liệu như chuẩn hóa biến thể và lift over.
- **Khả năng mở rộng**: Có thể thêm User-defined Functions, một tính năng của Apache Spark.
- **Kết nối hệ sinh thái tin sinh học và dữ liệu lớn**: Các phương pháp tốt nhất được sử dụng bởi kỹ sư dữ liệu và nhà khoa học dữ liệu trong các ngành.
- Được xây dựng trên Apache Spark và Delta Lake, cho phép tính toán phân tán và lưu trữ dữ liệu di truyền.
- **Tích hợp**: Có thể kết hợp với các tập dữ liệu như hồ sơ y tế điện tử, bằng chứng thực tế, và hình ảnh y tế để tạo ra thông tin chi tiết bổ sung.
