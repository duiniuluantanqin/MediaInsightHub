> 程序在持续优化中，如遇异常，请您将源文件上传到issue，我会定时查看并解决。

# MediaInsight 使用说明

## 简介

MediaInsight 是一款在线可视化多媒体格式解析工具，专为音视频开发者和学习者打造。无需安装任何软件，打开浏览器即可深入分析各类媒体文件的内部结构。通过直观的树形结构、十六进制数据对照、图表统计等功能，帮助用户快速理解音视频封装格式的原理，定位开发中遇到的各类问题。

## 核心特性

### 广泛的格式支持

支持主流的音视频封装格式，包括：
- **视频容器**：FLV、MP4、fMP4、MKV/WebM、MPEG-TS、MPEG-PS
- **音频格式**：MP3、M4A/AAC
- **流媒体**：RTP、HLS (m3u8)、HTTP-FLV
- **裸流**：H.264/H.265 Annex B

### 多维度解析视图

- **媒体信息总览**：快速查看文件的基本属性，如时长、码率、分辨率、编码格式等
- **逐帧分析**：深入每一帧的详细信息，包括 PTS/DTS 时间戳、帧类型、帧大小等
- **逐包查看**：针对 TS、RTP 等格式，支持查看原始数据包结构
- **容器结构**：可视化展示 MP4 Box 树、FLV Tag、MKV EBML 元素等容器层级结构
- **字段级详情**：每个解析字段都可展开查看，并高亮显示对应的十六进制原始数据

### 音视频预览

- **视频帧预览**：基于 WebCodec 技术，支持逐帧解码显示视频画面
- **音频播放**：支持单帧播放和连续播放，覆盖 MP3、AAC、G.711 等编码格式

### 数据可视化

- **码率分布图**：直观展示视频码率随时间的变化趋势
- **帧间隔图表**：分析帧间隔分布，快速发现卡顿或丢帧问题
- **帧类型统计**：I/P/B 帧占比一目了然

---

## 各格式详细解析说明

### FLV (Flash Video)

FLV 是一种流媒体封装格式，结构简单，常用于直播场景。MediaInsight 提供完整的 FLV 结构解析：

**文件头 (FLV Header)**
- `Signature`：文件签名，固定为 "FLV"
- `Version`：版本号，通常为 1
- `TypeFlags`：标志位，指示是否包含音频/视频
- `DataOffset`：数据区起始偏移，通常为 9

**Tag 结构**
每个 FLV Tag 包含以下字段：
- `TagType`：标签类型（8=音频，9=视频，18=脚本数据）
- `DataSize`：数据区大小（3 字节）
- `Timestamp`：时间戳（3 字节 + 1 字节扩展）
- `StreamID`：流 ID，始终为 0
- `PreviousTagSize`：前一个 Tag 的大小

**音频 Tag 详情**
- `SoundFormat`：音频编码格式（如 10=AAC, 2=MP3, 7=G.711 A-law）
- `SoundRate`：采样率（0=5.5kHz, 1=11kHz, 2=22kHz, 3=44kHz）
- `SoundSize`：采样精度（0=8bit, 1=16bit）
- `SoundType`：声道数（0=单声道, 1=立体声）
- `AACPacketType`：AAC 包类型（0=序列头, 1=原始数据）

**视频 Tag 详情**
- `FrameType`：帧类型（1=关键帧, 2=非关键帧）
- `CodecID`：编码格式（7=AVC/H.264, 12=HEVC/H.265）
- `AVCPacketType`：AVC 包类型（0=序列头, 1=NALU, 2=序列结束）
- `CompositionTime`：CTS 偏移量

**脚本数据 (onMetaData)**
- `duration`：时长
- `width`/`height`：视频尺寸
- `framerate`：帧率
- `videodatarate`/`audiodatarate`：码率

  <img width="3804" height="1748" alt="image" src="https://github.com/user-attachments/assets/299dc7fc-bd88-43b5-8113-19bb1bc151c1" />

---

### MP4 / fMP4

MP4 基于 ISO Base Media File Format，采用 Box（也称 Atom）嵌套结构。MediaInsight 以树形视图展示完整的 Box 层级：

**顶层 Box**
- `ftyp`：文件类型，包含品牌标识和兼容性信息
- `moov`：元数据容器，包含所有轨道信息
- `mdat`：媒体数据区，存储实际的音视频数据
- `moof`：分片元数据（fMP4 特有）

**moov 内部结构**
- `mvhd`：影片头，包含时长、时间刻度、创建时间等
- `trak`：轨道容器，每个音视频流对应一个 trak
  - `tkhd`：轨道头，包含轨道 ID、时长、宽高
  - `mdia`：媒体信息容器
    - `mdhd`：媒体头，包含时间刻度、时长、语言
    - `hdlr`：处理器类型（vide=视频, soun=音频）
    - `minf`：媒体信息
      - `stbl`：采样表，核心数据结构
        - `stsd`：采样描述，包含编码参数（如 avc1, hvc1, mp4a）
        - `stts`：时间到采样映射
        - `stss`：同步采样表（关键帧索引）
        - `stsc`：采样到 Chunk 映射
        - `stsz`：采样大小表
        - `stco`/`co64`：Chunk 偏移表

