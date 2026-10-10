# Apple Music Downloader

适用于 Apple Silicon Mac 的本地 Apple Music 下载工具集，包含网页图形界面、音质查询和启动脚本。

## 安装发布版

1. 在 [GitHub Releases](https://github.com/wangchuanao2008/applemusicdownloader/releases) 下载 `applemusicdownloader-v0.1.0-macos-arm64.tar.gz` 并解压。
2. 安装 Python 3、QEMU 和 ffmpeg：`brew install python qemu ffmpeg`。
3. 在终端进入解压后的 `applemusicdownloader` 文件夹，执行 `./login-wrapper.sh` 完成首次登录。
4. 运行 `./start-wrapper.sh` 启动后端，再在另一个终端运行 `./start-gui.sh`。
5. 浏览器打开 <http://127.0.0.1:8890>。

运行包包含 `amdl`、本地 QEMU 启动器、内核、初始文件系统和全新的空白登录磁盘模板；QEMU 与 ffmpeg 使用 Homebrew 安装的系统版本。首次启动命令行或图形界面时会从 `config.example.yaml` 创建个人配置 `config.yaml`。

Git 仓库保存图形界面、启动脚本、配置模板和文档。直接克隆仓库后还需从 Release 获取运行文件。个人配置、账号登录状态和下载的音乐均不包含在仓库或发布包中。第三方组件说明见 [THIRD_PARTY.md](THIRD_PARTY.md)。

## 本地使用

运行包包含适用于 Apple Silicon Mac 的 `amdl` v0.1.0、`wrapper-lite` macOS ARM64 构建包、配置模板和启动脚本。后端默认地址为 `http://127.0.0.1:12340`。

## 首次登录

在终端进入本目录，执行：

```bash
cd /path/to/applemusicdownloader
./login-wrapper.sh
```

脚本会在终端读取 Apple ID 和密码；如出现两步验证码提示，请在同一终端输入验证码。密码不会写进此目录的配置文件。上游启动器登录时会暂时把凭据放进受限的参数文件，脚本退出时会清理；请不要在多人共用的 Mac 上运行。登录时应先停止正在运行的后端服务。

## 每次使用

在一个终端中启动后端，并保持窗口开启：

```bash
./start-wrapper.sh
```

另开终端检查后端：

```bash
curl http://127.0.0.1:12340/status
```

`regions` 有值后，可运行下载器：

```bash
./run-amdl.sh "https://music.apple.com/你有权访问的歌曲或专辑链接"
```

文件默认保存在本目录下的 `AM-Lossless`、`AM-Atmos`、`AM-AAC` 或 `AM-MV`。如需更改地区、输出路径、歌词等选项，编辑 `config.yaml`。电台功能另需在 `general.media-user-token` 中填写自己的令牌。

## 图形界面（可选）

除了命令行，也可以用本目录新增的本地网页界面完成「查音质 → 选音质 → 下载」。上面的命令行方式完全保留，两者互不影响。

```bash
./start-gui.sh        # 启动后打开 http://127.0.0.1:8890（仅监听本机）
```

界面功能：

- **查询可用音质**：粘贴歌曲或专辑链接点「查询音质」。查询调用 `amdl --debug`，只读取 Apple 服务器上的档位信息，不下载任何文件（因此查询本身不需要后端在线）；结果按曲目列出 AAC / 无损 / Hi-Res / 杜比全景声的可用情况。链接带 `?i=` 单曲参数时只显示对应那一首的音质（下载同样只会下载这一首）。
- **选择音质下载**：从可用档位中单选后点「下载所选音质」，等价于命令行的 `--aac`、`--alac-max`、`--atmos` 参数。任务一次只跑一个，可随时停止；完成后列出文件并可在 Finder 中显示。
- **下载目录**：默认下载到本工具目录下的 `AM-Lossless / AM-Atmos / AM-AAC / AM-MV`（按音质分文件夹，内有 歌手/专辑 子目录）。在「下载目录」卡片点「浏览…」弹出 macOS 原生选择窗口（选中即自动保存），或直接填绝对路径（支持 `~`）后保存：界面下载的音频文件（m4a）**直接存到该目录**，不建任何子文件夹。该设置保存在 `gui/settings.json`，只对图形界面生效——不修改 `config.yaml`，命令行下载不受影响；留空即恢复默认。注意：完全平铺时不同专辑的同名曲目会互相覆盖，同一首歌重复下载会跳过。
- **不保存独立封面文件**：图形界面的下载只保留音频文件，amdl 生成的同时独立封面图（cover.jpg 等，仅限本次下载新生成的）会自动删除；封面仍内嵌在音频文件内，播放器照常显示专辑图。命令行下载行为不变。
- **任务日志**：实时滚动 amdl 输出，出错时可直接看到原因。

注意事项：下载仍需先按上文启动 `start-wrapper.sh` 后端；查询和下载共用一个任务队列；登录（Apple ID、两步验证）仍使用 `./login-wrapper.sh`，界面不处理凭据。

## 音质查看与选择

### 查看这首歌提供哪些音质

下载前可以先用 `--debug` 查询：只读取 Apple 服务器上的档位信息，**不下载任何文件**，也不需要后端在线。

```bash
cd /path/to/applemusicdownloader
./run-amdl.sh --debug 'https://music.apple.com/us/album/the-nights/1440829274?i=1440829549'
```

输出中的 `Available Audio Formats` 一段就是各档位的可用情况：

```
Available Audio Formats:
------------------------
AAC             : AAC | 2 Channel | 256 Kbps
Lossless        : ALAC | 2 Channel | 24-bit/44 kHz
Hi-Res Lossless : Not Available
Dolby Atmos     : Not Available
Dolby Audio     : Not Available
------------------------
```

- `Not Available` 表示 Apple 未提供该版本，对应参数也下不到；其余行的文字就是该曲目的实际规格（位深/采样率、声道、码率）。
- AAC 256 kbps 和无损（ALAC）绝大多数歌曲都有；Hi-Res（24 bit/48 kHz 以上）和杜比全景声因歌而异。
- 专辑链接会逐曲各输出一张表，各曲目可用档位可能不同。
- 带 `?i=` 单曲参数的链接：`--debug` 仍会输出整张专辑每首的表（下载模式则只下 `?i=` 指定的那一首）；图形界面在单曲链接下会自动只显示那一首。
- 图形界面的「查询音质」按钮与本命令等价，结果更易读。

### 选择音质下载

先启动 `wrapper-lite` 后端，再在下载器终端选择模式。以下以单曲《The Nights》为例：

```bash
cd /path/to/applemusicdownloader
song_url='https://music.apple.com/us/album/the-nights/1440829274?i=1440829549'

./run-amdl.sh "$song_url"                       # 默认：ALAC 无损（上限取 config.yaml 的 media.alac-max）
./run-amdl.sh --alac-max 44100 "$song_url"      # 无损，采样率上限 44.1 kHz（CD 级）
./run-amdl.sh --alac-max 96000 "$song_url"      # Hi-Res，采样率上限 96 kHz
./run-amdl.sh --aac "$song_url"                 # AAC 有损，保存到 AM-AAC
./run-amdl.sh --atmos "$song_url"               # 杜比全景声，歌曲提供该版本时才可下载
```

`--alac-max` 是采样率**上限**，单位为 Hz；配置文件 `media.alac-max` 当前为 `192000`（192 kHz）。它不会把 44.1 kHz 的源音频提升为 96 或 192 kHz，也不能强制 Apple Music 提供不存在的版本。需要限制全景声码率时可用 `--atmos-max 2448`；AAC 类型可在 `config.yaml` 的 `media.aac-type` 中设置。更改同一种格式的上限后，如果已有同名文件，请先将旧文件移到别处，再重新下载对比。

`.m4a` 是文件容器，不等于 AAC 编码；ALAC 和 AAC 都可以存放在 `.m4a` 中。当前 `convert.after-download: false`，没有进行下载后转码。查看其他文件的实际编码和采样率：

```bash
ffprobe -v error -select_streams a:0 \
  -show_entries stream=codec_name,bits_per_raw_sample,sample_rate \
  -of default=noprint_wrappers=1 '音频文件路径.m4a'
```

项目文档：[下载器 README](https://github.com/zhaarey/apple-music-downloader/blob/main/README-CN.md) · [wrapper-lite README](https://github.com/WorldObservationLog/wrapper/tree/lite) · [后端官方构建记录](https://github.com/WorldObservationLog/wrapper/actions/runs/36431829492)
