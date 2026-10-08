# Tiger OpenAPI C++ SDK — Quickstart / 快速入门

> C++ SDK 快速入门 / Quick Start for C++ SDK
> GitHub: https://github.com/tigerfintech/openapi-cpp-sdk

## 依赖 / Dependencies

| 依赖 Dependency | 版本 Version | 说明 |
|----------------|-------------|------|
| C++ | **14**（构建基线） | SDK 以 C++14 编译；使用方项目可用 C++17+ |
| CMake | 3.15+ | 构建系统 |
| Boost | **1.86.0** | thread/log/program_options/chrono/filesystem |
| cpprestsdk | 源码构建（git master） | HTTP client（Microsoft REST SDK） |
| Protobuf | **5.28.3** | 推送消息序列化 |
| Abseil | **20240722.0**（固定版本） | Protobuf 依赖，必须与之匹配 |
| OpenSSL | 3.x 推荐 | TLS 支持 |

> ⚠️ **Abseil 版本必须固定在 20240722.0**。Homebrew 的新版 Abseil 需要 C++17 头文件，
> 与 C++14 基线冲突，会导致编译失败或 `absl::string_view` / `std::string_view` ABI 不匹配。
> Abseil must stay pinned at 20240722.0; newer Homebrew builds require C++17 headers.

---

## 安装依赖 / Install Dependencies

**推荐用仓库自带的构建脚本**，它会按固定版本拉取并编译 Boost / cpprestsdk /
Abseil / Protobuf，避免手动安装造成版本错配。
Use the bundled build scripts — they pin every dependency version.

### macOS / Linux

```bash
git clone https://github.com/tigerfintech/openapi-cpp-sdk.git
cd openapi-cpp-sdk

chmod +x scripts/build_linux_mac.sh
./scripts/build_linux_mac.sh                      # 默认 Debug，SDK 同时产出 Debug+Release
BUILD_TYPE=Release ./scripts/build_linux_mac.sh   # 仅 Release
SKIP_DEMO=1 ./scripts/build_linux_mac.sh          # 跳过 demo
./scripts/build_linux_mac.sh --demo-only          # 只编 demo
```

其他可用环境变量：`SKIP_DEPS=1`、`SKIP_PROTO_REGEN=1`、`NUM_JOBS`、`INSTALL_PREFIX`、
`LOCAL_OPT_PREFIX`（默认 `/usr/local/opt`）。

### Windows

```powershell
.\scripts\build_windows.ps1 -Triplet x64-windows -BuildType Release -Runtime MD
```

`-Triplet` 可选 `x64-windows` / `x86-windows` / `arm64-windows`（及对应 `-static`），
`-Runtime` 可选 `MD` / `MT`；另有 `-SkipDeps` / `-SkipDemo` / `-ProtobufProvider Source`。

也可直接走 MSBuild（8 种配置：`{Debug,Release}-{MD,MT}` × `{x64,Win32}`）：

```powershell
msbuild openapi-cpp-sdk.vcxproj /t:Rebuild /p:Configuration=Release-MD /p:Platform=x64 /m /nr:false
```

### 构建产物 / Build Output

- macOS：`output/Mac/<Config>/{lib,include}`（如 `output/Mac/Release/lib/libtigerapi.a`）
- Linux：`output/Linux/<Config>/{lib,include}`
- Windows：`output/Windows/{x64,Win32}/<Config>/`（`openapi-cpp-sdk.dll` / `.lib`）

仓库也提供预编译包：`output/Mac/{Debug,Release}.zip` 等。

---

## 配置 / Configuration

### 配置文件 / Config File

在项目目录创建 `tiger_openapi_config.properties`：

```properties
tiger_id=your_tiger_id
private_key_pk8=MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJd...
account=your_account_number
```

> C++ SDK 同时支持 **PKCS#8**（`private_key_pk8`）和 **PKCS#1**（`private_key_pk1`）格式，优先读取 `private_key_pk8`。
> 两种格式均可使用裸 Base64 DER 字符串（无需 PEM 头尾行），SDK 会自动识别并添加正确的 PEM 头。
>
> **PKCS#8 转 PKCS#1：**
> ```bash
> openssl rsa -in pkcs8_private.pem -out pkcs1_private.pem
> ```
> **PKCS#1 转 PKCS#8：**
> ```bash
> openssl pkcs8 -topk8 -inform PEM -outform DER -nocrypt -in pk1.pem | base64 | tr -d '\n'
> ```

### 代码加载配置 / Load Config in Code

```cpp
#include "tigerapi/client_config.h"

using namespace TIGER_API;

// 从配置文件目录加载（第一个参数固定传 false）
ClientConfig config(false, "path/to/config/dir/");
```

`ClientConfig` 构造参数：
- 第一个参数（bool）：**固定传 `false`**
- 第二个参数（string_t）：配置文件目录路径

