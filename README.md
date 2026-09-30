# dsh-skill-github-releases

> 一个 DSH 技能（Skill）：**把闭源项目发到 GitHub** —— 源码留本地私有，
> 公开仓库只放说明文档、界面截图、版本历史和安装包。

给那些"想给别人用、但不想公开源码"的项目的发布流程规范。

---

## 它解决什么问题

想发到 GitHub 给别人下载，Releases 就必须在 **Public** 仓库里；
可源码一起传上去，就违背了"不公开源码"的初衷。

这个技能给出一套完整做法：**拆成两个仓库** ——
公开的发布仓库（说明 + 截图 + 安装包）+ 私有的源码仓库，互不干扰。

---

## 安装

### 方式一：装成 DSH 技能（推荐，装一次所有项目都能用）

把仓库里的 `SKILL.md` 放进任一技能根目录：

| 作用域 | 路径 | 优先级 |
| --- | --- | --- |
| 当前项目 | `<项目根>/.dsh/skills/github-releases-private-source/SKILL.md` | 100 |
| 当前项目 | `<项目根>/.agents/skills/github-releases-private-source/SKILL.md` | 200 |
| **所有项目** | **`~/.dsh/skills/github-releases-private-source/SKILL.md`** | **400** |
| **所有项目** | **`~/.agents/skills/github-releases-private-source/SKILL.md`** | **500** |

Windows 上 `~` 是 `C:\Users\<你的用户名>`。

装好后 DSH **立刻就能识别**（无需重启），技能目录里会出现 `github-releases-private-source`。

### 方式二：给别的 AI 读（不装，直接读文件）

把这条链接发给它，并说"读一下这个，按里面的流程做"：

```
https://raw.githubusercontent.com/lyq-05/dsh-skill-github-releases/main/SKILL.md
```

**注意用 `raw.githubusercontent.com` 而不是 `github.com`**：

| 链接形式 | 返回什么 | 适不适合 AI |
| --- | --- | --- |
| `github.com/.../blob/main/SKILL.md` | 一整页 HTML（正文夹在里面） | 能用，但要先剥壳 |
| **`raw.githubusercontent.com/.../main/SKILL.md`** | **纯 Markdown 文本** | **直接可用** |

三个前提，缺一不可：

1. 那个 AI **有联网能力**（不能联网就只能把文件直接给它）
2. 它**支持读网页/文件**（能 fetch URL）
3. 你得**明确告诉它去读** —— 它不会自己发现这个链接

> 这种方式的效果是"**照着流程做**"，而不是"**装了一个技能**"。
> 区别：装成技能后，AI 会在合适的时机**自动想起来用它**；
> 只给链接的话，每次都得你提醒一句。

### 方式三：直接拷文件

把 `SKILL.md` 下载下来，塞进对方项目的 `.dsh/skills/` 下即可。

---

## 里面有什么

单个 `SKILL.md`，一份自包含的流程规范：

| 章节 | 内容 |
| --- | --- |
| 双仓库结构 | 公开仓库 vs 私有源码仓库怎么分 |
| 动手前要问的三件事 | 目标仓库、**有哪些平台**、源码放哪 |
| 目录骨架 | README / CHANGELOG / docs / releases 怎么摆 |
| **产物命名规范** | 多平台多产物的 ASCII 命名法 |
| README 模板 | 含多平台下载表格、徽章、FAQ |
| CHANGELOG 模板 | 倒序 + 新增/优化/修复三节 |
| Release 正文模板 | 下载 / 升级说明 / 本次更新 / 已知问题 / 界面 |
| 完整命令 | 建仓库、双 git init、推送、用 API 建 Release 并上传产物 |
| 收尾自检清单 | 8 条，逐条过 |
| 常见坑表 | 7 个真实踩过的坑及处理 |

**多平台是内置的**，不是只针对安卓：Windows / macOS（区分 Intel 与 Apple Silicon）/
Linux / Android / iOS 的产物类型、命名、安装说明差异都覆盖到了。
只有一个平台时，模板里的表格退化成一行即可。

---

## 它从哪来

这套流程不是凭空写的，是在一个真实项目上跑通之后提炼的
（`kemai-tu`：Android 应用，公开仓库放说明和安装包、源码仓库私有私有），
把过程中**实际踩过的坑**都记进去了，例如：

- GitHub 的 Release 附件名**会剥掉非 ASCII 字符**（中文名传上去变成 `_v1.0.0.apk`）
- 代理环境下 `git push` 报连接重置 → 需要 `git config http.version HTTP/1.1`
- PowerShell 的 `Get-Content -Raw` 会给字符串挂 ETS 属性，导致 JSON 请求体被污染
- `releases/latest/download/<文件>` 这种"永远指向最新版"的写法，下一版文件名一变就 404

---

## 适用与不适用

**适用**：个人项目 / 小团队项目，想给别人下载用，但暂时不打算开源。

**不适用**：
- 本来就打算开源的 → 直接一个仓库就行，不需要拆
- 要上架应用商店的 → 商店有自己的发布流程
- 需要 CI 自动构建的 → 这份文档只覆盖手动发布流程

---

## 许可

随意取用、修改、再分发。踩到新坑欢迎提 Issue 补充。
