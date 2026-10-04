# 闭源项目发 GitHub

一个可复用的 Agent Skill：**公开展示与下载，源码保持私有。**

公开仓库只放 README、截图、版本说明和安装包；源码留在本地，也可经用户明确同意备份到独立 GitHub 私有仓库。支持多平台，按项目实际产物调整。

## 展示格式已内置

无需再给 AI 找参考项目。技能内置以下展示格式：

| 区域 | 默认展示 |
| --- | --- |
| 页首 | 应用名、居中徽章、宣传语与产品简介 |
| 下载 | 平台、带版本标签的附件直链、真实包大小、安装与升级步骤 |
| 界面 | 2～3 列功能截图表格，只用虚构示例数据 |
| 功能 | 按用户场景分组的功能表、权限与数据说明、常见问题 |
| 版本历史 | 倒序“版本／主题”简表与完整 CHANGELOG |
| Release | 更新主题、下载、升级说明、具体变化、截图、验证与已知问题 |

历史安装包丢失时仍保留能核实的说明，不制造失效下载链接。规范还包含中文徽章参数编码、预发布版本显示与 GitHub 图片代理检查。

## 安装

**安装完整技能目录，不要只复制 SKILL.md。** 引用的展示规范是必需文件；agents/openai.yaml 是可选的客户端元数据。

```text
github-releases-private-source/
├── SKILL.md
├── references/
│   └── presentation.md
└── agents/
    └── openai.yaml
```

将目录放入客户端支持的技能位置。以下以 ~/.agents/skills 为例；使用其他目录的客户端请替换安装路径。

**Windows PowerShell：首次安装**

```powershell
$skillRoot = Join-Path $HOME '.agents/skills'
New-Item -ItemType Directory -Force $skillRoot | Out-Null
git clone https://github.com/lyq-05/agent-skill-github-releases.git (Join-Path $skillRoot 'github-releases-private-source')
```

**macOS / Linux：首次安装**

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/lyq-05/agent-skill-github-releases.git ~/.agents/skills/github-releases-private-source
```

目标目录已存在时先检查其来源和本地改动，不覆盖或删除。也可下载仓库 ZIP，把完整目录命名为 github-releases-private-source 后安装。

不同客户端的发现和刷新方式可能不同。安装后检查技能列表，必要时刷新客户端或开启新会话；已有会话可能仍保留之前读入的版本。

## 更新到其他电脑

GitHub main 分支是远程版本。本地修改不会自动上传，另一台电脑已有的安装也不会自动更新。

Git 克隆安装可在技能目录执行：

```bash
git status --short
git pull --ff-only
```

有本地修改时先保留并合并，不用强制重置。ZIP 安装则重新下载完整目录，保留自己的修改后更新。之后让 AI 重新读取技能及展示规范。

## 使用

在客户端选择 github-releases-private-source，或者明确说：

> 使用 github-releases-private-source，把这个项目发布到 GitHub，公开下载但不公开源码。采用技能内置的完整展示格式。

不安装时，也可以把这两份链接交给有联网读取能力的 AI：

- [SKILL.md 原文](https://raw.githubusercontent.com/lyq-05/agent-skill-github-releases/main/SKILL.md)
- [展示规范原文](https://raw.githubusercontent.com/lyq-05/agent-skill-github-releases/main/references/presentation.md)

这是临时读取流程，不等于安装。AI 需要同时读取入口与引用规范。

## 发布与源码边界

- 完整源码工程保持原位，优先另建独立公开发布目录。
- 公开仓库不包含源码、密钥、真实账单、私人样张或本机环境。
- 源码备份使用独立 Private 仓库，先核实私有属性再推送。
- 公开发布授权不自动包含源码上传授权。
- 使用客户端登录或凭据管理器，不要求在聊天中粘贴 token。
- 发布后检查远程提交、附件下载、图片代理与 README，不把本地写入成功当作线上已生效。

## 最近完善

- 内置统一展示规范，不再依赖外部参考项目。
- 补齐徽章编码和 GitHub 图片代理校验。
- 保留历史说明，缺失附件不提供下载。
- 修正旧仓库链接、只复制单文件的安装说明与跨客户端立即生效的承诺。
- 网络失败先检查实际环境，不将 HTTP/1.1 当作固定修复。

## 适用范围

适合个人或小团队的闭源应用下载发布。已计划开源、应用商店上架或需要完整 CI 自动构建的项目，应采用相应流程；本技能不替代那些流程。

## 许可

随意取用、修改、再分发。欢迎通过 Issue 反馈实际问题。
