# Sparkle Chrome 插件方案：展示当前页面是否走代理及代理链

## 1. 目标

基于 Sparkle 现有 Mihomo 连接数据，新增一个 Chrome MV3 插件，在浏览网页时实时展示：

- 当前标签页是否命中代理（DIRECT / PROXY）
- 命中的代理链（`chains`）
- 命中的规则信息（`rule` / `rulePayload`）
- 最近更新时间与匹配状态

## 2. 现状与可复用能力

Sparkle 已具备所需核心字段：

- 连接详情页展示了 `规则` 与 `代理链`。
- 连接列表卡片可展示链路中的节点信息。
- 类型定义中 `ControllerConnectionDetail` 已包含：
  - `chains: string[]`
  - `rule: string`
  - `rulePayload: string`
  - `metadata.host / sniffHost / destinationIP / remoteDestination`

结论：**数据已具备，关键在于提供给扩展的查询接口与匹配逻辑。**

## 3. 总体架构

```text
Chrome Tab URL/Host
      │
      ▼
Chrome Extension (MV3: service worker + popup)
      │  HTTP(Bearer Token)
      ▼
Sparkle Local Extension API (127.0.0.1)
      │
      ▼
Mihomo connections stream/cache
      │
      ▼
match(host/url) -> isProxied + chains + rule
```

## 4. Sparkle 侧改造

### 4.1 新增本地只读接口

建议新增 `Extension API`（仅监听 `127.0.0.1`），例如：

- `GET /ext/v1/connection?url=<encoded_url>`

返回：

```json
{
  "matched": true,
  "host": "www.example.com",
  "isProxied": true,
  "chain": ["🚀 HK", "🔁 Auto"],
  "rule": "DOMAIN-SUFFIX",
  "rulePayload": "example.com",
  "connectionId": "...",
  "timestamp": 1730000000000
}
```

未命中：

```json
{
  "matched": false,
  "host": "www.example.com",
  "isProxied": false,
  "chain": [],
  "timestamp": 1730000000000
}
```

### 4.2 匹配算法（核心）

输入 `url` 后取 `host`，并按优先级匹配活动连接（最近优先）：

1. `metadata.host === host`
2. `metadata.sniffHost === host`
3. `metadata.destinationIP / remoteDestination`（当 host 是 IP 或可解析为 IP 时）

优先选择：

- `isActive = true` 的连接
- `start` 最新

`isProxied` 判定建议：

- `chain` 为空或首节点为 `DIRECT` 视为直连
- 否则为代理

### 4.3 安全设计（必须）

避免任何网页都可探测本机代理行为：

- 要求 `Authorization: Bearer <token>`
- token 在 Sparkle 中生成/保存，可重置
- 可选：增加扩展 ID 白名单校验（`Origin: chrome-extension://<id>`）
- API 仅开放查询，不提供写接口
- 日志中不打印 token 明文

### 4.4 缓存策略

- 主进程维护最近 N 条连接（如 500）
- 匹配时先查活动集合，再查最近关闭集合（可选）
- 响应附带 `timestamp` 以便扩展端做“新鲜度”提示

## 5. Chrome 插件方案（MV3）

目录建议：`extension/`

- `manifest.json`
- `service_worker.js`
- `popup.html`
- `popup.js`
- `options.html` / `options.js`（填写 token 与 API 地址）

### 5.1 权限建议

- `permissions`: `tabs`, `activeTab`, `storage`
- `host_permissions`: `http://127.0.0.1/*`

### 5.2 交互流程

1. 用户点击插件图标，popup 获取当前 tab URL
2. 向 `http://127.0.0.1:<port>/ext/v1/connection?url=...` 发起请求
3. 展示状态：
   - 代理中 / 直连 / 未匹配
   - 代理链（`A > B > C`）
   - 规则与规则参数
   - 更新时间

### 5.3 异常处理

- 本地服务不可达：提示“Sparkle 未运行或扩展 API 未启用”
- 401：提示“token 无效，请在设置页更新”
- 未匹配：展示“当前暂无连接样本（可能页面空闲）”

## 6. 实施步骤（分阶段）

### Phase 1（最小可用）

- Sparkle: 提供 `/ext/v1/connection`
- Extension: popup 展示当前页面代理状态与代理链
- 通过手动 token 完成鉴权

### Phase 2（可用性提升）

- 选项页可配置 token / API 地址
- service worker 监听 `tabs.onUpdated` 做预拉取缓存
- UI 增加状态图标（绿=代理，灰=直连，黄=未匹配）

### Phase 3（增强）

- Sparkle 增加匹配调试信息（命中字段、排序分数）
- 可导出最近站点代理记录（隐私开关）
- 多浏览器兼容（Edge 同 MV3 基本兼容）

## 7. 验收标准

- 打开任意网站，popup 可在 1s 内显示“代理/直连 + 代理链”
- 与 Sparkle “连接详情”中的 `代理链/规则`一致
- Sparkle 未启动、token 错误等异常路径有明确提示
- 未暴露可写能力，接口只读且受 token 保护

## 8. 风险与注意事项

- “当前页面”与“连接采样”存在时间差，可能短时未命中
- 某些站点多域名并发请求，需要按 host 选择最佳连接
- 直连与代理判定依赖 chain 命名规范，建议统一 DIRECT 判定逻辑

## 9. 推荐下一步

1. 先在 Sparkle 主进程实现 `/ext/v1/connection`（只读 + token）
2. 同步创建 `extension/` 最小 MV3 popup
3. 用常见站点（Google/GitHub/国内站）做对比验证

