# bmv-panel

B站MV（`com.doubao.bmv`）的**线上面板仓库**，同时存放 1.0 基线的维护资料。

- `panel.html`：面板结构与逻辑（App 启动时拉取）
- `panel.css`：面板样式，与 `panel.html` 同源热更；改这里不用重装 APK
- `docs/`：1.0 基线资料库
  - `docs/index.md` 总索引
  - `docs/usage.md` 使用说明
  - `docs/maintenance.md` 维护手册（构建 / 版本策略 / 热更 / 调音 / 发布清单）
  - `docs/changelog.md` 变更记录
  - `docs/baseline.md` 基线指纹（SHA-256）

改面板：编辑 `panel.html` / `panel.css` 并提交 main，App 启动即生效。
改壳能力才发 APK（versionCode 从 32 往上递增，签名沿用 `bmv.keystore`）。
