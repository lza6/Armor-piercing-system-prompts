# search_offer_by_keyword — 通过关键词搜索店铺商品

## 功能说明

通过关键词搜索当前店铺的商品，支持分页查询。可用于快速定位特定商品，作为选品、商品诊断等场景的辅助查询工具。

## 前置条件

- 已配置 AK（通过设置环境变量 `ALI_1688_AK`）

## CLI 调用

```bash
python {baseDir}/cli.py search_offer_by_keyword --keyword "测试" --page 1 --page_size 10
```

**参数说明**：

| 参数 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `--keyword` | 是 | — | 搜索关键词 |
| `--page` | 否 | `1` | 页码 |
| `--page_size` | 否 | `10` | 每页返回数量 |

## 输出格式

### 成功输出

```json
{
  "success": true,
  "markdown": "商品搜索成功",
  "data": {
    "data": {
      "total": 25,
      "items": [
        {
          "itemId": 1001683074980,
          "title": "xs 测试女装连衣裙",
          "brief": "面料名称:灯芯绒、 主图来源:实拍有模特、 ...",
          "type": "SALE",
          "mainImage": "https://cbu01.alicdn.com/img/ibank/xxx.jpg",
          "imageList": [
            "https://cbu01.alicdn.com/img/ibank/xxx.jpg",
            "https://cbu01.alicdn.com/img/ibank/yyy.jpg"
          ],
          "minPrice": 0.1,
          "maxPrice": 0.1,
          "attributes": [
            { "fid": 100031521, "name": "面料名称", "value": "灯芯绒" },
            { "fid": 3216, "name": "颜色", "value": "白色" }
          ],
          "detailUrl": "https://itemcdn.tmall.com/desc/...",
          "memberId": "b2b-123",
          "status": "PUBLISHED"
        }
      ]
    }
  }
}
```

### items 数组中每个商品字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `itemId` | Number | 商品 ID |
| `title` | String | 商品标题 |
| `brief` | String | 商品属性摘要（key:value 拼接） |
| `type` | String | 商品类型，如 `SALE` |
| `mainImage` | String | 主图 URL |
| `imageList` | Array\<String\> | 商品图片列表 |
| `minPrice` | Number | 最低价（元） |
| `maxPrice` | Number | 最高价（元） |
| `attributes` | Array\<Object\> | 商品属性列表，每项含 `fid`(属性ID)、`name`(属性名)、`value`(属性值) |
| `detailUrl` | String | 商品详情页 URL |
| `memberId` | String | 卖家会员 ID |
| `status` | String | 商品状态，如 `PUBLISHED` |

### 失败输出

```json
{
  "success": false,
  "markdown": "❌ AK 未配置，无法搜索商品。\n\n请补充有效 AK 或检查鉴权配置后重试",
  "data": {
    "data": {}
  }
}
```

## 异常处理

| 异常场景 | 处理方式 |
|---------|---------|
| AK 未配置 | 提示用户配置 AK |
| 签名无效（401） | 提示用户检查 AK 是否有效 |
| 请求被限流（429） | 建议用户等待 1-2 分钟后重试 |
| keyword 为空 | 提示用户提供搜索关键词 |

## 使用注意

1. 此技能为只读操作，不会修改任何数据
2. 搜索结果为当前店铺内的商品，非全平台搜索
