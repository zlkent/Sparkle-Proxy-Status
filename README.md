# Sparkle Proxy Status

适配 Sparkle 的 Chrome MV3 扩展，用来查看当前页面是否走代理、命中了哪条规则、最终使用了哪条代理链路。

扩展通过 Sparkle 提供的本地只读 Extension API 查询连接状态，不改系统设置，不写代理配置，只做读取和展示。

## 使用前说明

这个扩展必须配合修改版的 Sparkle 客户端使用，不能使用官方版 Sparkle。

修改版 Sparkle 下载与源码地址：

`https://github.com/zlkent/sparkle`

在 Sparkle 中开启功能的位置：

`设置 -> 更多设置 -> 浏览器扩展 API`

## 功能

- 查看当前标签页是否走代理
- 显示命中的规则和代理链路
- 支持选项页配置本地 API 地址和 Bearer Token
- 支持后台预拉取和角标状态提示
- 只访问本机 `127.0.0.1` 的 Sparkle API

## 仓库结构

这个仓库只保留发布 Chrome 扩展需要的内容：

- `extension/`：扩展源文件

以下目录仅用于本地开发或上游代码同步，不会提交到这个发布仓库：

- `docs/`
- `scripts/`
- `upstream-sparkle/`

## 本地加载

1. 打开 `chrome://extensions`
2. 开启“开发者模式”
3. 点击“加载已解压的扩展程序”
4. 选择本仓库里的 `extension/` 目录

## Sparkle 侧配置

这里必须使用修改版 Sparkle，不能使用官方版本。

修改版下载与源码：

`https://github.com/zlkent/sparkle`

在 Sparkle 中打开：

`设置 -> 更多设置 -> 浏览器扩展 API`

然后完成下面几步：

1. 开启浏览器扩展 API
2. 确认端口，默认一般是 `14123`
3. 复制 Token
4. 建议把允许的 Origin 设置为 `chrome-extension://<你的扩展ID>`

## 扩展侧配置

打开扩展的 Options 页面，填写：

- `Extension API Base URL`：`http://127.0.0.1:14123`
- `Bearer Token`：直接粘贴 Sparkle 里的 token，不要自己再加 `Bearer `

填完后点击“测试连接”。

## 如何打包

Chrome Web Store 要求 zip 包根目录直接包含 `manifest.json`。  
所以不要把整个仓库打包，也不要把 `extension/` 文件夹本身包成一层壳。

正确做法是把 `extension/` 目录里面的文件打进 zip。

### 命令行

在仓库根目录执行：

```bash
cd extension
zip -r ../extension.zip .
```

生成的 `extension.zip` 就可以直接上传。

### 你应该看到的 zip 根目录

- `manifest.json`
- `popup.html`
- `popup.js`
- `options.html`
- `options.js`
- `service_worker.js`

## 每次发布前检查

1. 确认 Sparkle 的浏览器扩展 API 已开启
2. 确认 token 可用
3. 确认弹窗能正确显示代理状态
4. 确认设置页“测试连接”通过
5. 发布前更新 `extension/manifest.json` 里的版本号

## 说明

- 扩展权限只访问 `http://127.0.0.1/*`
- 鉴权头格式是 `Authorization: Bearer <token>`
- 这是只读状态查询，不负责改代理规则
