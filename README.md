# Data Engineering Immersion Day

A hands-on workshop from the [AWS First Cloud Journey](https://github.com/vanhoangkha/aws-first-cloud-journey) series.

**Read the workshop online:** https://vanhoangkha.github.io/000105-Data-Engineering-Immersion-Day/

## Overview

Đây là một bài hands-on labs có chứa các module tập trung vào việc sinh, nhập, load, khám phá và sử dụng dữ liệu trong data lake trên AWS

Data Engineering Immersion day cung cấp thời gian hands-on trên các dịch vụ phân tích dữ liệu như Amazon Kinesis Services để truyển tải và phân tích dữ liệu, AWS Data Migration service, AWS Glue, Amazon Athena, Amazon Quicksight. Data Engineering Immersion day giúp xây dựng serverless data lake trên nền tảng đám mây.

## Workshop contents

1. Giới thiệu
2. Lab. Phát hiện luồng click chuột bất thường bằng Amazon Managed Service for Apache Flink
   - Phát hiện luồng click chuột bất thường bằng Amazon Managed Service for Apache Flink
   - Lab
   - Lab. Streaming ETL with Glue
   - Real-time Streaming with Kinesis
   - Clickstream Analytics using MSK
3. Nhập data với DMS
   - Option1: DMS Migration Lab
   - Option 3: Skip DMS Lab
   - Option 2: AutoComplete DMS Lab
4. Lab: Transforming data with Glue
   - Data Validation and ETL
   - Incremental Data Processing with Hudi
5. Lab: Query and Visualize
   - Athena and QuickSight
   - Athena Federated query
   - Athena and SageMaker
6. Lab: Data Lake Automation
   - Lake Formation Lab
   - Lake Formation Lab for Apache Hudi Tables
   - Lake Formation Lab for Apache Iceberg Tables
   - Lake Formation Lab for Delta Tables
7. Bonus Lab: Glue DataBrew
   - DataBrew Pre-Lab
   - Data preparation with Glue DataBrew
8. Clean up

## Prerequisites

- An AWS account with permissions to create the resources used in the labs
- Basic familiarity with the AWS Management Console
- [Hugo](https://gohugo.io/installation/) (extended edition) if you want to run the site locally

## Run locally

```bash
git clone --recurse-submodules https://github.com/vanhoangkha/000105-Data-Engineering-Immersion-Day.git
cd 000105-Data-Engineering-Immersion-Day
hugo server
```

Then open http://localhost:1313 in your browser.

## Repository structure

| Path | Purpose |
| --- | --- |
| `content/` | Workshop pages |
| `static/` | Images and other static assets |
| `layouts/` | Hugo layout overrides |
| `themes/` | Hugo theme |
| `config.toml` | Site configuration |

## Cost and cleanup

The labs create AWS resources that may incur charges. Complete the **Clean up** chapter when you finish to remove them.

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

To report a security issue, see [SECURITY.md](SECURITY.md).

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).
