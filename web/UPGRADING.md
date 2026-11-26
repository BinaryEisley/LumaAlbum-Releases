# v4 打包与升级协议说明

v4（当前 `4.2.0`）采用 **Docker Hub 4.2.0 多架构基线镜像 + `.luma` 无签名在线升级包** 的分离发布架构：

- **首次容器部署**：直接从 Docker Hub 拉取 `binaryeisley/luma-album:4.2.0`（支持 `linux/amd64` 与 `linux/arm64`，基于 `debian:bookworm-slim` / glibc `2.36`），配套模板为 `web/compose.nas.yaml` 与 `web/compose.orangepi5plus.yaml`。
- **日常版本迭代**：无论是本机还是 GitHub Actions，都**只构建 `.luma` 升级包与桌面端程序**，不再重新构建或推送 Docker 镜像。

## 本地构建（零 Docker）

在仓库根目录执行（需安装 `rustup` Linux 目标、`zig` 与 `cargo-zigbuild`）：

```powershell
# 一键编译三端二进制并打包 + 校验全部 .luma 升级包
powershell -ExecutionPolicy Bypass -File web/scripts/build-app-update.ps1 -Target All

# 仅单独构建某个平台（已编译前端时可加 -SkipFrontend）
powershell -ExecutionPolicy Bypass -File web/scripts/build-app-update.ps1 -Target Windows
powershell -ExecutionPolicy Bypass -File web/scripts/build-app-update.ps1 -Target LinuxAmd64
powershell -ExecutionPolicy Bypass -File web/scripts/build-app-update.ps1 -Target OrangePi5Plus
```

Linux 二进制统一通过 `cargo zigbuild --target <triple>.2.36` 交叉编译，保证与 Docker Hub `binaryeisley/luma-album:4.2.0` 内的 glibc `2.36` 100% ABI 兼容。

## GitHub Actions 构建（零 Docker）

工作流位于 `.github/workflows/release-updates.yml`：
- 推送 `web-v*` / `v*` 标签自动构建全部 `.luma` 升级包、Windows 桌面端 `.exe`、`manifest.json` 与 `SHA256SUMS.txt` 并发布 GitHub Release；
- 也可在 Actions 页面手动 `workflow_dispatch` 触发构建。

## v4 更新协议与数据策略

- 常规升级包默认声明 `dataPolicy=preserve`，网页端替换时完整保留 `/data` 下的相册源、索引缓存、收藏、轮播方案与保险箱数据；照片目录始终独立挂载，不受影响。
- 更新器保留 `LUMAUPD2` 包头、gzip 分片边界、相对路径安全检查（拒绝绝对路径与 `..`），以及新进程健康检查与失败自动回滚。
- 启动器 `app_update_agent.py` 支持自动比较版本：当手动拉取了比 `/updates` 更高版本的 Docker 镜像时，自动切回新镜像 `base` 运行。
