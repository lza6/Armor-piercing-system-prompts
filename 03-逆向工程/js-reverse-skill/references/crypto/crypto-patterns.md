# 常见加密模式识别与还原（Node.js / Python 双语言）

> **两层识别**：先用「输出特征识别表」从密文长度/字符集判型；判不出时再到源码里 grep「常量指纹速查表」中的魔数，命中即定算法。

## 输出特征识别表（看密文）

| 特征 | 可能的算法 | 验证方法 |
|------|-----------|---------|
| 32位十六进制 | MD5 | 用已知输入验证 |
| 40位十六进制 | SHA-1 | 同上 |
| 64位十六进制 | SHA-256 | 同上 |
| 128位十六进制 | SHA-512 | 同上 |
| `=` 结尾，含 `A-Za-z0-9+/` | Base64 | 直接 atob 解码 |
| `=` 结尾，含 `A-Za-z0-9-_` | Base64url | 替换 `-_` 为 `+/` 后解码 |
| 固定长度密文，16字节倍数 | AES (128-bit key) | 寻找 key/iv |
| 固定长度密文，8字节倍数 | DES / 3DES | 寻找 8字节 key |
| 超长数字字符串 | RSA | 寻找公钥 |
| `CryptoJS` 关键词 | CryptoJS 库 | 直接搜索源码 |

## 响应体编码识别表（看响应 body）

> **何时用**：目标接口响应体不是明文 JSON（`code` 非 0/非 1、`data` 是二进制或乱码）时，先判 body 的编码特征再套解密算法。别一看到源码里有 AES 密钥串就默认 body 是 AES。

| 特征 | 含义 | 处理 |
|---|---|---|
| base64 解码后 `1f 8b` 开头 | gzip 压缩 | gunzip 解压 |
| base64 解码后 `78 9c` / `78 01` 开头 | zlib 压缩 | inflate 解压 |
| 长度 `mod16 == 0` | 可能是 AES-ECB/CBC | 找 key/iv，但需确认密钥确实作用于该字段 |
| 长度 `mod16 != 0` | **基本排除 AES-ECB** | 转查压缩/位移/异或/自定义编码，别截断凑块 |
| 仅字母错位的乱码（每字母偏移） | 凯撒位移 | 位移量常在 `code` 字段（如 `code - 100`） |
| `code` 字段非 0 且落在某个区间 | 可能是加密容器/模式标识 | 先搜前端 `if(code...)` 分支，别当错误码 |

> **排查顺序**：先看响应拦截器 `code` 分支 → 再判 data 编码特征（magic number / mod16）→ 最后套算法。**反向禁止**：不要先假设"code 是错误码"或"data 是 AES"，那会把人引向 TLS/风控/参数长度的无效排查。

## 常量指纹速查表（看源码）

> **何时用**：输出特征判不出算法、或代码混淆严重读不清逻辑时，直接在源码里 grep 下列魔数。命中即高度疑似对应算法，再拿标准实现验证。
> **比读上下文判断更可靠**：魔数是算法的数学指纹，混淆器改变量名/控制流但改不掉常量。

### 哈希类

| 魔数 / 指纹 | 算法 | grep 关键词 | 备注 |
|---|---|---|---|
| `0x67452301` `0xefcdab89` `0x98badcfe` `0x10325476` | MD5 | `0x67452301\|0xefcdab89` | 4 个初始化向量（小端序） |
| `0x67452301` `0xefcdab89` `0x98badcfe` `0x10325476` `0xc3d2e1f0` | SHA-1 | `0xc3d2e1f0` | 比 MD5 多第 5 个 IV，单独 grep 它即可区分 |
| `0x6a09e667` `0xbb67ae85` `0x3c6ef372` `0xa54ff53a` | SHA-256 | `0x6a09e667\|0xbb67ae85` | 8 个 IV 的前两个 |
| `0x5be0cd19` `0x1f83d9ab` | SHA-512 | `0x5be0cd19` | 64 位 IV |
| `0x9e3779b9` | XXTEA / TEA 系列 | `0x9e3779b9` | 黄金分割常量 delta，TEA/XTEA/XXTEA 共用 |
| `0x0123456789abcdeffedcba9876543210` | SM3 | `0x0123456789abcdef` | 国密哈希 IV |

