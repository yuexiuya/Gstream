# 一、目标
本章阐述一个新的 GStream 的概念，允许在运行中构建一个 pipeline，而不是在一开始定义一个完整的管道。

本章重点：

1. 如何在连接元素时获得更好的控制；
2. 如何订阅你感兴趣的事件；
3. element的各种状态；

# 二、介绍

正如你所将要看到的那样，本章的 pipeline 在播放之前 都没有被完整的创建，也就是本章的核心——Dynamic pipeline。这是可行的，如果这样做，pipeline 将会在 data 流到末尾时产生一个 Error 并且 stop。

在这个例子中，我们将打开一个 multiplexed (or muxed) 文件，混流(muxed) 意味这 audio 和 video 是存储在同一个文件里的。负责打开这种文件的 element叫做 demuxers，这种容器的格式( 存储audio & video 的容器)有 Matroska(MKV), Quick Time(QT.MOV), Ogg, or Advanced Systems Format(ASF, WMV,WMA)

如果一个 container 中嵌入 multiple streams (如 one video and two audio tracks, for exmaple), demuxers 将会分离它到不同的输出通道去。在这种方式中，管道中将创建不同的分支去处理不同类型的 data。

Element 中的输入输出口叫做 GstPad。如 source elements 只有 source pads，只负责输出；sink elements 只有 sink pads，只负责接受； 而 filter elements 两者都有。

