# Lakehouse evidence (lightweight path)

Bằng chứng thay cho ảnh chụp MinIO console (đường Spark không dùng) — theo rubric,
đường lightweight nộp `tree _lakehouse/` + nội dung một file `_delta_log/*.json`.

## 1. `tree _lakehouse/` (thư mục, độ sâu 4)

```
_lakehouse
_lakehouse/blobs
_lakehouse/bronze
_lakehouse/bronze/agent_traces
_lakehouse/bronze/agent_traces/_delta_log
_lakehouse/bronze/docs_multimodal
_lakehouse/bronze/docs_multimodal/_delta_log
_lakehouse/bronze/llm_calls_raw
_lakehouse/bronze/llm_calls_raw/_delta_log
_lakehouse/gold
_lakehouse/gold/agent_performance
_lakehouse/gold/agent_performance/_delta_log
_lakehouse/gold/llm_daily_metrics
_lakehouse/gold/llm_daily_metrics/_delta_log
_lakehouse/gold/llm_daily_metrics/date=2026-04-01
_lakehouse/gold/llm_daily_metrics/date=2026-04-02
_lakehouse/gold/llm_daily_metrics/date=2026-04-03
_lakehouse/gold/llm_daily_metrics/date=2026-04-04
_lakehouse/gold/llm_daily_metrics/date=2026-04-05
_lakehouse/gold/llm_daily_metrics/date=2026-04-06
_lakehouse/gold/llm_daily_metrics/date=2026-04-07
_lakehouse/iceberg
_lakehouse/iceberg/nb5
_lakehouse/iceberg/nb5/warehouse
_lakehouse/iceberg/nb5/warehouse/lake
_lakehouse/iceberg/nb6
_lakehouse/iceberg/nb6/warehouse
_lakehouse/iceberg/nb6/warehouse/lake
_lakehouse/iceberg/nb8
_lakehouse/iceberg/nb8/warehouse
_lakehouse/iceberg/nb8/warehouse/lake
_lakehouse/scratch
_lakehouse/scratch/customers_tt
_lakehouse/scratch/customers_tt/_delta_log
_lakehouse/scratch/docs_cdf
_lakehouse/scratch/docs_cdf/_change_data
_lakehouse/scratch/docs_cdf/_delta_log
_lakehouse/scratch/docs_intable
_lakehouse/scratch/docs_intable/_delta_log
_lakehouse/scratch/emb_f32
_lakehouse/scratch/emb_f32/_delta_log
_lakehouse/scratch/emb_int8
_lakehouse/scratch/emb_int8/_delta_log
_lakehouse/scratch/events_smallfiles
_lakehouse/scratch/events_smallfiles/_delta_log
_lakehouse/scratch/maint_events
_lakehouse/scratch/maint_events/_delta_log
_lakehouse/scratch/media_inline
_lakehouse/scratch/media_inline/_delta_log
_lakehouse/scratch/media_pointer
_lakehouse/scratch/media_pointer/_delta_log
_lakehouse/scratch/users_delta
_lakehouse/scratch/users_delta/_delta_log
_lakehouse/scratch/vector_index_external
_lakehouse/scratch/vector_index_external/_delta_log
_lakehouse/silver
_lakehouse/silver/agent_trajectories
_lakehouse/silver/agent_trajectories/_delta_log
_lakehouse/silver/agent_trajectories/agent_version=policy-v2
_lakehouse/silver/agent_trajectories/agent_version=policy-v3
_lakehouse/silver/llm_calls
_lakehouse/silver/llm_calls/_delta_log
_lakehouse/silver/llm_calls/date=2026-04-01
_lakehouse/silver/llm_calls/date=2026-04-02
_lakehouse/silver/llm_calls/date=2026-04-03
_lakehouse/silver/llm_calls/date=2026-04-04
_lakehouse/silver/llm_calls/date=2026-04-05
_lakehouse/silver/llm_calls/date=2026-04-06
_lakehouse/silver/llm_calls/date=2026-04-07
_lakehouse/silver/training_corpus_governed
_lakehouse/silver/training_corpus_governed/_delta_log
_lakehouse/silver/training_corpus_governed/provenance_bucket=UNCLASSIFIED
_lakehouse/silver/training_corpus_governed/provenance_bucket=licensed
_lakehouse/silver/training_corpus_governed/provenance_bucket=public_domain
_lakehouse/silver/training_corpus_governed/provenance_bucket=scraped_optout_checked
_lakehouse/silver/training_corpus_governed/provenance_bucket=synthetic
```

## 2. Nội dung `_delta_log/00000000000000000000.json` — bảng Bronze `llm_calls_raw`

Commit đầu tiên (version 0) khi `make data` ghi 200,000 dòng vào Bronze (NB1/NB4).
File Delta log là NDJSON — mỗi dòng một action riêng (`commitInfo`, `protocol`, `metaData`, `add`):

```json
{"commitInfo":{"timestamp":1787066714749,"operation":"WRITE","operationParameters":{"mode":"Overwrite"},"engineInfo":"delta-rs:py-1.6.2","operationMetrics":{"execution_time_ms":157,"num_added_files":1,"num_added_rows":200000,"num_partitions":0,"num_removed_files":0},"clientVersion":"delta-rs.py-1.6.2"}}
{"protocol":{"minReaderVersion":1,"minWriterVersion":2}}
{"metaData":{"id":"65f4a3d6-6248-4d89-8ab1-bc3426cfae3c","name":null,"description":null,"format":{"provider":"parquet","options":{}},"schemaString":"{\"type\":\"struct\",\"fields\":[{\"name\":\"request_id\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}},{\"name\":\"ts\",\"type\":\"timestamp\",\"nullable\":true,\"metadata\":{}},{\"name\":\"raw_json\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}}]}","partitionColumns":[],"createdTime":1787066714591,"configuration":{}}}
{"add":{"path":"part-00000-01c525d6-0044-4006-9a8e-5b1a3b6df4cd-c000.snappy.parquet","partitionValues":{},"size":13987087,"modificationTime":1787066714749,"dataChange":true,"stats":"{\"numRecords\":200000,\"minValues\":{\"ts\":\"2026-04-01T00:00:00Z\",\"raw_json\":\"{\\\"model\\\": \\\"claude-haiku-4-5\\\", \\\"user_id\\\": \\\"u_1\\\", \\\"usage\\\": {\\\"input\",\"request_id\":\"00003d04-bcde-4801-882c-f7b905ad20d6\"},\"maxValues\":{\"raw_json\":\"{\\\"model\\\": \\\"claude-sonnet-4-6\\\", \\\"user_id\\\": \\\"u_999\\\", \\\"usage\\\": {\\\"io\",\"request_id\":\"ffffe51d-c9bd-4a64-992d-38f077901f95\",\"ts\":\"2026-04-07T23:59:56Z\"},\"nullCount\":{\"raw_json\":0,\"ts\":0,\"request_id\":0}}","tags":null,"baseRowId":null,"defaultRowCommitVersion":null,"clusteringProvider":null}}
```

Đọc nhanh: `commitInfo` ghi engine (`delta-rs:py-1.6.2`) + số dòng thêm (200,000);
`add` trỏ tới file Parquet vật lý và mang theo min/max/null-count stats dùng cho
data-skipping — cùng định dạng JSON mà Spark/Databricks ghi ra.
