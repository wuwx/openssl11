# openssl11 RPM for CentOS 6

自动构建 `openssl11`（el6）RPM 并推送到 PackageCloud。宿主上下载
`openssl11-1.1.1k-7.el7.src.rpm`，在 `centos:6` 容器里解包出 sources / patches，
用仓库里的 `openssl11.spec` 构建，最后用 `package_cloud` 推送。

## 配置

在仓库 **Settings → Secrets and variables → Actions** 里设置：

- `PACKAGECLOUD_REPO`（variable）：目标仓库，如 `wuwx/openssl11`
- `PACKAGECLOUD_TOKEN`（secret）：PackageCloud 的 push token（带写权限）

## 触发

- `main` 分支：只构建；
- `v*` tag（如 `v1.1.1k-7`）或手动 `workflow_dispatch`：构建并推送。

## 说明

逻辑全在 `.github/workflows/build.yml`，没有独立脚本。

- 用 `docker run centos:6` 而不是 `container:`：JS action（node ≥ 20）要 glibc ≥ 2.28，
  el6 只有 2.12，`actions/checkout` 在 el6 容器里跑不起来。
- src.rpm 在宿主上下载，不下沉到 el6：el6 的 rpm/curl 抓 dl.fedoraproject.org 会被回 404。
- 容器里装了 `epel-release` 并把源改指 EPEL 6 archive，只为满足 spec 里的
  `perl-interpreter`（el7 风格的名字，EPEL 6 用一个零文件的壳包提供它）。
- el6 的 rpmbuild 没有 `--nocheck`，`%check` 会照常执行（spec 里 `make test` 已注释）。
- distro 固定 `el/6`（`distro_version_id=123`，CentOS 6 与 RHEL 6 共用）。
- `centos:6` 拉不动时可换 `quay.io/centos/centos:6`。
