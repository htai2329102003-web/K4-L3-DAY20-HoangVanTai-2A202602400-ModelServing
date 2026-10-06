# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=4` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 33 | 0.58 | 13000 | 22000 | 23000 | 8.4 | 0.0% |
| 50 | 34 | 0.58 | 37000 | 57000 | 57000 | 19.2 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## Your reading

Server bị bão hòa (saturate) ở mức tải rất sớm (ngay khoảng 10 users). Bằng chứng rõ ràng nhất là khi tăng lượng request gửi đến gấp 5 lần (từ 10 lên 50 users), thông lượng (throughput) thực tế của server KHÔNG HỀ tăng (RPS giữ nguyên ở mức 1.00x - 0.58 req/s). Trong khi đó, độ trễ P95 tăng vọt lên 2.59 lần. Điều này chứng tỏ từ sau 10 users, các request mới bị đẩy hết vào hàng đợi (queue time) chứ server không thể xử lý nhanh hơn được nữa.

Để tăng Goodput, giải pháp tối ưu nhất lúc này không phải là tinh chỉnh thông số server (vì server đã đạt ngưỡng giới hạn xử lý vật lý), mà là đổi sang một model nhỏ hơn/lượng tử hóa mạnh hơn hoặc nâng cấp phần cứng.
