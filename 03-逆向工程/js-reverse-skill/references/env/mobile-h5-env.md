# 移动端 H5 环境补全

> **触发条件**：目标 UA 为移动端浏览器（Mobile Safari / Chrome for Android / 微信 X5 / QQ B / UC 等）时读。需要补移动端专属 API、screen/viewport、TLS 指纹对齐时也读。

## 适用范围

移动端 H5（手机浏览器访问的网页）是 Web 场景，完全在本 skill 能力范围内。包括：
- Mobile Safari（iPhone / iPad）
- Chrome for Android
- 微信内置浏览器（X5 内核 / iOS WKWebView）
- QQ 浏览器（QB）
- UC 浏览器
- 小米 / 华为 / OPPO / vivo 自带浏览器

**不适用**：App 内 JS（React Native bundle / JSCore）、小程序容器 JS（微信/支付宝/百度小程序）。这些运行在非浏览器 JS 引擎中，API 集与浏览器差异大，超出 skill 边界。

## 移动端 UA 矩阵

### Mobile Safari（iPhone）

**UA 示例**：
```
Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1
```

**必补**（jsdom 缺失但真实 Mobile Safari 有）：

- ✅ `window.DeviceMotionEvent` / `DeviceOrientationEvent`（构造函数 + 事件）
- ✅ `window.TouchEvent` / `window.PointerEvent`
- ✅ `window.orientation`（iOS 独有，已废弃但仍被检测）
- ✅ `screen.orientation`（`{ type: 'portrait-primary', angle: 0 }`）
- ✅ `navigator.standalone`（PWA 模式检测，boolean）
- ✅ `navigator.maxTouchPoints`（iOS 通常 5）
- ✅ `window.webkit.messageHandlers`（iOS WKWebView 桥接，H5 中常为 `{}`）
- ✅ `document.ontouchstart` / `document.ontouchmove` 等（通常为 null）
- ✅ `Animation`（iOS 17+）
- ✅ `VisualViewport`（含 `offsetLeft`/`offsetTop`/`pageLeft`/`pageTop`）

**禁补**（Mobile Safari 没有）：

- ❌ `window.chrome`（Safari 无 chrome 对象）
- ❌ `navigator.userAgentData`（Chrome Client Hints，Safari 不支持）
- ❌ `navigator.connection`（Safari 不支持 Network Information API）
- ❌ `navigator.getBattery`（Safari 不支持）
- ❌ `performance.memory`（Safari 不支持）

### Chrome for Android

**UA 示例**：
```
Mozilla/5.0 (Linux; Android 13; Pixel 7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Mobile Safari/537.36
```

**必补**（与桌面 Chrome 重合 + 移动端额外）：

- ✅ `window.DeviceMotionEvent` / `DeviceOrientationEvent`
- ✅ `window.TouchEvent` / `window.PointerEvent`
- ✅ `screen.orientation`
- ✅ `navigator.maxTouchPoints`（Android 通常 5-10）
- ✅ `navigator.userAgentData`（含 `mobile: true`）
- ✅ `navigator.connection`（`{ effectiveType: '4g', rtt: 50, downlink: 10, saveData: false }`）
- ✅ `navigator.getBattery`（Android Chrome 支持）
- ✅ `window.chrome`（与桌面 Chrome 一致）

**禁补**（Mobile Chrome 没有）：

- ❌ `navigator.standalone`（iOS PWA 独有）
- ❌ `window.webkit.messageHandlers`（iOS 独有）

### 微信内置浏览器（X5 / WKWebView）

**UA 示例（Android X5）**：
```
Mozilla/5.0 (Linux; Android 13; ...) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/86.0.4240.99 MQQBrowser/6.2 TBS/046831 Mobile Safari/537.36 MMWEBID/xxxx MicroMessenger/8.0.40.2420(0x28002834) WeChat/arm64 WeChat GPUtil/1.1.2 WeChat/8.0.40(0x28002834) FloraLanguage/en
```

**UA 示例（iOS WKWebView）**：
```
Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Mobile/15E148 MicroMessenger/8.0.40(0x1800282f) NetType/WIFI Language/en
```

**必补**（微信容器专属）：

- ✅ `window.wx` / `window.WeixinJSBridge`（微信 JS-SDK 桥接）
- ✅ `window.WeixinJSBridge.invoke` / `window.WeixinJSBridge.on`
- ✅ `window.__wxjs_environment`（值为 `'miniprogram'` 或 undefined）
- ✅ `navigator.userAgent` 含 `MicroMessenger/x.x.x`
- ✅ `navigator.language` / `navigator.languages`
- ✅ X5 内核专属：`window.QBao` / `window.browser`（部分版本）

