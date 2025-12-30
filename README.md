# ComfyUI-FFmpegURLMedia


## 介绍

本插件提供了一套基于FFmpeg的ComfyUI节点，主要功能包括：

- 🎬 **从URL加载视频** - 支持直接从网络URL加载视频文件
- 🎵 **从URL加载音频** - 支持直接从网络URL加载音频文件
- 🖼️ **从URL加载图像** - 支持直接从网络URL加载图像文件
- 🔗 **多视频拼接** - 将多个视频片段按顺序合并成一个长视频
- 🎧 **音频处理** - 为视频添加背景音乐、替换原声或混合音频
- 📦 **兼容性强** - 输出与ComfyUI原生VIDEO/AUDIO类型兼容

---

## 说明

使用该节点**不需要手动安装FFmpeg**，插件首次运行时会**自动下载**FFmpeg到插件目录。

如果自动下载失败，可以手动下载FFmpeg并放置到 `ComfyUI/custom_nodes/ComfyUI-FFmpeg-VideoMerge/bin/` 目录下。

---

## 安装

#### 方法1：Git Clone

1. 进入节点目录 `ComfyUI/custom_nodes/`
2. 执行以下命令：

```bash
git clone https://github.com/hhayiyuan/ComfyUI-FFmpegURLMedia.git
```

3. 重启ComfyUI

#### 方法2：手动安装

1. 下载本仓库的ZIP压缩包
2. 解压到 `ComfyUI/custom_nodes/` 目录下
3. 确保文件夹名为 `ComfyUI-FFmpegURLMedia`
4. 重启ComfyUI

---

## 节点介绍

### FFmpeg Video URL 节点

**作用：从网络URL加载视频文件**

![FFmpeg Video URL节点截图](screenshots/video_url_node.png)

##### 参数说明

| 参数名 | 说明 | 示例 |
|-------|------|------|
| url | 视频文件的网络地址 | `https://example.com/video.mp4` |
| show_preview | 是否在节点中显示预览 | `true` |

##### 输出

| 输出名 | 类型 | 说明 |
|-------|------|------|
| VIDEO | VIDEO | 视频对象，可连接到其他视频处理节点 |

---

### FFmpeg Audio URL 节点

**作用：从网络URL加载音频文件**

![FFmpeg Audio URL节点截图](screenshots/audio_url_node.png)

##### 参数说明

| 参数名 | 说明 | 示例 |
|-------|------|------|
| url | 音频文件的网络地址 | `https://example.com/audio.mp3` |
| show_preview | 是否在节点中显示预览 | `true` |

##### 输出

| 输出名 | 类型 | 说明 |
|-------|------|------|
| AUDIO | AUDIO | 音频对象，可连接到Video Combine等节点 |

**注意：** 输出的AUDIO类型与ComfyUI原生AUDIO格式兼容，可以直接连接到VideoHelperSuite的Video Combine节点。

---

### FFmpeg Image URL 节点

**作用：从网络URL加载图像文件**

![FFmpeg Image URL节点截图](screenshots/image_url_node.png)

##### 参数说明

| 参数名 | 说明 | 示例 |
|-------|------|------|
| url | 图像文件的网络地址 | `https://example.com/image.png` |
| show_preview | 是否在节点中显示预览 | `true` |

##### 输出

| 输出名 | 类型 | 说明 |
|-------|------|------|
| IMAGE | IMAGE | 图像张量，格式为 `[B, H, W, C]` |

---

### FFmpeg Media Merge 节点

**作用：将多个视频和音频合并处理**

这是本插件**最核心的节点**，支持多种合并模式。

![FFmpeg Media Merge节点截图](screenshots/media_merge_node.png)

##### 参数说明

| 参数名 | 说明 | 默认值 |
|-------|------|--------|
| video_count | 视频输入数量（0-10） | 1 |
| audio_count | 音频输入数量（0-5） | 1 |
| audio_mode | 音频模式：`mix`混合 / `replace`替换 | mix |
| video_volume | 原视频音频音量（0.0-2.0） | 1.0 |
| audio_volume | 外部音频音量（0.0-2.0） | 1.0 |
| output_name | 输出文件名前缀 | merged_output |
| save_video | 是否保存到output目录 | true |

##### 音频模式说明

| 模式 | 说明 |
|-----|------|
| `mix` | 保留原视频声音，与外部音频混合播放 |
| `replace` | 删除原视频声音，仅使用外部音频 |

