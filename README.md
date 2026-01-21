# MediaInsight User Guide

[中文说明](README_zh.md)

## Introduction

MediaInsight is an online visual multimedia format analysis tool designed for audio/video developers and learners. Without installing any software, you can deeply analyze the internal structure of various media files directly in your browser. Through intuitive tree structures, hexadecimal data correlation, chart statistics, and other features, it helps users quickly understand the principles of audio/video container formats and locate various issues encountered during development.

## Core Features

### Extensive Format Support

Supports mainstream audio/video container formats, including:
- **Video Containers**: FLV, MP4, fMP4, MKV/WebM, MPEG-TS, MPEG-PS
- **Audio Formats**: MP3, M4A/AAC
- **Streaming**: RTP, HLS (m3u8), HTTP-FLV
- **Raw Streams**: H.264/H.265 Annex B

### Multi-dimensional Analysis Views

- **Media Info Overview**: Quickly view basic file properties such as duration, bitrate, resolution, codec format, etc.
- **Frame-by-Frame Analysis**: Dive into detailed information for each frame, including PTS/DTS timestamps, frame type, frame size, etc.
- **Packet-by-Packet View**: For formats like TS and RTP, supports viewing raw packet structures
- **Container Structure**: Visualize MP4 Box trees, FLV Tags, MKV EBML elements, and other container hierarchies
- **Field-Level Details**: Each parsed field can be expanded to view, with corresponding hexadecimal raw data highlighted

### Audio/Video Preview

- **Video Frame Preview**: Based on WebCodec technology, supports frame-by-frame decoding and display
- **Audio Playback**: Supports single-frame and continuous playback, covering MP3, AAC, G.711, and other codecs

### Data Visualization

- **Bitrate Distribution Chart**: Intuitively displays video bitrate trends over time
- **Frame Interval Chart**: Analyzes frame interval distribution to quickly identify stuttering or frame drops
- **Frame Type Statistics**: I/P/B frame ratios at a glance

---

## Detailed Format Analysis

### FLV (Flash Video)

FLV is a streaming media container format with a simple structure, commonly used in live streaming scenarios. MediaInsight provides complete FLV structure parsing:

**FLV Header**
- `Signature`: File signature, fixed as "FLV"
- `Version`: Version number, usually 1
- `TypeFlags`: Flag bits indicating whether audio/video is present
- `DataOffset`: Data area start offset, usually 9

**Tag Structure**
Each FLV Tag contains the following fields:
- `TagType`: Tag type (8=audio, 9=video, 18=script data)
- `DataSize`: Data area size (3 bytes)
- `Timestamp`: Timestamp (3 bytes + 1 byte extension)
- `StreamID`: Stream ID, always 0
- `PreviousTagSize`: Size of the previous tag

**Audio Tag Details**
- `SoundFormat`: Audio codec format (e.g., 10=AAC, 2=MP3, 7=G.711 A-law)
- `SoundRate`: Sample rate (0=5.5kHz, 1=11kHz, 2=22kHz, 3=44kHz)
- `SoundSize`: Sample precision (0=8bit, 1=16bit)
- `SoundType`: Channel count (0=mono, 1=stereo)
- `AACPacketType`: AAC packet type (0=sequence header, 1=raw data)

**Video Tag Details**
- `FrameType`: Frame type (1=keyframe, 2=inter frame)
- `CodecID`: Codec format (7=AVC/H.264, 12=HEVC/H.265)
- `AVCPacketType`: AVC packet type (0=sequence header, 1=NALU, 2=end of sequence)
- `CompositionTime`: CTS offset

**Script Data (onMetaData)**
- `duration`: Duration
- `width`/`height`: Video dimensions
- `framerate`: Frame rate
- `videodatarate`/`audiodatarate`: Bitrate

<img width="3804" height="1748" alt="image" src="https://github.com/user-attachments/assets/299dc7fc-bd88-43b5-8113-19bb1bc151c1" />

---

### MP4 / fMP4

MP4 is based on ISO Base Media File Format, using nested Box (also called Atom) structures. MediaInsight displays the complete Box hierarchy in a tree view:

**Top-Level Boxes**
- `ftyp`: File type, contains brand identifier and compatibility information
- `moov`: Metadata container, contains all track information
- `mdat`: Media data area, stores actual audio/video data
- `moof`: Fragment metadata (fMP4 specific)

**moov Internal Structure**
- `mvhd`: Movie header, contains duration, timescale, creation time, etc.
- `trak`: Track container, one trak for each audio/video stream
  - `tkhd`: Track header, contains track ID, duration, dimensions
  - `mdia`: Media information container
    - `mdhd`: Media header, contains timescale, duration, language
    - `hdlr`: Handler type (vide=video, soun=audio)
    - `minf`: Media information
      - `stbl`: Sample table, core data structure
        - `stsd`: Sample description, contains codec parameters (e.g., avc1, hvc1, mp4a)
        - `stts`: Time-to-sample mapping
        - `stss`: Sync sample table (keyframe index)
        - `stsc`: Sample-to-chunk mapping
        - `stsz`: Sample size table
        - `stco`/`co64`: Chunk offset table

