---
name: xhs-product-image-design-report
description: Use when the user provides one or more Xiaohongshu goods-detail links, product screenshots, or competitor product image folders and asks to collect public product fields/images, compare main images/detail images, explain why competitors designed them that way, and output a Chinese HTML product image design decision report for Xiaohongshu goods pages.
---

# 小红书商品图设计决策报告

## Goal

Turn multiple Xiaohongshu product links or supplied product image folders into a design decision report answering:

- 竞品主图/详情图分别在解决什么购买问题。
- 对方为什么这样设计。
- 我的商品主图、副图、详情页长图应该怎么拍、怎么排、哪些表达需要证据。

Default output is a standalone Chinese HTML report matching the confirmed v10 report structure: local images, product links, comparison tables, one-product-per-row evidence wall, and a simulated vertical mobile detail-page layout.

## Inputs

Accept only Xiaohongshu product-analysis sources. Do not use this skill for Taobao, Tmall, JD, Douyin, Pinduoduo, Amazon, Shopify, or other platforms.

- Xiaohongshu `https://www.xiaohongshu.com/goods-detail/...` URLs, preferably 3-10 links.
- Exported Xiaohongshu product screenshots/images for each product if live collection is blocked.
- The user's own product资料: product name, category, target customer, price band, SKU, material, packaging, shipping/after-sales rules.
- The user's evidence: real product photos, supplier specs, test reports, material certificates, packaging photos, customer review excerpts, sales/click/conversion data.

If own-product资料 or evidence is missing, continue with competitor analysis and mark assumptions/证据缺口 clearly.

When asking the user to provide screenshots manually, request these images for every Xiaohongshu product:

1. 商品首屏截图: include product main image, title, price, sold count if visible, and shop/product header if visible.
2. 商品图集截图 or saved carousel images: include all main images/sub-images in the product carousel, ideally 8-10 images per product.
3. 详情页长图/详情内容截图: scroll down and capture the product detail image area in order; use multiple screenshots if one long screenshot is not possible.
4. SKU/spec screenshot: capture color, size, material, package, quantity, single/pair options, or variant selector if visible.
5. 店铺/口碑/售后 screenshot: capture shop rating, shipping, return/after-sales, packaging, delivery promises, or service badges if visible.

Tell the user that screenshots should preserve the original order. Do not crop away price, link/title context, or image captions that explain material/specs.

## Collection Capability And Dependencies

Collection uses browser automation, not a Xiaohongshu official API.

Preferred capability:

- Playwright in a real browser session, through the local `playwright` skill/CLI or a Node.js Playwright script.
- Use existing logged-in browser state when public access is incomplete. If login/captcha/anti-bot blocks access, ask the user to open the links in their browser or provide screenshots/images.

Required local dependencies for scripted collection:

- Node.js and npm/npx.
- `playwright` package or the bundled Playwright CLI skill.
- Network access to `xiaohongshu.com`.
- A browser profile/session that can view the goods pages when pages are not publicly accessible.

Run the script with a Node.js environment that can resolve Playwright, for example:

```bash
node scripts/collect-xhs-goods.js --urls-file 输入资料/商品链接.txt --out-dir 输出结果
```

When the host provides a bundled Node/Playwright runtime instead of a normal `node` command, use that host's documented runtime path. Do not copy a machine-specific absolute path from another user's environment.

Data that can usually be collected from visible public pages:

- URL, page title, visible text.
- Product title, shop name, price, sold count, rating/口碑, SKU/spec text when visible.
- `<img>` image URLs, rendered image dimensions, alt text.
- CSS background image URLs.
- Full-page screenshots for evidence and fallback extraction.

Do not claim access to private platform data, comments, backend metrics, order data, or hidden fields unless the user provides them.

## Workflow

### 1. Create Work Folder

Create a timestamped folder with this structure:

```text
小红书商品分析-{YYYYMMDD-HHMMSS}/
├── 输入资料/
│   ├── 商品链接.txt
│   ├── 我的产品资料.md
│   ├── 我的真实证据/
│   └── 竞品图片/
└── 输出结果/
    ├── 公开字段抽取.json
    ├── 图片采集记录.md
    ├── 竞品逐图拆解表.csv
    ├── 小红书商品主图详情图设计决策分析报告.html
    └── 截图验证/
```

