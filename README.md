# dsh-filepannel-plugin

> DeepSeek Harness (DSH) 工作区文件面板插件 —— 在浏览器界面中管理当前工作区文件的可视化面板。

[English](README.en.md) | [中文](README.md)

一个运行在 DSH Web 界面中的**动态 Cordis 插件**：从屏幕右侧边缘滑出一个文件面板，浏览、上传、下载、预览、编辑、搜索、压缩…… 当前工作区的全部文件操作，都在浏览器里完成。

UI 无 emoji，全部使用 **SVG 线条图标**，配色全面接入 **DSH 主题令牌**（`--dsw-alias-*`），亮/暗主题自动适配。

## 功能一览

| 分类 | 功能 |
| --- | --- |
| 入口 | 屏幕最右边缘热区/标签，鼠标悬停 0.3s 滑出，移出面板 0.5s 自动收起；面板宽度可拖拽调整 |
| 浏览 | 面包屑导航、上级目录、刷新、目录内过滤、目录优先排序 |
| 传输 | **流式上传**（1MB/块、进度条 + 速率 + 可取消、上限 2GB、临时文件原子替换）、下载、浏览器新标签打开 |
| 预览 | PDF（内嵌浏览器阅读器）、图片、文本/代码（行号 + 语法高亮 + 编辑，Ctrl/Cmd+S 保存） |
| 文件操作 | 新建文件/文件夹、重命名、删除（二次确认）、复制路径、移动/复制到其他目录（目录选择器） |
| 批量 | 多选 → 批量下载 / 打包 / 移动 / 复制 / 删除 |
| 搜索 | **全工作区递归搜索**：文件名 + 文本内容匹配，结果一键直达 |
| 压缩 | 选中项打包为 zip；`.zip` 文件一键解压到当前目录 |
| 上传方式 | 按钮选择 + **拖拽上传**（拖入面板即传） |

## 截图

（暂无截图 —— 请打开面板自行体验）

## 工作原理

这是一个 **Host + Client 双端**的动态 Cordis 插件：

- **Host 半部**（`src/host.js`）：
  - 通过 `harness.handle` 注册 15 个 Package-private RPC 方法（列表、搜索、读写、移动/复制、zip/unzip、流式上传等）
  - 在 `webServer` 注册同源下载路由 `/__dsh__/filepanel/download`，带一次性令牌 + 工作区包含校验，支持 `inline` 预览（图片/PDF）与附件下载
  - 所有路径经 `fs.contains` 强制约束在会话工作区内；写操作遵循会话沙箱策略（`workspace-write`）
- **Client 半部**（`src/client.js`）：
  - 注册到 `shell.overlay` 插槽（浮动层，不替换任何自带 UI）
  - React 组件 + 内联 SVG 图标 + DSH 主题令牌，无需构建、无外部依赖

### RPC 方法表（`panel.*`）

| 方法 | 说明 |
| --- | --- |
| `list` | 列出目录（名称/类型/大小/绝对路径） |
| `search` | 递归搜索（文件名 + ≤256KB 文本文件内容；深度 ≤8，结果 ≤200） |
| `readText` / `writeText` | 文本读取（≤512KB 预览）/ 原子写入 |
| `createDir` / `remove` / `rename` | 新建目录 / 删除 / 重命名 |
| `move` / `copy` | 批量移动 / 递归复制（含同名冲突检查） |
| `zip` / `unzip` | 打包（`zip -r`）/ 解压（`unzip -o`） |
| `uploadStart` / `uploadChunk` / `uploadAbort` | 流式上传（base64 分块写入临时文件，完成后原子替换） |
| `token` | 下载路由一次性令牌 |

## 安装与使用

当前版本以 **DSH 动态插件**形式运行（无需改部署配置、无需构建）：

1. 在 DSH 会话中调用 `cordis_define`，提供 `src/host.js` 与 `src/client.js` 两段代码（函数体，不含外层 `export`）。
2. 调用 `cordis_run` 激活（Client 半部首次需在界面中批准）。
3. 激活后：**鼠标移到屏幕最右边缘**悬停片刻，文件面板即从右侧滑出；或点击右侧边缘的文件夹标签。

> 注意：动态插件是进程内临时扩展，重启后需重新定义/运行。若要持久化，可将该插件纳入 DSH 部署的 Cordis 组合（`cordis.yml`）。

## 依赖与限制

- Host 依赖服务：`fs`、`shell`、`webServer`、`sandboxPolicy`（DSH 标准能力，均随部署提供）
- 压缩功能依赖系统命令 `zip` / `unzip`（Linux 常见，未安装时仅打包/解压不可用）
- 上传上限 2GB；预览上限：文本 512KB、图片/PDF 256MB；全局搜索限深度 8、访问节点 4000、结果 200
- 「用系统应用打开」在无图形界面的服务器上不可用（会给出友好提示；请使用面板内预览或浏览器打开）

## 开发

```text
dsh-filepannel-plugin/
├── src/
│   ├── host.js        # Host 半部（RPC + 下载路由）
│   └── client.js      # Client 半部（面板 UI）
├── README.md          # 中文文档
├── README.en.md       # English docs
└── LICENSE            # MIT
```

提交 PR 前请保持：无 emoji、配色仅用 DSH 主题令牌、纯 JavaScript（无 TS/JSX/构建）。

## License

[MIT](LICENSE)
