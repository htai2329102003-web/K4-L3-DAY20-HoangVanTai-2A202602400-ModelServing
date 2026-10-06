# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=4` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3884 | 536 / 568 | 38.0 / 40.5 | 2913 / 2992 / 2992 | 26.3 |
| UD-Q2_K_XL | 0.39 | 5171 | 1437 / 1927 | 543.8 / 638.4 | 35827 / 42145 / 42145 | 1.8 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **14.61x SLOWER** than `Q4_K_M` here, despite being 0.11 GB smaller. That is a real result, not a mistake: fewer bits only buys speed when decode is limited by memory bandwidth. On a machine that is compute-limited instead — few cores, no GPU offload — the extra dequantization work of a heavily-quantized format can cost more than the bytes it saves. Say which case yours is.

## Your observation

Bản `UD-Q2_K_XL` (2-bit) chạy chậm hơn rất nhiều lần so với bản `Q4_K_M` (4-bit), dù nó chỉ tiết kiệm được vỏn vẹn 0.11 GB RAM. Lý do là vì máy bị giới hạn bởi năng lực tính toán của CPU (compute-bound) thay vì băng thông bộ nhớ. Việc phải giải mã (dequantize) một định dạng bị nén quá sâu như 2-bit tốn quá nhiều sức mạnh CPU, vượt quá phần lợi ích về dung lượng bộ nhớ. Do đó, trên cấu hình máy này, việc dùng bản 2-bit là hoàn toàn không đáng; bản 4-bit (`Q4_K_M`) mang lại trải nghiệm tốt hơn hẳn.
