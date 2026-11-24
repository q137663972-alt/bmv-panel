# 1.0 基线指纹

用于判断"手里这份是不是基线"。

## 产物

| 文件 | SHA-256 |
|---|---|
| `01_正式版1.0/B站MV_v1.0.apk` | `9f03330138776afc0427726758d7add439f41f99bea4c6285e847ee9259bc089` |
| `03_面板/panel.html` | `66527417766f2fefe647651121ff7ddf5cc0562114c957772e0a219e7d051946` |
| `03_面板/panel.css` | `b7fe8b9e04ccdfa0b6e22c9338a8a1e51aadfdff5febc169011b2cfd62988c23` |

## 签名

- APK 签名证书 SHA-256：`aa3996e27767d553acf6d7a1381dc90ca0328cb358beb83d92e89ae25fcd724e`
- keystore：`../tools/bmv.keystore`，alias `bmv`，口令 `bmv123456`

## 关键源码

| 文件 | SHA-256 |
|---|---|
| `build.sh` | `c12270ae400ecb09d10f99c848dfc28f7216d3c161ad016492601e370fefd7d8` |
| `AndroidManifest.xml` | `d865108baca589e96ed08e907ff26b298beb317f4862c4e9231b1e87a92f940b` |
| `src/com/doubao/bmv/PlayerActivity.java` | `565b2edff3c4b6635dbc9ae57a6eb9bdeded55c59f50abe927b06a27708ae636` |
| `src/com/doubao/bmv/AudioFx.java` | `2b495b80e3324b138ac5b11b6957804bf038283ef9bea44fb9426fa35ec946bf` |
| `src/com/doubao/bmv/Bili.java` | `352a0d51fedfce230850311b739e7859761827c585e1e6cab6deb955252ee6eb` |
| `src/com/doubao/bmv/Net.java` | `e8e324b3b8e85d96bbf1a6a1f9fdba4211b4e42f19d2ce2faee4132b14a5b69b` |
| `src/com/doubao/bmv/PanelActivity.java` | `16f2630ebcd4336274dcc4c51e08331de641759d87ee3f525a88b4610b6533d9` |
| `src/com/doubao/bmv/BgService.java` | `f796d0782fe6e54ac78e395cb6bfdc1bd23778f52c591832627e150a0aa15ffe` |

## 校验命令

```bash
sha256sum 01_正式版1.0/B站MV_v1.0.apk
../tools/bt/android-14/aapt2 dump badging 01_正式版1.0/B站MV_v1.0.apk | head -1
# 期望：versionCode='32' versionName='1.0'
```