![image](https://github.com/yuexiuya/Gstream/blob/master/image/Basic_tutorial3.png?raw=true)

## 2.1 GStreamer elements with their pads.

一个 demuxer 包含 一个 sink pad，用来接收 muxed data；多个 source pads，用来输出 contaniner 中存储的流。

![image](https://github.com/yuexiuya/Gstream/blob/master/image/Basic_tutorial3_1.png?raw=true)

## 2.2 一个完整的 pipeline 结构

![image](https://github.com/yuexiuya/Gstream/blob/master/image/Basic_tutorial3_2.png?raw=true)

## 2.3 动态创建的时机

处理 demuxers 的复杂性在于 它在接收到 data 之前无法生成任何信息。如果 demuxer 没有被任何 elements 连接，pipeline 就必然在这里终止。

解决办法：构建完整的 pipeline (link source -> dumuxer)。当 demuxer 接收到足够的信息时，它开始创建 source pads。

# 三、Example

```
#include <gst/gst.h>

/* Structure to contain all our information, so we can pass it to callbacks */
typedef struct _CustomData {
  GstElement *pipeline;
  GstElement *source;
  GstElement *convert;
  GstElement *sink;
} CustomData;

/* Handler for the pad-added signal */
static void pad_added_handler (GstElement *src, GstPad *pad, CustomData *data);

int main(int argc, char *argv[]) {
  CustomData data;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;
  gboolean terminate = FALSE;
  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  data.source = gst_element_factory_make ("uridecodebin", "source");
  data.convert = gst_element_factory_make ("audioconvert", "convert");
  data.sink = gst_element_factory_make ("autoaudiosink", "sink");

  /* Create the empty pipeline */
  data.pipeline = gst_pipeline_new ("test-pipeline");

  if (!data.pipeline || !data.source || !data.convert || !data.sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Build the pipeline. Note that we are NOT linking the source at this
   * point. We will do it later. */
  gst_bin_add_many (GST_BIN (data.pipeline), data.source, data.convert , data.sink, NULL);
  if (!gst_element_link (data.convert, data.sink)) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (data.pipeline);
    return -1;
  }

  /* Set the URI to play */
  g_object_set (data.source, "uri", "https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);

  /* Connect to the pad-added signal */
  g_signal_connect (data.source, "pad-added", G_CALLBACK (pad_added_handler), &data);

  /* Start playing */
  ret = gst_element_set_state (data.pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (data.pipeline);
    return -1;
  }
  /* Listen to the bus */
  bus = gst_element_get_bus (data.pipeline);
  do {
    msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
        GstMessageType(GST_MESSAGE_STATE_CHANGED | GST_MESSAGE_ERROR | GST_MESSAGE_EOS) );
    /* Parse message */
    if (msg != NULL) {
      GError *err;
      gchar *debug_info;

      switch (GST_MESSAGE_TYPE (msg)) {
        case GST_MESSAGE_ERROR:
          gst_message_parse_error (msg, &err, &debug_info);
          g_printerr ("Error received from element %s: %s\n", GST_OBJECT_NAME (msg->src), err->message);
          g_printerr ("Debugging information: %s\n", debug_info ? debug_info : "none");
          g_clear_error (&err);
          g_free (debug_info);
          terminate = TRUE;
          break;
        case GST_MESSAGE_EOS:
          g_print ("End-Of-Stream reached.\n");
          terminate = TRUE;
          break;
        case GST_MESSAGE_STATE_CHANGED:
          /* We are only interested in state-changed messages from the pipeline */
          if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data.pipeline)) {
            GstState old_state, new_state, pending_state;
            gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
            g_print ("Pipeline state changed from %s to %s:\n",
                gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));
          }
          break;
        default:
          /* We should not reach here */
          g_printerr ("Unexpected message received.\n");
          break;
      }
      gst_message_unref (msg);
    }
  } while (!terminate);
  while(1);
  /* Free resources */
  gst_object_unref (bus);
  gst_element_set_state (data.pipeline, GST_STATE_NULL);
  gst_object_unref (data.pipeline);
  return 0;
}

/* This function will be called by the pad-added signal */
static void pad_added_handler (GstElement *src, GstPad *new_pad, CustomData *data) {
  GstPad *sink_pad = gst_element_get_static_pad (data->convert, "sink");
  GstPadLinkReturn ret;
  GstCaps *new_pad_caps = NULL;
  GstStructure *new_pad_struct = NULL;
  const gchar *new_pad_type = NULL;

  g_print ("Received new pad '%s' from '%s':\n", GST_PAD_NAME (new_pad), GST_ELEMENT_NAME (src));

  /* If our converter is already linked, we have nothing to do here */
  if (gst_pad_is_linked (sink_pad)) {
    g_print ("  We are already linked. Ignoring.\n");
    goto exit;
  }

  /* Check the new pad's type */
  new_pad_caps = gst_pad_query_caps (new_pad, NULL);
  new_pad_struct = gst_caps_get_structure (new_pad_caps, 0);
  new_pad_type = gst_structure_get_name (new_pad_struct);
  if (!g_str_has_prefix (new_pad_type, "audio/x-raw")) {
    g_print ("  It has type '%s' which is not raw audio. Ignoring.\n", new_pad_type);
    goto exit;
  }

  /* Attempt the link */
  ret = gst_pad_link (new_pad, sink_pad);
  if (GST_PAD_LINK_FAILED (ret)) {
    g_print ("  Type is '%s' but link failed.\n", new_pad_type);
  } else {
    g_print ("  Link succeeded (type '%s').\n", new_pad_type);
  }

exit:
  /* Unreference the new pad's caps, if we got them */
  if (new_pad_caps != NULL)
    gst_caps_unref (new_pad_caps);

  /* Unreference the sink pad */
  gst_object_unref (sink_pad);
}
```


1. 在最开始调试这个问题的时候，我出现了 segmentfalut 的错误，查了一阵子，总结一下，基本原因无非是：非法的内存访问。

总结一下： 类型转换的时候还是使用 (目标类型)(data)的方式比较保险；

```
//一个类型转换的错我，我想当然的使用了GST_MESSGE_TYPE这个macro，根据这个macro最后获取的是一个指针
//Error
msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
    GST_MESSGE_TYPE(GST_MESSAGE_STATE_CHANGED | GST_MESSAGE_ERROR | GST_MESSAGE_EOS) );
//Right
msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
    GstMessageType(GST_MESSAGE_STATE_CHANGED | GST_MESSAGE_ERROR | GST_MESSAGE_EOS) );
```

2. g_signal_connect 的使用

其实 g_signal_connect 的定义中， G_CALLBACK 是一个 void func(void * ) 的参数。但是我们这里却定义了 static void pad_added_handler (GstElement * src, GstPad * new_pad, CustomData * data) {} 类型，那么这种 G_CALLBACK 的传参个数如何确定？？？？

```
static void pad_added_handler (GstElement *src, GstPad *new_pad, CustomData *data) {}

g_signal_connect (data.source, "pad-added", G_CALLBACK (pad_added_handler), &data);
```

这里监听的是 uridecodebin 元素 的 signal pad-added , 我们使用 gst-inspect-1.0 uridecodebin
