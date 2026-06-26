---
name: weibo-crawler
description: Use when the user wants to scrape a Weibo (微博) user's posts by UID/URL and export to Excel — covers recon, paginated fetching, the API's hidden depth limit, dedup, and export. Captures hard-won pitfalls from a prior manual run, not theoretical.
---

# 微博爬虫作业指导书

## 任务流程

```
用户提供 UID/URL → 侦测（昵称/total/翻页数/时间范围）→ 分批抓取 → 合并去重 → 导出 Excel
```

## 1. 侦测阶段

目标：拿到昵称、API 返回的 total、翻页数、时间范围。

做法：访问 `weibo.com/u/{UID}`，从 profile 拿昵称和 statuses_count；调用 `statuses/mymblog` API（cardlist 接口），看 total 和分页数。

注意：**API 返回的 total 经常远大于实际可抓到的数量**（账号已删/隐藏的微博也算在 total 里），翻页时遇到 `ok=0` 或 cards 为空，就是真正的边界，以此为准，不要信 total。

## 2. 抓取阶段

- **每批抓 10 页、count=10**：这个组合最稳定。count=20 时 API 页数上限会不成比例地缩水。
- **API 硬性深度限制**：count=10 时最多约能翻到 page 47（约 460 条）；count=20 时最多约 page 24。超出这个范围，深层页码的 API 不再返回数据。账号体量很大（比如 7000+ 条）时，只能从第 1 页顺序往后抓到翻页边界为止，没有跳页的捷径。
- **深层页面响应变慢是正常现象**：越往后翻页越慢，抓取速度会从每批 100 条降到 90+、甚至 30+，这是 API 行为本身导致的，不是抓取脚本出问题。
- **需要登录态/Cookie**：用真实浏览器登录后的 Cookie 去调 API，未登录或 Cookie 过期会返回 `ok:-100`。

## 3. 合并导出阶段

- 把所有分批抓到的原始 JSON 合并，按 `mid` 字段去重，按发布时间排序。
- 导出 Excel，建议字段：序号、mid、发布时间、微博正文、来源、转发数、评论数、点赞数。表头加粗、首行冻结、自动筛选、自适应列宽。

## 常见踩坑对照表

| 坑 | 现象 | 应对 |
|---|---|---|
| Cookie 过期 | API 返回 `ok:-100` | 重新登录获取最新 Cookie |
| API 深层限制 | page 47+（count=10）之后翻页返回空 | 这是硬限制，无解，只能从第 1 页顺序抓到边界为止 |
| total 不可信 | 翻页早早碰到空值，远低于宣称的 total | 以 `ok=0`/空 cards 为真实边界，不要按 total 估算进度 |
| 深层响应变慢 | 越往后翻页越慢，单批条数下降 | 正常现象，不是抓取出错，耐心等待或适当降低单批页数 |

## 执行检查清单

- [ ] Cookie 是否有效（未过期登录）
- [ ] 每批是否固定用 10 页、count=10
- [ ] 翻页是否推进到真正边界（ok=0 / 空 cards），而非按 total 估算
- [ ] 合并时是否按 mid 去重
- [ ] 导出 Excel 是否包含表头样式/冻结/筛选
