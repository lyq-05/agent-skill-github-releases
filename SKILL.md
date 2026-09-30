---
name: github-releases-private-source
description: 把闭源项目发到 GitHub —— 源码留本地私有，公开仓库只放说明文档、界面截图、版本历史和安装包，让外人能了解项目全貌并直接下载安装程序。适用于多平台（Android / iOS / Windows / macOS / Linux）的多产物发布。当用户提到「上传到 GitHub 但不想公开源码」「做个下载页」「发 Release」「版本说明」「安装包」「开源但源码私有」时使用本 skill。
---

# 闭源项目的 GitHub 发布仓库

## 这个 skill 解决什么

用户想把项目发到 GitHub 给别人用，但**不想公开源码**。矛盾在于：
Releases 要能被外人下载，仓库就必须是 Public —— 可源码一起传上去就违背了初衷。

解法是**拆成两个仓库**：

| 位置 | 内容 | 公开性 |
| --- | --- | --- |
| 项目根目录 | README、CHANGELOG、界面截图、Release 说明、安装包 | **公开** |
| 源码目录（如 `src/`、`app/`） | 完整源码 | **私有**，独立 git 仓库 |

根目录的 `.gitignore` 把源码目录排除掉，两个仓库互不干扰。

> 别把源码放进公开仓库再靠"以后删掉" —— git 历史里删不干净。

---

## 第 0 步：动手前先问清三件事

不要自己猜，直接问用户。**平台数量决定后面所有模板的写法**。

1. **目标仓库**：`<owner>/<repo>` 叫什么？已经在 GitHub 上建好了吗？
2. **有哪些平台、每个平台产出什么文件**？例如：
   - Android → `apk` / `aab`
   - iOS → `ipa`（或只能走 TestFlight，那就没有可下载文件，要写清楚）
   - Windows → `exe` 安装包 / `zip` 绿色版
   - macOS → `dmg` / `zip`（注意区分 Intel 与 Apple Silicon）
   - Linux → `AppImage` / `deb` / `tar.gz`
3. **源码放哪个子目录**？项目根就是源码根，还是嵌套一层？

如果用户只有一个平台、一个产物，照常做 —— 模板里的多平台表格退化成一行即可。

---

## 目录骨架

```
<projectRoot>/
├── README.md                 项目主页（门面，最重要）
├── CHANGELOG.md              版本历史，倒序
├── .gitignore                必须排除源码目录
├── docs/
│   ├── screenshots/          精选界面截图，NN-功能名.png
│   ├── RELEASE_v1.0.0.md     每个版本的 Release 正文
│   └── 发布流程.md            给自己看的流程备忘
├── releases/                 随仓库发布的安装包副本
│   └── <project>-v1.0.0-<平台>.apk
└── <源码目录>/                独立 git 仓库，被 .gitignore 排除
```

---

## 产物命名：**必须用 ASCII**

```text
<项目名>-v<版本>-<平台>[-<架构>].<扩展名>

kemai-tu-v1.7.1.apk                      单平台可省平台名
myapp-v2.0.0-windows-x64-setup.exe
myapp-v2.0.0-windows-x64-portable.zip
myapp-v2.0.0-macos-arm64.dmg
myapp-v2.0.0-macos-x64.dmg
myapp-v2.0.0-linux-x86_64.AppImage
```

**踩过的坑**：GitHub 的 Release 附件名**会剥掉非 ASCII 字符**。
「客脉图_v1.7.1_debug.apk」传上去变成「_v1.7.1_debug.apk」—— 中文全没了，只剩个下划线。
所以**文件必须用 ASCII 名**，中文只出现在 README 的文字描述里。

---

## 步骤

### 1. 在 GitHub 上建空仓库

必须由用户在网页上建（AI 通常没有建仓库的权限）：

1. https://github.com/new
2. Repository name 填仓库名
3. 选 **Public**（要给别人下载就必须 Public）
4. **不要**勾选 Add a README / .gitignore / license（会和本地冲突）
5. Create repository

### 2. 写 `.gitignore` —— 这一步决定源码会不会泄漏