**iOS 微信差异**：
- WKWebView 内核，性能与 Safari 一致
- `window.webkit.messageHandlers` 存在且含微信专属 handlers
- 不支持 `window.WeixinJSBridge` 的部分 Android X5 接口

### QQ 浏览器（QB）

**UA 示例**：
```
Mozilla/5.0 (Linux; Android 13; ...) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/86.0.4240.99 MQQBrowser/6.2 TBS/046831 Mobile Safari/537.36
```

**必补**：
- ✅ `window.QBao`（QB 专属对象）
- ✅ `window.browser`（QB 版本信息）
- ✅ `navigator.userAgent` 含 `MQQBrowser/x.x.x`

## 移动端 screen / viewport fixture

```javascript
// iPhone 14 Pro
const iPhoneScreen = {
    width: 393,
    height: 852,
    availWidth: 393,
    availHeight: 852,
    colorDepth: 32,
    pixelDepth: 32,
    orientation: { type: 'portrait-primary', angle: 0 },
};

const iPhoneViewport = {
    devicePixelRatio: 3,
    innerWidth: 393,
    innerHeight: 654,    // 减去地址栏和工具栏
    outerWidth: 393,
    outerHeight: 852,
    scrollX: 0,
    scrollY: 0,
    visualViewport: {
        width: 393,
        height: 654,
        offsetLeft: 0,
        offsetTop: 0,
        pageLeft: 0,
        pageTop: 0,
        scale: 1,
    },
};

// Android Pixel 7
const androidScreen = {
    width: 412,
    height: 915,
    availWidth: 412,
    availHeight: 915,
    colorDepth: 24,
    pixelDepth: 24,
    orientation: { type: 'portrait-primary', angle: 0 },
};

const androidViewport = {
    devicePixelRatio: 2.625,
    innerWidth: 412,
    innerHeight: 847,
    outerWidth: 412,
    outerHeight: 915,
};
```

## 移动端专属 API 补全

### DeviceMotion / DeviceOrientation

```javascript
// 重力感应/陀螺仪事件
class DeviceMotionEvent extends Event {
    constructor(type, init = {}) {
        super(type, init);
        this.acceleration = init.acceleration || { x: null, y: null, z: null };
        this.accelerationIncludingGravity = init.accelerationIncludingGravity ||
            { x: 0, y: -9.8, z: 0 };
        this.rotationRate = init.rotationRate || { alpha: null, beta: null, gamma: null };
        this.interval = init.interval || 16;
    }

    static requestPermission() { return Promise.resolve('granted'); }
}

class DeviceOrientationEvent extends Event {
    constructor(type, init = {}) {
        super(type, init);
        this.absolute = init.absolute || false;
        this.alpha = init.alpha || 0;
        this.beta = init.beta || 0;
        this.gamma = init.gamma || 0;
        this.webkitCompassHeading = init.webkitCompassHeading || 0;
    }
}

// 注册到 window
win.DeviceMotionEvent = DeviceMotionEvent;
win.DeviceOrientationEvent = DeviceOrientationEvent;
```

### Touch / Pointer 事件

```javascript
class Touch {
    constructor(init = {}) {
        this.identifier = init.identifier || 0;
        this.target = init.target || null;
        this.clientX = init.clientX || 0;
        this.clientY = init.clientY || 0;
        this.pageX = init.pageX || 0;
        this.pageY = init.pageY || 0;
        this.screenX = init.screenX || 0;
        this.screenY = init.screenY || 0;
        this.radiusX = init.radiusX || 1;
        this.radiusY = init.radiusY || 1;
        this.rotationAngle = init.rotationAngle || 0;
        this.force = init.force || 0;
    }
}

class TouchList {
    constructor(touches = []) {
        this._touches = touches;
    }
    get length() { return this._touches.length; }
    item(i) { return this._touches[i] || null; }
    namedItem(name) { return this._touches.find(t => t.identifier == name) || null; }
    [Symbol.iterator]() { return this._touches[Symbol.iterator](); }
}

class TouchEvent extends Event {
    constructor(type, init = {}) {
        super(type, init);
        this.touches = new TouchList(init.touches || []);
        this.targetTouches = new TouchList(init.targetTouches || []);
        this.changedTouches = new TouchList(init.changedTouches || []);
    }
}

win.Touch = Touch;
win.TouchList = TouchList;
win.TouchEvent = TouchEvent;
win.PointerEvent = PointerEvent; // 通常 jsdom 有，按需补

// document.ontouch* 设为 null（表示有触摸支持但无处理器）
doc.ontouchstart = null;
doc.ontouchmove = null;
doc.ontouchend = null;
doc.ontouchcancel = null;
```