### 2. Collect Public Fields And Images

Use `scripts/collect-xhs-goods.js` as the starting point when collecting from live URLs. Provide `--urls-file` and `--out-dir` per task.

For each product:

1. Open the URL with a mobile-like viewport first, because Xiaohongshu product pages are mobile-oriented.
2. Wait for content and lazy images to load.
3. Save full-page screenshot.
4. Extract visible text, image URLs, background images, and rendered dimensions.
5. Download or otherwise save usable product images into `输入资料/竞品图片/商品N_短标题/`.
6. Record failures and blocked fields in `图片采集记录.md`.

If live extraction is blocked, switch to user-provided screenshots/images and explicitly label the source as "用户提供图片/截图".

### 3. Normalize Product Evidence

Create one normalized record per product:

- 商品序号
- 商品链接
- 商品名称
- 价格/到手价/划线价
- 已售/销量口径
- 店铺名/口碑/粉丝数 if visible
- SKU/spec/material/packaging if visible
- Main image path
- Detail image paths in display order
- Notes about missing or uncertain fields

Keep raw fields separate from inferred conclusions.

### 4. Analyze Image Logic

Analyze each product across these dimensions:

- 主图任务: click, style, first impression, product clarity.
- 副图/详情图任务: wearing effect, SKU choice, size, material, structure, packaging, after-sales, FAQ.
- Purchase doubts solved: 好不好看, 能不能戴, 怎么选, 值不值, 是否真实, 收到什么.
- Design patterns: 氛围实拍, 上耳效果, SKU矩阵, 结构说明, 礼物包装, 真实交付.
- Evidence boundary: what can be said only with proof, e.g. 纯银, F136, 不过敏, 爆款, 久戴不痛.

The core judgment should not be "which image looks better"; it should be "which purchase problem this image is solving".

### 5. Produce Design Recommendation

Output an actionable plan for the user's own images:

- 主图 1+5 plan: 图序, 页面任务, 核心文案方向, 画面怎么设计, 需要素材/证据, 设计判断.
- 详情页 10 屏 order: first screen, style confirmation, doubts, structure, size, material, SKU, real delivery, after-sales, FAQ.
- Execution checklist: 必须拍, 必须确认, 不要写.
- Evidence gaps: what the user must provide before strong claims can be used.

### 6. Build HTML Report

Use a practical Chinese report UI:

- Top summary with key conclusion metrics.
- 商品级横向对比.
- 主图/详情图证据墙: follow the confirmed v10 layout, one product per row, product link visible, larger images than the earlier three-column wall.
- 报告维度核对.
- 对方为什么这样设计.
- 竞品打法地图.
- 我的商品图应该怎么设计.
- 详情页模拟布局: follow the confirmed v10 behavior, phone frame with vertical scroll only, because users browse goods detail pages top-to-bottom; external cards should be a tidy two-row grid on desktop.
- 执行清单.

For layout details, read `references/report-structure.md` when implementing the HTML.

### 7. Verify

Always run lightweight checks before reporting completion:

- Confirm the final HTML file exists.
- Confirm every product link remains present in the report.
- Confirm local image paths referenced by the report exist.
- Confirm required sections are present: 商品对比, 图片证据墙, 设计原因, 设计方案, 详情页模拟布局, 执行清单.
- Confirm the report states missing/uncertain fields instead of treating inferred data as fact.

Run Playwright/browser screenshot verification only when visual correctness matters, such as:

- The user asks for a polished HTML report, UI redesign, or responsive layout check.
- The report layout was newly created or substantially changed.
- The detail-page phone mockup or image wall layout was modified.
- The final artifact will be shared externally and needs visual QA.

When running browser verification, start a local static server from the analysis folder, open the HTML with Playwright, capture desktop/mobile screenshots, and check image loading, product links, horizontal overflow, and vertical phone-frame scrolling. Console errors are acceptable only for unrelated favicon 404; mention this explicitly.

## Output Naming

Use a clear final filename:

```text
输出结果/小红书商品主图详情图设计决策分析报告.html
```

If iterating UI versions, suffix with `_v2`, `_v3`, etc., and tell the user which version is recommended.
