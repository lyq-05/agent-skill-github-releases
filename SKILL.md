---
name: github-releases-private-source
description: 上传项目到 GitHub 但不想公开源码时使用 —— 公开仓库只放 README、界面截图、版本说明和安装包，源码留本地，可额外备份到独立 GitHub 私有仓库。也适用于「做个下载页」「发 Release」「写版本说明」「多平台安装包」（Windows / macOS / Linux / Android / iOS）。不适用于本来就打算开源、或要上架应用商店的项目。
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

如果现有项目根本身就是完整源码工程，优先保持开发目录不变，另建独立公开发布目录。源码仓库应包含重建项目需要的代码、资源、测试和构建脚本，不应只备份 `app/` 而漏掉根目录构建文件。两种布局都必须保证公开仓库不包含源码。

> 别把源码放进公开仓库再靠"以后删掉" —— git 历史里删不干净。

---

## 第 0 步：动手前先问清三件事

先从会话和项目核实以下信息，只询问仍缺失的部分，不重复询问已确认事项。**平台数量决定后面所有模板的写法**。

1. **目标仓库**：`<owner>/<repo>` 叫什么？已经在 GitHub 上建好了吗？
2. **有哪些平台、每个平台产出什么文件**？例如：
   - Android → `apk` / `aab`
   - iOS → `ipa`（或只能走 TestFlight，那就没有可下载文件，要写清楚）
   - Windows → `exe` 安装包 / `zip` 绿色版
   - macOS → `dmg` / `zip`（注意区分 Intel 与 Apple Silicon）
   - Linux → `AppImage` / `deb` / `tar.gz`
3. **源码放哪个子目录**？项目根就是源码根，还是嵌套一层？

如果用户只有一个平台、一个产物，照常做 —— 模板里的多平台表格退化成一行即可。

另外，若会话尚未确定源码备份方式，简短询问是否需要**额外备份到 GitHub 私有仓库**：它提供异地恢复和协作能力，本地 Git 只能保留本机版本历史。该步骤可选，不阻塞已授权的公开发布；用户已同意或拒绝时不重复询问。用户仅询问优势不等于授权上传源码。

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

先检查用户指定的仓库是否已存在，以及当前 GitHub 登录账号是否有写入权限。已有仓库直接复用，不覆盖已有历史。

用户已授权创建或上传项目，且仓库名、所属账号与可见性已明确时，优先使用可用的 GitHub 工具、`gh` 或 API 创建仓库，不要求用户手动建仓。创建公开下载仓库时设为 Public；源码远程备份必须明确使用另一个 Private 仓库，公开发布授权不自动包含私有源码上传。

若创建请求结果不明确，先查询目标仓库确认结果，避免重复创建。仅在未登录、缺少创建权限或需要用户完成账号验证时，请用户完成必要操作，并说明具体原因；不要在聊天中索取 token。

需要网页兜底时，提供 https://github.com/new，填写已确认的仓库名和可见性，创建空仓库，不勾选 README、.gitignore 或 License。用户已确认的选择无需反复询问。

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

### 3–5. 编写项目主页、版本历史与 Release 正文

创建或重排发布材料前，必须阅读 [GitHub 展示规范](references/presentation.md)。它完整定义了用户偏好的展示格式，包括居中徽章与宣传语、下载与截图表格、功能与权限说明、版本主题简表、逐版详细说明和 Release 模板；不需要再访问客脉图作为参考。

