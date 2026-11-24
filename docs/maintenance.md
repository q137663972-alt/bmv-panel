# B站MV 维护手册（基线 1.0）

这份手册只围绕 **1.0 这一个基线** 写，后续所有改动都在它上面长，不再保留 1.2~4.9 的历史分支。

## 1. 基线定义

| 项 | 值 |
|---|---|
| 版本 | 1.0，versionCode **32** |
| 包名 | `com.doubao.bmv` |
| 签名 SHA-256 | `aa3996e27767d553acf6d7a1381dc90ca0328cb358beb83d92e89ae25fcd724e` |
| APK SHA-256 | `9f03330138776afc0427726758d7add439f41f99bea4c6285e847ee9259bc089` |
| 内核 | Media3 / ExoPlayer 1.4.1 |
| 面板 | `panel.html` + `panel.css`，GitHub `q137663972-alt/bmv-panel` |
| 源码快照 | `02_源码/BMVPlayer_1.0/`（可直接 `bash build.sh`） |

签名必须沿用 `tools/bmv.keystore`（storepass / keypass `bmv123456`，alias `bmv`），否则老用户不能覆盖安装。

## 2. 版本号怎么涨

- **只改面板** → 不涨版本，不重发 APK（见第 5 节）。
- **改壳能力**（新增原生桥方法、权限、内核、音效链、后台策略、UI 结构）→ 涨版本：
  - `1.0.1`、`1.0.2`… 修 bug / 小改，versionCode 33、34…
  - `1.1`、`1.2`… 有可感知的新功能，同样只递增 versionCode
- 版本号必须**同时**改两处：`AndroidManifest.xml` 的 `versionCode/versionName` 与 `build.sh` 的 `VER_CODE/VER_NAME`（以及 `apksigner` 输出文件名）。漏一处会被 `aapt2 link` 参数或 manifest 覆盖，出现"装了没变化"的假象。
- 永远不降低 versionCode（Android 不允许降级安装）。

## 3. 构建

```bash
cd 资料库/02_源码/BMVPlayer_1.0        # 或工程目录 ../BMVPlayer
bash build.sh
# 产物：../build/B站MV_v<VER_NAME>.apk
```

链路：`aapt2 compile` → `aapt2 link`（`-A assets`、minSdk 21 / target 34）→ `javac`（`-source 8 -target 8 -bootclasspath android.jar`）→ `d8`（含 zxing、`tools/m3/*.jar`）→ dex 塞回 → `zipalign -f 4` → `apksigner`（v1+v2）。

工具全在 `../tools/`：`bt/android-14`、`plat/android-34/android.jar`、`jdk-17.0.21.1+2`、`zxing.jar`、`m3/`、`bmv.keystore`。

## 4. 源码结构与改动入口

| 文件 | 职责 | 什么时候动 |
|---|---|---|
| `PlayerActivity.java` | 播放页：播放器创建、DASH 取流、选集条、全屏、后台通知同步、比例计算 | 播放/交互/布局 |
| `AudioFx.java` | 系统级音效：EQ + DynamicsProcessing（三级降级）+ BassBoost/环绕/响度；`curveAt` 频响曲线 | 调音 |
| `Bili.java` | B 站接口：WBI 签名 `signQuery`、三条取流路线 `playurlDash`、`pickDash` / `parseDash` | 取流/画质 |
| `Net.java` | Cookie 管理、`api()` 412 自动修复、`saveFullCookies` / `syncFromBrowser` | 登录态 |
| `PanelActivity.java` | 面板容器：源列表、HTML/CSS 拉取与内联、缓存、JS 桥、热更 | 面板加载逻辑 |
| `BgService.java` | 后台播放前台服务、MediaStyle 通知、MediaSession | 后台策略 |
| `assets/panel.html` `assets/panel.css` | 内置面板（离线兜底） | 与线上面板同步 |
| `res/layout/activity_player.xml` | 播放页布局（`sbSub` / `sbAir` 两个滑杆） | 布局 |

几个必须记住的约束：
- 音效只能走系统 AudioEffect；ExoPlayer `AudioProcessor` 里做 PCM DSP 在本机会拖死音频线程。
- 播放器 session 由 `AudioManager.generateAudioSessionId()` 生成并 `setAudioSessionId`，效果器在 `STATE_READY` 挂载。
- 取流必须桌面 UA + `Referer: https://www.bilibili.com/video/<bvid>`，手机 UA 一律 403。
- 全屏按钮/点击区必须挂在根容器（`android.R.id.content`），挂在 `videoBox` 会被黑边裁走。
- 视频尺寸走"按容器实际宽高算比例 + 布局完成即校正"，不要再用固定延时。