```gitignore
# ---- 源码：独立仓库，绝不进公开仓库 ----
<源码目录>/

# ---- 构建产物 ----
build/
dist/
out/
target/
*.apk
*.aab
*.ipa
*.exe
*.msi
*.dmg
*.AppImage
*.deb

# ---- 依赖与工具链（可能几个 G）----
node_modules/
.gradle/
.venv/
venv/
__pycache__/
*.pyc

# ---- 本机环境（模拟器镜像、SDK 缓存等）----
.android/
*.log

# ---- 过程截图（只保留 docs/screenshots 下的精选图）----
shots/
screenshots-raw/
tmp/

# ---- 例外：随仓库发布的安装包 ----
!releases/*.apk
!releases/*.exe
!releases/*.dmg
!releases/*.zip
```

> `.gitignore` 里 `!` 放行的规则，**父目录本身不能被排除**，否则放行无效。

### 3. 写 README.md（项目门面）

用这个骨架，按平台数量增删表格行：

````markdown
# <项目名>

<p align="center">
  <a href="https://github.com/<owner>/<repo>/releases"><img alt="版本" src="https://img.shields.io/github/v/release/<owner>/<repo>?label=%E7%89%88%E6%9C%AC&color=blue"></a>
  <a href="https://github.com/<owner>/<repo>/releases"><img alt="总下载" src="https://img.shields.io/github/downloads/<owner>/<repo>/total?label=%E6%80%BB%E4%B8%8B%E8%BD%BD&color=brightgreen"></a>
  <img alt="平台" src="https://img.shields.io/badge/%E5%B9%B3%E5%8F%B0-Windows%20%7C%20macOS%20%7C%20Android-3DDC84">
</p>

<p align="center">
  <b><一句话卖点></b><br>
  <一句话补充说明>
</p>

---

一个 <一句话说清是什么>。<关键特性一句话>

> **当前版本 v1.0.0（YYYY-MM-DD 发布）**：<一句话主题>。
> <两三句说清这版最大的变化>
> 逐项说明见 **[v1.0.0 发布说明](docs/RELEASE_v1.0.0.md)**，完整历史见 **[CHANGELOG](CHANGELOG.md)**。

---

## 下载