### 对称加密类

| 魔数 / 指纹 | 算法 | grep 关键词 | 备注 |
|---|---|---|---|
| 256 元素数组初始化 + 两两交换循环 | RC4 | `Array.*256\|for.*256.*swap` | S-box 初始化是 RC4 最强指纹 |
| `0x63` `0x7c` `0x77` `0x7b`（S-box 首行） | AES | `0x63.*0x7c.*0x77` | AES S-box 前 4 字节 |
| `0xc66363a5` `0xf87c7c84` `0xee777799` | AES | `0xc66363a5` | Te0 表前 3 项（T-table 优化实现） |
| `0xd6` `0x90` `0xe9` `0xfe` | SM4 | `0xd6.*0x90.*0xe9` | SM4 S-box 前 4 字节 |
| `0x0123456789abcdeffedcba9876543210` | SM4 | `0x0123456789abcdef` | SM4 系数 FK（与 SM3 共用） |
| `0x9e3779b9` | XTEA | `0x9e3779b9` | 与 XXTEA 共用 delta，需看循环结构区分 |
| `0x9e3779b9` + 32 轮循环 | TEA | `0x9e3779b9` | TEA 是 32 轮，XTEA 也是 32 轮但密钥调度不同 |
| `0x61c8864700000000`（近似 `0x9e3779b9` 的高位） | XXTEA（64位） | `0x61c88647` | 64 位版本的 delta |

### 非对称加密类

| 魔数 / 指纹 | 算法 | grep 关键词 | 备注 |
|---|---|---|---|
| `10001`（65537） | RSA 公钥指数 | `0x10001\|65537` | 最常见的公钥指数 e |
| 大素数 `p` / `q` 硬编码 | RSA | 看 `BigInteger\|bigint` 上下文 | 自定义 RSA 常硬编码密钥 |

### 编码类

| 指纹 | 算法 | grep 关键词 | 备注 |
|---|---|---|---|
| `ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/` | 标准 Base64 | `ABCDEFGH.*abcdef` | 标准字母表 |
| `ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_` | Base64url | `ABCDEFGH.*-_` | URL 安全变体 |
| 任意 64 字符的可打印字符串 | 自定义 Base64 | 找 64 字符常量字符串 | 字母表被替换，需提取该表 |
| `0123456789abcdef`（16 字符） | Hex | `0123456789abcdef` | 十六进制编码表 |
| `ABCDEFGHIJKLMNOPQRSTUVWXYZ234567` | Base32 | `ABCDEFGH.*234567` | 32 字符表 |

### 识别流程

```
1. 先看输出特征（密文长度/字符集）→ 命中则直接验证
2. 判不出 → grep 源码中的常量指纹
   - 先 grep 0x9e3779b9（TEA/XXTEA，最常见）
   - 再 grep MD5/SHA 的 IV
   - 再 grep AES/SM4 的 S-box
   - 再找 64 字符字符串（自定义 Base64）
3. 命中 → 用标准实现跑同一输入，输出一致即确认
4. 不一致 → 可能是自定义变体（如魔改 MD5 的 chrsz、自定义 S-box），降级到 vm 沙箱执行原实现
```

### 注意事项

- **魔数可能被拆分**：混淆器会把 `0x67452301` 拆成 `0x67 << 24 | 0x45 << 16 | ...`，grep 不到完整常量时搜片段
- **多算法组合**：签名常是 `md5(aes(data, key) + salt)`，需分别识别
- **标准实现验证失败 ≠ 算法判断错误**：可能是参数拼接顺序/编码/填充不同，先排查输入再怀疑算法类型
- **常量指纹是充分不必要条件**：有 `0x9e3779b9` 不一定是 TEA（可能是无关常量），但 TEA 一定有 `0x9e3779b9`

## 1. MD5

```javascript
// Node.js
const crypto = require('crypto');
function md5(str) {
    return crypto.createHash('md5').update(str).digest('hex');
}
```

```python
# Python
import hashlib
def md5(text: str) -> str:
    return hashlib.md5(text.encode('utf-8')).hexdigest()
```

