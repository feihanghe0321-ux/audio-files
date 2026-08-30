# Seed-TTS 2.0 问题报告：音色 `en_male_jimmy_uranus_bigtts` 会念错文本

## 现象

用该音色合成时，**约 50% 的请求会把开头一句替换成与输入完全无关的内容**，后半句正常。
同一请求重复调用，每次编造的内容都不一样，**不是稳定的错误，而是随机的**。

- 端点：`POST https://voice.ap-southeast-1.bytepluses.com/api/v3/tts/unidirectional`
- `X-Api-Resource-Id: seed-tts-2.0`
- `speaker: en_male_jimmy_uranus_bigtts`

## 输入文本

```
You're right. This isn't working. But I'm not walking away.
```

## 实际念出来的（10 次里 5 次错，句首均被替换）

| 实际内容 | 与输入的相似度 |
|---|---|
| **You symbolize oil and greed,** and this isn't working. I'm not up for th… | 0.40 |
| **guarded deeply,** but I'm not walking away.（整个前半句丢失） | 0.59 |
| **Hear him.** This isn't working, but I'm not walking away. | 0.80 |
| **You're frustrated,** this isn't working, but I'm not walking away. | 0.90 |
| **All right,** this isn't working, but I'm not walking away. | 0.90 |
| **right here instead be on the lurking,** but I'm not walking away. | 0.55 |

换成另外两段完全不同的文本测试，同样出现句首被替换的情况，**与具体文本无关**。

## 范围：只有这一个 speaker

同一账号、同一次批量测试下：

| 范围 | 结果 |
|---|---|
| seed-tts-2.0 其余 17 个英语男声（Tim / Bob / Cedric / Quentin / Simba 等） | **17/17 正确** |
| 另一组 16 个对照音色 | **16/16 正确** |
| **`en_male_jimmy_uranus_bigtts`** | **10 次中 5 次错误** |

所有测试用的是同一段文本、同一套请求参数，只有 `speaker` 不同。

## 可供排查的 request id（`x-tt-logid`）

同一段输入连打 8 次，其中 **4 次念错**。以下是**念错那几次**的 `x-tt-logid`
（取自响应头，北京时间 2026-08-25 17:00 前后）：

| `x-tt-logid` | 与输入相似度 | 实际念出 |
|---|---|---|
| `20260825170049488E4298712E8881CB97` | 0.84 | cessarily. This isn't working, but I'm not walking away. |
| `20260825170102488E4298712E8881CEB4` | 0.00 | their 3D version and submit. At the end of the day, that's all what it |
| `20260825170122488E4298712E8881D3FD` | 0.00 | information for lesser governed and the order that rotting |
| `20260825170140488E4298712E8881DAFD` | 0.84 | Remy, this isn't working, but I'm not walking away. |

同一批里**正确**的 4 次，可作对照：

`20260825170044E196CAC3B87C281CF0BC`、`20260825170117488E4298712E8881D26E`、
`2026082517012999EB4E372F93627FFA58`、`2026082517013699EB4E372F93627FFD43`

同一账号、同一 speaker、同一段文本、同一套参数，**成功与失败混在同一批请求里**，
相隔仅几十秒。

## 复现方式

见随附的 `seed_tts_jimmy_repro.sh`：

```bash
export BYTEPLUS_API_KEY=<key>
bash seed_tts_jimmy_repro.sh          # 跑 10 次，生成 jimmy_1..10.mp3
```

对照组（预期 10 次全对）：

```bash
SPEAKER=en_male_tim_uranus_bigtts bash seed_tts_jimmy_repro.sh
```

## 随附证据

| 文件 | 对应 `x-tt-logid` | 说明 |
|---|---|---|
| `bad_sample_3_logid_20260825170102.mp3` | `20260825170102488E4298712E8881CEB4` | **相似度 0.00**，整段内容与输入无关 |
| `bad_sample_4_logid_20260825170122.mp3` | `20260825170122488E4298712E8881D3FD` | **相似度 0.00**，整段内容与输入无关 |
| `bad_sample_1.mp3` | —（早于本批） | 念成 “right here instead be on the lurking…” |
| `bad_sample_2.mp3` | —（早于本批） | 另一例整段跑偏 |
| `seed_tts_jimmy_repro.sh` | — | 复现脚本；含响应解码，并把每次的 `x-tt-logid` 记到 `logids.txt` |

## 请协助确认

1. `en_male_jimmy_uranus_bigtts` 这个音色当前是否可用、是否有已知问题？
2. 是否与该音色的某个版本有关，是否有修复计划或建议替代的音色？
3. 我们的请求参数（见脚本）是否有不当之处？

在收到答复前，我们已将该音色从可用列表中下线。
