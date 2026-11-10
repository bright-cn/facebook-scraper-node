# 供在此代码仓库中工作的编码智能体使用

在更改任何内容之前请先阅读本文。以下每一行均已于 2026-11-03 对照实时 API、`@brightdata/sdk` 1.2.0 或 Bright Data CLI 0.4.1 验证。如果 API 参考页面与实时 API 不一致，本文以实时 API 为准。

## 身份验证

- `token()` 会读取 `BRIGHTDATA_API_TOKEN`，然后读取存储在 `brightdata-cli/credentials.json` 中的密钥 `bdata login`（SDK 也读取同一文件）。不支持 `.env`；Node 20 可通过 `node --env-file=.env` 加载该文件。设置此变量或登录一次，是必须由人工完成的唯一一步；切勿将密钥粘贴到代码中。
- README 中的原始 HTTP 快速入门仅读取 `BRIGHTDATA_API_TOKEN`。它无法读取 CLI 登录信息。

## API 的工作方式

- 此软件包使用内置的 `fetch` 调用 `https://api.brightdata.com/datasets/v3`，且没有任何依赖项。请保持这一点。测试通过 `transport` 模拟网络。JavaScript SDK `@brightdata/sdk` 1.2.0 封装了 `client.scrape.facebook` 中 11 个 Facebook 爬虫工具里的 9 个，但不包括页面详情和广告资料库。Python SDK 的 `comments()` 会发送 `num_of_comments`，API 会以 400 验证错误拒绝该请求。
- 共有 11 个数据集，全部按 URL 采集；其中 3 个还支持发现：
  - 页面帖子 `gd_lkaxegm826bjpoo9m5`、群组帖子 `gd_lz11l67o2cb3r0lkj3`、按 URL 获取帖子 `gd_lyclm1571iy3mv57zw`、Reels `gd_lyclm3ey2q6rww027t`：使用 `num_of_posts` 限制数量。
  - 评论 `gd_lkay758p1eanlolqw8`：使用 `limit_records` 限制数量。
  - 页面 `gd_mf124a0511bauquyow`、个人资料 `gd_mf0urb782734ik94dz`。
  - 公司评价 `gd_m0dtqpiu1mbcyc2g86`：需要一个 `/reviews/` URL，使用 `num_of_reviews` 限制数量。
  - Marketplace `gd_lvt9iwuh6fbcwmx1a`：按 `keyword` 发现（必须提供 `keyword` 和 `city`），或按 `url`（分类 URL）发现。
  - 活动 `gd_m14sd0to1jz48ppm51`：按 `url`（活动搜索 URL）或按 `venue`（场地页面的 `/events` URL；仅提供页面 URL 会返回“There are no events.”）发现。
  - Meta 广告资料库 `gd_mtveskb62h7q65ftgz`：按 `keyword` 发现，必须提供 `country` 和 `ad_type`。接受 `ad_type` `"All ads"`；拒绝 `"all"`。
  - `bdata pipelines info <dataset_id> <method>` 会输出实时输入架构。
- 每次调用都是异步作业：`/trigger`、轮询 `/progress/<id>`，然后执行 `/snapshot/<id>?format=json`。运行耗时从 12 秒（评价、单条广告）到 234 秒（一个 Marketplace 分类）不等。`TIMEOUT_MS` 为 900000。
- 始终发送 `include_errors=true`。API 默认会丢弃错误行，因此失败的输入看起来会像空结果。
- `limit_per_input` 会限制发现数量和费用。达到限制时，仍在运行的爬虫会以包含错误“Crawl aborted on job cancel”的行返回。`split()` 不会将这些行计为失败。
- 许多品牌页面关闭了评价：Coca-Cola 返回“Reviews not found”，Starbucks 和 Shake Shack 返回失效页面。`TheGarden/reviews/` 可用。
- 活动搜索的匹配较宽松：“tech conference”返回了三个活动，但没有一个是会议。
- 因网络错误（`fetch failed`）而失败的状态轮询会持续重试，直至截止时间。HTTP 错误仍会导致拒绝。
- 架构可能随时更改，恕不另行通知。切勿硬编码字段列表。
- 完整文档索引，每个条目对应一个 `.md` 页面：https://docs.brightdata.com/llms.txt

## 在此代码仓库中工作

- `npm test` 可离线运行且不需要令牌。`npm run lint` 必须通过。
- CI 会在空白机器上执行 README 自身的命令。每周工作流会针对真实 API 执行 README 中的每个围栏代码块，每日工作流则运行较小的实时检查。每个代码片段都必须完整且可单独粘贴运行，所展示的输出也必须是真实结果。
- 群组帖子、评论和评价会包含普通人的姓名。README 代码片段仅输出数量、日期和页面名称，不输出谁写了什么。
- README 中的字段表位于 `<!-- fields:start -->` 和 `<!-- fields:end -->` 之间，由 `.github/scripts/field_tables.mjs` 每日根据 `scrapers-full.json` 重新生成。请勿手动编辑。
- README 顶部的“last verified”徽章行由每日运行重写。请勿手动编辑。
- 保持精简。除状态轮询外，请勿添加重试、去重、调度、数据库、异步示例或并发。