**自定义 MD5**（非标准实现）：某些网站修改了 MD5 的内部参数（如 chrsz=16），导致输出与标准 MD5 不同。识别：用相同输入对比标准 MD5 输出，不一致则为自定义实现。还原：必须提取原始 JS 实现，在 Node.js 中直接执行。

**常见签名模式**：
```javascript
// 模式1：简单拼接
sign = md5(param1 + param2 + timestamp + secret)

// 模式2：排序拼接
params = Object.keys(data).sort().map(k => k + '=' + data[k]).join('&')
sign = md5(params + secret)

// 模式3：嵌套哈希
sign = md5(md5(password) + timestamp)
```

## 2. HMAC

```javascript
// Node.js
const crypto = require('crypto');
function hmacSha256(message, secret) {
    return crypto.createHmac('sha256', secret).update(message).digest('hex');
}
```

```python
# Python
import hmac, hashlib
def hmac_sha256(message: str, secret: str) -> str:
    return hmac.new(secret.encode('utf-8'), message.encode('utf-8'), hashlib.sha256).hexdigest()
```

**识别特征**：搜索 `HMAC`、`createHmac`、`CryptoJS.HmacSHA256`

## 3. AES

### AES-CBC
```javascript
// Node.js
const crypto = require('crypto');
function aesEncrypt(plaintext, key, iv) {
    const cipher = crypto.createCipheriv('aes-128-cbc', key, iv);
    cipher.setAutoPadding(true); // PKCS7
    let encrypted = cipher.update(plaintext, 'utf8', 'base64');
    encrypted += cipher.final('base64');
    return encrypted;
}
```

```python
# Python
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad

def aes_cbc_encrypt(plaintext: str, key: str, iv: str) -> str:
    cipher = AES.new(key.encode('utf-8'), AES.MODE_CBC, iv.encode('utf-8'))
    padded = pad(plaintext.encode('utf-8'), AES.block_size)
    return base64.b64encode(cipher.encrypt(padded)).decode('utf-8')
```

### CryptoJS 兼容
```javascript
const CryptoJS = require('crypto-js');
const encrypted = CryptoJS.AES.encrypt(plaintext, CryptoJS.enc.Utf8.parse(key), {
    iv: CryptoJS.enc.Utf8.parse(iv),
    mode: CryptoJS.mode.CBC,
    padding: CryptoJS.pad.Pkcs7
}).toString();
```

**关键参数识别**：
- key 长度：16字节=AES-128，24字节=AES-192，32字节=AES-256
- iv 长度：固定16字节
- 模式：CBC（需要iv）、ECB（不需要iv）、CTR、GCM
- 填充：PKCS7（最常见）、ZeroPadding、NoPadding
- 输出格式：Base64（最常见）、Hex

## 4. DES / 3DES

```javascript
// Node.js
function desEncrypt(plaintext, key) {
    const cipher = crypto.createCipheriv('des-ecb', key, null); // 8字节key
    cipher.setAutoPadding(true);
    let encrypted = cipher.update(plaintext, 'utf8', 'base64');
    encrypted += cipher.final('base64');
    return encrypted;
}
```

```python
# Python（以下展示 3DES-CBC 变体）
import base64
from Crypto.Cipher import DES, DES3
from Crypto.Util.Padding import pad

def triple_des_cbc_encrypt(plaintext: str, key: str, iv: str) -> str:
    cipher = DES3.new(key.encode('utf-8'), DES3.MODE_CBC, iv.encode('utf-8'))
    padded = pad(plaintext.encode('utf-8'), DES3.block_size)
    return base64.b64encode(cipher.encrypt(padded)).decode('utf-8')
```

## 5. RSA

```javascript
// Node.js
const crypto = require('crypto');
function rsaEncrypt(plaintext, publicKey) {
    const buffer = Buffer.from(plaintext, 'utf8');
    const encrypted = crypto.publicEncrypt({
        key: publicKey,
        padding: crypto.constants.RSA_PKCS1_PADDING
    }, buffer);
    return encrypted.toString('base64');
}
```

```python
# Python
import base64
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_v1_5

def rsa_encrypt(plaintext: str, public_key_pem: str) -> str:
    key = RSA.import_key(public_key_pem)
    cipher = PKCS1_v1_5.new(key)
    encrypted = cipher.encrypt(plaintext.encode('utf-8'))
    return base64.b64encode(encrypted).decode('utf-8')
```