**Codec Configuration Parsing**
- `avcC`: H.264 decoder configuration, contains SPS/PPS
- `hvcC`: H.265 decoder configuration, contains VPS/SPS/PPS
- `esds`: AAC audio configuration, contains AudioSpecificConfig

<img width="3814" height="1744" alt="image" src="https://github.com/user-attachments/assets/4af6b454-f385-4660-81a8-2a84dc215757" />

---

### MKV / WebM

MKV uses EBML (Extensible Binary Meta Language) structure, a binary format similar to XML:

**Top-Level Elements**
- `EBML`: EBML header, contains version and document type
- `Segment`: Segment container, contains all media data

**Segment Internal Structure**
- `SeekHead`: Index table for quick location of other elements
- `Info`: Segment information
  - `TimestampScale`: Timestamp precision (nanoseconds)
  - `Duration`: Total duration
  - `MuxingApp`/`WritingApp`: Creation tool information
- `Tracks`: Track definitions
  - `TrackEntry`: Track entry
    - `TrackNumber`: Track number
    - `TrackType`: Type (1=video, 2=audio, 17=subtitle)
    - `CodecID`: Codec identifier (e.g., V_MPEG4/ISO/AVC)
    - `CodecPrivate`: Codec private data
    - `Video`: Video parameters (dimensions, display size)
    - `Audio`: Audio parameters (sample rate, channels, bit depth)
- `Cluster`: Data cluster, contains actual media data
  - `Timestamp`: Cluster timestamp
  - `SimpleBlock`/`BlockGroup`: Data blocks

---

### MPEG-TS

MPEG-TS is a transport stream format commonly used in broadcast television and streaming, with a fixed 188-byte packet structure:

**TS Packet Header (4 bytes)**
- `sync_byte`: Sync byte, fixed at 0x47
- `transport_error_indicator`: Transport error flag
- `payload_unit_start_indicator`: Payload unit start flag
- `transport_priority`: Transport priority
- `PID`: Packet identifier (13 bits)
- `transport_scrambling_control`: Scrambling control
- `adaptation_field_control`: Adaptation field control
- `continuity_counter`: Continuity counter (4 bits)

**Adaptation Field**
- `adaptation_field_length`: Adaptation field length
- `discontinuity_indicator`: Discontinuity flag
- `random_access_indicator`: Random access point flag
- `PCR_flag`: PCR presence flag
- `PCR`: Program Clock Reference (42 bits)

**PSI Tables**
- `PAT` (PID=0x0000): Program Association Table, maps program numbers to PMT PIDs
- `PMT`: Program Map Table, defines streams within a program
  - `program_number`: Program number
  - `PCR_PID`: PID containing PCR
  - `stream_type`: Stream type (0x1B=H.264, 0x24=H.265, 0x0F=AAC)
  - `elementary_PID`: Elementary stream PID
  - `descriptors`: Descriptors

**PES Packet**
- `packet_start_code_prefix`: Start code prefix (0x000001)
- `stream_id`: Stream identifier
- `PES_packet_length`: Packet length
- `PTS`/`DTS`: Presentation/Decoding timestamps

<img width="3824" height="1748" alt="image" src="https://github.com/user-attachments/assets/9a53d69b-1402-4094-841d-dd5c9a5be9d1" />

---

### MPEG-PS

MPEG-PS is a program stream format, commonly used for DVDs and surveillance recordings (e.g., GB28181):

**Pack Header**
- `pack_start_code`: Pack start code (0x000001BA)
- `system_clock_reference`: System Clock Reference (SCR)
- `program_mux_rate`: Mux rate
- `pack_stuffing_length`: Stuffing length

**System Header**
- `system_header_start_code`: System header start code (0x000001BB)
- `rate_bound`: Rate upper bound
- `audio_bound`: Maximum number of audio streams
- `video_bound`: Maximum number of video streams
- `stream_id`/`buffer_bound_scale`/`buffer_size_bound`: Stream buffer information

**Program Stream Map (PSM)**
- `stream_type`: Stream type
- `elementary_stream_id`: Elementary stream ID
- Supported codecs: H.264, H.265, G.711, AAC, etc.

**PES Packet**
Same structure as PES in MPEG-TS

<img width="3824" height="1744" alt="image" src="https://github.com/user-attachments/assets/304a6ef9-6716-4edf-b1d5-fe8ef5a916b0" />

---

### RTP

RTP (Real-time Transport Protocol) is used for real-time audio/video transmission:

**RTP Header (12 bytes)**
- `V`: Version number (2 bits), fixed at 2
- `P`: Padding flag (1 bit)
- `X`: Extension flag (1 bit)
- `CC`: CSRC count (4 bits)
- `M`: Marker bit (1 bit), usually indicates frame end
- `PT`: Payload type (7 bits)
  - 0 = PCMU (G.711 μ-law)
  - 8 = PCMA (G.711 A-law)
  - 96-127 = Dynamic types (H.264, H.265, AAC, etc.)