**编码配置解析**
- `avcC`：H.264 解码配置，包含 SPS/PPS
- `hvcC`：H.265 解码配置，包含 VPS/SPS/PPS
- `esds`：AAC 音频配置，包含 AudioSpecificConfig

<img width="3814" height="1744" alt="image" src="https://github.com/user-attachments/assets/4af6b454-f385-4660-81a8-2a84dc215757" />

---

### MKV / WebM

MKV 采用 EBML (Extensible Binary Meta Language) 结构，类似 XML 的二进制格式：

**顶层元素**
- `EBML`：EBML 头，包含版本和文档类型
- `Segment`：片段容器，包含所有媒体数据

**Segment 内部结构**
- `SeekHead`：索引表，快速定位其他元素
- `Info`：片段信息
  - `TimestampScale`：时间戳精度（纳秒）
  - `Duration`：总时长
  - `MuxingApp`/`WritingApp`：创建工具信息
- `Tracks`：轨道定义
  - `TrackEntry`：轨道条目
    - `TrackNumber`：轨道编号
    - `TrackType`：类型（1=视频, 2=音频, 17=字幕）
    - `CodecID`：编码标识（如 V_MPEG4/ISO/AVC）
    - `CodecPrivate`：编码私有数据
    - `Video`：视频参数（宽高、显示尺寸）
    - `Audio`：音频参数（采样率、声道数、位深）
- `Cluster`：数据簇，包含实际媒体数据
  - `Timestamp`：簇时间戳
  - `SimpleBlock`/`BlockGroup`：数据块

---

### MPEG-TS

MPEG-TS 是广播电视和流媒体常用的传输流格式，采用固定 188 字节的包结构：

**TS 包头 (4 字节)**
- `sync_byte`：同步字节，固定为 0x47
- `transport_error_indicator`：传输错误标志
- `payload_unit_start_indicator`：负载单元起始标志
- `transport_priority`：传输优先级
- `PID`：包标识符（13 位）
- `transport_scrambling_control`：加扰控制
- `adaptation_field_control`：自适应域控制
- `continuity_counter`：连续计数器（4 位）

**自适应域 (Adaptation Field)**
- `adaptation_field_length`：自适应域长度
- `discontinuity_indicator`：不连续标志
- `random_access_indicator`：随机访问点标志
- `PCR_flag`：PCR 存在标志
- `PCR`：节目时钟参考（42 位）

**PSI 表**
- `PAT` (PID=0x0000)：节目关联表，映射节目号到 PMT PID
- `PMT`：节目映射表，定义节目内的流
  - `program_number`：节目号
  - `PCR_PID`：PCR 所在的 PID
  - `stream_type`：流类型（0x1B=H.264, 0x24=H.265, 0x0F=AAC）
  - `elementary_PID`：基本流 PID
  - `descriptors`：描述符

**PES 包**
- `packet_start_code_prefix`：起始码前缀 (0x000001)
- `stream_id`：流标识
- `PES_packet_length`：包长度
- `PTS`/`DTS`：显示/解码时间戳

<img width="3824" height="1748" alt="image" src="https://github.com/user-attachments/assets/9a53d69b-1402-4094-841d-dd5c9a5be9d1" />

---

### MPEG-PS

MPEG-PS 是节目流格式，常用于 DVD 和监控录像（如 GB28181）：

**Pack Header**
- `pack_start_code`：包起始码 (0x000001BA)
- `system_clock_reference`：系统时钟参考 (SCR)
- `program_mux_rate`：复用速率
- `pack_stuffing_length`：填充长度

**System Header**
- `system_header_start_code`：系统头起始码 (0x000001BB)
- `rate_bound`：速率上限
- `audio_bound`：音频流数量上限
- `video_bound`：视频流数量上限
- `stream_id`/`buffer_bound_scale`/`buffer_size_bound`：流缓冲信息

**Program Stream Map (PSM)**
- `stream_type`：流类型
- `elementary_stream_id`：基本流 ID
- 支持的编码：H.264, H.265, G.711, AAC 等

**PES 包**
与 MPEG-TS 中的 PES 结构相同

<img width="3824" height="1744" alt="image" src="https://github.com/user-attachments/assets/304a6ef9-6716-4edf-b1d5-fe8ef5a916b0" />

---

### RTP

RTP (Real-time Transport Protocol) 用于实时音视频传输：

