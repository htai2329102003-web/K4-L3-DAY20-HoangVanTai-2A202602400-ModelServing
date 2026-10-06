# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **4 physical · 8 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 22.7 | 99% |
| 2 | 22.8 | 99% |
| 4 | 22.6 | 98% |
| 8 | 23.0 | 100% |
| 16 | 22.6 | 98% |

**Best**: `-t 8` at 23.0 tok/s
**Slowest tested**: `-t 16` at 22.6 tok/s (1.02x spread)
**Against the physical-core default** (`-t 4`, 22.6 tok/s): 1.02x

Use this in your run:

```bash
LAB_N_THREADS=8 make bench
```

## Your explanation

Đường biểu diễn tốc độ là một đường thẳng (flat) với mức chênh lệch gần như không đáng kể (1.02x) giữa 1 luồng và 8 luồng. Điều này cho thấy hệ thống đã hoàn toàn bị giới hạn bởi băng thông bộ nhớ (memory bandwidth bound) ngay cả khi chỉ chạy bằng 1 luồng duy nhất. Do CPU đủ nhanh để xử lý hết luồng dữ liệu truyền từ RAM, việc thêm số lượng luồng không thể tăng thêm tốc độ vì nút thắt lúc này nằm ở tốc độ truyền dữ liệu của bộ nhớ.
