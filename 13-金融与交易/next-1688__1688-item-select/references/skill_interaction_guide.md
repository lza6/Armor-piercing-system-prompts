# Skill 交互接入快速指南 (Quick Start)

本指南旨在帮助 Skill 开发者**快速接入** Newton Agent 的客户端交互能力。只需两步：**声明 Metadata** 和 **正文引导**。

---

## 1. 第一步：在 Metadata 中声明交互 (必选)

在 `SKILL.md` 的 YAML Frontmatter 中添加 `interactions` 列表。这是框架识别和大模型发现交互能力的**唯一入口**。

### 核心字段速查

| 字段 | 说明 | 示例值 |
| :--- | :--- | :--- |
| `name` | **交互唯一 ID**，大模型调用时使用 | `select_products` |
| `type` | **组件类型**：`table` (表格), `card` (卡片), `input` (问答) | `table` |
| `selectionType` | **数据类型标识**，决定云端存哪里 | `product`, `merchant` |
| `description` | **业务语义**，告诉大模型何时用 | "从结果中选择商品" |
| `required_data` | **数据槽位**，简要描述需要的数据 | `{ products: "商品列表" }` |

### 快速复制模板

```yaml
---
metadata:
  interactions:
    - name: select_products_for_inquiry
      type: table
      selectionType: product
      description: "从搜索结果中选择要询盘的商品"
      required_data:
        products: "商品列表数组，每项包含 id, title, price"
---
```

---

## 2. 第二步：在正文中引导大模型 (可选但推荐)

在 `SKILL.md` 正文中，用自然语言告诉大模型**何时触发**、**如何填数**，以及**去哪里查规范**。

### 引导话术示例

> **触发时机**：
> “在执行 `text_search` 获得商品列表后，若用户表现出挑选意向，请调用 `show_interaction` 并设置 `name='select_products_for_inquiry'`。”
>
> **数据填充**：
> “将 `text_search` 返回的 `items` 数组赋值给 `products` 槽位。**具体的字段映射规则与组件渲染数据结构请查阅 [`references/interaction-specs.md`](./references/interaction-specs.md) 中对应交互的章节。**”

### ⚠️ 关键约定

正文中**必须明确告诉大模型**：组件的渲染数据结构、Props 字段、映射规则等技术细节，统一存放在 `references/interaction-specs.md`，调用前需查阅该文档。这样可以避免在 SKILL.md 主体中堆砌大段 JSON Schema，节省 Token 并提升可维护性。

---

## 3. 三种组件的快速参考

根据你的业务场景，选择对应的 `type`：

### A. Table (表格选择) - 适合结构化数据多选
*   **场景**：商品列表、订单筛选。
*   **Metadata 示例**：
    ```yaml
    - name: select_items
      type: table
      selectionType: product
      required_data:
        title: "表格标题"
        columns: "列定义数组"
        rows: "数据行数组"
    ```

### B. Card (选项卡片) - 适合快速偏好选择
*   **场景**：风格选择、平台确认。端侧会自动追加“输入其他”。
*   **Metadata 示例**：
    ```yaml
    - name: choose_style
      type: card
      selectionType: style
      required_data:
        questions: "问题列表，如 [{ question: '喜欢什么风格？', options: ['休闲', '商务'] }]"
    ```

### C. Input (澄清问答) - 适合细节确认与自由输入
*   **场景**：预算确认、备注填写。
*   **Metadata 示例**：
    ```yaml
    - name: clarify_budget
      type: input
      selectionType: requirement
      required_data:
        questions: "问题列表，如 [{ question: '预算多少？', options: ['1万内', '1-5万'] }]"
    ```

---

## 4. 关键注意事项 (避坑指南)

1.  **`selectionType` 必须准**：填 `product` 还是 `merchant` 直接决定数据存入哪张表，请勿随意填写。
2.  **Table 不能为空**：调用 `table` 类型交互前，务必确保 `rows` 有真实数据，否则前端会报错。
3.  **细节下沉**：Metadata 中只写**槽位名**和**简要描述**。复杂的 JSON 结构、Props 定义和数据映射规则应写入 `references/interaction-specs.md`，实现声明与实现的解耦。

   **推荐目录结构**：
   ```text
   my-skill/
   ├── SKILL.md                      # 轻量级声明与业务逻辑
   └── references/
       └── interaction-specs.md      # 详细的组件 Props 与数据格式定义
   ```

---

## 5. `references/interaction-specs.md` 编写规范

该文档是 Skill 内部的**交互详细说明书**，供大模型在调用 `show_interaction` 前查阅，确保数据结构正确。

### 编写原则

1.  **一个交互一节**：每个在 Metadata 中声明的 `interactions` 条目，对应文档中的一个章节，标题与 `name` 一致。
2.  **聚焦数据结构**：只写**渲染数据结构**和**字段映射规则**，不要重复 Metadata 中已有的业务描述。
3.  **给出真实示例**：提供一段可直接拷贝的 JSON 示例，便于大模型对照填充。

### 标准模板

```markdown
# 交互组件详细规范

本文档定义了本 Skill 中所有交互组件的具体数据结构与映射规则。

## 1. <交互 name> (<组件类型> 组件)

### 组件类型
`type: table` | `card` | `input`

### 数据槽位定义
- **`<槽位名>`**:
  - 类型: `Array<Object>` / `String` / ...
  - 映射规则: 描述该槽位的数据从哪个 Tool 的哪个字段转换而来。
  - 必须字段: 列出对象内必须包含的 key。

### 框架自动填充的字段（如有）
说明框架会根据 `selectionType` 自动注入哪些默认值（如 Table 的 `columns`），Skill 无需手动指定。

```json
[
  { "key": "imageUrl", "label": "图片", "width": 80 },
  { "key": "title", "label": "商品标题" },
  { "key": "price", "label": "价格" }
]
```

### 完整数据示例
```json
{
  "products": [
    { "id": "p1", "title": "示例商品", "price": 99, "imageUrl": "https://..." }
  ]
}
```
```

### 编写要点速查

| 章节 | 必须包含 | 说明 |
| :--- | :--- | :--- |
| 组件类型 | ✅ | 与 Metadata 的 `type` 保持一致 |
| 数据槽位定义 | ✅ | 每个 `required_data` 槽位都要展开 |
| 映射规则 | ✅ | 明确指出数据来源的 Tool 与字段路径 |
| 框架自动填充 | ⭕ | 仅 `selectionType` 有内置模板时需要写 |
| 完整数据示例 | ✅ | 一段可直接拷贝的 JSON |

---

*更多详细技术实现请参考：[SIMPLE_INTERACTION_PROTOCOL.md](./SIMPLE_INTERACTION_PROTOCOL.md)*