| 平台 | 安装包 | 大小 | 说明 |
| :---: | --- | :---: | --- |
| Windows | **⬇ [myapp-v1.0.0-windows-x64-setup.exe](https://github.com/<owner>/<repo>/releases/download/v1.0.0/myapp-v1.0.0-windows-x64-setup.exe)** | 132 MB | 点文件名**直接开始下载** |
| macOS（Apple 芯片） | **⬇ [myapp-v1.0.0-macos-arm64.dmg](...)** | 149 MB | 首次打开需右键→打开 |
| macOS（Intel） | **⬇ [myapp-v1.0.0-macos-x64.dmg](...)** | 151 MB | 同上 |
| Android 8.0+ | **⬇ [myapp-v1.0.0.apk](...)** | 11 MB | 点文件名**直接开始下载** |

> 想看历史版本、以及每个版本具体改了什么 → **[前往 Releases 页面](https://github.com/<owner>/<repo>/releases)**

**安装步骤**

<按平台分条写，每个平台 3-5 步，要具体到点哪里>

---

## 界面

| <功能1> | <功能2> | <功能3> |
| :---: | :---: | :---: |
| ![功能1](docs/screenshots/01-功能1.png) | ![功能2](docs/screenshots/02-功能2.png) | ![功能3](docs/screenshots/03-功能3.png) |

---

## 功能

<分类分表格列，每条一句话说清"做了什么、有什么用">

---

## 常见问题

**Q：<用户最可能踩的坑>？**
A：<直说，不要绕>

---

## 版本历史

见 **[CHANGELOG.md](CHANGELOG.md)**。

---

## 说明

- 本项目为**个人项目**，暂不公开源码
- <数据/隐私相关的一句话承诺>
````

**下载区那两行是关键**（见下方「直链」一节）：文件名是**直链**，点下去直接下载；
Releases 页面单独给一个链接，只管"看历史版本"。

### 4. 写 CHANGELOG.md

倒序，最新的在最上面。每版分「新增 / 优化 / 修复」三节：

```markdown
# 版本历史

## [v1.0.1] — <一句话主题>

### 修复

- **<问题描述>**：<根因是什么> → <怎么修的>

### 优化

- <改了什么，为什么>

---

## [v1.0.0] — 首个版本

- <功能清单>
```

> **写根因，不要只写"修了 bug"**。"修了闪退问题"没有价值；
> "空列表时数组越界导致闪退，已加空判断"才有价值 —— 半年后的自己能看懂。

### 5. 写 `docs/RELEASE_vX.Y.Z.md`（Release 正文）

用这个结构：

````markdown
# <项目名> v1.0.1

> **<一句话主题>**

**发布类型**：修复 / 功能更新 / 重大更新
**上一版本**：v1.0.0
**系统要求**：<各平台最低要求>

---

## 📦 下载

| 平台 | 安装包 | 大小 | 说明 |
| --- | --- | --- | --- |
<和 README 的下载表一致>

---

## ⚠️ 升级说明

- **直接覆盖安装即可**，数据不会丢
- <要不要先卸载、要不要迁移数据、有没有不兼容变更>

---

## 🚀 本次更新

### 新增
### 优化
### 修复

<每条写清：现象 → 根因 → 处理>

---

## 🚨 已知问题

<诚实列出。写清楚"这是系统限制"还是"还没做">

---

## 📱 界面

| <变化前> | <变化后> |
| :---: | :---: |
| ![前](https://raw.githubusercontent.com/<owner>/<repo>/main/docs/screenshots/xx.png) | ![后](...) |

---

## 🔗 相关链接

- [完整变更历史](https://github.com/<owner>/<repo>/blob/main/CHANGELOG.md)
- [项目首页](https://github.com/<owner>/<repo>)
- [上一版 v1.0.0](https://github.com/<owner>/<repo>/releases/tag/v1.0.0)
````

> Release 正文里的图片**必须用绝对 URL**（`raw.githubusercontent.com/...`），
> 因为 Release 页面不在仓库文件树里，相对路径找不到图。

### 6. 初始化两个 git 仓库

```powershell
# ---- 公开仓库（项目根）----
cd <projectRoot>
git init -b main
git config user.name  "<GitHub 用户名>"
git config user.email "<用户名>@users.noreply.github.com"
git config core.autocrlf false
git add -A

# ★★★ 关键一步：自检，确认源码没被带进来 ★★★
git ls-files | Select-String "<源码目录名>"
# 应该没有任何输出。有输出就回去改 .gitignore，然后 git rm -r --cached <源码目录>

git commit -m "v1.0.0 发布"
git tag -a v1.0.0 -m "<项目名> v1.0.0"

# ---- 私有仓库（源码目录）----
cd <源码目录>
git init -b main
git add -A
git commit -m "<项目名> 源码 v1.0.0"
git tag -a v1.0.0 -m "源码 v1.0.0"
# 先不配 remote，将来要开源再加
```

### 7. 推送

```powershell
cd <projectRoot>
git remote add origin https://github.com/<owner>/<repo>.git

# ★ 有代理/加速器时 HTTP/2 常连不上，强制 HTTP/1.1
git config http.version HTTP/1.1

git push -u origin main
git push origin --tags
```

**如果推送报 `Recv failure: Connection was reset` 或 `Could not connect to server`**：

- 先 `curl.exe -sS -o NUL -w "%{http_code}" https://github.com` 确认网络本身通
- 通了还推不上，就是 HTTP/2 的问题，执行上面那条 `http.version`
- 域名解析到 `198.18.x.x` 之类的地址是正常的（代理软件的 fake-IP 段）

### 8. 建 Release 并上传产物

**没有 `gh` CLI 时用 GitHub API + curl**：

```powershell
# ---- 取凭据（绝不打印内容）----
$lines = "protocol=https`nhost=github.com`n`n" | git credential fill 2>$null
$tok = (($lines | Where-Object { $_ -like "password=*" }) -replace "^password=","") -join ""
if (-not $tok) { throw "没有可用凭据，让用户生成一个 Personal Access Token（勾 repo 权限）" }

$H = @("-H","Authorization: Bearer $tok",
       "-H","Accept: application/vnd.github+json",
       "-H","User-Agent: release-bot")

# ---- 构造 payload ----
# ★ 用 .NET 读文件，不要用 Get-Content -Raw ★
#   Get-Content 会给字符串挂上 PSPath 等 ETS 属性，
#   ConvertTo-Json 之后整个对象被序列化进 body，请求直接失败。
$body = [System.IO.File]::ReadAllText("<projectRoot>\docs\RELEASE_v1.0.0.md",
                                      [System.Text.Encoding]::UTF8)
$payload = [ordered]@{
  tag_name   = "v1.0.0"
  name       = "<项目名> v1.0.0 · <一句话主题>"
  body       = $body
  draft      = $false
  prerelease = $false
} | ConvertTo-Json -Depth 4 -Compress
[System.IO.File]::WriteAllText("$env:TEMP\_payload.json", $payload,
                               (New-Object System.Text.UTF8Encoding($false)))

# ---- 建 Release ----
$rel = (& curl.exe -sS --max-time 60 -X POST @H `
        -H "Content-Type: application/json; charset=utf-8" `
        --data-binary "@$env:TEMP\_payload.json" `
        "https://api.github.com/repos/<owner>/<repo>/releases" | ConvertFrom-Json)
if (-not $rel.id) { throw "创建失败" }
$rel.html_url

# ---- 逐个上传产物（多平台就循环几次）----
$assets = @(
  @{ path = "<projectRoot>\releases\myapp-v1.0.0-windows-x64-setup.exe";
     name = "myapp-v1.0.0-windows-x64-setup.exe";
     type = "application/octet-stream" },
  @{ path = "<projectRoot>\releases\myapp-v1.0.0.apk";
     name = "myapp-v1.0.0.apk";
     type = "application/vnd.android.package-archive" }
)
foreach ($a in $assets) {
  $u = "https://uploads.github.com/repos/<owner>/<repo>/releases/$($rel.id)/assets?name=$($a.name)"
  $r = (& curl.exe -sS --max-time 600 -X POST @H -H "Content-Type: $($a.type)" `
        --data-binary "@$($a.path)" $u | ConvertFrom-Json)
  if ($r.browser_download_url) { "OK  $($r.name)  $([math]::Round($r.size/1MB,2)) MB" }
  else { "FAIL  $($a.name)" }
}
```

> **超时**要按文件大小给足：100 MB 以上的产物用 `--max-time 1800`，
> 否则传到一半被 curl 掐断。

### 9. 更新 README 的下载直链

**直链格式**（点下去直接下载，不会跳页面）：

```text
https://github.com/<owner>/<repo>/releases/download/<tag>/<文件名>
```

每发一版要同步改 README 表格里的 **文件名 / URL 里的 tag / URL 里的文件名 / 文件大小** 四处。

> **不要用** `releases/latest/download/<文件名>`：它要求"最新那个 Release 里必须有这个文件名"。
> 下一版文件名变了（版本号进了文件名），旧链接就 404。带 tag 的虽然要手改，但永远不会坏。

---

## 收尾自检清单

发布完成后，**逐条过一遍**，不要凭印象：

- [ ] `git ls-files | Select-String "<源码目录>"` **无输出**（源码没泄漏）
- [ ] `git ls-files` 里没有 `node_modules/`、`build/`、`.venv/`、模拟器镜像等大目录
- [ ] README 的下载直链**逐个点开**，确认是"开始下载"而不是"跳到 Releases 页"
- [ ] 每个 Release 附件的名字是**纯 ASCII**
- [ ] 所有产物的 **Content-Type 正确**（apk / exe / dmg 各不同）
- [ ] README、CHANGELOG、RELEASE 说明里的版本号**三处一致**
- [ ] Release 正文里的图片用**绝对 URL**，在 Release 页面能正常显示
- [ ] 仓库里**没有** API token、密钥、`.env`、个人隐私数据

---

## 常见坑（都是真踩过的）

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| 附件名变成 `_v1.0.0.apk` | GitHub 剥掉非 ASCII 字符 | 产物一律用 ASCII 文件名 |
| `git push` 报连接重置 | 代理环境下 HTTP/2 不通 | `git config http.version HTTP/1.1` |
| 建 Release 返回 400 | `Get-Content -Raw` 的 ETS 属性污染了 JSON | 改用 `[System.IO.File]::ReadAllText` |
| 上传大文件中断 | curl 默认超时太短 | `--max-time 1800` |
| README 下载链接点开是 Releases 页 | 写成了 `releases/latest` 页面链接 | 改成 `releases/download/<tag>/<文件>` 直链 |
| 上传后附件少了一个 | 中途失败但脚本继续跑了 | 上传后逐个校验 `browser_download_url` 存在 |
| 改了已发布的内容但线上没变 | 改完没提交/没推送 | 每次改完 `git status` 确认干净 |

---

## 一条纪律

**改完任何已经发布出去的内容（README、Release 正文、下载链接），必须回头验证一遍再回复用户** ——
打开页面看一眼，或者 `curl` 请求一次，确认线上真的是改后的样子。

编辑工具/脚本返回"成功"**不等于**线上生效：可能没提交、没推送、或者被后续的读写覆盖了。
这个坑踩过两次，都是用户先发现的。
