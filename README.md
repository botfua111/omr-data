# OMR Data · 免费情报数据

每日同步「免费额度」清单，解析为结构化 JSON，通过 jsDelivr CDN 对小程序端提供只读访问。
本仓只放数据产物，抓取管线在私有仓维护。

## 小程序端读取

```
https://cdn.jsdelivr.net/gh/botfua111/omr-data@main/data/free-tokens/latest.json
```

- `latest.json`：最新一版数据（每次运行后刷新 jsDelivr 缓存）
- `<北京日期>.json`：按日快照，如 `data/free-tokens/2026-11-26.json`，可作回退/回溯
- `logos/<host>.png`：渠道图标，随 JSON 一起分发

## 更新机制

每天北京时间 09:40 由上游管线拉取清单、解析校验后写入本仓 `data/`：

- 栏目条目数必须与声明的「N 个」一致
- 总条数 ≥ 40；与上一版偏离 ±50% 视为异常
- 每条必填 提供方 / 能领多少 / 去领，去领必须是 https URL
- `meta.syncedAt` 每次运行都刷新，因此每天必有一次数据提交（即使上游无更新）
- 提交后主动刷新 jsDelivr 的 `latest.json` 缓存

> 注意：purge 返回 `finished` 后，边缘节点传播可能仍需数分钟才全部生效。
> 时效敏感场景可用按日快照 URL（新文件名必为缓存未命中，回源即最新）。

## 数据结构

```jsonc
{
  "meta": {
    "sourceUpdatedAt": "2026-11-26 09:30",   // 清单声明的更新时间（北京时间）
    "syncedAt": "2026-11-26T02:40:31.000Z",  // 每日管线运行时间，无条件刷新（客户端据此判定「今日包」）
    "fetchedAt": "2026-11-26T02:41:00.000Z", // 内容最后变更时的抓取时间，内容不变则保留旧值
    "conversionRule": "valueCny 取自「能领多少」中的 ≈¥N / ¥N……",
    "etag": "\"…\"",
    "totalItems": 64
  },
  "sections": [
    {
      "key": "daily",          // daily / monthly / other / secN
      "name": "每天都能领",
      "declaredCount": 16,
      "items": [
        {
          "id": "a1b2c3d4e5",  // 标题 md5 前 10 位，跨天稳定可去重
          "title": "…",
          "provider": "…",     // 提供方
          "amountText": "…",   // 能领多少（原文）
          "amountNote": "…",   // 能领多少括号内注记
          "valueCny": 10,      // ≈¥N 折算，"不折钱" → null
          "howTo": "…",        // 怎么领
          "scope": "…",        // 能用在
          "tags": ["门槛1"],   // 门槛按「、」切分
          "validity": "…",     // 有效期
          "deadline": "…",     // 截止（可选）
          "verifiedAt": "2026-11-26",
          "verifiedSource": "…",
          "claimUrl": "https://…",
          "logo": "https://cdn.jsdelivr.net/…/logos/<host>.png"
          // 渠道 favicon（PNG）。按 claimUrl host 抓取，全部失败 → null（小程序端显示提供方首字）
        }
      ]
    }
  ]
}
```

`data/raw/` 为原文快照，仅本地排查用，不入库。