这个构造函数是唯一会自动读取 properties 文件与 token 的重载。
This is the only ctor that loads the properties file and token.

---

## 客户端创建 / Create Clients

```cpp
#include "tigerapi/client_config.h"
#include "tigerapi/quote_client.h"
#include "tigerapi/trade_client.h"
#include "tigerapi/push_client.h"

using namespace std;
using namespace web;
using namespace web::json;
using namespace TIGER_API;

ClientConfig demo_config(false, "demo/openapi_cpp_test/");

// 行情客户端 / Quote client
auto quote_client = make_shared<QuoteClient>(demo_config);

// 交易客户端 / Trade client
auto trade_client = make_shared<TradeClient>(demo_config);

// 推送客户端：用静态工厂，不能 new / use the static factory, not `new`
shared_ptr<IPushClient> push_client = IPushClient::create_push_client(demo_config);
```

---

## 字符串与 JSON / Strings and JSON

C++ SDK 使用 cpprestsdk 的 `web::json::value`，跨平台字符串用 `utility::string_t`：

```cpp
#include "cpprest/details/basic_types.h"
using namespace web::json;

// U() 宏：将字符串字面量转为 string_t（跨平台）
utility::string_t symbol = U("AAPL");

// 构建 JSON 数组 / Build array
value symbols = value::array();
symbols[0] = value::string(U("AAPL"));
symbols[1] = value::string(U("TSLA"));

// 输出：ucout 替代 std::cout
ucout << symbols << std::endl;
```

---

## 直接调用 API / Raw API Call

**优先用 SDK 封装好的方法**，只有 SDK 未封装某个 API 时才退回 `TigerClient::post()`。
Prefer the SDK's first-class wrappers; fall back to `TigerClient::post()` only for
APIs the SDK does not wrap.

```cpp
#include "tigerapi/quote_client.h"

// 推荐：SDK 已封装的接口直接调 wrapper（如市场状态）
// Preferred: call the wrapper when one exists (e.g. market state).
auto qc = make_shared<QuoteClient>(config);
value state = qc->get_market_state(U("US"));
ucout << state << endl;
```

```cpp
#include "tigerapi/tiger_client.h"

// 兜底：SDK 未封装的接口才用裸 post
// Fallback: raw post() for APIs without a wrapper.
// 直接用 TigerClient（QuoteClient/TradeClient 均继承自它）
auto client = make_shared<TigerClient>(config);

value obj = value::object(true);
obj[U("market")] = value::string(U("US"));

// 第一个参数为 API 名称常量，在 service_types.h 中定义
value result = client->post(MARKET_STATE, obj);
ucout << result << endl;
```

常用 API 名称常量（在 **`tigerapi/service_types.h`** 中，共 89 个）：

| 常量 | API |
|------|-----|
| `MARKET_STATE` | 市场状态 |
| `BRIEF` | 实时报价 |
| `KLINE` | K 线 |
| `POSITIONS` | 持仓 |
| `ORDERS` | 订单 |
| `ASSETS` | 资产 |

---

## 工程集成 / CMakeLists.txt Example

在自己的项目中使用已编译好的 SDK（`output/Mac/Release/`）：

```cmake
cmake_minimum_required(VERSION 3.15)
project(my_tiger_app CXX)
# SDK 本身以 C++14 编译；使用方项目可以用 C++17 或更高
set(CMAKE_CXX_STANDARD 17)

find_package(Boost REQUIRED COMPONENTS thread log program_options chrono filesystem)
find_package(cpprestsdk CONFIG REQUIRED)
find_package(Protobuf REQUIRED)
find_package(OpenSSL REQUIRED)

# SDK 头文件和库路径
set(TIGERAPI_DIR /path/to/openapi-cpp-sdk/output/Mac/Release)
include_directories(${TIGERAPI_DIR}/include)
link_directories(${TIGERAPI_DIR}/lib)

add_executable(my_app main.cpp)
target_link_libraries(my_app
    tigerapi
    Boost::thread Boost::log Boost::program_options Boost::chrono Boost::filesystem
    cpprestsdk::cpprest
    ${Protobuf_LIBRARIES}
    OpenSSL::SSL OpenSSL::Crypto
)
```

---

## 错误处理 / Error Handling

```cpp
try {
    value symbols = value::array();
    symbols[0] = value::string(U("AAPL"));
    value result = quote_client->get_brief(symbols);
    ucout << result << endl;
} catch (const std::exception& e) {
    std::cerr << "Error: " << e.what() << std::endl;
} catch (...) {
    std::cerr << "Unknown error occurred" << std::endl;
}
```

---

## 前置条件 / Prerequisites

1. 老虎证券账户 + 开发者 API 权限：https://developer.itigerup.com/
2. 准备好 `tiger_id`、RSA 私钥（2048位）、账户号
3. 行情数据需要对应市场的行情权限
