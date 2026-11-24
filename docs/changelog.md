# 变更记录（自 1.0 基线起）

只记录 1.0 及之后的版本。1.2~4.9 的演进过程已归档到 `../../_归档/历史版本_1.2-4.9/CHANGELOG.md`，不进入本库。

## 1.0（基线，versionCode 32）· 2026-11-23

正式定为 1.0，作为后续所有维护更新的起点。能力清单：

- **取流/画质**：WBI 签名修正（`wts` 参与排序 + 过滤 `!'()*`），三条取流路线（wbi 签名接口 → 老 `playurl` → `platform=android`）+ WebView 兜底 `dashViaWeb`；DASH ≥480P 即启用（避免回落 MP4 降音质）；登录态用 `saveFullCookies` 全量 Cookie 并 `syncFromBrowser` 覆盖。
- **音效**：系统级 AudioEffect 链路（EQ + DynamicsProcessing 三级降级 + Virtualizer + LoudnessEnhancer），五套预设 + 超低频下潜 / 通透度两个滑杆，频响为 38Hz + 52Hz 双峰；BassBoost 停用。
- **后台播放**：`BgService` 前台服务 + MediaStyle 通知 + MediaSession（锁屏、蓝牙耳机键）+ PARTIAL 唤醒锁；`onPause` 不暂停。
- **全屏**：16:9 等比不拉伸，右上角「⤡ 小屏」胶囊，最左 26dp 点击区弹「接下来」浮层（贴左、最多 4 首）；播完自动下一首。
- **面板**：`panel.html` + `panel.css` 分离，同源热更，CSS 内联注入渲染，双缓存 + 内置兜底。

## 后续版本（按此格式追加）

### x.y（versionCode N）· 日期

- 类型：壳能力 / 面板热更
- 改动：
- 影响范围：
- 验证方式：
- 产物链接：
