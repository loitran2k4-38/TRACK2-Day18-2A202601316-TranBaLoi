# Reflection

Anti-pattern tôi dễ vướng nhất: **coi "chạy job retention/expire" là đã dọn xong storage**, trong khi nó thực chất chỉ xoá metadata.

Dữ liệu của tôi (`llm_calls_raw`, 200K dòng, ghi liên tục theo request) giống pipeline logging LLM thật: mỗi lần append tạo một snapshot mới, tần suất commit cao. Ở NB6, sau `expire_snapshots`, số snapshot Iceberg giảm 20 → 3, nhưng **0 file `.avro` bị xoá thật** — dung lượng data vẫn nguyên, chỉ metadata gọn lại. Nếu chỉ nhìn số snapshot giảm mà kết luận "đã dọn sạch, hoá đơn S3 sẽ giảm", tôi sẽ bất ngờ khi bill không đổi — đúng như Job 3 (expiry) và Job 4 (orphan sweep) là một cặp bắt buộc, thiếu Job 4 thì Job 3 gần như vô nghĩa.

Rủi ro này lớn với dữ liệu của tôi vì log LLM ghi liên tục 24/7 → snapshot/orphan tích luỹ rất nhanh nếu thiếu job maintenance định kỳ, và lỗi âm thầm (không crash, không cảnh báo) — chỉ lộ ra khi so dung lượng S3 thực tế với kỳ vọng, lúc đó đã khó truy vết.