按该规范生成 README.md、CHANGELOG.md 和 docs/RELEASE_v版本.md。旧安装包缺失时保留能核实的更新说明，不编造下载链接；所有事实、平台、数据行为和版本状态按当前项目调整。徽章也属于交付内容，必须检查 GitHub 图片代理实际加载，不能只验证 Shields 源站。

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
# 未授权远程源码备份时不配 remote；已授权则按下一节配置独立 Private 仓库
```

### 6.1 可选：额外备份到 GitHub 私有远程仓库

用户明确同意后执行。默认可采用同账号下的 `<公开仓库名>-source`，先说明实际使用的名字；用户指定名字时按其要求。

1. **确定范围与历史**：检查源码仓库状态、已有 remote 和提交历史。备份代码、资源、测试、构建脚本和开发文档；排除签名密钥、token、`.env`、本机配置、构建缓存、模拟器、安装包和用户私人样张。检查即将推送的历史，不只是当前工作目录；发现凭据时先处理，不能仅靠新增 `.gitignore` 隐藏历史中的秘密。签名密钥应另行安全备份，不放入源码仓库。
2. **创建或复用 Private 仓库**：先查询目标是否存在。新建时明确传 `private: true`，随后重新读取仓库属性，确认账号、名称及私有属性无误再推送。若同名仓库是 Public，不向其推送源码，也不擅自修改可见性；先与用户确认处理方式。已有非空仓库先核对用途和历史，禁止用强制推送覆盖。
3. **配置独立 remote**：只在源码仓库配置私有远程地址，不能使用公开发布仓库的 remote。已有 `origin` 时不擅自替换；可另加明确命名的备份 remote。提交已确认的源码快照，推送目标分支和对应版本标签，不盲目使用 `--mirror` 或推送所有分支。
4. **验证结果**：重新确认 GitHub 仓库仍为 Private；核对远程分支提交 SHA 与本地一致、版本标签指向正确，检查远程文件清单包含所需源码且没有排除项。公开仓库应继续只包含发布材料。
5. **交付说明**：提供私有仓库链接、备份版本及验证结果，说明密钥和本机环境未包含。明确这只是当前快照；后续修改需要提交并推送才会同步，不要声称已经开启自动备份。

创建或推送返回超时时，先查询远程仓库和引用确认是否已经成功，再决定重试。登录或权限受限时说明实际阻碍，不索取用户在聊天中粘贴凭据。

### 7. 推送公开发布仓库

```powershell
cd <projectRoot>
git remote add origin https://github.com/<owner>/<repo>.git
git push -u origin main
git push origin --tags
```

**推送失败怎么排查 —— 按这个顺序，不要跳步**

**第一步：检查实际网络环境。只有使用代理时才检查代理状态，不预设失败原因。**

```powershell
Resolve-DnsName github.com -Type A          # 解析出来是 198.18.x.x 吗？
Test-NetConnection 127.0.0.1 -Port 7897     # 代理端口在监听吗？（7890 也常见）
```

- 解析成 `198.18.x.x` / `2001:2::x` = 机器上装了 Clash / Surge 这类代理工具，
  它把 GitHub 的域名**接管成了自己的"假 IP"段**。
- **代理没开的时候，这些假 IP 哪儿也不通** —— 表现就是"域名能解析、连接却被重置"。
- 最典型的场景：**为了下载文件临时关了代理，之后忘了开回来**。
- 处理：把代理打开，重试即可。

**第二步：确认网络本身通**

```powershell
curl.exe -sS -o NUL -w "%{http_code}" --max-time 30 https://github.com
```

**第三步：以上都正常还失败，才考虑 HTTP/1.1 兜底**

```powershell
git config http.version HTTP/1.1
```

> ⚠️ **不要一上来就改这个，更不要把它当成"代理环境下的通病"。**
> 实测：代理正常时 **HTTP/2 工作完全正常**（`git -c http.version=HTTP/2 ls-remote` 返回 0）。
> 把它归因成 HTTP/2 问题是**错误归因** —— 会让你忽略真正的原因（代理没开），
> 下次换个项目还会踩同一个坑。

**为什么这里特别容易误判**

「改 `http.version`」和「把代理打开」经常发生在同一两分钟内。
两个变量一起变，人就很容易把功劳记到错的那个头上 —— 当时看到"改完就通了"，
其实真正起作用的是代理被打开了。

**排查网络问题的纪律：一次只改一个变量，再验证。**
改完先问自己：刚才是不是还动了别的东西？

### 8. 建 Release 并上传产物

**没有 `gh` CLI 时用 GitHub API + curl**：

```powershell
# ---- 取凭据（绝不打印内容）----
$lines = "protocol=https`nhost=github.com`n`n" | git credential fill 2>$null
$tok = (($lines | Where-Object { $_ -like "password=*" }) -replace "^password=","") -join ""
if (-not $tok) { throw "没有可用凭据，请用户通过 gh auth login 或客户端登录完成认证；不要在聊天中索取 token" }

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
- [ ] 若执行了源码远程备份：目标仓库确认为 Private，分支 SHA 和版本标签与本地一致，公开仓库没有混入源码；未授权时没有额外上传源码

---

## 常见坑（都是真踩过的）

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| 附件名变成 `_v1.0.0.apk` | GitHub 剥掉非 ASCII 字符 | 产物一律用 ASCII 文件名 |
| `git push` 报 `Connection was reset` / `Could not connect to server` | 先排查网络和已配置的代理；假 IP 且代理未运行是一种已遇到的原因，不能凭同一报错断定根因 | 见「步骤 7 · 推送失败怎么排查」 |
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
