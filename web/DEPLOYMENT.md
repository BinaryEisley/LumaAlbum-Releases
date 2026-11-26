# v4 Docker 部署（Docker Hub 基线镜像）

v4 首次部署统一使用发布在 Docker Hub 上的多架构基线镜像：

- **`binaryeisley/luma-album:4.2.0`**（同时包含 `linux/amd64` 与 `linux/arm64`，Docker 会自动匹配宿主机架构）
- 底座为 `debian:bookworm-slim`（glibc `2.36`），内置 `ImageMagick`、`libheif1`、非 root 运行用户（`10001`）与 v4 无签名网页更新代理（`app_update_agent.py`）。

无需手动下载或导入 `.tar` 镜像归档。

## 群晖 NAS（x86_64 / amd64）

将以下文件放在同一个部署目录，下面的命令从该目录执行：

- `compose.nas.yaml`
- `.env`
- `secrets/web-password.txt`

准备照片路径、数据路径和更新目录后执行：

```bash
docker compose -f compose.nas.yaml config --quiet
docker compose -f compose.nas.yaml up -d
curl -fsS http://127.0.0.1:3939/health
```

容器使用 `linux-amd64-v1` 运行时。

## 香橙派 5 Plus / iStoreOS（ARM64）

将以下文件放在同一个部署目录，下面的命令从该目录执行：

- `compose.orangepi5plus.yaml`
- `.env`
- `secrets/web-password.txt`

执行：

```bash
docker compose -f compose.orangepi5plus.yaml config --quiet
docker compose -f compose.orangepi5plus.yaml up -d
curl -fsS http://127.0.0.1:3939/health
```

容器使用 `linux-arm64-v1` 运行时。照片目录只读挂载到 `/photos`，应用数据（`/data`）和更新目录（`/updates`）必须分别可写。

## 后续版本升级（`.luma` 在线更新）

容器基于 `binaryeisley/luma-album:4.2.0` 启动后，**后续版本更新不再需要重新拉取 Docker 镜像**：

1. 从 GitHub Releases 下载对应架构的 `.luma` 升级包（`linux-amd64-v1` 或 `linux-arm64-v1`）。
2. 打开拾光网页 → **偏好设置 → 应用更新** → 选择 `.luma` 文件 → **准备更新** → **替换并更新**。
3. 常规升级包默认使用 `dataPolicy=preserve`，升级时完整保留 `/data` 下的相册源、索引缓存、收藏、轮播方案与保险箱数据；挂载在 `/photos` 的原始照片目录只读独立。

> 如果未来手动拉取了版本号更高的 Docker 镜像，容器内的 `app_update_agent.py` 会在启动时自动识别新镜像版本高于 `/updates` 旧版本并切回新镜像 `base`，同时保留 `/data` 数据不变。
