# Reflection — Lab 20 (Personal Report)

> **Đây là báo cáo cá nhân.** Mỗi học viên chạy lab trên laptop của mình, với spec của mình. Số liệu của bạn không so sánh được với bạn cùng lớp — chỉ so sánh **before vs after trên chính máy bạn**. Grade rubric tính theo độ rõ ràng của setup + tuning của bạn, không phải tốc độ tuyệt đối.

---

**Họ Tên:** Nguyễn Hoàng Khải Minh
**Cohort:** A20-K1
**Ngày submit:** 6/5/2026

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

> Paste output của `python 00-setup/detect-hardware.py` vào đây, hoặc điền thủ công:

```
(.venv) nhkmi@MSI:~/Day20-Track2-ModelServing-Lab$ python3 00-setup/detect-hardware.py
────────────────────────────────────────────────────────────
  Platform : Linux 6.6.87.2-microsoft-standard-WSL2 (x86_64)
  CPU      : AMD Ryzen 7 5800H with Radeon Graphics
             6 physical · 6 logical cores
             AVX2 available
  RAM      : 4.8 GB
  GPU      : CPU only (no discrete accelerator)
  Docker   : no (compose: no)
────────────────────────────────────────────────────────────

Recommended paths for your hardware:
  • 01-llama-cpp-quickstart
  • 02-llama-cpp-server
  • 03-milestone-integration
  • BONUS-llama-cpp-optimization

Recommended model: TinyLlama-1.1B (Q4_K_M)
llama.cpp backend: CPU (AVX/NEON tuning)
────────────────────────────────────────────────────────────
```

Saved hardware.json — other lab scripts will read this.

**Setup story** (≤ 80 chữ): những gì cần thay đổi để lab chạy được trên máy bạn (vd: dùng WSL2, install CUDA Toolkit, fall back sang Vulkan vì ROCm phiên bản kén, tắt antivirus để pip install nhanh hơn, v.v.):

_Answer here._

Sử dụng WSL2 Ubuntu để chạy lab. Tăng RAM WSL từ 4GB lên 5GB trong `.wslconfig` để phù hợp với model. Điều chỉnh này giúp build ổn định và chạy server thành công.

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

> Paste bảng từ `benchmarks/01-quickstart-results.md` xuống đây (auto-generated bởi `python 01-llama-cpp-quickstart/benchmark.py`).

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|---:|---:|---:|---:|---:|
| tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf | 5793 | 161 / 242 | 36.4 / 41.0 | 2410 / 2764 / 2869 | 27.5 |
| tinyllama-1.1b-chat-v1.0.Q2_K.gguf | 1109 | 231 / 306 | 27.8 / 28.4 | 1957 / 2035 / 2040 | 35.9 |

**Một quan sát** (≤ 50 chữ): Q4_K_M vs Q2_K trên máy bạn — số liệu nói gì? Quality đáng đánh đổi không?

_Answer here._

Q2_K load nhanh hơn 5x (1109ms vs 5793ms) và decode nhanh hơn 30% (35.9 vs 27.5 tok/s), nhưng E2E P50 chỉ nhanh hơn ~450ms. Với tác vụ chat thông thường, Q2_K đáng dùng nếu ưu tiên tốc độ; nếu cần output chất lượng hơn thì Q4_K_M xứng đáng đánh đổi.

---

## 3. Track 02 — llama-server load test

> Chạy 2 lần locust ở concurrency 10 và 50, paste tóm tắt bên dưới.

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.12 | 28000 | 28000 | 28000 | 0 (0.00%) |
| 50 | 0.12 | 28000 | 44000 | 44000 | 0 (0.00%) |

**KV-cache observation** (từ `record-metrics.py`): peak `llamacpp:kv_cache_usage_ratio` ở concurrency 50 = _<0.XX>_, nghĩa là …

_Answer here._

---

## 4. Track 03 — Milestone integration

> Paste output của `make pipeline`

