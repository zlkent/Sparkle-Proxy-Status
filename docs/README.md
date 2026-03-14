## Sparkle Chrome 扩展（独立目录）

本仓库根目录的 `extension/` 是一个 Chrome MV3 扩展（不再放在 Sparkle 源码仓库里），用于调用 Sparkle 本机的只读 Extension API：

- `GET http://127.0.0.1:<port>/ext/v1/connection?url=...`
- Header：`Authorization: Bearer <token>`

### 安装

1. 打开 `chrome://extensions`
2. 开启「开发者模式」
3. 点击「加载已解压的扩展程序」，选择本仓库的 `extension/` 目录

### 配置

Sparkle：设置 → 更多设置 → 浏览器扩展 API

- 开启开关
- 复制 Token（或重置）
- 端口默认 `14123`
- 建议把「允许的 Origin」设置为 `chrome-extension://<你的扩展ID>`

扩展：打开 Options

- Base URL：`http://127.0.0.1:14123`
- Token：粘贴 Sparkle 里的 token（不需要写 `Bearer`）

