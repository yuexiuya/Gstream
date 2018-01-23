# 一、Goal

本章将介绍一系列工具

- 如何应用 命令行 搭建一个 GStreamer pipelines
- 如何找出您可用的GStreamer元素及其功能
- 如何发现媒体文件的内部结构

# 二、gst-launch-1.0

该工具用于接收管道的文本描述，然后它帮助你快速检查管道是否可用，并调用 GStreamer API调用执行实际的操作。

记住，它只能创建简单的管道。特别是，它只能模拟管道与应用程序之间的交互，达到一定的水平。无论如何，快速测试管道非常方便，并且每天世界各地的GStreamer开发人员都在使用。

请注意，gst-launch-1.0主要是开发人员的调试工具。您不应该在它之上构建应用程序。相反，使用GStreamer API的gst_parse_launch()函数作为从管道描述构建管道的简单方法。<font color="0xFFFF"> —— 不要把他写到程序里面去 </font>

虽然构建管道描述的规则非常简单，但是多个元素的连接可以很快使这些描述类似于黑魔法。不要害怕，因为每个人最终都会学习gst-launch-1.0语法。<font color="0xFFFF"> —— 使用 gst-launch-1.0 搭建管道十分复杂，谨慎轻松的应对。</font>

下面将讲述 如何在命令行中配合 gst-launch-1.0 访问element、Properties 、pads、

## 2.1 Element

- 在简单的形式中，管道描述 是用感叹号(!)分隔的元素类型列表

```
gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink
```

- Named elements <font color="0xFFFF"> —— 为 Element 重命名，简化书写</font>

简化 tee -> t, 通过 queue 完成了多线程操作，记住使用变量时 使用 t. ， 记住这个 "." !!!

```
gst-launch-1.0 videotestsrc ! videoconvert ! tee name=t ! queue ! autovideosink t. ! queue ! autovideosink
```

## 2.2 Properties

- 属性可以附加到元素中，形式*属性=值*(可以指定多个属性，由空格分隔)。

```
gst-launch-1.0 videotestsrc pattern=11 ! videoconvert ! autovideosink
```

## 2.3 Pads

GStreamer 会在链接两个元素时自动判定使用哪个Pad，但有时您可能需要直接指定Pad。您可以通过在元素的名称后面加上一个点加上Pad名称来实现这一点(它必须是一个命名的元素)。通过使用gst-inspection-1.0工具学习元素的名称。

- 从网络上获取一段视频，并把视频流存放到 sintel_video.mkv

```
gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! matroskademux name=d d.video_00 ! matroskamux ! filesink location=sintel_video.mkv
```

- 从网络上获取一段视频，并把音频流存放到 sintel_video.mka

```
gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! matroskademux name=d d.audio_00 ! vorbisparse ! matroskamux ! filesink location=sintel_audio.mka
```

## 2.4 Caps filters

当一个元素有多个输出垫时，可能会发生下一个元素的链接不明确:下一个元素可能有多个兼容的输入垫，或者它的输入垫可能与所有输出垫的衬垫上限兼容。在这些情况下，GStreamer将使用可用的第一个pad链接，这几乎等于说GStreamer将随机选择一个输出pad。<font color="0xFFFF">——如何指定一个 Cap filters </font>

- 无 Caps filters

```
gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! matroskademux ! filesink location=test
```

- 有 Caps filters

```
gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! matroskademux ! video/x-vp8 ! matroskamux ! filesink location=sintel_video.mkv
```

Caps filters 就像是一个 元素通道 —— 只接受特定流的元素通道。在上面的 command 中 在 matroskademux 和 matroskamux 之间 有一个 video/x-vp8 的过滤器，有效的隔离了流的属性。

## 2.5 例子

```
gst-launch-1.0 playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm
```

```
gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! matroskademux name=d ! queue ! vp8dec ! videoconvert ! autovideosink d. ! queue ! vorbisdec ! audioconvert ! audioresample ! autoaudiosink
```

```
gst-launch-1.0 uridecodebin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm name=d ! queue ! theoraenc ! oggmux name=m ! filesink location=sintel.ogg d. ! queue ! audioconvert ! audioresample ! flacenc ! m.
```

```
gst-launch-1.0 uridecodebin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! queue ! videoscale ! video/x-raw-yuv,width=320,height=200 ! videoconvert ! autovideosink
```


# 三、gst-inspect-1.0

gst-inspect-1.0 有三种使用方式：

- 没有参数： 将列出所有可用的元素
- 参数是一个文件：<font color="red">使用失败</font>
- 参数是一个元素：列出元素所有的属性

