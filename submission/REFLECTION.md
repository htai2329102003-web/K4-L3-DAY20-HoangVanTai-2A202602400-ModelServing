# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Hoàng Văn Tài
**MSSV:** 2A202602400
**Cohort:** K4-L3
**Ngày submit:** 2026-10-06

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** _Windows 10 (AMD64)_
- **CPU:** _4 physical · 8 logical cores_
- **Cores:** _4 / 8_
- **CPU extensions:** _AVX2_
- **RAM:** _8 GB_
- **Accelerator:** _CPU only_
- **llama.cpp asset đã tải:** _llama-b10488-bin-win-avx2-x64.zip_
- **Model đã dùng:** _Qwen3.5 0.8B_ (`LAB_MODEL=`_qwen35-0.8b_)
- **Quantization:** _Q4_K_M_ + _UD-Q2_K_XL_ (từ `models/active.json`)

**Chạy ở đâu:** _laptop của tôi_
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

_Quá trình setup ban đầu gặp lỗi do lệnh `python` trên Windows bị dính alias của Microsoft Store và mã hóa CP1252 làm crash log. Đã giải quyết bằng cách thay đổi lệnh thành `py` trong file script và chuyển ký tự unicode sang ascii trong thư viện labkit._

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 4221 | 636 / 1256 | 41.9 / 48.6 | 3112 / 4045 / 4045 | 23.9 |
| UD-Q2_K_XL | 0.39 | 4352 | 1291 / 2815 | 519.6 / 559.6 | 34022 / 38070 / 38070 | 1.9 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

_Bản 2-bit chạy chậm hơn tận 12.58 lần so với bản 4-bit (1.9 tok/s vs 23.9 tok/s), dù chỉ tiết kiệm được 0.11 GB RAM. Đánh đổi này hoàn toàn không đáng vì máy tính bị giới hạn bởi CPU compute-bound, quá trình dequantize 2-bit tốn quá nhiều sức mạnh CPU._

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 0.58 | 13000 | 22000 | 23000 | 8.4 | 0.0% |
| 50 | 0.58 | 37000 | 57000 | 57000 | 19.2 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** _1.00_×
- **P95 tăng:** _2.59_×
- **Effective concurrency ở 50 users:** _19.2_ so với `--parallel` = _4_ slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): _3.88_ / _4_ slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

_Server đã bão hòa dưới mức 10 users. Bằng chứng là khi tăng tải lên 5 lần, RPS giữ nguyên (1.00x) nhưng P95 tăng 2.59x. Vì throughput không đổi nên latency tăng thêm hoàn toàn là queue time. Để nâng goodput, tôi sẽ giảm model size hoặc nâng phần cứng để tăng tốc compute._

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | - | stub |
| N17 Data pipeline | - | stub |
| N18 Lakehouse | - | stub |
| N19 Vector + features | - | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: _0.0 ms_
- retrieve: _0.1 ms_
- llm: _6005.6 ms_
- **stage chiếm nhiều nhất:** _llm_ (_100_% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

_Bottleneck nằm hoàn toàn ở bước gọi LLM (chiếm 100% thời gian xử lý), hoàn toàn khớp với kỳ vọng. Nếu muốn giảm latency của hệ thống RAG này đi một nửa, tôi chắc chắn phải tập trung vào việc tối ưu LLM inference._

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
`LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** _Tăng số lượng luồng (threads) xử lý từ 4 lên 16_

```
before:  23.6 tok/s (4 threads)
after:   29.7 tok/s (16 threads)
speedup: 1.26×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Thông thường, tốc độ tối ưu nhất của CPU đạt được khi số lượng luồng bằng đúng với số physical cores (tương ứng là 4 luồng trên máy tôi). Tuy nhiên, kết quả đo lại cho thấy tốc độ cao nhất (29.7 tok/s) đạt được ở mức 16 luồng (oversubscription)._

_Điều này xảy ra là do quá trình LLM inference đôi khi bị nghẽn ở bước chờ dữ liệu từ RAM (memory bandwidth bound). Khi số lượng luồng lớn hơn rất nhiều so với số core thực tế, hệ điều hành tận dụng cơ chế hyperthreading và context switch liên tục. Nhờ vậy, trong lúc một luồng đang phải chờ I/O từ memory, CPU lập tức chuyển sang tính toán cho luồng khác. Cơ chế này giúp các ALU của CPU luôn được lấp đầy công việc, không bị bỏ trống, từ đó mang lại tốc độ tổng thể cao hơn 1.26 lần so với việc dùng đúng 4 luồng vật lý._

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _(để trống)_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_Phiên bản lượng tử hóa nhỏ hơn (2-bit) lại chạy chậm hơn bản 4-bit tới 12.58 lần do chi phí giải nén quá lớn trên CPU. Điều này đi ngược hoàn toàn với suy nghĩ thông thường là "càng nhỏ thì càng nhanh"._

---

## 8. Self-check trước khi push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [x] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [x] `benchmarks/02-server-results.md` committed (`make load-report`)
- [x] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [x] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [x] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [x] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [x] 5 screenshots trong `submission/screenshots/`
- [x] `make verify` → **exit 0**
- [x] Repo tên đúng mẫu `K4-L3-DAY20-HoVaTen-MSSV-ModelServing` (xem `docs/SUBMISSION.md`)
- [x] Repo GitHub ở chế độ **public**
- [ ] Đã push và paste public URL vào VinUni LMS **trước 23:59 (UTC+7) ngày làm lab**
- [x] **Không** commit `models/*.gguf`, `runtime/` hay `.env` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

---

## 9. Khai báo sử dụng AI  *(xem `docs/RULES.md` §3)*

_Sử dụng AI Assistant (Antigravity) để debug các lỗi encode CP1252, sửa lỗi alias PowerShell và hỗ trợ lập luận các chỉ số từ hệ thống._
