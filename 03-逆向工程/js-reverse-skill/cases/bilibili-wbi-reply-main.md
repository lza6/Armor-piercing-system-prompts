# Case：B 站番剧评论接口 WBI 签名（w_rid）纯协议还原

> 难度：★★（纯算还原，坑点集中在 query 构造顺序与密钥来源接口变更）
> 还原方案：A 纯算还原（标准 MD5 + 参数排序 + 密钥混排派生，零补环境）
> 实现语言：Node.js（零第三方依赖，`node:crypto` + `node:https`）
> 最后验证日期：2026-08-16（样本反推一致 + 真实 API 5/5 次 GET 200，每页 19~20 条评论，翻页游标生效）
> 平台类型：视频/番剧社区（bilibili.com，番剧播放页评论区）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

- [x] 请求参数 `w_rid` = `md5(query + mixin_key)`，32 位 hex；`wts` = `Math.round(Date.now()/1000)` 秒级时间戳
- [x] query 构造：**先注入 wts 再按 key 升序排序**；value 为字符串时**先剔除 `!'()*` 四个字符再 `encodeURIComponent`**；空值也参与（如 `seek_rpid=`）
- [x] `mixin_key` = 固定 64 索引混排表对 `(img_key + sub_key)` 重排后取前 32 位（混排表见实现）
- [x] 密钥来源：`GET /x/web-interface/nav` 返回 `data.wbi_img`（`img_url`/`sub_url` 文件名去扩展名）；**未登录也返回**；密钥按日轮换，实现内 12h 缓存即可
- [x] **`/x/web-interface/wbi/keys` 已废弃**（2026-08-16 实测 404），不要用
- [x] 签名生成端：`web.min.js`（unios 请求库）的 `UNIOS_WBI_ENCODE` 中间件；请求链 `bili-comments.js` 的 `Im` 构造 → `web.min.js` 的 `D` 函数签名 → fetch GET
- [x] `web_location=1315875` 为评论组件硬编码固定值

## 加密方案

- 路径：A 纯算还原（标准 MD5 + 参数排序 + 混排密钥派生），无需补环境、无需浏览器
- 框架：无（Node 内置 `node:crypto` / `node:https` + keepAlive Agent）
- TLS 客户端：普通 Node `https` 直连即可（公开评论读取，无 TLS 指纹检测）
- 核心思路：评论组件 `bili-comments.js` 只传参（`oid/type/mode/pagination_str/plat/seek_rpid/web_location`），实际签名在 `web.min.js` 的 `UNIOS_WBI_ENCODE` 中间件里完成；密钥从 nav 接口动态取，运行时派生 mixin_key。

### 签名公式
```
mixin_key = 混排表[46,47,18,...,52]（64 索引）对 (img_key + sub_key) 重排，取前 32 位
wts       = Math.round(Date.now()/1000)          # 先注入 wts，再排序
params    = { ...业务参数, wts }
query     = keys(params).sort()
             每项: encodeURIComponent(k) + "=" + encodeURIComponent(v)   # v 先剔除 !'()*
             以 "&" 连接（空值也参与，如 seek_rpid=）
w_rid     = md5(query + mixin_key)
```

### 评论接口参数
```
GET https://api.bilibili.com/x/v2/reply/wbi/main?
  oid=<番剧 aid>            # 如 114543489451440
  type=1
  mode=3                     # 3=热度排序（2=时间排序）
  pagination_str={"offset":"<游标>"}   # 第一页 offset 为空串
  plat=1
  seek_rpid=                 # 空值也参与签名
  web_location=1315875       # 固定
  wts=<秒级时间戳>
  w_rid=<md5 签名>
```

---

## 踩坑记录（IMPLEMENT 必查）

| # | 坑 | 现象 | 正确做法 |
|---|---|---|---|
| 1 | 评论接口需滚动触发 | 自动 trace 第一轮只抓到播放器 eval（5 行），无评论请求 | 取证时滚动页面到评论区再采集；或从 Step1 抓包直接拿目标请求 |
| 2 | 用 `wbi/keys` 取密钥 | 返回 404 | 已废弃，改用 `/x/web-interface/nav`（未登录也返回 `wbi_img`） |
| 3 | 翻页游标用 rpid | 每页返回相同评论 | 游标是 `cursor.pagination_reply.next_offset`（protobuf 编码，如 `CAEiAggC`，每次 +1），不是 rpid |
| 4 | 排序后才注入 wts | 签名校验失败 | **先注入 wts 再排序**，wts 参与签名 |
| 5 | value 未剔字符直接编码 | 签名校验失败 | value 字符串先剔除 `!'()*` 再 `encodeURIComponent`（web.min.js 原始行为） |
| 6 | 空值参数被丢弃 | 签名校验失败 | `seek_rpid=` 等空值也要进 query 参与签名 |
| 7 | `node -e` 跑验证脚本 | `process.argv[1]` 为 undefined 报错 | 用独立脚本文件，别用 `node -e`（ESM 入口判断依赖 argv[1]） |
| 8 | ESM 跨目录相对 import | 路径解析到 case 目录 | 数清层级或改用绝对路径；最终入口统一 `result/final.js` |

---

## 可验证事实清单（经验资产，同站升级时逐条核对）

1. `w_rid = md5(sorted_query + mixin_key)`，`sorted_query` 按 key 升序、`encodeURIComponent(k)=encodeURIComponent(v)`、`&` 连接
2. 混排表为固定 64 索引：`[46,47,18,2,53,8,23,32,15,50,10,31,58,3,45,35,27,43,5,49,33,9,42,19,29,28,14,39,12,38,41,13,37,48,7,16,24,55,40,61,26,17,0,1,60,51,30,4,22,25,54,21,56,59,6,63,57,62,11,36,20,34,44,52]`
3. `mixin_key = 混排(img_key + sub_key)[:32]`，img/sub key 来自 nav 的 `wbi_img.img_url/sub_url`（取文件名去扩展名）
4. `/x/web-interface/wbi/keys` 2026-08-16 实测 404 废弃；`/x/web-interface/nav` 未登录也返回 wbi_img
5. `web_location=1315875` 为评论组件硬编码，参与签名
6. 翻页游标 = `cursor.pagination_reply.next_offset`（protobuf 编码，每次 +1），评论总数在 `cursor.all_count`
7. `mode=3` 热度排序、`mode=2` 时间排序；`oid` 为番剧 aid
8. 样本验证（2026-08-16）：`wts=1786868551` 时 `w_rid=6fb72a5f42bb01a6f8e5b03db455894d`，与真实请求一致
9. 公开评论读取无需登录、无人工验证码；短时高频请求可能被限流

---

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/crypto/crypto-entry.md` | 请求方向四层（source→entry→builder→writer），本案例 writer 在 `web.min.js` unios 中间件 |
| `references/crypto/crypto-patterns.md` | 参数排序 + MD5 签名模式 |
| `references/workflow/decision-tree.md` | 纯算还原（A）路径判定 |
| `references/network/session-chain.md` | keepAlive Session 复用与关闭清理 |
