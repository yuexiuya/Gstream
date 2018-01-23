# 一、Goal

对 gstreamer 的调试，我们可能会期待 bus 上 Error 的抛出，但是大多数时候，事情总没有那么顺利。但其实 gstream 有大量的调试信息，这里将演示如何进行调试。

- 如何得到更多的 GStreamer 日志
- 如何打印自己的 debug 信息 到 GStreamer 的日志中
- 如何得到 pipeline 的图表

# 二、Printing debug information

## 2.1 日志等级说明

GStreamer和它的插件都包含了调试跟踪，时间戳、过程、类别、源代码文件、函数和元素信息都会被打印到控制台。

<font color="red">调试信息由 GST_DEBUG 控制</font>

但是由于 GStreamer 的日志是冗长的，如果全部打印到 console 或者 重定向到文件会使得程序无法正常运行，因此请根据你的需求选择合适的调试等级。

GST_DEBUG 等级一共分为 8 个等级

- 0=none <font color="0xFFFF">没有任何输出</font>
- 1=ERROR <font color="0xFFFF">打印出所有致命的错误。如果APP处理了这些错误，它可以继续运行</font>
- 2=WARNING <font color="0xFFFF">这些问题是非致命的。用于警告User存在一些异常</font>
- 3=FIXME <font color="0xFFFF">打印出 “fixme” 信息。一般会打印出发生错误的代码路径</font>
- 4=INFO <font color="0xFFFF">打印出所有的 information，这里指那些只打印一次，或者重要的信息</font>
- 5=DEBUG <font color="0xFFFF">打印出所有的 debug, 这里指那些有限次的信息</font>
- 6=LOG <font color="0xFFFF">打印出所有的 log, 这里指在生命周期里重复打印的</font>
- 7=TRACE <font color="0xFFFF">打印出所有的 trace，这里指常常打印的信息</font>
- 8=MEMDUMP <font color="0xFFFF">打印出所有的 memory dump信息，这是最繁杂的日志等级，可能会包含内存块的回溯</font>

另外我们可以为某个特定的元素定制他的 GST_DEBUG 等级

- audiotestsrc 调试等级 = 6， 其他都是 2
```
GST_DEBUG=2,audiotestsrc:6
```

- 可以使用通配符

```
GST_DEBUG=2,audio*:6
```

- <font color="red"> 使用 GST_DEBUG 一定要注意把他放到前面</font>

```
GST_DEBUG=5 gst-launch-1.0 playbin uri=file:///home/joshua/music/mayday.mp4
```

- 有些时候，我们的系统不支持颜色。此时要使用 GST_DEBUG_NO_COLOR 不输出颜色

```
GST_DEBUG_NO_COLOR=1 GST_DEBUG_FILE=/home/joshua/music/gst.log GST_DEBUG=5 gst-launch-1.0 playbin uri=file:///home/joshua/music/mayday.mp4
```

## 2.2 日志解析

```
0:00:00.868050000  1592   09F62420 WARN                 filesrc gstfilesrc.c:1044:gst_file_src_start:<filesrc0> error: No such file "non-existing-file.webm"
```


- 0:00:00.868050000 <font color="0xFFFF">时间戳信息</font>
- 1592 <font color="0xFFFF">打印该信息的 进程 id</font>
- 09F62420 <font color="0xFFFF">打印该信息的 线程 id</font>
- WARN <font color="0xFFFF">日志等级</font>
- filesrc <font color="0xFFFF">打印出该日志的 element </font>
- gstfilesrc.c:1044 <font color="0xFFFF">打印该日志的文件和行数 </font>
- gst_file_src_start <font color="0xFFFF">打印该日志的函数 </font>
- filesrc0 <font color="0xFFFF">具体gobject对象</font>


# 三、把你自己的日志添加到 Gstream 日志系统里面去

- 使用 GST_ERROR(), GST_WARNING(), GST_INFO(), GST_LOG() 和 GST_DEBUG() 宏

- 在代码最上面加上

```
GST_DEBUG_CATEGORY_STATIC (my_category);
#define GST_CAT_DEFAULT my_category
```

- 在调用gst_init之后，call this function
```
GST_DEBUG_CATEGORY_INIT (my_category, "my category", 0, "This is my very own");
```

# 四、得到 pipeline 的图表

## 4.1 简介

试着想要这样一种情况，当你的 pipeline 变的越来越庞大 并且 你已经不知道 什么元素 和 什么元素 相连接时该怎么办？ 这时候就需要 graph。这些是 .dot 文件，可以用 GraphViz 查看。

对于分析 playbin or uridecodebin 这样封装度比较高的元素，使用 graph 查看会非常方便。我们可以通过 .dot 去了解这些封装度非常高的元素内部的功能。

## 4.2 如何获得 .dot 文件

只需要设置 GST_DEBUG_DUMP_DOT_DIR 变量重定向文件的输出路劲。 gst-launch-1.0 会在每一个 state 改变的时候创建 .dot 文件，以便于我们分析每一个 caps 的协商过程。取消设置变量以禁用此功能。在应用程序中，可以使用 GST_DEBUG_BIN_TO_DOT_FILE() 和 GST_DEBUG_BIN_TO_DOT_FILE_WITH_TS() 宏来生成 .dot文件。

- 设置环境变量后，即可在目标目录下面获得.dot文件
```
export GST_DEBUG_DUMP_DOT_DIR=/home/joshua/music/   
```

- 可以使用这个命令将其转换成图片

```
dot -Tpng 0.00.00.414815397-gst-launch.PAUSED_PLAYING.dot > pipeline.png
```
