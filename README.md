# MovieClaw Android 预编译原生库

[MovieClaw](https://github.com/movieclaw/MovieClaw) 安卓客户端（`apps/android`）构建时需要的
一组 **arm64-v8a** 预编译二进制。它们体积大（压缩包约 38MB、解压后约 116MB），不宜进代码仓库，
因此单独放在这里：构建脚本会自动下载 + 校验 sha256 + 解压进 `app/src/main/jniLibs/arm64-v8a/`。

| 文件 | 作用 |
| --- | --- |
| `libmp2.so` | mpv 内核（libmpv 接口），应用运行时 `dlopen` 它 |
| `libavcodec` `libavformat` `libavfilter` `libavutil` `libswscale` `libswresample` `libavdevice` | FFmpeg 共享库，`libmp2.so` 的 `NEEDED` 依赖 |
| `libc++_shared.so` | C++ 运行库（mpv / FFmpeg 需要；客户端的 JNI 库不依赖它） |
| `libass.so` | 字幕渲染（仅供已废弃的 `cpp/libass_bridge.cpp`，可不带） |

**少任何一个 `libmp2.so` 的 NEEDED 都会让 `dlopen` 失败**——动态链接器解析依赖时不看上游是否
真的用到它，所以不能按需裁剪。

## 用法

客户端仓库 `apps/android/gradle.properties` 里写的就是这里的地址与校验和：

```
nativeLibsUrl=https://github.com/anlan-home/movieclaw-android-libs/releases/download/android-native-libs/movieclaw-android-native-arm64-v8a.zip
nativeLibsSha256=c64766c609d6e8b1096385e0fdac46ba810e06a30b81176d809ad96ff7277f72
```

拉不到**不会中断构建**，只是产出一个仅 Exo 内核的包（ISO / BDMV 原盘直读、VC-1/MPEG-2/TrueHD/PGS
软解、HDR 与 ASS 特效字幕的 MPV 路径不可用），构建日志里会写明；联网后重跑即会重试。
想从别处取（镜像 / 本地文件），可用 `-PnativeLibsUrl=… -PnativeLibsSha256=…` 或
`local.properties` 覆盖，不必改客户端仓库里的文件（见其 README 的「预编译依赖」一节）。

## 许可与来源

包内是第三方项目的二进制产物，按各自许可原样再分发：

- mpv — **GPL-2.0+**
- FFmpeg — **LGPL-2.1+ 或 GPL-2.0+**（视构建开关）
- libass — **ISC**
- libc++ — **Apache-2.0 with LLVM exception**

压缩包根目录的 `NOTICE.md` 有逐项来源与构建说明。
