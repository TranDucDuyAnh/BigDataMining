# Schema dữ liệu

## Tổng quan
Dữ liệu NLP tài chính sau khi xử lí được lưu trữ dưới dạng Parquet và Apache Iceberg trong MinIO.

## Cấu trúc lưu trữ
```
finance/
├── raw/                          # Dữ liệu thô (JSON, PDF)
│   ├── news/
│   └── reports/
├── processed/                    # Dữ liệu đã xử lí
│   └── spacy_pipeline/
│       └── parsed_txt/
└── parquet/                      # Parquet
    └── spacy_pipeline/
        └── parsed_txt/
            ├── company=AAPL/
            └── company=AMZN/
iceberg/                          # Iceberg table (local.spacy_parsed)
    └── spacy_parsed/
```

## Schema: `spacy_parsed`

| Cột | Loại | Có thể null | Mô tả |
|--------|------|----------|-------------|
| TOKEN | string | yes | Từ/token gốc từ văn bản |
| LEMMA | string | yes | Dạng gốc của token |
| POS | string | yes | POS tag (NOUN, VERB, etc.) |
| TAG | string | yes | Penn Treebank POS tag chi tiết |
| DEP | string | yes | Mối quan hệ phụ thuộc với head token |
| HEAD | string | yes | Head token mà từ này phụ thuộc vào |
| source_file | string | no | Đường dẫn đầy đủ của tệp nguồn đến S3 |
| year | integer | yes | Năm báo cáo trích từ tên file |
| company | string | no | Mã chứng khoán công ty (AAPL, AMZN, etc.) |

## Chất lượng dữ liệu
- Số hàng: 1,055,474
- Cột HEAD có 6,356 giá trị null — hay gặp đối với các mã thông báo ROOT trong quá trình phân tích cú pháp phụ thuộc (dependency parsing)
- Chia theo `company` để query công ty hiệu quả hơn

## Cách để đọc

### Parquet
```python
df = spark.read.parquet("s3a://finance/parquet/spacy_pipeline/parsed_txt/")
```

### Iceberg
```python
# Full table
spark.sql("SELECT * FROM local.spacy_parsed")

# Filter by company
spark.sql("SELECT * FROM local.spacy_parsed WHERE company = 'AAPL'")

# Filter by POS tag
spark.sql("SELECT * FROM local.spacy_parsed WHERE POS = 'NOUN'")
```

## Companies
| Ticker | Row Count |
|--------|-----------|
| AAPL | 667,363 |
| AMZN | 388,111 |
| **Total** | **1,055,474** |