### window.orientation

```javascript
// iOS 独有，已废弃但仍被检测
Object.defineProperty(win, 'orientation', {
    get() { return 0; }, // 0=竖屏, 90=左横, -90=右横
    configurable: true,
});

// screen.orientation 是现代标准
Object.defineProperty(win.screen, 'orientation', {
    get() {
        return {
            type: 'portrait-primary',
            angle: 0,
            onchange: null,
            addEventListener: () => {},
            removeEventListener: () => {},
        };
    },
    configurable: true,
});
```

### navigator.connection（Android Chrome）

```javascript
// Network Information API（仅 Android Chrome 支持）
Object.defineProperty(win.navigator, 'connection', {
    get() {
        return {
            effectiveType: '4g',    // 'slow-2g'/'2g'/'3g'/'4g'
            rtt: 50,                // 往返时延 ms
            downlink: 10,           // 下行带宽 Mbps
            saveData: false,        // 节省流量模式
            type: 'wifi',           // 'bluetooth'/'cellular'/'ethernet'/'wifi'/'wimax'/'other'/'unknown'/'none'
            addEventListener: () => {},
            removeEventListener: () => {},
            dispatchEvent: () => false,
        };
    },
    configurable: true,
});
```

### 微信 WeixinJSBridge

```javascript
// 微信容器专属
win.WeixinJSBridge = {
    invoke: function(api, params, callback) {
        // 按需实现：模拟微信原生调用
        if (callback) setTimeout(() => callback({}), 0);
    },
    on: function(event, callback) {
        // 注册事件
    },
    call: function(api) {
        // 调用微信 API
    },
    log: function(msg) {
        console.log('[WeixinJSBridge]', msg);
    },
};

// window.wx（JS-SDK）
win.wx = {
    config: function(opts) {},
    ready: function(fn) { setTimeout(fn, 0); },
    error: function(fn) {},
    // 按需补全业务用到的 API
};

win.__wxjs_environment = undefined; // 非小程序环境为 undefined
```

## 移动端 TLS 指纹对齐

移动端 TLS 指纹与桌面不同，需单独对齐：

| 平台 | TLS 库推荐 | 说明 |
|---|---|---|
| Mobile Safari | `curl_cffi` + `impersonate='safari_ios_16_0'` 或 `safari_ios_17_0` | iOS Safari ja3 |
| Chrome Android | `curl_cffi` + `impersonate='chrome120_android'` | Android Chrome ja3 |
| 微信 X5 (Android) | `cyCronet` 优先（Cronet 内核） | X5 基于 Cronet，curl_cffi 难模拟 |
| 微信 iOS | `curl_cffi` + `impersonate='safari_ios_17_0'` | iOS 微信用 WKWebView |

**cyCronet 适用场景**：
- 微信 X5 / 字节系 Android App 内 WebView
- Chrome Android（Cronet 是 Chrome 的网络栈）
- 需要精确模拟 HTTP/2 Akamai fingerprint 的移动端

详见 `references/network/tls-validation.md` 的 cyCronet 段。

## 自检命令

### Mobile Safari UA 下

```bash
# 不应有 Chrome 独有 API
grep -c "userAgentData\|navigator\.connection\|getBattery\|window\.chrome\|performance\.memory" env-patch.js
# 必须 = 0

# 应有 iOS 独有 API
grep -c "DeviceMotionEvent\|DeviceOrientationEvent\|window\.orientation\|navigator\.standalone\|maxTouchPoints" env-patch.js
# 必须 > 0
```

### Chrome Android UA 下

```bash
# 应有 Chrome + 移动端 API
grep -c "userAgentData\|navigator\.connection\|DeviceMotionEvent\|maxTouchPoints" env-patch.js
# 必须 > 0

# userAgentData.mobile 必须为 true
grep "mobile.*true" env-patch.js
```

### 微信 UA 下

```bash
# 应有微信容器对象
grep -c "WeixinJSBridge\|window\.wx\|__wxjs_environment" env-patch.js
# 必须 > 0

# UA 中应含 MicroMessenger
grep "MicroMessenger" case/notes/headers.json
```

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/env/env-object-model.md` | 桌面端 API 对象模型（移动端在此基础上扩展） |
| `references/fingerprint/fingerprint-baseline-consistency.md` | 指纹基线对比（移动端 fixture） |
| `references/network/tls-validation.md` | TLS 指纹对齐（cyCronet 段） |
| `references/network/ip-risk-control.md` | 移动 IP 风控（移动代理通过率高） |
| `cases/jsvmp-xhr-interceptor-env-emulation.md` | 桌面 UA 分支矩阵（移动端补充见本文档） |
