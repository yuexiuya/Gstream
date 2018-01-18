# Goal

用GStreamer构建的管道不需要完全关闭。数据可以被注入到管道中，并在任何时候以多种方式从它中提取出来。

本教程展示了:

- 如何将外部数据注入通用的GStreamer管道。

- 如何从通用的GStreamer管道中提取数据。

- 如何访问和操作这些数据。

(Playback tutorial 3: short-cutting the pipeline 说明了如何在基于在 playbin 中实现相同的目标。)

# 一、介绍

Applications 和 pipeline 中的 data 交互的方式有很多种，这里介绍最简单的一种。

用于 Applications 向 Gstreamer 注入 data 的 Element = appsrc ， 从 Gstreamer 中提取 data 的 element = appsink。 appsrc 和 appsink 是多功能的，他们有自己的API，可以通过 gstreamer 的库来访问。在本教程中，我们将使用更加简单的方法并通过信号来控制它们。

appsrc 可以在多种模式下工作 : ( 主要是可以控制流入数据的速度 )

- push mode : 应用程序以自己的速度推动数据
- pull mode : 它每次需要时都从应用程序请求数据。在这个模式下，应用程序也可以选择 blocked。 它也可以监听 enough-data 和 need-data 信号来控制流。

# 二、Buffers

data 在 GStreamer pipeline 中通过 buffers 传输。这个例子将演示如何产生并消耗数据，这里我们了解一下 gstbuffer。

source pads 产生 buffers， sink pads 消耗 buffers。buffers 会在 element 之间传递。A buffer 只是代表一块数据，我们必须有这样一种思想： 不要去假定 buffer 的大小都是相同的，不要去假定 buffer 消耗的时间都是相同的，也不要假定如果 buffer 传入 element， 这个 buffer 必然会被输出。因为 Elements 可以随意去处理 buffer。gstbuffer也可能包含多个实际内存缓冲区。实际的内存缓冲区用GstMemory对象抽象出来，而GstBuffer可以包含多个GstMemory对象。

每一个 buffer 都包含 time-stamps(时间戳) 和 duration， 描述着这些 buffer 在何时应该被解码，显示。time-stamps是一个非常复杂的主题，但是这里把他简化了。

这里先描述一个简单的流程，例如：filesrc(a GStreamer element that reads files) 生产 buffer，但是这些 buffer 并不带有任何的 caps 和 time-stamping 信息。这个 buffer 经过 demuxing 分流后 (see Basic tutorial 3: Dynamic pipelines),它就可能就带有了一些特定的属性，比如说 “video/x-h264”。 它 在经过 decoding 解码后，每一个 buffer 将会包含一个 "video/x-raw-yuv" 和 time stamps 。

# 三、This tutorial
本教程扩展了 Basic tutorial 7: Multithreading and Pad Availability 。

1. auidotestsrc 将被 appsrc 替代，由 appsrc 产生 audio data。

2. tee 有了一个新的分支，data 可以继续输出到 audio sink 并且 wave display 被复制到 appsink 中。appsink 可以上报给APP该信息已经收到，我们可以新起一个复杂的线程去处理它。