```
(.venv) nhkmi@MSI:~/Day20-Track2-ModelServing-Lab$ make pipeline

=== Why is goodput more useful than throughput? ===
  contexts: ['n20-paged', 'n20-radix', 'n20-disagg']
  timings : {'retrieve': 0.0, 'llm': 10888.1, 'total': 10889.2}
  answer  : Answer: Goodput (the amount of data transferred in a given time period) is a more useful metric for measuring the performance of a network than throughput (the total number of bytes transmitted during a given time period). Throughput measures the rate at which data can be transmitted, while goodput

=== What problem does PagedAttention actually solve? ===
  contexts: ['n20-paged', 'n20-radix', 'n20-disagg']
  timings : {'retrieve': 0.5, 'llm': 3316.9, 'total': 3317.6}
  answer  : The question does not provide a specific problem that PagedAttention solves. It only asks about the solution provided by PagedAttention, which is KV cache like virtual memory pages, eliminating 60-80% fragmentation and storing KV in a prefix trie, cache hit on shared prefix lets the engine skip pref

=== When should I think about disaggregated serving? ===
  contexts: ['n20-disagg', 'n20-paged', 'n20-radix']
  timings : {'retrieve': 0.0, 'llm': 6462.2, 'total': 6462.3}
  answer  : Answer: When the serving-engineering tutors have a high degree of fragmentation and prefilling is causing significant overhead, such as 60-80% fragmentation. In this case, disaggregated serving would be a viable option to address the issue.

Context: The tutors are serving multiple applications that
```

- **N16 (Cloud/IaC):** stub: localhost only
- **N17 (Data pipeline):** stub: in-memory dict
- **N18 (Lakehouse):** stub: in-memory
- **N19 (Vector + Feature Store):** stub: TOY_DOCS

**Nơi tốn nhiều ms nhất** trong pipeline (đo bằng `time.perf_counter` trong `pipeline.py`):

- embed: ~0 ms
- retrieve: ~0.5 ms
- llama-server: ~3317 – 10888 ms

**Reflection** (≤ 60 chữ): Bottleneck nằm hoàn toàn ở llama-server (LLM inference), chiếm >99% tổng thời gian. Retrieve gần như tức thì. Kết quả đúng kỳ vọng vì model chạy trên CPU không có GPU offload.

---

## 5. Bonus — The single change that mattered most

**Change:** Tuning `-t` (thread count) từ default xuống 3 threads thông qua thread sweep trên CPU 6-core.

**Before vs after** (paste 2-3 dòng từ sweep output):

```
before: -t 1  →  14.2 tok/s
after:  -t 3  →  28.3 tok/s
speedup: ~2.0×
```

**Tại sao nó work** (1–2 đoạn ngắn — đây là phần grader đọc kỹ nhất):

LLM decode là workload **memory-bandwidth-bound**, không phải compute-bound — mỗi token sinh ra phải load toàn bộ weight từ RAM, còn phép tính thực sự rất ít. Vì vậy thêm thread chỉ có ích khi mỗi thread đang khai thác một memory channel khác nhau.

Trên máy này (6 physical cores), peak xảy ra ở `-t 3` chứ không phải `-t 6`. Lý do: 6 threads cùng tranh nhau memory bus, gây contention — các thread phải chờ nhau fetch data thay vì chạy song song thật sự. Kết quả `-t 6` (27.9) thậm chí còn chậm hơn `-t 3` (28.3), đúng với lý thuyết memory-bandwidth ceiling từ lecture. Sweet spot nằm ở số thread vừa đủ để saturate bandwidth mà không gây contention.

---

## 6. (Optional) Điều ngạc nhiên nhất

_(1–2 câu — không bắt buộc, nhưng người grader đọc tất cả)_

_Answer here._

---

## 7. Self-graded checklist

- [x] `hardware.json` đã commit
- [x] `models/active.json` đã commit (hoặc paste path snapshot vào section 1)
- [x] `benchmarks/01-quickstart-results.md` đã commit
- [x] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit
- [x] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep)
- [x] Ít nhất 6 screenshots trong `submission/screenshots/` (xem `submission/screenshots/README.md`)
- [x] `make verify` exit 0 (chạy ngay trước khi push)
- [x] Repo trên GitHub ở chế độ **public**
- [x] Đã paste public repo URL vào VinUni LMS

---

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Nếu private, grader không xem được → 0 điểm.
