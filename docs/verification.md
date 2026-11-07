# v1.19.0 验证记录

适配 CPR `>=3.43.0,<3.44.0`，Linux x86_64、glibc 2.36+。SDK 与官方打包工具固定至 v3.43.0 提交 `402bc37fbb5c83378cc83cec89e9fa251e4a99c1`。发布目标为 [DNAlec/cpr-jzjc v1.19.0](https://github.com/DNAlec/cpr-jzjc/releases/tag/v1.19.0)。

## 合同核对

对照 v3.42.0 与 v3.43.0 官方源码，Rust SDK、插件运行时生产代码及官方 CLI 无变化。前端桥接新增 `openExternal`，既有管理请求、主题事件与模型桥接保持兼容；本插件没有外链跳转调用，无需业务迁移。

运维错误日志接口保持不变，继续读取 `/api/admin/operations/errors`。OpenAI 账户选择器的变化增加了排队失败迁移绑定的诊断记录，`required_account` 仍排除自动换号路径，检测继续固定到提交的账户。保留 RPC v7、管理能力 v2、180 秒维护预算及既有私有状态 schema。前端组件库继续使用 UI 0.8.0。

## 构建

使用 Rust 1.99.1、Node.js 24.19.0、pnpm 11.21.0、Zig 0.15.2、cargo-zigbuild 0.24.1。编译绑定 CPU 0，Rust 使用 `CARGO_BUILD_JOBS=1`；官方 CLI 从 CPR v3.43.0 源码和其 workspace 锁文件构建。

从已通过包摘要及全部资源摘要校验的 1.18.0 安装包复用 Python/NumPy 运行时，并刷新评分与上游源码。重新收集第三方许可证后核对正文集合未变，保留既有排列。Rust 与前端依赖保持锁定。

## 验证结果

- Rust 格式检查、Clippy `--all-targets --all-features -- -D warnings`、25 项单元测试通过。
- 前端锁文件安装、TypeScript 类型检查及生产构建通过；Python 评分 6 项单元测试及分页接口回归通过。
- 最终安装包摘要、全部资源摘要、RPC v7、宿主范围、18 个模型离线校验、Rust 运行时解包与完整评分通过。ELF 引用的 GLIBC 符号最高版本为 2.34。
- 1.18.0 → 1.19.0 升级验证通过：配置、计划、历史及指纹库保留，提交幂等性保留，无检测请求重放。
- 浏览器使用最终包生成的模拟数据及包内前端，39 组检查通过，页面错误为 0；覆盖筛选、提交、配置、报告、进度、明暗主题与移动布局。
- 最终包二进制的 75 项 RPC 集成场景全部通过；分三组各 25 项运行，耗时分别为 106.100、207.889、130.468 秒。覆盖账户绑定、日志触发、复测、取消、长响应与并发预算、保存和重启恢复。

安装包：`dist/personal.modeltrace-health-1.19.0-x86_64-unknown-linux-gnu.tar.gz`，30,830,230 字节。

SHA-256：`1dc977a2e1b53ba9ee5b1a53069bd567378f89ff9486d3df1e560fbf25c45ff9`。

## 复现命令

激活 `.venv`，安装 `requirements-dev.txt` 及构建工具后执行：

```bash
cargo fmt --manifest-path backend/Cargo.toml --check
cargo clippy --manifest-path backend/Cargo.toml --locked --all-targets --all-features -- -D warnings
cargo test --manifest-path backend/Cargo.toml --locked
pnpm --dir frontend install --frozen-lockfile
pnpm --dir frontend build
python -m unittest discover -s engine/tests -v
node scripts/api_smoke.mjs
python scripts/refresh_runtime.py dist/personal.modeltrace-health-1.18.0-x86_64-unknown-linux-gnu.tar.gz
CARGO_BUILD_JOBS=1 RAYON_NUM_THREADS=1 taskset -c 0 cargo zigbuild \
  --manifest-path backend/Cargo.toml --locked --release --target x86_64-unknown-linux-gnu.2.36
CARGO_BUILD_JOBS=1 taskset -c 0 cargo build \
  --manifest-path .cache/codex-proxy-rs-v3.43.0/backend/Cargo.toml \
  -p codex-proxy-plugin-cli --release --locked
.cache/codex-proxy-rs-v3.43.0/backend/target/release/cpr-plugin package \
  --manifest plugin.json --binary backend/target/x86_64-unknown-linux-gnu/release/modeltrace-health \
  --target x86_64-unknown-linux-gnu --resource-map web=frontend/dist --output-dir dist
python scripts/verify_artifact.py dist/personal.modeltrace-health-1.19.0-x86_64-unknown-linux-gnu.tar.gz
python scripts/upgrade_smoke.py \
  dist/personal.modeltrace-health-1.18.0-x86_64-unknown-linux-gnu.tar.gz \
  dist/personal.modeltrace-health-1.19.0-x86_64-unknown-linux-gnu.tar.gz
mkdir -p .cache/pkg-v1.19.0
tar -xzf dist/personal.modeltrace-health-1.19.0-x86_64-unknown-linux-gnu.tar.gz -C .cache/pkg-v1.19.0
python scripts/protocol_smoke.py --binary .cache/pkg-v1.19.0/bin/modeltrace-health -v
python scripts/ui_fixture.py --binary .cache/pkg-v1.19.0/bin/modeltrace-health
PLAYWRIGHT_HOST_PLATFORM_OVERRIDE=ubuntu24.04-x64 \
  PLAYWRIGHT_BROWSERS_PATH="$PWD/.cache/browser-bin" UI_DIST=.cache/pkg-v1.19.0/web \
  node scripts/ui_smoke.mjs
```

浏览器使用 Playwright 1.58.2。当前 Ubuntu 26.04 未被该版识别，显式使用 Ubuntu 24.04 的 Chromium 构建。浏览器需允许本地进程通信。

## 验证边界

集成验证使用最终安装包二进制、生产前端与模拟宿主，未在生产 CPR 3.43.0 宿主安装，也未访问真实账号。浏览器使用模拟数据。

## 历史验证

[1.18.0](verification-v1.18.0.md)、[1.17.0](verification-v1.17.0.md)、[1.16.0](verification-v1.16.0.md)。
