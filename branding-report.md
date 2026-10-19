# FiberOcean Remote Support 品牌定制报告

日期：2026-10-15 ｜ 目标：GitHub Actions 编译出零配置直连自建服务器的 Windows 客户端
品牌名：**FiberOcean Remote Support** ｜ 服务器：`74.211.102.155`（relay 同）｜ key：`6ipCia2LP6hk5lYxoqvb1CPiM2D0bou9eaiixNvYWps=`

---

## 一、替换清单

### 1. 图标（图片类，34 个文件，全部经 PIL 重新打开验证可读）

| 文件 | 替换内容 | 验证方式 |
|---|---|---|
| `flutter/windows/runner/resources/app_icon.ico` | 品牌多尺寸 ICO 直接拷贝（Runner.rc 编译进 exe） | PIL 打开 256×256 ✅ |
| `res/icon.ico` | 由 `icon.png` 生成 7 尺寸 ICO（16/24/32/48/64/128/256） | PIL 读取 7 帧 ✅ |
| `res/tray-icon.ico` | 由 `tray-icon.png` 生成 4 尺寸 ICO（16/24/32/48）；`src/tray.rs:50` `include_bytes!` 编译进 Windows 托盘 | PIL 可读 ✅ |
| `flutter/assets/icon.ico`（新增） | 多尺寸 ICO；`win32_window.cpp:40` 运行时优先从 `flutter_assets/assets/icon.ico` 加载窗口图标 | 文件存在 ✅ |
| `flutter/assets/icon.png`（新增） | 1024×1024 应用内图标（`common.dart` `loadIcon()`） | gitignore 例外已加 ✅ |
| `res/icon.png` / `res/32x32.png` / `res/64x64.png` / `res/128x128.png` / `res/128x128@2x.png` | 按原尺寸缩放替换（`pubspec.yaml` `flutter_launcher_icons` 的 `image_path` 源 + Linux 打包） | PIL 尺寸核对 ✅ |
| `res/mac-icon.png` / `res/mac-tray-dark-x2.png` / `res/mac-tray-light-x2.png` | 按原尺寸（1024/60/48）缩放替换（mac 打包用，顺带） | PIL 尺寸核对 ✅ |
| `flutter/android/.../mipmap-*/ic_stat_logo.png`（5 档） | 通知栏图标，按原尺寸（24/36/48/72/96）缩放 | PIL 抽验 ✅ |
| `flutter/android/.../mipmap-*/ic_launcher.png` / `ic_launcher_round.png` / `ic_launcher_foreground.png`（15 个） | Android 桌面图标，按各文件原尺寸缩放 | PIL 抽验 ✅ |

### 2. Logo（SVG→PNG 策略）

| 文件 | 处理 | 说明 |
|---|---|---|
| `flutter/assets/logo.png`（新增） | `logo-banner.png`（2748×1000 透明底横版）拷入 | flutter 侧 logo 唯一加载点：`common.dart:3757` `_kDefaultLogoAsset='assets/logo.png'`（另有可选 `logo_light/logo_dark.png`，未提供时明暗主题都回退到 `logo.png`）。`pubspec.yaml` 已整体注册 `assets/` 目录，**无需改 pubspec**。显示约束 maxWidth 300 × maxHeight 60，`BoxFit.contain` |
| `res/logo.svg` | 替换为内嵌 base64 PNG 的 SVG 包装（26×26，tray-icon 内容） | 全仓 grep 确认**无任何构建引用**（仅文档/旧 sciter 素材） |
| `res/logo-header.svg` | 替换为内嵌 base64 banner PNG 的 SVG 包装（1000×286.6） | 同上，无构建引用 |

### 3. 应用名显示串（技术标识符一律未动）

| 文件 | 替换 | 验证 |
|---|---|---|
| `libs/hbb_common/src/config.rs`（子模块，走补丁） | `APP_NAME "RustDesk"` → `"FiberOcean Remote Support"` | 补丁 git apply 验证 ✅（见第四节） |
| `flutter/lib/desktop/widgets/tabbar_widget.dart:654` | 标签栏硬编码 `"RustDesk"` → `"FiberOcean Remote Support"` | grep ✅ |
| `flutter/windows/runner/main.cpp:83` | 窗口标题回退值 `L"RustDesk"` → `L"FiberOcean Remote Support"`（正常运行时由核心 DLL `get_rustdesk_app_name` 覆盖为 APP_NAME，此仅为兜底） | grep ✅ |
| `flutter/windows/runner/Runner.rc` | `FileDescription`/`ProductName` → FiberOcean Remote Support；`CompanyName`/`LegalCopyright` → FiberOcean。**`InternalName`/`OriginalFilename` 保持 rustdesk.exe 不动** | grep ✅ |
| `flutter/android/app/src/main/AndroidManifest.xml` | `android:label="RustDesk"` → `"FiberOcean Remote Support"`；无障碍服务 `"RustDesk Input"` → `"FiberOcean Input"` | grep ✅ |
| `src/lang/en.rs:204` | `powered_by_me` 值 `"Powered by RustDesk"` → `"Powered by FiberOcean"`（该词条被翻译系统排除在自动替换外，须手改） | grep ✅ |