##### 输入端口

根据 `video_count` 和 `audio_count` 的设置，节点会显示相应数量的输入端口：

- `video_1` ~ `video_10`：视频输入（按顺序拼接）
- `audio_1` ~ `audio_5`：音频输入（按顺序拼接）

##### 合并规则

| 输入组合 | 处理方式 |
|---------|---------|
| 2+个视频，无音频 | 视频按顺序拼接 |
| 无视频，2+个音频 | 音频按顺序拼接，输出MP3 |
| 1个视频 + 1+个音频 | 视频添加音频轨道 |
| 2+个视频 + 1+个音频 | 先拼接视频，再添加音频 |

**注意：** 至少需要2个有效的媒体文件（视频或音频）才能执行合并。

---

### FFmpeg Single AV Merge 节点

**作用：将单个视频与单个音频合并**

相比Media Merge节点提供了更精细的控制，特别是**音频循环**功能。

![FFmpeg Single AV Merge节点截图](screenshots/single_av_merge_node.png)

##### 参数说明

| 参数名 | 说明 | 默认值 |
|-------|------|--------|
| video_path | 视频输入（必需） | - |
| audio_path | 音频输入（必需） | - |
| output_name | 输出文件名前缀 | video_with_audio |
| audio_mode | 音频模式：`replace`替换 / `mix`混合 | replace |
| video_volume | 原视频音量（0.0-2.0） | 1.0 |
| audio_volume | 外部音频音量（0.0-2.0） | 1.0 |
| loop_audio | 是否循环音频以匹配视频时长 | true |
| save_video | 是否保存到output目录 | true |

##### loop_audio 参数说明

| 设置 | 效果 |
|-----|------|
| `true` | 如果音频比视频短，音频会自动循环播放直到视频结束 |
| `false` | 如果音频比视频短，音频结束后将保持静音直到视频结束 |

---

## 工作流示例

### 示例1：为视频添加背景音乐

```
FFmpeg Video URL ──┐
                   ├──> FFmpeg Single AV Merge ──> 输出视频
FFmpeg Audio URL ──┘
                   
设置：
- audio_mode: mix
- video_volume: 1.0
- audio_volume: 0.3
- loop_audio: true
```

### 示例2：多个视频拼接

```
FFmpeg Video URL (视频1) ──┐
FFmpeg Video URL (视频2) ──┼──> FFmpeg Media Merge ──> 拼接后的视频
FFmpeg Video URL (视频3) ──┘

设置：
- video_count: 3
- audio_count: 0
```

### 示例3：视频配音（替换原声）

```
FFmpeg Video URL ──┐
                   ├──> FFmpeg Single AV Merge ──> 配音后的视频
FFmpeg Audio URL ──┘

设置：
- audio_mode: replace
- loop_audio: false
```

---

## 输出规格

| 属性 | 规格 |
|-----|------|
| 视频编码 | H.264 (libx264) |
| 音频编码 | AAC |
| 分辨率（Media Merge） | 1080×1920（竖屏） |
| 帧率 | 30 FPS |
| 音频采样率 | 44100 Hz |

---

## 支持的格式

**视频格式**
- MP4（推荐）、MOV、AVI、MKV、WebM

**音频格式**
- MP3（推荐）、WAV、AAC、FLAC、OGG、M4A

**图像格式**
- JPG、PNG（支持透明通道）、WebP、GIF（静态帧）

---

## 常见问题

### Q: 节点在菜单中找不到？

确认插件文件夹已正确放置在 `custom_nodes` 目录下，然后重启ComfyUI。

### Q: 提示"Need at least 2 media files"？

Media Merge节点需要至少2个有效的媒体输入。请检查：
- `video_count` 和 `audio_count` 设置是否正确
- 是否已连接足够数量的视频/音频节点

### Q: URL加载失败？

- 确保URL是视频/音频文件的直接链接
- URL需要可公开访问，不支持需要登录的链接
- 检查网络连接是否正常

### Q: 输出视频没有声音？

- 检查音频节点的URL是否正确
- 确认 `audio_volume` 不为0
- 如果使用mix模式，确保原视频有音频轨道

---

## 更新日志

### v1.0.0
- 初始版本发布
- 支持从URL加载视频/音频/图像
- 支持多视频拼接（最多10个）
- 支持视频音频混合/替换
- 支持音频循环功能
- 输出类型兼容ComfyUI原生VIDEO/AUDIO

---

## License

MIT License