**RTP 头 (12 字节)**
- `V`：版本号（2 位），固定为 2
- `P`：填充标志（1 位）
- `X`：扩展标志（1 位）
- `CC`：CSRC 计数（4 位）
- `M`：标记位（1 位），通常表示帧结束
- `PT`：负载类型（7 位）
  - 0 = PCMU (G.711 μ-law)
  - 8 = PCMA (G.711 A-law)
  - 96-127 = 动态类型（H.264, H.265, AAC 等）
- `sequence_number`：序列号（16 位）
- `timestamp`：时间戳（32 位）
- `SSRC`：同步源标识（32 位）

**扩展头**
- `extension_profile`：扩展配置
- `extension_length`：扩展长度
- `extension_data`：扩展数据

---

### H.264/H.265 Annex B 裸流

Annex B 是 H.264/H.265 的字节流格式，使用起始码分隔 NAL 单元：

**起始码**
- 3 字节：0x000001
- 4 字节：0x00000001（通常用于 SPS/PPS/VPS/IDR）

**NAL Unit Header**

H.264:
- `forbidden_zero_bit`：禁止位，必须为 0
- `nal_ref_idc`：参考级别（2 位）
- `nal_unit_type`：NAL 类型（5 位）
  - 1 = 非 IDR 图像片
  - 5 = IDR 图像片
  - 6 = SEI
  - 7 = SPS
  - 8 = PPS

H.265:
- `forbidden_zero_bit`：禁止位
- `nal_unit_type`：NAL 类型（6 位）
  - 0-9 = 非 TSA/STSA 片
  - 16-21 = BLA/IDR/CRA 片
  - 32 = VPS
  - 33 = SPS
  - 34 = PPS
  - 39-40 = SEI
- `nuh_layer_id`：层 ID（6 位）
- `nuh_temporal_id_plus1`：时域 ID（3 位）

**SPS 关键字段**
- `profile_idc`/`level_idc`：档次和级别
- `pic_width_in_mbs`/`pic_height_in_map_units`：图像尺寸
- `frame_mbs_only_flag`：仅帧编码标志
- `chroma_format_idc`：色度格式
- `bit_depth_luma`/`bit_depth_chroma`：位深

---

### MP3

MP3 采用帧结构，每帧独立可解码：

**帧头 (4 字节)**
- `sync`：同步字（11 位全 1）
- `version`：MPEG 版本（2 位）
  - 00 = MPEG 2.5
  - 10 = MPEG 2
  - 11 = MPEG 1
- `layer`：层（2 位）
  - 01 = Layer III
  - 10 = Layer II
  - 11 = Layer I
- `protection_bit`：CRC 保护
- `bitrate_index`：码率索引（4 位）
- `sampling_frequency`：采样率索引（2 位）
- `padding_bit`：填充位
- `private_bit`：私有位
- `channel_mode`：声道模式
  - 00 = 立体声
  - 01 = 联合立体声
  - 10 = 双声道
  - 11 = 单声道
- `mode_extension`：模式扩展
- `copyright`：版权标志
- `original`：原始标志
- `emphasis`：强调模式

**帧大小计算**
- Layer I: `(12 * bitrate / samplerate + padding) * 4`
- Layer II/III: `144 * bitrate / samplerate + padding`

<img width="3818" height="1744" alt="image" src="https://github.com/user-attachments/assets/a4a5ec12-31f4-44a3-a809-b2362f7180f0" />

---

## 字段级详情与十六进制对照

MediaInsight 的一大特色是字段级的详细解析。当您在树形视图中点击任意字段时：

1. **字段值显示**：显示解析后的值，如数字、字符串、枚举名称
2. **位置信息**：显示该字段在文件中的字节偏移和位偏移
3. **十六进制高亮**：右侧的十六进制视图会自动高亮对应的原始字节
4. **位级解析**：对于位域字段，显示具体的位范围和二进制值

这种对照方式让您能够：
- 验证解析结果的正确性
- 学习各字段的二进制编码方式
- 快速定位异常数据的具体位置

---

## 隐私安全

MediaInsight 采用纯前端解析技术，所有文件处理均在浏览器本地完成，**不会上传任何数据到服务器**，充分保障您的数据隐私和安全。

## 适用场景

- **音视频入门学习**：通过可视化界面直观理解各种封装格式的内部结构
- **开发调试**：快速定位编码异常、时间戳错乱、容器结构错误等问题
- **质量分析**：评估码率分布、帧间隔稳定性等指标
- **协议分析**：分析流媒体协议的数据包结构

## 快速开始

1. 访问 [MediaInsight](https://mediainsight360.com/) 网站
2. 点击「打开文件」选择本地媒体文件，或点击「打开链接」输入流媒体地址
3. 等待解析完成，即可浏览各项分析结果
4. 点击左侧树形结构中的字段，查看详细信息和十六进制数据
5. 切换到「帧列表」查看逐帧信息，点击帧可预览视频画面或播放音频
6. 切换到「图表」查看码率分布和帧间隔统计

---

MediaInsight —— 让多媒体格式解析变得简单直观。
