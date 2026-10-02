# openssl11 RPM for CentOS 6

自动构建 `openssl11`（el6）RPM 并推送到 PackageCloud。构建时用 `docker run` 起
`centos:6`，在里面从 Fedora 归档下载 `openssl11-1.1.1k-7.el7.src.rpm`，
`rpm -ivh` 解包出 sources / patches，spec 用仓库里的 `openssl11.spec`，再用
`rpmbuild` 构建，最后用 `package_cloud` CLI 推送。

## 仓库配置

在 GitHub 仓库的 **Settings → Secrets and variables → Actions** 里设置：

- **Repository variables**
  - `PACKAGECLOUD_REPO`：目标仓库，格式 `用户名/仓库名`，例如 `wuwx/openssl11`
- **Repository secrets**
  - `PACKAGECLOUD_TOKEN`：PackageCloud 的 push token（带写权限）

## 触发方式

- 推送 `main` 分支：只构建，不推送；
- 推送 `v*` 形式的 tag（如 `v1.1.1k-7`）：构建并推送到 PackageCloud；
- Actions 页面手动 `workflow_dispatch`：构建并推送。

## 说明

构建逻辑全部在 `.github/workflows/build.yml` 里，没有独立脚本。

- 构建走 `docker run centos:6`，而不是 workflow 的 `container:`：JS action（node ≥ 20）
  要求 glibc ≥ 2.28，el6 只有 2.12，`actions/checkout` 在 el6 容器里跑不起来。
- 构建固定加 `--nocheck`（跳过 OpenSSL 测试套件，避免 el6 环境差异导致失败）。
- PackageCloud 的 distro 固定为 `Enterprise Linux 6`，即推送路径里的 `el/6`
  （对应 `distro_version_id=123`，CentOS 6 与 RHEL 6 共用）。如需更改改推送命令里的路径。
- 如果 `centos:6` 镜像无法拉取，可改用 `quay.io/centos/centos:6`。
