# GitHub 展示规范

这是用户指定的默认展示格式，已提炼自客脉图的布局。新项目按此文档即可完成，不需要再访问客脉图或向用户索要参考。用户对当前项目另有要求时以当前要求为准。内容必须来自项目事实，不能照搬参考项目的功能、权限或正式版承诺。

## README 的顺序与布局

1. 一级标题：应用中文名。
2. 居中徽章行：版本、总下载、Stars、最后提交、平台、最低系统、安装包大小、发布状态；“不联网”等标记仅在已核实时加入。不要漏掉徽章。
3. 居中宣传语：粗体一句话，换行后补一句产品价值说明。
4. 简介：是什么、解决什么问题、数据存储方式。随后引用块写当前版本、更新主题，链接到该版详细说明和 CHANGELOG。
5. 水平分隔线，然后下载区：平台／安装包直链／大小／说明表格；单独给 Releases 历史链接；具体安装与覆盖升级步骤。
6. 界面区：精选实际界面截图，按 2～3 列组成带功能名称的表格。只用虚构示例数据，手机截图同宽，避免一张超长图占满页面。
7. “它解决什么问题”：用用户场景说明价值，保持简短。
8. 功能区：按产品功能分组，每组用“功能／说明”表格，描述用户能做什么。
9. 权限／数据说明：权限、用途、何时需要、是否可选。没有相关权限就写实际数据行为，不虚构一套权限表。
10. 常见问题：安装、升级、备份迁移、权限失败与已知限制等实际问题，直接给操作路径。
11. 版本历史：不能只有一个 CHANGELOG 链接；必须有倒序“版本／主题”简表，版本链接到详细说明。下面给 CHANGELOG 和 Releases 入口。
12. 项目说明：个人项目、源码是否公开等已确定信息。

主要区域之间用 `---` 分隔。正文使用清晰短段落，功能与版本信息用表格，不写空章节或空泛卖点。

### 页首模板

下面的 OWNER、REPO、VERSION 等需要换成实际值。徽章 URL 中的中文采用 UTF-8 百分号编码；HTML 属性里的 `&` 写成 `&amp;`。不要把整条 URL 编码，也不要二次编码已有 `%xx`。

```html
<p align="center">
  <a href="https://github.com/OWNER/REPO/releases"><img alt="版本" src="https://img.shields.io/github/v/release/OWNER/REPO?label=%E7%89%88%E6%9C%AC&amp;color=blue"></a>
  <a href="https://github.com/OWNER/REPO/releases"><img alt="总下载" src="https://img.shields.io/github/downloads/OWNER/REPO/total?label=%E6%80%BB%E4%B8%8B%E8%BD%BD&amp;color=brightgreen"></a>
  <a href="https://github.com/OWNER/REPO/stargazers"><img alt="Stars" src="https://img.shields.io/github/stars/OWNER/REPO?style=social"></a>
  <a href="https://github.com/OWNER/REPO/commits/main"><img alt="最后提交" src="https://img.shields.io/github/last-commit/OWNER/REPO/main?label=%E6%9C%80%E5%90%8E%E6%8F%90%E4%BA%A4"></a>
  <img alt="平台" src="https://img.shields.io/badge/%E5%B9%B3%E5%8F%B0-Android-3DDC84">
  <img alt="最低系统" src="https://img.shields.io/badge/Android-8.0%2B-3DDC84">
</p>
<p align="center">
  <b>应用宣传语</b><br>
  一句话说明产品价值
</p>
```

平台只是示例，必须替换；补上当前实际安装包大小和体验版／正式版状态徽章。使用 `/badge/标签-内容-颜色` 生成静态徽章，各字段先编码，字段里的连字符按 Shields 规则转义为双连字符。

仅有预发布版时，版本徽章添加 `include_prereleases`；可使用 `sort=semver`。不要为了徽章显示而将体验版改成正式版。下载量代表附件下载次数，不是用户人数。包大小为静态数据，每次发布随下载表一起更新；MB／MiB 保持一致。默认分支不一定是 main，按实际分支写最后提交链接。

## CHANGELOG

所有可核实版本按倒序列出，每版有“版本号 — 一句话主题”。按实际变化设新增／优化／修复章节，不留空标题。修复要说明用户遇到的现象、已确认的原因和处理后的行为；未知原因不要猜。

历史记录来源：既有 Release、提交记录、开发文档和已核实的安装包信息。安装包丢失不等于删除版本说明：保留说明，标注“安装包未保留，不提供下载”。同版本下多次构建可合并记录。不要猜造历史版本号、发布日期、提交标签或测试结果。

## 每版详细说明与 GitHub Release

保存 `docs/RELEASE_v版本.md`，将相同正文用于对应 Release：

```markdown
# 应用名 v版本

> 一句话更新主题

**发布类型**：修复／功能更新／重大更新；注明体验版或正式版
**上一版本**：已核实的版本
**系统要求**：实际最低系统

---

## 📦 下载

| 平台 | 安装包 | 大小 | 说明 |
| --- | --- | --- | --- |
| 实际平台 | 带版本 tag 的附件直链 | 实际大小 | 安装说明 |

## ⚠️ 升级说明

说明覆盖安装条件、数据兼容和备份路径，不无条件保证数据绝不丢失。

## 🚀 本次更新

按变化分段，必要时用“用户操作／现在的行为”表格解释流程修复。

## 📱 界面

用绝对 raw.githubusercontent.com 地址展示本版相关截图。

## 验证与已知问题

区分已验证和待真机验证，说明实际测试范围。

## 🔗 相关链接

项目首页、CHANGELOG、真实存在的上一版说明或 Release。
```

历史无附件版本可以只有详细文档，不制造假下载链接或将当前源码冒充旧版打 tag。GitHub Release 标题使用“应用名 v版本 · 更新主题”。

## 发布后的展示验证

- 检查 README、CHANGELOG、详细说明和 Release 的版本、文件大小、下载 tag 一致。
- 实际请求下载链接，确认附件可取回；无附件历史版本不得有下载链接。
- 检查线上 README 已是新内容，截图与徽章能加载。
- 徽章源站返回成功不代表 GitHub 页面可显示。若用户看到破图，检查页面 HTML 中的 GitHub Camo 图片代理 URL，同时检查 SVG 内容而非仅 HTTP 状态。重点检查查询参数中的未编码中文、HTML 转义与预发布参数。
- SVG 可能没有 `<title>`，不能因此判失败；确认返回的是 SVG，而非 HTML 拦截页，并检查是否含 inaccessible、invalid、not found 等错误文字。
- 修改 URL 后代理缓存可能暂时保留旧结果，重新获取页面中的新代理 URL 验证。最多两轮针对性修正；仍不可访问时明确报告限制，不声称已显示正常。
