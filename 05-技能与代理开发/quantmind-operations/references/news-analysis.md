# 新闻驱动分析参考

## RSS/新闻数据流

```
RSSHub / myrss 订阅源
      ↓ (RSS 抓取)
Huntly (lcomplete/huntly) — 存储 + 阅读器
      ↓ (API 代理)
quantmind 容器 /api/v1/news/* — 富化(情感/事件/行业) + 对外 API
      ↓
前端新闻面板 / QuantBot 分析
```

- **Huntly** 容器：`quantmind-huntly`，端口 `8090`
- **新闻源管理**：`/api/v1/news/sources`（CRUD）+ `/admin/sources`（管理端）
- **RSSHub** 容器：`quantmind-rsshub`，提供通用网站订阅

## 情感/事件富化

`/api/v1/news/enrichment/stats` 显示富化覆盖情况。
`/api/v1/news/enrichment/run` 触发增量富化（对未处理文章做 NLP）。

富化后的字段：`sentiment`（bullish/bearish/neutral）、`event_tags`（事件标签）、
`tickers`（关联股票）、`industries`（行业）。

## 分析示例

### 分析某只股票近期新闻情绪
```bash
curl -s -H "$AUTH" "$BASE/api/v1/news/articles" -G \
  --data-urlencode "tickers=600519.SH" \
  --data-urlencode "sort=sentiment_bullish" \
  --data-urlencode "since=$(date -d '-7 days' +%Y-%m-%dT00:00:00Z)" \
  --data-urlencode "page=1"
```
按利好强度排序返回 7 天内关于该股的文章。

### 抓取某行业强信号新闻
```bash
curl -s -H "$AUTH" "$BASE/api/v1/news/articles" -G \
  --data-urlencode "industries=半导体" \
  --data-urlencode "strong_only=true" \
  --data-urlencode "sort=time_desc" \
  --data-urlencode "page=1"
```

### 事件驱动（财报季）
```bash
curl -s -H "$AUTH" "$BASE/api/v1/news/articles" -G \
  --data-urlencode "event_tags=财报,业绩预增" \
  --data-urlencode "date_entities=2026-Q3" \
  --data-urlencode "sentiment=bullish" \
  --data-urlencode "page=1"
```

## 与模型联动

新闻分析结果可与模型推理交叉验证：
1. 新闻强利好（sentiment=bullish, strong_only）+ 模型高分 → 高置信买入候选
2. 新闻利空 + 模型分数走低 → 规避
3. 行业新闻热度高 + 行业因子（ind_strength）走强 → 板块机会

## 常见问题

- **sources 为空**：需要先在 Huntly 网页（8090 端口）或管理端添加订阅源
- **富化未跑**：调用 `/news/enrichment/run` 触发
- **ticker 格式**：用 suffix 格式（`600519.SH`），与全站一致
