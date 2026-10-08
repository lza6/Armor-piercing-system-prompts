# JS 混淆识别与还原

本文件在分析阶段发现目标 JS 存在混淆、加密打包、控制流平坦化、自定义 VM、JSVMP 等情况时读取。目标是快速识别混淆类型并选择对应还原策略，不反编译字节码，不盲目展开全部代码。

## 总原则

- **不反编译字节码**：JSVMP / 自定义 VM 字节码只通过 I/O 定位和 Hook 关键操作理解行为，不尝试还原源码。
- **最小还原**：只还原影响参数生成和入口定位的部分，不做全文件美化。
- **运行时优先**：能直接在 Node.js / 浏览器中运行的解释器，直接运行并 Hook 出口，比静态还原更可靠。
- **AST 工具辅助**：OB 混淆、字符串数组、计算属性等可批量处理的，使用 AST 工具一次还原。
- **分流策略**：
  - 纯算场景：静态分析 + AST 还原即可。
  - 需动态定位场景：ruyiPage `search_code` + `set_breakpoint_via_hook` + `trace_function` 辅助定位。
  - JSVMP / WASM 场景：必须用 RuyiTrace 捕获执行轨迹，按 I/O 反推算法。

## 混淆类型速查表

| 类型 | 识别特征 | 还原策略 | 工具 |
|---|---|---|---|
| OB (obfuscator.io) | `_0x` 前缀变量、十六进制字符串数组、旋转函数 | 定位数组→执行旋转→全局替换→AST 美化 | `assets/ast-patterns/scripts/run-pipeline.js`（含 OB 变种 pass） |
| 控制流平坦化 (CFF) | `while(true) switch(state)` 状态机 | 按状态转移顺序还原顺序代码 | ruyiPage `trace_function` 追踪状态转移 |
| eval/Function 打包 | `eval(function(p,a,c,k,e,d){...})` | Hook eval/Function 拦截实际代码 | `references/hooks/hook-templates.md` 的 eval/Function Hook |
| AAEncode | 日文颜文字字符 `ﾟωﾟﾉ` | 直接执行或替换执行为输出 | 浏览器 console |
| JJEncode | 全是 `$` 和特殊字符 | 直接执行或替换执行为输出 | 浏览器 console |
| JSFuck | 仅使用 `[]()!+` 六种字符 | 直接执行 | 浏览器 console |
| 自定义 VM | 超大数组作字节码、解释器循环、switch/查找表 | 不反编译，Hook I/O 反推 | RuyiTrace 模式 |
| JSVMP | 200KB+ 文件、自定义操作码表、改写原生 API | 不反编译，Hook 出口反推 | RuyiTrace |

## 1. OB 混淆还原

**识别特征**：
- 大量 `_0x` 前缀变量名（如 `_0x4a3b2c`）
- 顶部十六进制字符串数组（`var _0xabc = ['...', '...', ...]`）
- 字符串数组旋转函数
- 十六进制属性访问（`obj['_0x1234']` 替代 `obj.method`）

**还原步骤**：
1. 定位字符串数组和旋转函数。
2. 执行旋转函数得到最终字符串数组。
3. 全局替换十六进制索引为实际字符串。
4. 简化数学表达式和逻辑运算。
5. 变量重命名提高可读性。

**AST 还原模板**：

```javascript
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const generate = require('@babel/generator').default;
const t = require('@babel/types');

function deobfuscate(code) {
    const ast = parser.parse(code);

    traverse(ast, {
        // 还原十六进制字符串
        StringLiteral(path) {
            if (/^\\x/.test(path.node.extra?.raw || '')) {
                path.node.extra = undefined;
            }
        },
        // 还原计算属性为点号访问
        MemberExpression(path) {
            if (t.isStringLiteral(path.node.property) && /^[a-zA-Z_$]/.test(path.node.property.value)) {
                path.node.computed = false;
                path.node.property = t.identifier(path.node.property.value);
            }
        },
        // 折叠常量表达式
        BinaryExpression(path) {
            if (t.isNumericLiteral(path.node.left) && t.isNumericLiteral(path.node.right)) {
                const result = eval(`${path.node.left.value} ${path.node.operator} ${path.node.right.value}`);
                if (typeof result === 'number' && isFinite(result)) {
                    path.replaceWith(t.numericLiteral(result));
                }
            }
        }
    });

    return generate(ast, { comments: false }).code;
}
```

完整 AST 模式见 `assets/ast-patterns/` 目录。

## 2. 控制流平坦化 (CFF) 还原

**识别特征**：

```javascript
var state = initialState;
while (true) {
    switch (state) {
        case 'A': /* ... */ state = 'C'; break;
        case 'B': /* ... */ state = 'D'; break;
        case 'C': /* ... */ state = 'B'; break;
    }
}
```

**还原步骤**：
1. 找到初始状态值。
2. 按状态转移顺序排列代码块。
3. 去掉 switch-case 包装，还原为顺序代码。
4. 简化多余的变量赋值。

**ruyiPage 辅助**：

```
ruyiPage: set_breakpoint_via_hook(target_function="状态机入口函数")
ruyiPage: trace_function(function_path="状态机函数", log_args=true, log_return=true)
ruyiPage: get_trace_data → 查看状态值变化
```

## 3. eval / Function 打包还原

**识别特征**：

```javascript
eval(function(p,a,c,k,e,d){...}('encoded_string',...))
// 或
new Function('return ' + decryptedCode)()
```

**还原步骤**：
1. Hook `eval` 和 `Function` 构造器，拦截实际执行的代码。
2. 或者将 `eval()` 替换为 `console.log()` 查看解密后的代码。
3. 可能有多层嵌套，需要逐层解包。