**常见 RSA 公钥格式**：
- PEM 格式（`-----BEGIN PUBLIC KEY-----`）
- 模数(n) + 指数(e) 格式（需要手动构造 PEM）

## 6. Base64 变体

```javascript
// Node.js
const encoded = Buffer.from(str).toString('base64');

// Base64url（URL安全变体）
function base64url(str) {
    return Buffer.from(str).toString('base64')
        .replace(/\+/g, '-').replace(/\//g, '_').replace(/=/g, '');
}

// 自定义字符表 Base64
function customBase64(str, table) {
    const std = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
    const b64 = Buffer.from(str).toString('base64');
    return b64.split('').map(c => {
        const idx = std.indexOf(c);
        return idx >= 0 ? table[idx] : c;
    }).join('');
}
```

```python
# Python
import base64
encoded = base64.b64encode(text.encode('utf-8')).decode('utf-8')
encoded_url = base64.urlsafe_b64encode(text.encode('utf-8')).decode('utf-8').rstrip('=')
```

## 7. 异或加密（XOR）

```javascript
// Node.js
function xorEncrypt(plaintext, key) {
    const result = [];
    for (let i = 0; i < plaintext.length; i++) {
        result.push(plaintext.charCodeAt(i) ^ key.charCodeAt(i % key.length));
    }
    return Buffer.from(result).toString('hex');
}
```

## 8. RC4

> **快速识别**：源码里出现 `Array(256)` 初始化 + 两两交换循环 → 高度疑似 RC4（见常量指纹速查表）。

```javascript
// Node.js
function rc4(data, key) {
    const s = Array.from({ length: 256 }, (_, i) => i);
    let j = 0;
    for (let i = 0; i < 256; i++) {
        j = (j + s[i] + key.charCodeAt(i % key.length)) % 256;
        [s[i], s[j]] = [s[j], s[i]];
    }
    let i = 0; j = 0;
    const result = [];
    for (let k = 0; k < data.length; k++) {
        i = (i + 1) % 256;
        j = (j + s[i]) % 256;
        [s[i], s[j]] = [s[j], s[i]];
        result.push(data.charCodeAt(k) ^ s[(s[i] + s[j]) % 256]);
    }
    return Buffer.from(result);
}
```

```python
# Python（使用 pycryptodome）
from Crypto.Cipher import ARC4
def rc4_encrypt(data: str, key: str) -> bytes:
    cipher = ARC4.new(key.encode('utf-8'))
    return cipher.encrypt(data.encode('utf-8'))
```

## 9. 时间戳处理

```javascript
// Node.js
const tsMs = Date.now();                      // 毫秒级（13位）
const tsSec = Math.floor(Date.now() / 1000);  // 秒级（10位）
```

```python
# Python
import time
ts_ms = int(time.time() * 1000)  # 毫秒级（13位）
ts_sec = int(time.time())        # 秒级（10位）
```

**注意**：某些网站使用服务端时间戳，需要从响应头或接口获取。

## 10. 参数签名常见拼接模式

```javascript
// 模式1：固定格式
sign = md5(`page=${page}&t=${timestamp}&key=${secret}`)

// 模式2：所有参数排序拼接
const params = { page: 1, size: 10, t: Date.now() };
const str = Object.keys(params).sort().map(k => `${k}=${params[k]}`).join('&');
sign = md5(str + secret)

// 模式3：JSON 字符串
sign = md5(JSON.stringify(data) + secret)

// 模式4：管道分隔
sign = md5(`${page}|${timestamp}|${secret}`)
```

```python
# Python
import hashlib, json, time

def md5(text: str) -> str:
    return hashlib.md5(text.encode('utf-8')).hexdigest()

# 模式2：所有参数排序拼接
params = {"page": 1, "size": 10, "t": int(time.time())}
param_str = "&".join(f"{k}={params[k]}" for k in sorted(params))
sign = md5(param_str + secret)

# 模式3：JSON 字符串（注意 separators 控制格式）
sign = md5(json.dumps(data, separators=(',', ':'), sort_keys=True) + secret)
```
