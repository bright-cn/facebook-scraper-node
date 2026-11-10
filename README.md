# 适用于 Node.js 和 JavaScript 的 Facebook 爬虫工具

使用 Node.js 和 JavaScript 将 Facebook 帖子、评论、Reels、Marketplace、活动和 Meta 广告资料库抓取为 JSON。无需登录，无需 Graph API。基于 [Bright Data Facebook 爬虫工具 API](https://www.bright.cn/products/web-scraper/facebook?utm_source=github&utm_medium=referral&utm_content=facebook-scraper-node) 构建。新账户每月可获得 5,000 点免费额度。

[![实时检查](https://github.com/bright-cn/facebook-scraper-node/actions/workflows/live.yml/badge.svg)](https://github.com/bright-cn/facebook-scraper-node/actions/workflows/live.yml) [![最后验证时间](https://img.shields.io/badge/last%20verified-9%20Nov%202026-brightgreen)](https://github.com/bright-cn/facebook-scraper-node/actions/workflows/live.yml) <!-- verified: rewritten by the daily run -->

[快速入门](#如何使用-nodejs-抓取-facebook-帖子) · [无需登录](#无需登录或-graph-api-即可抓取-facebook) · [CLI](#批量-facebook-页面帖子-cli) · [端点](#抓取-facebook-帖子评论marketplace活动和广告) · [数据](#facebook-帖子数据字段json) · [错误](#facebook-爬虫工具错误) · [MCP 和 Claude Code](#facebook-mcp-服务器和-claude-code-技能) · [免费额度](#这个-facebook-爬虫工具免费吗) · [文档](https://docs.brightdata.com/products/scrapers/facebook/introduction) · [支持](#支持)

在 JavaScript 中以 JSON 获取 Facebook 数据：页面和群组帖子、评论、Reels、页面详情、个人资料、评价、Marketplace 商品、活动以及 Meta 广告资料库。涵盖全部 11 个 Bright Data Facebook 爬虫工具，每种功能对应一个函数。无需 Facebook 登录，无需浏览器，没有依赖项。基于 [Bright Data Facebook 爬虫工具 API](https://www.bright.cn/products/web-scraper/facebook?utm_source=github&utm_medium=referral&utm_content=facebook-scraper-node) 构建。

此外还提供：用于页面帖子的单命令 CLI，以及完全不需要 JavaScript 的 [Bright Data CLI](#facebook-mcp-服务器和-claude-code-技能)。

## 如何使用 Node.js 抓取 Facebook 帖子

需要 Node 20 或更高版本，无需安装任何内容：这里使用内置的 `fetch`。由于使用了顶层 `await`，请将其保存为 `.mjs` 文件。

```bash
export BRIGHTDATA_API_TOKEN=YOUR_API_KEY
```

从 [Bright Data 控制面板](https://www.bright.cn/cp/setting/users)获取令牌。还没有账户？[创建账户](https://www.bright.cn/cp/start)；新账户每月可获得[5,000 点免费额度](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。

```javascript
const API = "https://api.brightdata.com/datasets/v3";
const HEADERS = {
  Authorization: `Bearer ${process.env.BRIGHTDATA_API_TOKEN}`,
  "Content-Type": "application/json",
};

async function call(path, body) {
  const init = body === undefined ? { headers: HEADERS } : { method: "POST", headers: HEADERS, body: JSON.stringify(body) };
  const response = await fetch(API + path, init);
  if (!response.ok) throw new Error(`HTTP ${response.status}: ${await response.text()}`);
  return response.json();
}

const page = { url: "https://www.facebook.com/Meta", num_of_posts: 3 };
const job = await call("/trigger?dataset_id=gd_lkaxegm826bjpoo9m5&include_errors=true", [page]);
while (!["ready", "failed"].includes((await call(`/progress/${job.snapshot_id}`)).status)) {
  await new Promise((done) => setTimeout(done, 10_000));
}
for (const post of await call(`/snapshot/${job.snapshot_id}?format=json`)) {
  if (post.error) throw new Error(post.error);
  console.log(post.likes, post.num_comments, post.url);
}
```

```
725 343 https://www.facebook.com/Meta/posts/pfbid0evGHWPACd3JninWg3j3nucQjdFxoHWwJSPgFXKjZXaUiU8MctREDPRZZsAps6CGwl
4169 717 https://www.facebook.com/reel/2300984144015832/
7679 819 https://www.facebook.com/reel/29074877795478111/
```

预计耗时一到三分钟：API 会运行作业，循环会等待作业完成。按[每条帖子](https://www.bright.cn/pricing/web-scraper)计费。没有令牌时，第一个请求会失败并返回 `HTTP 401`。

该代码片段中的两项设置并非可选。

`num_of_posts` 会限制运行数量和费用。如果没有它，页面会返回抓取工具能访问到的所有帖子，并按全部帖子计费。

`include_errors=true` 会让 API 将失效或私密页面报告为包含 `error` 键的行。API 默认关闭此功能，因此失败页面看起来会与没有帖子的页面相同。

Bright Data [JavaScript SDK](https://github.com/bright-cn/bright-data-sdk-js) 封装了 11 个 Facebook 爬虫中的 9 个，但不包括页面详情和广告资料库。因此，这里和本软件包都使用原生 HTTP。

## 无需登录或 Graph API 即可抓取 Facebook

Facebook 爬虫工具 API 不需要 Facebook 账户、Facebook 登录、Meta 应用或 Graph API 访问令牌。唯一凭证是 Bright Data API 密钥，通过 `BRIGHTDATA_API_TOKEN` 设置。本软件包及其命令还可以找到由 `bdata login` 存储的密钥。它会采集未登录访客可见的内容：私密群组或个人资料会返回错误行。

## 批量 Facebook 页面帖子 CLI

此代码仓库中的命令会同时为多个页面运行 Node.js 快速入门，并写入一个 JSON 文件。

```bash
npm install -g github:bright-cn/facebook-scraper-node
facebook-scraper Meta Microsoft --limit 2
```

```
Fetching up to 2 recent posts per page, for: Meta, Microsoft
Usually one to three minutes each. Billed per post, 5,000 free credits a month.
asking  Meta...
got     Meta: 2 posts
asking  Microsoft...
got     Microsoft: 2 posts

Saved 4 posts as JSON to facebook.json (44 fields per post)
```

在终端中，`asking` 行会被原地更新的加载动画和运行时钟替代，因此你可以看到它正在运行以及已经运行了多长时间。

```
--limit N      posts per page, default 5, minimum 1
--out PATH     output file, default facebook.json
```

页面可以是名称、`@Meta`、`facebook.com/Meta` 或完整 URL。该命令会在 `BRIGHTDATA_API_TOKEN` 中查找令牌；如果你运行过一次 `npx -p @brightdata/cli bdata login`，它也会从已存储的登录信息中查找，供你及该终端中的任何编码智能体使用。它不会读取 `.env` 文件；Node 会通过 `node --env-file=.env` 为其加载该文件。

你也可以导入它而不是直接运行，以便按页面获取 `ok`、`note` 和 `error`，而非原始行。本软件包使用 ESM，因此请使用 `import`，不要使用 `require`。`scrape` 不会因单个错误页面而拒绝；读取 `posts` 之前请检查 `ok`：

```javascript
import { scrape } from "@brightdata/facebook-scraper-node";

for (const outcome of await scrape(["Meta", "zzqqxx-no-such-page-404"], { limit: 1 })) {
  if (outcome.ok) {
    console.log(`${outcome.page}: ${outcome.posts.length} posts`);
  } else {
    console.log(`${outcome.page} failed: ${outcome.error}`);
  }
}
```

```
Meta: 1 posts
zzqqxx-no-such-page-404 failed: Posts not found
```

不存在的页面会作为错误行“Posts not found”返回，而非零条帖子。该次运行依次处理两个页面，共耗时 209 秒。当一次运行返回部分帖子和部分错误行时，帖子会被保留，错误则会写入 `note`。

## 抓取 Facebook 帖子、评论、Marketplace、活动和广告

Bright Data 提供 11 个 Facebook 爬虫工具，每个工具都记录在 [Facebook 爬虫工具 API 文档](https://docs.brightdata.com/products/scrapers/facebook/introduction)中。本软件包针对每种调用方式提供一个函数。以下每个代码片段都完整可用：在上面的 `npm install` 之后按原样粘贴即可。每个片段都会在每周一的 Actions 中运行，每隔一天还会执行较小的检查，顶部徽章显示最新结果。

| 你已有 | 你想要 | 方法 |
| --- | --- | --- |
| 页面 URL | 其近期帖子 | `pagePosts(urls, { limit: 5 })` |
| 公开群组 URL | 其近期帖子 | `groupPosts(urls, { limit: 5 })` |
| 帖子 URL | 这些帖子 | `posts(urls)` |
| 帖子 URL | 其评论 | `comments(urls, { limit: 10 })`，按每条评论计费 |
| 页面 URL | 其近期 Reels | `reels(urls, { limit: 5 })` |
| 页面 URL | 页面详情：关注者、类别、联系信息 | `pages(urls)` |
| 个人资料 URL | 公开个人资料详情 | `profiles(urls)` |
| 页面的 `/reviews/` URL | 其评价 | `reviews(urls, { limit: 10 })` |
| 关键词和城市 | Marketplace 商品 | `marketplaceSearch("bicycle", "New York", { limit: 5 })` |
| Marketplace 分类 URL | 其商品 | `marketplaceCategory(url, { limit: 5 })` |
| Marketplace 商品 URL | 这些商品 | `marketplace(urls)` |
| 场地页面的 `/events` URL | 其即将举行的活动 | `eventsByVenue(url, { limit: 5 })` |
| 活动搜索 URL | 匹配的活动 | `eventsSearch(url, { limit: 5 })` |
| 活动 URL | 这些活动 | `events(urls)` |
| 关键词和国家/地区 | Meta 广告资料库中的广告 | `adsSearch("coffee", { country: "US", limit: 5 })` |
| 广告资料库广告 URL | 这些广告 | `ads(urls)` |

每个函数都接受一个 URL 或 URL 数组。一个数组对应一个作业，而不是每个 URL 一个作业。每个函数都会解析为行数组；API 无法采集的行会携带 `error` 键，而非记录。接受数量限制的函数还会通过同一个选项对象传递 API 接受的其他输入，并使用 API 中的字段名称：`pagePosts(url, { limit: 10, start_date: "10-01-2026" })`。

### 在一个作业中抓取多个页面的 Facebook 帖子

```javascript
import { pagePosts } from "@brightdata/facebook-scraper-node";

const urls = ["https://www.facebook.com/Meta", "https://www.facebook.com/Microsoft"];
for (const post of await pagePosts(urls, { limit: 1 })) {
  console.log(post.page_name, post.date_posted, post.likes, post.url);
}
```

```
Microsoft 2025-04-04T22:18:26.000Z 40838 https://www.facebook.com/Microsoft/posts/pfbid0347J66TDSqDpBtUS5Vmhu17t7bpCWTRzgi9yWw1387gGCJuwhAdKfsLzByvA93uRcl
Meta 2026-10-30T19:41:31.000Z 725 https://www.facebook.com/Meta/posts/pfbid0evGHWPACd3JninWg3j3nucQjdFxoHWwJSPgFXKjZXaUiU8MctREDPRZZsAps6CGwl
```

### 抓取 Facebook 群组帖子

仅限公开群组。记录中会包含发帖人的姓名；此示例仅输出群组和数量。

```javascript
import { groupPosts } from "@brightdata/facebook-scraper-node";

for (const post of await groupPosts("https://www.facebook.com/groups/532855527252024", { limit: 2 })) {
  console.log(post.group_name.slice(0, 40), post.group_members, post.date_posted, post.num_comments);
}
```

```
Jobs, Work and Vacancies in Cape Town an 238700 2026-11-03T18:53:25.000Z 0
Jobs, Work and Vacancies in Cape Town an 238700 2026-11-03T18:48:29.000Z 0
```

### 按 URL 抓取 Facebook 帖子

```javascript
import { posts } from "@brightdata/facebook-scraper-node";

const url = "https://www.facebook.com/Meta/posts/pfbid0evGHWPACd3JninWg3j3nucQjdFxoHWwJSPgFXKjZXaUiU8MctREDPRZZsAps6CGwl";
const [post] = await posts(url);
console.log(post.user_username_raw, post.likes, post.num_comments, post.num_shares);
console.log(post.content.slice(0, 70));
```

```
Meta 595 343 194
Meta Ray-Ban Display has been out for a while, so if you’re a develope
```

### 抓取 Facebook 帖子中的评论

按每条评论计费，因此请先检查帖子上的 `num_comments`。限制值会作为 `limit_records` 传递给 API。

```javascript
import { comments } from "@brightdata/facebook-scraper-node";

const url = "https://www.facebook.com/Meta/posts/pfbid0evGHWPACd3JninWg3j3nucQjdFxoHWwJSPgFXKjZXaUiU8MctREDPRZZsAps6CGwl";
const rows = await comments(url, { limit: 3 });
console.log(rows.length, "comments; first:", rows[0].date_created, rows[0].num_likes, "likes");
```

```
3 comments; first: 2026-10-31T01:48:29.000Z 10 likes
```

### 抓取 Facebook 页面的 Reels

```javascript
import { reels } from "@brightdata/facebook-scraper-node";

for (const reel of await reels("https://www.facebook.com/Meta", { limit: 2 })) {
  console.log(reel.date_posted, reel.video_view_count, "views", reel.url);
}
```

```
2026-10-27T16:44:32.000Z 447000 views https://www.facebook.com/reel/2300984144015832
2026-10-22T16:25:44.000Z 1200000 views https://www.facebook.com/reel/29074877795478111
```

### 抓取 Facebook 页面数据

```javascript
import { pages } from "@brightdata/facebook-scraper-node";

const [page] = await pages("https://www.facebook.com/Meta");
console.log(page.page_name, page.followers, "followers,", page.primary_category);
```

```
Meta 106000000 followers, Company
```

### 抓取 Facebook 个人资料

仅抓取公开个人资料详情，即未登录访客可见的内容。

```javascript
import { profiles } from "@brightdata/facebook-scraper-node";

const [profile] = await profiles("https://www.facebook.com/zuck");
console.log(profile.name, "|", profile.city);
```

```
Mark Zuckerberg | Palo Alto
```

### 抓取企业 Facebook 页面的评价

传入页面的 `/reviews/` URL。许多品牌页面关闭了评价，因而会返回错误行“Reviews not found”或失效页面，而不是零行。此示例输出评价结论，不输出评价者。

```javascript
import { reviews } from "@brightdata/facebook-scraper-node";

for (const review of await reviews("https://www.facebook.com/TheGarden/reviews/", { limit: 3 })) {
  console.log(review.company_name, review.review_time, "recommends:", review.recommends);
}
```

```
Madison Square Garden 2026-10-31T14:43:54.000Z recommends: true
Madison Square Garden 2026-10-21T12:28:03.000Z recommends: true
Madison Square Garden 2026-10-19T22:29:17.000Z recommends: false
```

### 抓取 Facebook Marketplace 商品

```javascript
import { marketplaceSearch } from "@brightdata/facebook-scraper-node";

for (const item of await marketplaceSearch("bicycle", "New York", { limit: 3 })) {
  if (!item.error) {
    // rows the limit cancelled carry only an error
    console.log(item.final_price, item.currency, item.title.slice(0, 40), "|", item.location);
  }
}
```

```
30 USD Bike | Hackensack, NJ
40 USD Huffy Nel Lusso Cruiser Bike | Long Beach, NY
120 USD fat tire bicycle | Uniondale, NY
```

`marketplaceCategory(url, { limit: 5 })` 可从分类 URL（例如 `https://www.facebook.com/marketplace/category/vehicles`）执行相同操作，`marketplace(urls)` 则会采集你已有 URL 的商品。

### 抓取场地的 Facebook 活动

传入场地页面的 `/events` URL。仅提供页面 URL 无法找到活动。

```javascript
import { eventsByVenue } from "@brightdata/facebook-scraper-node";

for (const event of await eventsByVenue("https://www.facebook.com/TheGarden/events", { limit: 3 })) {
  console.log(event.event_date, event.title);
}
```

```
2026-11-04T00:00:00.000Z NY Rangers vs St. Louis Blues Premium Seating
2026-11-05T00:30:00.000Z New York Knicks vs. Chicago Bulls
2026-11-04T00:00:00.000Z New York Rangers vs. St. Louis Blues
```

`eventsSearch(url, { limit: 5 })` 接受活动搜索 URL，例如 `https://www.facebook.com/events/search/?q=marathon`。其匹配较宽松：请检查返回的标题。

### 抓取 Facebook 广告资料库

```javascript
import { adsSearch } from "@brightdata/facebook-scraper-node";

for (const ad of await adsSearch("coffee", { country: "US", limit: 3 })) {
  if (!ad.error) {
    // rows the limit cancelled carry only an error
    console.log(ad.advertiser_name, "| active:", ad.is_active, "| since", ad.start_date);
  }
}
```

```
Dame Fine Coffee | active: true | since 2026-06-11T07:00:00.000Z
Biggby Coffee of Wickliffe Ohio | active: true | since 2025-04-29T07:00:00.000Z
Wall Stadium | active: true | since 2026-04-22T07:00:00.000Z
```

`ad_type` 默认为 `"All ads"`，这是 API 验证通过的值；`"all"` 会被拒绝。发现达到限制时，会取消仍在运行的爬虫，每个爬虫都会作为错误为“Crawl aborted on job cancel”的行返回。`split()` 会将这些行视为限制已生效，而非失败。

### 立即触发，稍后获取

对于超过几个页面的任务，不要让进程持续等待一小时。触发任务、保存快照 ID，并在就绪后获取。快照可下载 30 天。

```javascript
import { PAGES, fetchSnapshot, status, trigger } from "@brightdata/facebook-scraper-node";

const snapshot = await trigger(PAGES, [{ url: "https://www.facebook.com/Microsoft" }]);
console.log("snapshot:", snapshot);
let state;
while (!["ready", "failed"].includes((state = await status(snapshot)))) {
  await new Promise((done) => setTimeout(done, 10_000));
}
console.log("status:", state);
console.log("fetched:", (await fetchSnapshot(snapshot))[0].page_name);
```

```
snapshot: sd_mw133n4lv23hvuqep
status: ready
fetched: Microsoft
```

`trigger` 接受任意数据集、`discoverBy` 和 `limitPerInput`，因此可访问表中的每个爬虫工具。每项输入均已于 2026-11-03 通过实时 API 验证，其中多项与 API 参考页面不同：

| 爬虫工具 | dataset_id | discover_by | 输入 |
| --- | --- | --- | --- |
| 页面帖子 | `gd_lkaxegm826bjpoo9m5` | 无 | `{"url", "num_of_posts"}` |
| 群组帖子 | `gd_lz11l67o2cb3r0lkj3` | 无 | `{"url", "num_of_posts"}` |
| 按 URL 获取帖子 | `gd_lyclm1571iy3mv57zw` | 无 | `{"url"}` |
| 评论 | `gd_lkay758p1eanlolqw8` | 无 | `{"url", "limit_records"}`。`num_of_comments` 会被拒绝 |
| Reels | `gd_lyclm3ey2q6rww027t` | 无 | `{"url", "num_of_posts"}` |
| 页面 | `gd_mf124a0511bauquyow` | 无 | `{"url"}` |
| 个人资料 | `gd_mf0urb782734ik94dz` | 无 | `{"url"}` |
| 公司评价 | `gd_m0dtqpiu1mbcyc2g86` | 无 | `{"url", "num_of_reviews"}`，一个 `/reviews/` URL |
| Marketplace 商品 | `gd_lvt9iwuh6fbcwmx1a` | 无 | `{"url"}` |
| Marketplace 搜索 | `gd_lvt9iwuh6fbcwmx1a` | `keyword` | 必须提供 `{"keyword", "city"}` |
| Marketplace 分类 | `gd_lvt9iwuh6fbcwmx1a` | `url` | `{"url"}` |
| 活动 | `gd_m14sd0to1jz48ppm51` | 无 | `{"url"}` |
| 活动搜索 | `gd_m14sd0to1jz48ppm51` | `url` | `{"url"}`，一个 `/events/search/?q=` URL |
| 场地活动 | `gd_m14sd0to1jz48ppm51` | `venue` | `{"url", "upcoming_events_only"}`，一个 `/events` URL |
| 广告 | `gd_mtveskb62h7q65ftgz` | 无 | `{"url"}` |
| 广告搜索 | `gd_mtveskb62h7q65ftgz` | `keyword` | `{"keyword", "country", "ad_type"}`, `ad_type` `"All ads"` |

`bdata pipelines info <dataset_id> <method>` 会输出 API 验证的输入字段。

## Facebook 帖子数据字段（JSON）

多数人需要的页面帖子字段：

```
url  date_posted  content  likes  num_comments  num_shares  page_name  page_followers
```

代码不会硬编码任何字段列表。API 返回的所有内容都会写入每一行以及命令生成的文件中。

<!-- fields:start -->
每天根据已发布的架构 [scrapers-full.json](https://docs.brightdata.com/scrapers-full.json) 重新生成，因此不会过时。记录包含适用于它的字段，以及 `timestamp` 和 `input`；示例文件的第一条帖子有 44 个键。

<details>
<summary>全部 52 个页面帖子字段，<code>gd_lkaxegm826bjpoo9m5</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `url` | url | Facebook 帖子 URL |
| `post_id` | text | 每条 Facebook 帖子的唯一标识符 |
| `user_url` | url | 每条 Facebook 帖子的唯一标识符 |
| `user_username_raw` | text | 发布内容的用户的用户名 |
| `content` | text | 帖子的文本或媒体内容 |
| `date_posted` | date | 帖子发布日期和时间 |
| `hashtags` | array | 帖子中使用的标签 |
| `num_comments` | number | 帖子的评论数 |
| `num_shares` | number | 帖子的分享数 |
| `num_likes_type` | object | 收到的点赞类型：数据点“count_reactions_type”包含收到的所有回应。 |
| `page_name` | text | 发布帖子的 Facebook 页面名称 |
| `profile_id` | text | 用户个人资料的唯一标识符 |
| `page_intro` | text | Facebook 页面的简介或描述 |
| `page_category` | text | Facebook 页面的类别 |
| `page_logo` | image | Facebook 页面的徽标 |
| `page_external_website` | text | 与 Facebook 页面关联的外部网站 |
| `page_likes` | number | Facebook 页面的点赞数 |
| `page_followers` | number | Facebook 页面的关注者数量 |
| `page_is_verified` | boolean | Facebook 页面的验证状态 |
| `original_post` | object | 指示帖子是原创内容还是转发内容 |
| `attachments` | array | 帖子中包含的媒体附件 |
| `other_posts_url` | array | 同一用户其他帖子的网页地址 |
| `post_external_link` | url | 帖子中包含的外部链接 |
| `post_external_title` | text | 外部链接的标题 |
| `post_external_image` | url | 帖子外部链接的图片 |
| `page_url` | url | Facebook 页面的网页地址 |
| `header_image` | image | Facebook 页面的页眉图片 |
| `avatar_image_url` | text | Facebook 页面头像图片的 URL |
| `profile_handle` | text | Facebook 页面的短名或用户名 |
| `has_handshake` | text | 指示帖子是否包含握手标记 |
| `is_sponsored` | boolean | 指示帖子是否为赞助内容 |
| `sponsor_name` | text | 如果帖子为赞助内容，则为赞助商名称 |
| `shortcode` | text | 帖子 URL 的短代码 |
| `video_view_count` | number | 如果帖子包含视频，则为视频观看次数 |
| `likes` | number | 帖子的点赞数 |
| `post_image` | image | 帖子图片 URL（如果存在） |
| `post_type` | text | 帖子类型 |
| `following` | number | 关注的个人资料数量 |
| `link_description_text` | text | 链接标题下方显示的文本 |
| `count_reactions_type` | array | 收到的回应类型 |
| `is_page` | boolean | 帖子来自页面还是个人资料 |
| `page_phone` | text | 页面上列出的公开电话号码 |
| `page_email` | text | 页面上列出的公开电子邮件地址 |
| `page_creation_time` | date | 页面创建日期 |
| `page_reviews_score` | text | 页面平均评价分数 |
| `page_reviewers_amount` | number | 页面评价者总数 |
| `page_price_range` | text | 页面上显示的价格范围标签 |
| `about` | array | 从页面“简介”部分提取的键值对 |
| `active_ads_urls` | array | 指向页面在 Facebook 广告资料库中投放中广告的链接 |
| `delegate_page_id` | text | 显示为对此页面负责的页面 ID |
| `privacy_and_legal_info` | text | 个人资料的隐私和法律信息 |
| `play_count` | number | 表示视频开始播放（或被点击播放）的次数 |

</details>

<details>
<summary>全部 44 个群组帖子字段，<code>gd_lz11l67o2cb3r0lkj3</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `url` | url |  |
| `post_id` | text | 每条 Facebook 帖子的唯一标识符 |
| `user_url` | url | 每条 Facebook 帖子的唯一标识符 |
| `user_username_raw` | text | 发布内容的用户的用户名 |
| `content` | text | 帖子的文本或媒体内容 |
| `date_posted` | date | 帖子发布日期和时间 |
| `hashtags` | array | 帖子中使用的标签 |
| `num_comments` | number | 帖子的评论数 |
| `num_shares` | number | 帖子的分享数 |
| `num_likes_type` | object | 收到的点赞类型（例如回应、点赞） |
| `group_name` | text | 发布帖子的 Facebook 群组名称 |
| `group_id` | text | 用户个人资料的唯一标识符 |
| `group_url` | url | Facebook 群组的网页地址 |
| `group_intro` | text | Facebook 群组的简介或描述 |
| `group_category` | text | Facebook 群组的类别 |
| `group_logo` | image | Facebook 群组的徽标 |
| `group_external_website` | text | 与 Facebook 群组关联的外部网站 |
| `group_members` | number | Facebook 群组的成员数 |
| `group_created_at` | date | 群组创建日期 |
| `user_is_verified` | boolean | 发帖 Facebook 用户的验证状态 |
| `original_post` | object | 指示帖子是原创内容还是转发内容 |
| `attachments` | array |  |
| `other_posts_url` | array | 同一用户其他帖子的网页地址 |
| `post_external_link` | url | 帖子中包含的外部链接 |
| `post_external_title` | text | 外部链接的标题 |
| `post_external_image` | url | 帖子外部链接的图片 |
| `header_image` | image | Facebook 页面的页眉图片 |
| `avatar_image_url` | text | Facebook 页面头像图片的 URL |
| `profile_handle` | text | Facebook 页面的短名或用户名 |
| `has_handshake` | text | 指示帖子是否包含握手标记 |
| `is_sponsored` | boolean | 指示帖子是否为赞助内容 |
| `sponsor_name` | text | 如果帖子为赞助内容，则为赞助商名称 |
| `shortcode` | text | 帖子 URL 的短代码 |
| `video_view_count` | number | 如果帖子包含视频，则为视频观看次数 |
| `likes` | number | 帖子的点赞数 |
| `post_image` | image | 帖子图片 URL（如果存在） |
| `post_type` | text | 帖子类型 |
| `following` | number | 关注的个人资料数量 |
| `price` | number | 与帖子关联的价格 |
| `location` | text | 与帖子关联的位置 |
| `profile_id` | text |  |
| `publisher_image_url` | url |  |
| `num_reaction_type` | array | 收到的点赞类型（例如回应、点赞） |
| `delegate_page_id` | text | delegate_page_id |

</details>

<details>
<summary>全部 38 个按 URL 获取的帖子字段，<code>gd_lyclm1571iy3mv57zw</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `url` | url | URL |
| `post_id` | text | 每条 Facebook 帖子的唯一标识符 |
| `user_url` | url | 每条 Facebook 帖子的唯一标识符 |
| `user_username_raw` | text | 发布内容的用户的用户名 |
| `content` | text | 帖子的文本或媒体内容 |
| `date_posted` | date | 帖子发布日期和时间 |
| `hashtags` | array | 帖子中使用的标签 |
| `num_comments` | number | 帖子的评论数 |
| `num_shares` | number | 帖子的分享数 |
| `num_likes_type` | array | 收到的点赞类型（例如回应、点赞） |
| `profile_id` | text | 用户个人资料的唯一标识符 |
| `page_logo` | url | Facebook 页面的徽标 |
| `page_likes` | number | Facebook 页面的点赞数 |
| `page_followers` | number | Facebook 页面的关注者数量 |
| `page_is_verified` | boolean | Facebook 页面的验证状态 |
| `original_post` | object | 指示帖子是原创内容还是转发内容 |
| `attachments` | array | 附件 |
| `post_external_link` | url | 帖子中包含的外部链接 |
| `post_external_title` | text | 外部链接的标题 |
| `page_url` | url | Facebook 页面的网页地址 |
| `header_image` | url | Facebook 页面的页眉图片 |
| `avatar_image_url` | text | Facebook 页面头像图片的 URL |
| `profile_handle` | text | Facebook 页面的短名或用户名 |
| `has_handshake` | text | 指示帖子是否包含握手标记 |
| `is_sponsored` | boolean | 指示帖子是否为赞助内容 |
| `sponsor_name` | text | 如果帖子为赞助内容，则为赞助商名称 |
| `shortcode` | text | 帖子 URL 的短代码 |
| `video_view_count` | number | 如果帖子包含视频，则为视频观看次数 |
| `likes` | number | 帖子的点赞数 |
| `post_image` | url | 帖子图片 URL（如果存在） |
| `post_type` | text | 帖子类型 |
| `user_handle` | text | 用户短名 |
| `play_count` | number | 播放次数 |
| `video_title` | text | 视频标题 |
| `marketplace_price` | price | Marketplace 商品的价格。此值适用于分享到群组中的 Marketplace 商品。 |
| `delegate_page_id` | text | 被指定为“对此页面负责”的页面 ID |
| `marketplace_is_sold` | text | 指示商品是否已售出，仅适用于 Marketplace 帖子 |
| `comments` | array | 帖子的评论 |

</details>

<details>
<summary>全部 22 个评论字段，<code>gd_lkay758p1eanlolqw8</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `url` | url | 包含评论的页面或帖子的 URL |
| `post_id` | text | 每条帖子的唯一标识符 |
| `post_url` | url | 特定帖子的 URL |
| `comment_id` | text | 每条评论的唯一标识符 |
| `user_name` | text | 发表评论的用户名 |
| `user_id` | text | 每个用户的唯一标识符 |
| `user_url` | url | 用户个人资料的 URL |
| `date_created` | date | 评论创建日期和时间 |
| `comment_text` | text | 评论的实际文本内容 |
| `num_likes` | number | 评论收到的点赞或回应数量 |
| `num_replies` | number | 对评论的直接回复数 |
| `attached_files` | array | 与评论关联的任何文件或附件 |
| `video_length` | number | 视频内容的时长（如适用） |
| `source_type` | text | 来源类型（例如 Facebook、外部链接） |
| `subtype` | text | 帖子或评论的子类别或具体类型 |
| `type` | text | 帖子或评论的一般类别 |
| `commentator_profile` | url | 评论者个人资料 |
| `comment_link` | url | 评论 URL |
| `reply` | boolean | 回复 |
| `parent_comment_id` | text | 父评论 ID |
| `commentator_profile_url` | url | 评论者个人资料图片的 URL |
| `attached_images` | array | 与评论关联的任何图片 |

</details>

<details>
<summary>全部 27 个 Reel 字段，<code>gd_lyclm3ey2q6rww027t</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `url` | url | Reel 的 URL |
| `post_id` | text | 每个 Facebook Reel 的唯一标识符 |
| `user_url` | url | 每个 Facebook Reel 的唯一标识符 |
| `user_username_raw` | text | 发布内容的用户的用户名 |
| `content` | text | Reel 的文本或媒体内容 |
| `date_posted` | date | Reel 发布日期和时间 |
| `hashtags` | array | Reel 中使用的标签 |
| `num_comments` | number | 帖子的评论数 |
| `num_shares` | number | 帖子的分享数 |
| `video_view_count` | number | 观看次数 |
| `likes` | number | Reel 的点赞数 |
| `page_name` | text | 发布 Reel 的 Facebook 页面名称 |
| `profile_id` | text | 用户个人资料的唯一标识符 |
| `page_logo` | url | Facebook 页面徽标的 URL |
| `page_likes` | number | Facebook 页面的点赞数 |
| `page_followers` | number | Facebook 页面的关注者数量 |
| `page_is_verified` | boolean | Facebook 页面的验证状态 |
| `thumbnail` | url | Reel 缩略图的 URL |
| `external_link` | url | Reel 中包含的外部链接 |
| `page_url` | url | Facebook 页面的网页地址 |
| `header_image` | url | Facebook 页面页眉图片的 URL |
| `avatar_image_url` | url | Facebook 页面头像图片的 URL |
| `profile_handle` | text | Facebook 页面的短名或用户名 |
| `shortcode` | text | 帖子 URL 的短代码 |
| `length` | number | Reel 的时长 |
| `audio` | text | Reel 的音频 |
| `video_url` | url | Reel 视频 URL |

</details>

<details>
<summary>全部 28 个页面字段，<code>gd_mf124a0511bauquyow</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `id` | text | 内部记录 ID |
| `url` | url | Facebook 页面 URL |
| `page_name` | text | 页面显示名称 |
| `username` | text | 页面短名（用户名） |
| `entity_type` | text | 页面或个人资料 |
| `summary_text` | text | 简介标语文本 |
| `primary_category` | text | 主要类别 |
| `work` | array | 工作经历行 |
| `education` | array | 教育经历行 |
| `current_city` | text | 当前城市 |
| `hometown` | text | 家乡 |
| `relationship` | object | 关系信息 |
| `address` | object | 企业/地点地址 |
| `phones` | array | 电话号码 |
| `emails` | array | 电子邮件地址 |
| `websites` | array | 网站和社交链接 |
| `hours` | object | 营业时间 |
| `rating` | object | 评分摘要 |
| `contact_and_basic_info` | object | 简介 > 联系方式和基本信息 |
| `page_transparency` | object | 简介 > 页面透明度 |
| `details_about` | object | 简介 > 详细信息 |
| `privacy_and_legal_info` | text | 隐私/条款/法律说明 |
| `followers` | number | 关注者数量 |
| `logo` | url | 页面徽标 |
| `is_verified` | boolean | 页面是否已验证 |
| `group_members` | number | Facebook 群组的成员数（页面为群组页面时适用） |
| `group_created_at` | date | 群组创建日期（页面为群组页面时适用） |
| `num_posts_last_month` | number | 最近一个月的帖子数（页面为群组页面时适用） |

</details>

<details>
<summary>全部 11 个个人资料字段，<code>gd_mf0urb782734ik94dz</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `url` | url | URL |
| `name` | text | 人员姓名 |
| `id` | text | Facebook 个人资料 ID |
| `profile_photo` | image | 个人资料照片 |
| `cover_photo` | image | 个人资料封面照片 |
| `work` | array | 个人资料中的工作地点 |
| `college` | array | 个人资料中的大学数据 |
| `high_school` | array | 个人资料中的高中数据 |
| `photos` | array | 个人资料中的照片 |
| `location` | text | 个人资料所属地点 |
| `city` | text | 个人资料所属城市 |

</details>

<details>
<summary>全部 18 个评价字段，<code>gd_m0dtqpiu1mbcyc2g86</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `company_name` | text | 被评价公司的名称。此字段用于标识评价涉及的企业，有助于按公司对反馈进行分类和分析。 |
| `company_id` | text | 每家被评价公司的唯一标识符。此字段有助于区分不同公司，并可轻松引用和汇总与同一家公司相关的评价 |
| `company_url` | url | 被评价公司的 URL |
| `url` | url | Facebook 平台上特定评价的 URL。通过此链接，用户可直接查看原始评价 |
| `review_time` | date | 评价发布日期和时间。这有助于了解反馈的时间线和新近程度 |
| `recommends` | boolean | 指示评价者是否推荐该公司（true 表示推荐，false 表示不推荐） |
| `review_content` | text | 评价的文本内容，详细描述评价者对公司的体验和看法 |
| `review_attachments` | array | 一个 URL 数组，指向评价中包含的任何附件，例如图片或视频。这些附件可提供与评价相关的额外背景或证据 |
| `number_of_reactions_to_review` | number | 其他 Facebook 用户对该评价作出的回应总数（例如赞、爱心、哇）。这反映了评价的互动程度和可见度 |
| `review_reaction_types` | array | 一个数组，详细列出评价收到的回应类型（例如赞、爱心、哈哈、哇、难过、愤怒）。这有助于了解受众对评价的情绪反应。 |
| `reviewer_profile_url` | url | 评价者 Facebook 个人资料的 URL。通过此链接可访问评价者的个人资料以了解更多背景 |
| `reviewer_name` | text | reviewer_name |
| `reviewer_profile_picture_url` | url | reviewer_profile_picture_url |
| `recommended_perc` | text | 推荐评价的百分比 |
| `likes` | number | 页面的点赞数 |
| `followers` | number | 页面的关注者数量 |
| `total_reviews` | number | 评价总数 |
| `replies` | array | replies |

</details>

<details>
<summary>全部 23 个 Marketplace 商品字段，<code>gd_lvt9iwuh6fbcwmx1a</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `url` | url | Facebook Marketplace 商品页面的直接 URL |
| `title` | text | 卖家发布的商品标题 |
| `initial_price` | number | 商品在任何折扣前的原始标价 |
| `final_price` | number | 商品的当前价格或折扣价 |
| `currency` | text | 标价所使用的货币 |
| `product_id` | text | Facebook Marketplace 分配给商品的唯一标识符。 |
| `breadcrumbs` | array | 表示商品分类层级的导航面包屑项目列表 |
| `condition` | text | 卖家报告的商品状况 |
| `description` | text | 卖家撰写的商品完整文本描述 |
| `location` | text | 大致位置 |
| `country_code` | text | 两字母 ISO 3166-1 alpha-2 国家/地区代码 |
| `root_category` | text | 商品在 Facebook Marketplace 中所属的根类别 |
| `images` | array | 一个 URL 列表，指向卖家上传的商品图片 |
| `seller_description` | text | 卖家 Facebook Marketplace 个人资料中的简短介绍或描述 |
| `color` | text | 商品颜色（如适用且卖家已提供） |
| `brand` | text | 商品品牌或制造商（如适用） |
| `videos` | array | 一个 URL 列表，指向卖家作为商品信息一部分上传的任何视频内容 |
| `profile_id` | text | 发布商品的卖家的唯一 Facebook 个人资料 ID |
| `listing_date` | date | 商品最初发布到 Facebook Marketplace 的日期和时间 |
| `car_miles` | number | 卖家报告的车辆里程。仅与汽车/车辆商品相关 |
| `is_sold` | boolean | 指示商品是否已售出 |
| `transmission` | text | 车辆详情中显示的变速箱类型。仅与汽车/车辆商品相关 |
| `vehicle_details` | object | 从 Facebook Marketplace 汽车商品的“关于此车辆”部分提取的详细车辆专属属性。非车辆商品为 Null。 |

</details>

<details>
<summary>全部 30 个活动字段，<code>gd_m14sd0to1jz48ppm51</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `event_id` | text | 活动的唯一标识符 |
| `url` | url | 活动的直接 URL 链接 |
| `main_image` | url | 活动主图片的 URL |
| `event_date` | date | 活动日期 |
| `title` | text | 活动名称或标题 |
| `people_responded` | number | 标记为已“回应”活动的人数 |
| `event_by` | array | 用于指定活动组织者的对象数组，每个对象均包含名称和 URL 字段 |
| `location` | object | 包含地址和 URL 的嵌套对象 |
| `access_level` | array | 指示活动的隐私或访问级别（例如公开、私密） |
| `description` | object | 包含文本、链接和标签字段的对象，用于提供详细活动描述 |
| `hashtags` | array | 与描述分开，这是一个提供活动相关通用标签的数组 |
| `hosts` | array | 用于指定活动主办方的对象数组，包括名称、URL 和验证状态 |
| `suggested_events` | array | 列出其他推荐或类似活动的数组，每个活动均包含名称、URL、日期、感兴趣人数和地点字段 |
| `tickets` | object | 详述票务信息的对象，包括 URL、提供商、最低/最高价格和货币 |
| `duration` | object | 定义活动持续时间的对象 |
| `main_image_downloadable` | url | 可下载的主图片 |
| `responded_object` | object | 回应对象 |
| `event_start_time` | date | 活动开始时间 |
| `event_end_time` | date | 活动结束时间 |
| `event_end_date` | date | 活动持续超过一天时的结束日期 |
| `full_address` | text | 活动地址 |
| `longitude` | text | 经度 |
| `latitude` | text | 纬度 |
| `event_timezone` | text | 活动所在时区 |
| `unformatted_description_text` | text | 保留原始结构的描述文本 |
| `alternative_dates` | array | 其他日期 |
| `alternative_date_time` | array | 其他日期时间 |
| `redirected_event_url` | url | 重定向后的活动 URL |
| `main_video` | url | 主视频（如果有） |
| `is_cancelled` | boolean | 指示活动是否已取消 |

</details>

<details>
<summary>全部 32 个广告资料库广告字段，<code>gd_mtveskb62h7q65ftgz</code></summary>

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `url` | url | Meta 广告资料库广告详情页面 URL |
| `library_id` | text | 唯一 Meta 广告资料库 ID（以文本形式保留，因为超出安全整数范围） |
| `is_active` | boolean | 指示广告目前是否正在投放 |
| `status` | text | 广告状态标签（投放中、未投放） |
| `start_date` | date | 广告开始投放日期 |
| `end_date` | date | 广告停止投放日期（投放期间为空） |
| `total_active_time` | text | 显示的总投放时长（例如 8 小时） |
| `ad_category` | text | 特殊广告类别（住房、就业、信贷、社会议题、选举或政治）；商业广告为空 |
| `has_multiple_versions` | boolean | 指示广告是否有多个创意版本 |
| `versions_count` | number | 创意版本数量（来自“第 1 个，共 N 个”指示器） |
| `creative_reuse_count` | number | 使用此创意和文案的广告数量 |
| `platforms` | array | 广告投放平台 |
| `platforms_count` | number | 广告投放的平台数量 |
| `description` | text | 主要广告正文 |
| `title` | text | 广告标题 |
| `link_description` | text | 标题下方显示的链接描述 |
| `caption` | text | 标题上方显示的域名（例如 ADIDAS.EU） |
| `cta_text` | text | 显示的号召性用语按钮文本（例如“立即购买”） |
| `cta_type` | text | 标准化号召性用语类型（例如 SHOP_NOW、LEARN_MORE、SIGN_UP） |
| `link_url` | url | 广告的目标 URL |
| `image_url` | url | 主要创意图片 URL |
| `video_url` | url | 主要创意视频 URL |
| `video_preview_image` | url | 视频缩略图 URL |
| `ad_versions` | array | 广告的所有创意版本或轮播卡片 |
| `advertiser_name` | text | “关于广告主”部分中的广告主名称 |
| `advertiser_profile_picture` | url | 广告主个人资料图片 URL |
| `advertiser_description` | text | 广告主“更多信息”描述文本 |
| `advertiser_social_accounts` | array | “关于广告主”部分中列出的广告主社交账户 |
| `advertiser_and_payer` | array | 广告主和付款方条目（仅限欧盟及相关地区） |
| `eu_total_reach` | number | 广告在欧盟的总覆盖人数 |
| `transparency_by_location` | array | 按地点划分的覆盖和定向详情（仅限欧盟及相关地区） |
| `is_eu_ad` | boolean | 指示广告是否在欧盟投放并包含 DSA 透明度部分 |

</details>
<!-- fields:end -->

<details>
<summary>真实输出文件的开头，来自 <code>facebook-scraper Meta --limit 1</code></summary>

```json
{
  "generated_at": "2026-11-03T19:30:56.535Z",
  "pages": [
    {
      "page": "Meta",
      "posts": [
        {
          "url": "https://www.facebook.com/Meta/posts/pfbid0evGHWPACd3JninWg3j3nucQjdFxoHWwJSPgFXKjZXaUiU8MctREDPRZZsAps6CGwl",
          "post_id": "1181668677855674",
          "user_url": "https://www.facebook.com/Meta",
          "user_username_raw": "Meta",
          "content": "Meta Ray-Ban Display has been out for a while, so if you’re a developer, this one’s for you.\n\nbuild a game, a productivity tool, a science app — really, anything you love — and watch it run on an in-lens display, controlled by the Meta Neural Band.\n\nneed inspo? here’s what people have already cooked up: \n\nMeta Geology by @inna.sparrow.xr\nVistathlon by @stspanho\nSee DJ by @levinriegner\nSightline by @fasai.creates",
          "date_posted": "2026-10-30T19:41:31.000Z",
          "num_comments": 343,
          "num_shares": 194,
          "num_likes_type": {
            "type": "Like",
            "num": 594
          },
          "page_name": "Meta",
  ...
```

包含一条帖子及其所有字段的完整文件位于 [examples/sample_output.json](examples/sample_output.json)。

</details>

## Facebook 爬虫工具错误

| 你看到的内容 | 含义 |
| --- | --- |
| `API token required but not found.` | 在发出任何请求前以代码 2 退出。请设置令牌或运行 `bdata login`。 |
| `failed  page: Posts not found` | 以代码 1 退出。页面不存在、页面为私密页面，或页面没有公开帖子。请检查名称。 |
| `failed  page: timeout: snapshot ... not ready after 900s` | 以代码 1 退出。请重新运行，或稍后使用 `fetchSnapshot()` 获取快照。 |
| `failed  page: HTTP 401: ...` | 令牌已设置，但不正确。 |

任何失败都会以代码 1 退出，因此可安全地使用运行结果来控制脚本是否继续。

通过函数调用时，相同情况如下所示：

| 你看到的内容 | 含义 |
| --- | --- |
| `TokenMissing` | 没有 `BRIGHTDATA_API_TOKEN`，也没有 `bdata login`。 |
| `ApiError: HTTP 400: ...validation_error...` | API 不接受的输入字段。消息会指出该字段；请与上表比较。 |
| 包含 `error` 键的行 | API 针对某一输入的响应：“Posts not found”、失效页面、“Reviews not found”或“There are no events.”。其他行不受影响。 |
| `ApiError: timeout: ...` | 作业超过了 900 秒等待时间。向 `run` 传递 `{ timeoutMs }`，或使用 `trigger` 并稍后获取。 |

## Facebook MCP 服务器和 Claude Code 技能

无需 JavaScript，无需安装任何内容。粘贴这两行；第一行会打开一次浏览器，也可以在 SSH 和 CI 中使用 `bdata login --device`：

```bash
npx -p @brightdata/cli bdata login
npx -p @brightdata/cli bdata pipelines run gd_lkaxegm826bjpoo9m5 collect_by_url --url "https://www.facebook.com/Meta" --num_of_posts 2 --pretty
```

2 条记录，最多 38 个字段，2026 年 11 月 3 日耗时 113 秒。CLI 会省略值为 null 的字段。

CLI 按数据集 ID 运行爬虫工具，其中包括每个 Facebook 爬虫。`bdata pipelines search facebook` 会列出它们的 ID 和方法，`--limit-per-input N` 会限制发现数量，默认为 10。

`npx skills add brightdata/skills` 会向 Claude Code、Cursor 和 Codex 说明这些命令及文档，之后即可使用自然语言操作。完整指南：[面向编码智能体的 Bright Data](https://docs.brightdata.com/quickstart-coding-agent)。

托管式助手无需终端？[Bright Data MCP 服务器](https://github.com/bright-cn/brightdata-mcp#which-tool-to-use)可通过其 `scrapers` 组中的 `search_scrapers` 和 `run_scraper` 运行任何预构建爬虫工具，包括 Facebook 爬虫：

    https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN&groups=scrapers

智能体还可以自行开设账户，无需注册表单：[智能体注册](https://brightdata.com/auth.md)。Bright Data 支持的其他所有连接（从 LangChain 到 Zapier 和 n8n）：[集成](https://docs.brightdata.com/integrations/introduction)。

## 这个 Facebook 爬虫工具免费吗？

新的 Bright Data 账户每月可获得[5,000 点免费额度](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。每个 Facebook 爬虫工具均按记录计费：按每条帖子、评论、Reel、页面、个人资料、评价、商品、活动或广告计费。[定价](https://www.bright.cn/pricing/web-scraper)列出了具体费率。

## 支持

此代码仓库中的错误：[提交 Issue](https://github.com/bright-cn/facebook-scraper-node/issues)，[CONTRIBUTING.md](CONTRIBUTING.md) 说明了需要提供哪些信息。有关 API、你的账户或额度的任何问题，请联系 [Bright Data 支持团队](https://brightdata.zendesk.com/hc/en-us/requests/new)。

## 许可证

[Apache 2.0](LICENSE)。

## 适用于 Python 的 Facebook 爬虫工具

同一款 Facebook 爬虫工具也提供 Python 版本：[brightdata/facebook-scraper](https://github.com/brightdata/facebook-scraper)。

## Bright Data 提供的更多开源爬虫工具

采用相同方式构建并包含实时验证示例的其他 Node.js 爬虫工具：

- [适用于 Node.js 的 Instagram 爬虫工具](https://github.com/bright-cn/instagram-scraper-node)
- [适用于 Node.js 的 TikTok 爬虫工具](https://github.com/brightdata/tiktok-scraper-node)
- [适用于 Node.js 的 LinkedIn 爬虫工具](https://github.com/bright-cn/linkedin-scraper-node)
