# Bonus — Thread sweep

Model: `tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf`  ·  GPU layers: `0`

| threads | tg\d+ (tok/s) |
|---:|---:|
| 1 | 14.2 |
| 2 | 24.7 |
| 3 | 28.3 |
| 6 | 27.9 |

**Best**: `-t 3` at 28.3 tok/s.

Look at the curve. If it peaks around your **physical** core count and drops as you go higher, that's the memory-bandwidth ceiling: extra threads fight over the same memory channels and slow each other down.
