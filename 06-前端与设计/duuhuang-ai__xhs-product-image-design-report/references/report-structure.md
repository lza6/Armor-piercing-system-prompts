# HTML Report Structure

Use this structure for the final Chinese report.

## Required Sections

1. 报告摘要
   - State the main design decision in 1-2 sentences.
   - Include metrics: product count, image sample count, reusable tactics, recommended detail-page screens.

2. 商品级横向对比
   - One card per product.
   - Include product image, title, price, sold count, shop info, material/spec, tactic summary, and the original goods link.

3. 主图 / 详情图证据墙
   - One product per row.
   - Show 8-10 images per product with captions.
   - Keep the original goods link visible in the row header.
   - Desktop images should be large enough for visual review; mobile can wrap into 2 columns.

4. 报告维度核对
   - 已覆盖: fields/images that support conclusions.
   - 待补充: own product info, proof, reviews, platform performance, complete long-detail images.

5. 对方为什么这样设计
   - Table columns: 设计做法, 为什么这样做, 解决的购买问题, 我的商品能不能学.

6. 竞品打法地图
   - Summarize reusable tactics across products.

7. 我的商品图应该怎么设计
   - Combine table and design judgment in the same table.
   - Columns: 图序, 页面任务, 核心文案方向, 画面怎么设计, 需要素材/证据, 设计判断.

8. 详情页模拟布局
   - Left: phone frame with vertical scrolling long detail-page mockup.
   - Right: 10 cards in a two-row desktop grid.
   - Do not use horizontal carousel for the phone; Xiaohongshu goods detail pages are read vertically.

9. 执行清单
   - 必须拍, 必须确认, 不要写.

## UI Requirements

- Chinese business-report tone, not a landing page.
- Avoid decorative complexity.
- Use product images as the main visual evidence.
- Keep product links near the corresponding product data/images.
- Use `object-fit: contain` for product images.
- Avoid nested cards where possible.
- Verify mobile screenshots; no text overlap or whole-page horizontal overflow.

## Analysis Language

Prefer direct conclusions:

- "主图必须优先拍真人上耳。"
- "详情页不要先堆参数，先处理风格和痛点。"
- "材质、不过敏、爆款等强表达必须有证据。"
- "包装和售后只能展示真实交付。"

Avoid vague wording:

- "高级感"
- "氛围拉满"
- "提升转化" without saying which purchase doubt is solved.