- `sequence_number`: Sequence number (16 bits)
- `timestamp`: Timestamp (32 bits)
- `SSRC`: Synchronization source identifier (32 bits)

**Extension Header**
- `extension_profile`: Extension profile
- `extension_length`: Extension length
- `extension_data`: Extension data

---

### H.264/H.265 Annex B Raw Stream

Annex B is the byte stream format for H.264/H.265, using start codes to separate NAL units:

**Start Codes**
- 3 bytes: 0x000001
- 4 bytes: 0x00000001 (typically used for SPS/PPS/VPS/IDR)

**NAL Unit Header**

H.264:
- `forbidden_zero_bit`: Forbidden bit, must be 0
- `nal_ref_idc`: Reference level (2 bits)
- `nal_unit_type`: NAL type (5 bits)
  - 1 = Non-IDR slice
  - 5 = IDR slice
  - 6 = SEI
  - 7 = SPS
  - 8 = PPS

H.265:
- `forbidden_zero_bit`: Forbidden bit
- `nal_unit_type`: NAL type (6 bits)
  - 0-9 = Non-TSA/STSA slices
  - 16-21 = BLA/IDR/CRA slices
  - 32 = VPS
  - 33 = SPS
  - 34 = PPS
  - 39-40 = SEI
- `nuh_layer_id`: Layer ID (6 bits)
- `nuh_temporal_id_plus1`: Temporal ID (3 bits)

**Key SPS Fields**
- `profile_idc`/`level_idc`: Profile and level
- `pic_width_in_mbs`/`pic_height_in_map_units`: Picture dimensions
- `frame_mbs_only_flag`: Frame-only coding flag
- `chroma_format_idc`: Chroma format
- `bit_depth_luma`/`bit_depth_chroma`: Bit depth

---

### MP3

MP3 uses a frame structure where each frame is independently decodable:

**Frame Header (4 bytes)**
- `sync`: Sync word (11 bits, all 1s)
- `version`: MPEG version (2 bits)
  - 00 = MPEG 2.5
  - 10 = MPEG 2
  - 11 = MPEG 1
- `layer`: Layer (2 bits)
  - 01 = Layer III
  - 10 = Layer II
  - 11 = Layer I
- `protection_bit`: CRC protection
- `bitrate_index`: Bitrate index (4 bits)
- `sampling_frequency`: Sample rate index (2 bits)
- `padding_bit`: Padding bit
- `private_bit`: Private bit
- `channel_mode`: Channel mode
  - 00 = Stereo
  - 01 = Joint stereo
  - 10 = Dual channel
  - 11 = Mono
- `mode_extension`: Mode extension
- `copyright`: Copyright flag
- `original`: Original flag
- `emphasis`: Emphasis mode

**Frame Size Calculation**
- Layer I: `(12 * bitrate / samplerate + padding) * 4`
- Layer II/III: `144 * bitrate / samplerate + padding`

<img width="3818" height="1744" alt="image" src="https://github.com/user-attachments/assets/a4a5ec12-31f4-44a3-a809-b2362f7180f0" />

---

## Field-Level Details and Hexadecimal Correlation

A key feature of MediaInsight is field-level detailed parsing. When you click on any field in the tree view:

1. **Field Value Display**: Shows the parsed value, such as numbers, strings, enum names
2. **Position Information**: Shows the byte offset and bit offset of the field in the file
3. **Hexadecimal Highlighting**: The hex view on the right automatically highlights the corresponding raw bytes
4. **Bit-Level Parsing**: For bit fields, shows the specific bit range and binary value

This correlation allows you to:
- Verify the correctness of parsing results
- Learn the binary encoding of each field
- Quickly locate the exact position of anomalous data

---

## Privacy & Security

MediaInsight uses pure frontend parsing technology. All file processing is done locally in the browser, **no data is uploaded to any server**, fully protecting your data privacy and security.

## Use Cases

- **Audio/Video Learning**: Intuitively understand the internal structure of various container formats through the visual interface
- **Development Debugging**: Quickly locate encoding anomalies, timestamp disorders, container structure errors, and other issues
- **Quality Analysis**: Evaluate bitrate distribution, frame interval stability, and other metrics
- **Protocol Analysis**: Analyze streaming protocol packet structures

## Quick Start

1. Visit the [MediaInsight](https://mediainsight360.com/) website
2. Click "Open File" to select a local media file, or click "Open URL" to enter a streaming address
3. Wait for parsing to complete, then browse the analysis results
4. Click on fields in the left tree structure to view detailed information and hexadecimal data
5. Switch to "Frame List" to view frame-by-frame information; click a frame to preview video or play audio
6. Switch to "Charts" to view bitrate distribution and frame interval statistics

---

MediaInsight — Making multimedia format analysis simple and intuitive.