**Hook 注入**（见 `references/hooks/hook-templates.md`）：

```javascript
// eval/Function Hook
const originalEval = window.eval;
window.eval = function(code) {
    console.log('[eval intercepted]', code);
    return originalEval.call(this, code);
};

const OriginalFunction = window.Function;
window.Function = function(...args) {
    console.log('[Function intercepted]', args.join('\n'));
    return new OriginalFunction(...args);
};
```

## 4. AAEncode / JJEncode / JSFuck

这三类都是编码型混淆，还原方式一致：

- **AAEncode**：全是日文颜文字字符 `ﾟωﾟﾉ`。
- **JJEncode**：全是 `$` 和特殊字符。
- **JSFuck**：仅使用 `[]()!+` 六种字符。

**还原**：直接在浏览器 console 执行，或去掉最外层执行函数改为 `console.log` 输出。

## 5. 自定义 VM / 字节码解释器

**识别特征**：
- 超大数组作为"字节码"。
- 解释器循环，包含 `switch` 或函数查找表。
- 通常在 IIFE 中。
- 无法通过简单字符串替换还原。

**还原策略**：
1. **不要尝试反编译字节码**。
2. 找到解释器的输入和输出接口。
3. 通过 Hook 解释器的关键操作（函数调用、赋值、返回）来理解行为。
4. 直接在 Node.js 中运行字节码解释器。

**RuyiTrace 辅助**：

```
ruyiPage: trace_function(function_path="解释器函数", log_args=true, log_return=true)
ruyiPage: get_trace_data → 观察每步操作的输入输出
ruyiPage: set_breakpoint_via_hook(target_function="解释器核心函数") → 捕获关键调用
```

RuyiTrace 模式下，可捕获 C++ 内核级 NDJSON 轨迹，记录每个属性访问、函数调用、赋值操作，适合深度分析字节码执行流程。

## 6. JSVMP（JS 虚拟机保护）

**识别特征**：
- 超大 JS 文件（200KB+）。
- 包含自定义解释器和操作码表。
- 函数名和变量名完全无意义。
- 改写浏览器原生 API。

**还原策略**：
1. **不要反编译，通过 I/O 定位**。
2. Hook 所有出口（XHR、Cookie、fetch）。
3. 追踪加密函数的输入和输出。
4. 用已知 I/O 反推算法。

**RuyiTrace 黄金路径**：

```
ruyiPage: inject_hook_preset(preset="xhr") → 一键 Hook XHR 请求
ruyiPage: get_request_initiator(request_id=N) → 获取请求的 JS 调用栈（黄金路径）
ruyiPage: add_init_script → 注入全局 Hook
```

JSVMP 场景必须使用 RuyiTrace 模式，参见 `references/workflow/trace-flow.md`。

## ruyiPage 工作流

混淆代码分析的 ruyiPage 标准工作流：

```
1. save_script → 保存混淆代码到本地
2. search_code → 搜索可能的入口函数
3. set_breakpoint_via_hook → 在入口设伪断点
4. get_breakpoint_data → 查看捕获的参数和返回值
5. evaluate_js → 在浏览器执行还原操作
6. trace_function → 追踪关键函数调用链
```

## AST 反混淆工具链

`assets/ast-patterns/` 目录提供分层 AST 还原流水线，所有脚本位于 `assets/ast-patterns/scripts/`：

| 脚本 | 用途 |
|---|---|
| `detect-patterns.js` | 模式检测，判断命中的站点或混淆家族 |
| `run-pipeline.js` | 流水线入口，执行选中的步骤并输出报告 |
| `inline-literals.js` | 字面量内联（字符串表、十六进制、计算属性转点号） |
| `normalize-structure.js` | 结构标准化（逗号表达式、IIFE、语句提升） |
| `inline-dispatchers.js` | dispatcher 对象内联 |
| `flatten-array-control-flow.js` | 数组驱动控制流平坦化还原 |
| `if-chain-to-switch.js` | `if (literal === opcode)` 链转 switch |
| `prune-fake-branches.js` | 虚假常量分支清理（死代码消除） |
| `rename-identifiers.js` | 变量重命名 |
| `collect-residue-metrics.js` | 残留症状统计 |
| `compare-with-reference.js` | 与参考产物对比 |
| `pipeline-config.js` | 流水线配置项 |
| `patterns/` 子目录 | 站点专用 pass（reese84/dingxiang/geetest4/tonghuashun/yidun/xiaohongshu/OB 变种） |

使用方式：

```bash
# 1. 检测模式
node assets/ast-patterns/scripts/detect-patterns.js input.js [hint]

# 2. 执行流水线（自动选择命中家族的专用 pass）
node assets/ast-patterns/scripts/run-pipeline.js input.js output-dir [hint]

# 3. 查看残留症状
node assets/ast-patterns/scripts/collect-residue-metrics.js output-dir/decoded.js
```

详细规则文档见 `assets/ast-patterns/` 下的 `pattern-layering.md`、`safe-rewrite-rules.md`、`string-array-and-minimal-eval.md`、`control-flow-and-opcode-patterns.md`、`sequence-normalization.md`。

## 输出要求

阶段报告记录：

```markdown
## 混淆识别与还原

- 混淆类型：OB / CFF / eval / AAEncode / JJEncode / JSFuck / 自定义 VM / JSVMP
- 还原策略：AST 还原 / Hook 拦截 / 直接执行 / trace 反推
- 还原范围：全文件 / 入口部分 / I/O 接口
- 使用工具：AST 脚本 / ruyiPage trace / RuyiTrace
- 还原后可读性：高 / 中 / 低
- 是否影响参数生成：是 / 否
```
