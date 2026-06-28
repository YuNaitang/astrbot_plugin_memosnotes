# MemosNotes — AstrBot Memos 备忘录/日记/知识库集成插件

通过 AI 自然语言或命令管理你的自建 [Memos](https://usememos.com/) 实例。

**作者**: YuNaitang（鱼鼐棠）
**仓库**: https://github.com/YuNaitang/astrbot_plugin_memosnotes

---

## 📥 安装

### 方式一：AstrBot 插件市场（推荐）
1. 打开 AstrBot 管理面板 → **插件管理**
2. 搜索 `memosnotes`
3. 点击安装

### 方式二：手动安装
```bash
astrbot plug install https://github.com/YuNaitang/astrbot_plugin_memosnotes
```

**前置依赖**：你需要在服务器上自建一个 [Memos](https://usememos.com/) 实例并获取 API Token。

---

## 🚀 快速开始

### 第 1 步：配置连接
在 AstrBot 管理面板 → **插件配置** 中填写：

| 配置项 | 说明 |
|--------|------|
| `memos_url` | Memos 实例地址，例如 `https://memos.example.com` |
| `memos_token` | API 访问令牌（Memos 设置 → 我的账户 → 访问令牌） |

### 第 2 步：验证
```bash
/mn help          # 查看所有命令
/mn create 你好    # 创建第一条备忘录
/mn list          # 查看最近公开内容
```

### 第 3 步：让 AI 自然语言操作
```
💬 "帮我记一下今天学会了 Docker"
💬 "查一下我之前记录的 Nginx 配置"
💬 "写日记：今天去公园散步了"
```

---

## 📖 命令参考

### 基本操作

| 命令 | 说明 | 示例 |
|------|------|------|
| `/mn create <内容>` | 创建私有备忘录（默认） | `/mn create 今天天气真好` |
| `/mn create -p <内容>` | 创建**公开**备忘录 | `/mn create -p 公告通知` |
| `/mn create --public <内容>` | 同上 | `/mn create --public 周报` |
| `/mn create --protected <内容>` | 登录可见 | `/mn create --protected 内部消息` |
| `/mn list [n]` | 列出最近 **n** 条**公开**备忘录 | `/mn list 5` |
| `/mn list [n] --all` | 包含私有内容（右侧显示 🆀） | `/mn list --all` |
| `/mn get <ID>` | 查看单条详情（合并转发/纯文本） | `/mn get #abc123` |
| `/mn update <ID> <内容>` | 更新内容 | `/mn update #abc123 新内容` |
| `/mn delete <ID>` | 删除 | `/mn delete #abc123` |

> 💡 ID 输入时支持 `#` 前缀，`/mn get #abc` 和 `/mn get abc` 都可以。

### 置顶与归档

| 命令 | 说明 |
|------|------|
| `/mn pin <ID>` | 置顶备忘录 |
| `/mn unpin <ID>` | 取消置顶 |
| `/mn archive <ID>` | 归档（隐藏） |
| `/mn restore <ID>` | 恢复归档 |

> ⚠️ 这些操作可通过 WebUI 配置的 `allow_delete` 开关统一禁用

### 快捷别名

`/mn` 等同于 `/memos`，少打三个字母。

---

## 🤖 AI 自然语言功能

这是本插件的核心能力。你不需要记住命令，直接对 bot 说就行：

### ✍️ 写日记
```
你: "写日记：今天去了海边，天气很好"
→ AI 调用 write_diary
→ 返回：今日海边日记
       #abc123
       https://notes.xxx/m/abc123
```
AI 会自动以 **Markdown + Front Matter** 格式保存：
```markdown
---
标题: 今日海边日记
作者: Naitang
日期: 2026-06-28
摘要: 今天去了海边，天气很好
---
今天去了海边……
```

### 📚 保存知识
```
你: "记住，服务器 SSH 端口改成 2222 了"
→ AI 调用 save_knowledge
→ 标签自动提取：#SSH #运维
→ 下次问"SSH 端口多少"就能查到
```
支持通过 `visibility` 参数控制公开/私有。AI 会根据你的语气判断是否公开。

### 🔍 搜索知识
```
你: "查一下我之前记录的 Docker 笔记"
→ AI 调用 search_memos
→ 从 Memos 检索相关内容并回复你
```

### AI 返回值格式

每次 AI 保存后会返回三行信息：

```
Nginx 配置指南
#abc123
https://notes.yunaitang.top/m/abc123
```

| 行 | 内容 |
|---|------|
| 第 1 行 | 标题（从 Front Matter / 内容自动提取） |
| 第 2 行 | 备忘录 ID（复制后用 `/mn get` 查看） |
| 第 3 行 | 公网链接（点击在浏览器打开） |

---

## 🎛 WebUI 配置

在 AstrBot 管理面板 → **插件配置** 中，可以看到三个额外的设置：

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `enable_auth` | ✅ 开关 | off | 启用后**只允许白名单用户**使用插件 |
| `allowed_origins` | ✏️ 文本 | `*` | 允许的 `unified_msg_origin` 列表（逗号/换行分隔），格式：`platform:type:session` |
| `allow_delete` | ✅ 开关 | on | 关闭后 `delete/archive/pin/unpin/restore` 全部不可用，LLM 也不会看到这些功能 |

**白名单 origin 示例**：
```
webchat:friend:astrbot      ← 网页端管理员
aiocqhttp:group:123456789   ← QQ 群
```

---

## 📦 存储格式

AI 写入的内容采用 **Markdown + Front Matter** 格式：

```markdown
---
标题: xxx
作者: Naitang
日期: 2026-06-28
摘要: xxx
---

# 正文标题

正文内容……
```

标签通过 `#标签` 语法在内容中标记，Memos 会自动提取。你可以在 Memos 网页端按标签筛选。

---

## 🔒 安全建议

1. **API Token 不要泄露** — `data/config/` 下的配置已被 `.gitignore` 排除
2. `enable_auth` 建议开启，避免非授权用户读写你的知识库
3. `allow_delete` 设为 off 可以防止误删除
4. 你的 Memos 实例建议配置 HTTPS

---

## 🧩 平台适配

| 平台 | 显示方式 |
|------|----------|
| QQ (aiocqhttp) | `/mn get` 用**合并转发三段式**（Front Matter / 正文 / 附件） |
| WebChat / Telegram / 其他 | 纯文本 + `──────` 分隔线 |

---

## 📅 版本历史

| 版本 | 内容 |
|------|------|
| 1.8.0 | list 默认仅公开，智能分页补齐；--all 参数 |
| 1.7.1 | LLM 工具返回标题+ID+公网链接 |
| 1.7.0 | Markdown + Front Matter 格式，可见性控制 |
| 1.6.0 | 白名单鉴权、功能开关、WebUI 配置 |
| 1.5.1 | 按平台选择发送方式 |
| 1.5.0 | get 合并转发三段式 |
| 1.4.0 | 标签系统、置顶归档 |
| 1.3.0 | 显示格式优化 |
| 1.2.0 | LLM 知识库工具 |
| 1.1.0 | 可见性控制 -p |
| 1.0.1 | 修复 API 兼容 |
| 1.0.0 | 初始版本 |