## 5. 面板热更（不用发 APK）

改 `panel.css`（样式）或 `panel.html`（结构/逻辑），提交到仓库根目录即可；App 启动自动拉取，或面板页脚「检查热更」→「立即应用」。

- 拉取时会把当前生效源 URL 里的 `panel.html` 换成 `panel.css` 取同源样式，取回后**内联注入**渲染，不依赖 WebView 相对路径。
- 缓存两份：`panel_cache`（HTML）、`panel_css_cache`（CSS）；远程 CSS 失败回退内置 `assets/panel.css`。
- 老版内联 CSS 的 HTML 会被识别为"未分离"，原样使用，不叠加样式。
- 建议每次改完把 `assets/` 里那份内置面板同步更新，并顺手改页脚 `rev`，方便判断是否生效。

源的新鲜度（实测）：

| 源 | 生效延迟 | 备注 |
|---|---|---|
| GitHub Pages | 30~60 秒 | 首选 |
| gh-proxy.com | 近实时 | 国内可达 |
| ghproxy.net | 数分钟 | 边缘缓存 |
| jsDelivr | 小时级 | 兜底 |
| raw.githubusercontent | 数分钟 | 兜底（提交后 API 立即可见，raw 有缓存） |
| api.github.com | 实时 | 匿名 60 次/小时 |

## 6. 音效调参速查

预设行格式：`bass, sub, mid, high, widen, gain, comp`。

| 预设 | bass | sub | mid | high | widen |
|---|---|---|---|---|---|
| 原声直通 | 0 | 0 | 0 | 0 | 0 |
| 清澈人声（耳机默认） | 1.0 | 0 | 4.0 | 2.0 | 0.02 |
| 超重低音 | 7.5 | 9.0 | -1.0 | 1.5 | 0.10 |
| 3D丽音 | 4.0 | 2.5 | 1.0 | 1.5 | 0.22 |
| 深夜舒缓 | 5.0 | 3.0 | 0.5 | -1.0 | 0.06 |

`curveAt` 关键项：低架 `bass/(1+(f/65)^1.5)`；超低频双峰 `sub*(0.55*exp(-(ln(f/38)/0.60)^2) + 0.65*exp(-(ln(f/52)/0.55)^2))`；140Hz 凹口 −0.6；320Hz 凹口 −1.0；高架 3.5kHz 起抬。

诊断字段：`压缩链 ✓1` 完整链 / `✓2` 预均衡+限幅 / `✓3` 仅限幅 / `✗` 不支持；`EQ最低 xxHz` 决定超低频滑杆能否落地。

调参原则：B 站音轨多为 128k AAC，靠抬增益换来的"沉"很容易变成"假"，优先动 Q 值和凹口，其次动增益；总增益别超过 +11dB。

## 7. 发布清单

1. 改完源码 → 改 `AndroidManifest.xml` 与 `build.sh` 的版本号 → `bash build.sh`
2. `aapt2 dump badging build/xxx.apk | head -1` 确认 versionCode/versionName
3. `apksigner verify` 确认签名指纹未变
4. 装到真机走一遍：1080P / 音效面板 / 后台播放 / 全屏进出 / 自动下一首
5. 三件套复制到 `01_正式版x.x/`：APK、安装包 zip、源码 zip（源码 zip 含 `src res assets build.sh 使用说明.md 更新规范.md`）
6. 更新 `04_文档/变更记录.md` 与 `使用说明` 里受影响的描述
7. 上传（本地路径必须走上传拿链接，纯本地路径用户看不到）

## 8. 回滚

任何旧版本都在 `../_归档/历史版本_1.2-4.9/`（1.2~4.9 的 APK / 安装包 / 源码齐全，附 `清单.csv`）。签名一致，可直接覆盖安装回退；但要保证 versionCode 不低于已装版本，否则需先卸载。

## 9. 已知边界

- 播放器实例活在 Activity 里，从后台任务列表划掉会停止播放；要做到划掉不停需把内核搬进 `MediaSessionService`。
- 后台仍在解码视频轨，比纯听歌耗电。
- 部分老合集本身最高 480P（如 `BV1XW411w7TB`），属片源限制。
- B 站 CDN 与取流接口随时可能变，出问题先看播放页画质行的诊断输出。
