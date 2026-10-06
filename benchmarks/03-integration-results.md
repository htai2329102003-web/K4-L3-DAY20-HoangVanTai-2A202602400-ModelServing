# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.1 | 6861.3 | 6861.4 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 4731.2 | 4731.3 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 6424.3 | 6424.4 |

Mean per stage (ms): embed **0.0** · retrieve **0.1** ·
llm **6005.6** · total **6005.7**
Dominant stage: **llm** (100% of total)

## Which N16-N19 pieces are real

- Cả embed (0.0ms) và retrieve (0.1ms) đều là STUB, thời gian chạy gần như bằng 0.
- Chỉ có LLM generation là REAL (6005.6ms), chiếm 100% tổng độ trễ.
- Do đó, trong một hệ thống RAG thực tế, LLM vẫn sẽ là nút thắt cổ chai (bottleneck) lớn nhất về mặt thời gian.