**自动覆盖的部分（无需逐条改）**：`src/lang.rs:253` 翻译层内置定制客户端逻辑——`APP_NAME != "RustDesk"` 时，所有语言词条里的 "RustDesk" 自动替换为 APP_NAME（关于页标题、权限引导、提示语等全部生效）。`is_rustdesk()` 仅用于此判断，无其他副作用（已全仓 grep 确认）。

**明确未动**：`Cargo.toml` package name、`rustdesk://` scheme、`ORG "com.carriez"`、目录名、exe 文件名、`src/lang/` 其余语言文件（自动替换覆盖）。

### 4. 零配置服务器默认值（编译内置）

hbb_common 是 **git 子模块**（本仓仅记录指针 `5d52530b`），CI 用 `submodules: recursive` 从 GitHub 拉取——直接改本地文件进不了 CI。采用**补丁方案**：

| 文件 | 内容 |
|---|---|
| `patches/hbb_common-branding.patch`（新增） | 对子模块定点版本 `src/config.rs` 改 3 行：`APP_NAME`→品牌名；`RENDEZVOUS_SERVERS`→`&["74.211.102.155"]`；`RS_PUB_KEY`→我们的 key。已用 `git apply --check` + 实应用 + grep 三行验证 ✅ |
| `.github/workflows/flutter-build.yml` | 全部 **13 处** `submodules: recursive` checkout 后插入补丁步骤：`git -C libs/hbb_common apply ../../patches/hbb_common-branding.patch`（`shell: bash`，Win/Linux/mac runner 均可用）。YAML 解析校验通过（18 jobs） ✅ |

relay 未显式设置时 RustDesk 自动用同 host 的 21117 端口，与"relay 同 host"一致。

**文件名法兜底**（即使补丁失效也能零配置）：`src/custom_server.rs` 支持从 exe 文件名解析——把编出的 exe 改名为
`FiberOcean-host=74.211.102.155,key=6ipCia2LP6hk5lYxoqvb1CPiM2D0bou9eaiixNvYWps=.exe`
即可注入 host/key（但 APP_NAME 显示名只能靠补丁）。

### 5. 其他

| 文件 | 改动 | 原因 |
|---|---|---|
| `.gitignore` | `*png` 后新增例外 `!flutter/assets/icon.png`、`!flutter/assets/logo.png` | **关键坑**：仓库默认忽略所有 png，新拷入的两个 assets 不提交则 CI 缺图。已验证 `git check-ignore` 返回 1（不再忽略） ✅ |

---

## 二、不确定项（需 CI 编译后验证）

1. **补丁与子模块漂移**：补丁锚定 hbb_common `5d52530b`。若日后主仓 bump 子模块版本，`git apply` 会失败导致 CI 中断（此时需重新生成补丁——这也算好事，显式失败而非静默回退）。
2. **Android 圆形/自适应图标**：`ic_launcher_round.png` 未做圆形蒙版、`ic_launcher_foreground.png` 未留自适应安全边距，真机上可能显示为方形或贴边。Windows 不受影响。
3. **logo.png 暗色主题可读性**：banner 为透明底+FIBER OCEAN 文字，暗色主题下文字对比度需目验；不行就补一张 `logo_dark.png`（浅色文字版）进 `flutter/assets/`。
4. **mac 托盘图标语义**：mac-tray 惯例为单色模板图，用彩色 tray-icon 缩放的观感需 mac 构建目验（本次目标 Windows，优先级低）。
5. **窗口标题/关于页实际文案**：依赖 APP_NAME 补丁在 CI 生效 + 翻译层自动替换，需编译产物目验。
6. **`.gitmodules` 替代方案**（可选更稳）：把 hbb_common fork 到自己账号、改 `.gitmodules` URL 指向 fork 并直接在 fork 里改 config.rs，可去掉补丁步骤。需要用户有一个 GitHub fork，未默认采用。

## 三、给 GitHub Actions 的注意事项

- 用 `.github/workflows/flutter-build.yml`（或 tag 触发 `flutter-tag.yml`，其 checkout 同构但**未**插补丁步骤——只走 flutter-build.yml 最稳）。
- 补丁步骤在每次 checkout 后、构建前执行；13 个 job 全覆盖，任一 job 单独跑也没问题。
- 编译产物 exe 默认名仍是 `rustdesk.exe`（技术标识符，未动）；分发给客户前可按上文文件名法改名获得双保险。
- 首次 CI 若失败在补丁步，优先检查子模块 commit 是否已漂移（见不确定项 1）。

## 四、本地验证记录

- 34 个图标文件 PIL 全部重新打开验证（尺寸/帧数符合预期）
- 5 处文字替换 grep 命中数核对（tabbar 1 / main.cpp 1 / Runner.rc 2 / Manifest 1+1 / en.rs 1）
- `patches/hbb_common-branding.patch`：对原始 config.rs `git apply --check` 通过、实应用后 3 行目标值 grep 命中
- `flutter-build.yml` 插入后 Python YAML 解析通过（18 jobs）
- `.gitignore` 例外生效（`git status` 已能看到两个新 PNG 为未跟踪待提交状态）
- 本机无 Flutter/Rust 工具链，未编译（按约定走 CI）