```
gst-inspect-1.0 vp8dec

Factory Details:
  Rank                     primary (256)
  Long-name                On2 VP8 Decoder
  Klass                    Codec/Decoder/Video
  Description              Decode VP8 video streams
  Author                   David Schleef <ds@entropywave.com>, Sebastian Dröge <sebastian.droege@collabora.co.uk>

Plugin Details:
  Name                     vpx
  Description              VP8 plugin
  Filename                 /usr/lib64/gstreamer-1.0/libgstvpx.so
  Version                  1.6.4
  License                  LGPL
  Source module            gst-plugins-good
  Source release date      2016-04-14
  Binary package           Fedora GStreamer-plugins-good package
  Origin URL               http://download.fedoraproject.org

GObject
 +----GInitiallyUnowned
       +----GstObject
             +----GstElement
                   +----GstVideoDecoder
                         +----GstVP8Dec

Pad Templates:
  SINK template: 'sink'
    Availability: Always
    Capabilities:
      video/x-vp8

  SRC template: 'src'
    Availability: Always
    Capabilities:
      video/x-raw
                 format: I420
                  width: [ 1, 2147483647 ]
                 height: [ 1, 2147483647 ]
              framerate: [ 0/1, 2147483647/1 ]


Element Flags:
  no flags set

Element Implementation:
  Has change_state() function: gst_video_decoder_change_state

Element has no clocking capabilities.
Element has no URI handling capabilities.

Pads:
  SINK: 'sink'
    Pad Template: 'sink'
  SRC: 'src'
    Pad Template: 'src'

Element Properties:
  name                : The name of the object
                        flags: readable, writable
                        String. Default: "vp8dec0"
  parent              : The parent of the object
                        flags: readable, writable
                        Object of type "GstObject"
  post-processing     : Enable post processing
                        flags: readable, writable
                        Boolean. Default: false
  post-processing-flags: Flags to control post processing
                        flags: readable, writable
                        Flags "GstVP8DecPostProcessingFlags" Default: 0x00000403, "mfqe+demacroblock+deblock"
                           (0x00000001): deblock          - Deblock
                           (0x00000002): demacroblock     - Demacroblock
                           (0x00000004): addnoise         - Add noise
                           (0x00000400): mfqe             - Multi-frame quality enhancement
  deblocking-level    : Deblocking level
                        flags: readable, writable
                        Unsigned Integer. Range: 0 - 16 Default: 4
  noise-level         : Noise level
                        flags: readable, writable
                        Unsigned Integer. Range: 0 - 16 Default: 0
  threads             : Maximum number of decoding threads
                        flags: readable, writable
                        Unsigned Integer. Range: 1 - 16 Default: 0
```

# 四、gst-dicoverer-1.0

它从命令行接收一个URI，并打印关于GStreamer可以提取的媒体的所有信息。找出哪些容器和编解码器被用来制作媒体是很有用的，因此，您需要将哪些元素放入管道中来播放它。

```
gst-discoverer-1.0 https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm -v

Analyzing https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm
Done discovering https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm
Topology:
  container: video/webm
    audio: audio/x-vorbis, channels=(int)2, rate=(int)48000
      Codec:
        audio/x-vorbis, channels=(int)2, rate=(int)48000
      Additional info:
        None
      Language: en
      Channels: 2
      Sample rate: 48000
      Depth: 0
      Bitrate: 80000
      Max bitrate: 0
      Tags:
        taglist, language-code=(string)en, container-format=(string)Matroska, audio-codec=(string)Vorbis, application-name=(string)ffmpeg2theora-0.24, encoder=(string)"Xiph.Org\ libVorbis\ I\ 20090709", encoder-version=(uint)0, nominal-bitrate=(uint)80000, bitrate=(uint)80000;
    video: video/x-vp8, width=(int)854, height=(int)480, framerate=(fraction)25/1
      Codec:
        video/x-vp8, width=(int)854, height=(int)480, framerate=(fraction)25/1
      Additional info:
        None
      Width: 854
      Height: 480
      Depth: 0
      Frame rate: 25/1
      Pixel aspect ratio: 1/1
      Interlaced: false
      Bitrate: 0
      Max bitrate: 0
      Tags:
        taglist, video-codec=(string)"VP8\ video", container-format=(string)Matroska;

Properties:
  Duration: 0:00:52.250000000
  Seekable: yes
  Tags:
      video codec: VP8 video
      language code: en
      container format: Matroska
      application name: ffmpeg2theora-0.24
      encoder: Xiph.Org libVorbis I 20090709
      encoder version: 0
      audio codec: Vorbis
      nominal bitrate: 80000
      bitrate: 80000
```
