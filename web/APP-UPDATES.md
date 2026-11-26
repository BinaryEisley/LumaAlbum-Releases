# v4 应用更新

v4 使用无签名更新协议，基于 Docker Hub 基线镜像 `binaryeisley/luma-album:4.2.0`（或 Windows Web 运行时）进行在线替换。更新包不需要 Ed25519 私钥、公钥或网页登录密码确认。

支持的运行时：

- `windows-amd64-v1`（Windows Web）
- `linux-amd64-v1`（群晖 NAS / x86_64 Docker）
- `linux-arm64-v1`（香橙派 5 Plus / iStoreOS ARM64 Docker）

三个运行时必须使用各自的 `.luma` 文件。

## 更新包产物

输出目录为 `release/app-updates`：

- `Luma-Web-<version>-windows-amd64-v1.luma`
- `Luma-Web-<version>-linux-amd64-v1.luma`
- `Luma-Web-<version>-linux-arm64-v1.luma`

常规升级包的清单默认包含 `dataPolicy=preserve`，安装时完整保留现有相册连接、收藏、轮播方案与保险箱数据（仅当显式指定 `--data-policy fresh` 时才会清空数据目录）。原照片目录始终外部只读挂载，不在更新包内。

## 网页升级步骤

1. 打开拾光网页，进入 **偏好设置 → 应用更新**。
2. 选择对应平台的 `.luma` 文件（Windows 选 `windows-amd64-v1`，群晖选 `linux-amd64-v1`，香橙派 / ARM64 选 `linux-arm64-v1`）。
3. 点击 **准备更新**，再点击 **替换并更新**。
4. 服务自动重启后刷新页面即可。Docker 环境下的登录口令继续由 `secrets/web-password.txt` 管理，无需重新设置。

更新器复用当前 Web 端口，关闭浏览器不会中断后台替换。

## 本地无 Docker 构建更新包

以下命令从仓库根目录执行，**不需要启动 Docker**（Linux 目标在 Windows 本机通过 `cargo-zigbuild` + `zig` 交叉编译并锁定 glibc `2.36`）：

```powershell
# 首次环境准备（仅需一次）
rustup target add x86_64-unknown-linux-gnu aarch64-unknown-linux-gnu
winget install --id zig.zig --exact
cargo install --locked cargo-zigbuild

# 一键构建全部三个平台 .luma 包 + manifest.json + SHA256SUMS.txt
powershell -ExecutionPolicy Bypass -File web/scripts/build-app-update.ps1 -Target All

# 或按平台单独构建（如已构建 React/dist 可追加 -SkipFrontend）
powershell -ExecutionPolicy Bypass -File web/scripts/build-app-update.ps1 -Target Windows
powershell -ExecutionPolicy Bypass -File web/scripts/build-app-update.ps1 -Target LinuxAmd64
powershell -ExecutionPolicy Bypass -File web/scripts/build-app-update.ps1 -Target OrangePi5Plus
```

## GitHub Actions 自动构建

推送 `web-v*` / `v*` 标签或在 Actions 页面运行 **Release Upgrade Packages**（`.github/workflows/release-updates.yml`），会自动在云端无 Docker 编译并产出全平台 `.luma` 升级包及 Windows 桌面端 `.exe`。
