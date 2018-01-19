# 一、Goal

没有什么比在屏幕上打印“Hello World”更好的了。

但是由于我们正在处理多媒体框架，我们将会播放一个视频。

不要被下面的代码所吓倒:只有4行是真正的工作。其余的是清理代码，在C中，这总是有点冗长。

无需进一步，为您的第一个GStreamer应用程序做好准备……

# 二、Hello world

本教程将打开一个窗口并显示一个电影，并附带音频。媒体是从因特网上获取的，因此，根据您的连接速度，窗口可能需要几秒钟的时间。此外，没有延迟管理(缓冲)，所以在缓慢的连接上，电影可能在几秒钟后停止。参见 Basic tutorial 12: Streaming solves this issue.


```
#include <gst/gst.h>
int main(int argc, char *argv[]) {
  GstElement *pipeline;
  GstBus *bus;
  GstMessage *msg;
  guint64 flags;
  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Build the pipeline */

  pipeline = gst_parse_launch ("playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_cropped_multilingual.webm", NULL);
  g_print(" ====> %x \n",flags);
  /* Start playing */
  gst_element_set_state (pipeline, GST_STATE_PLAYING);

  /* Wait until error or EOS */
  while(1);
  bus = gst_element_get_bus (pipeline);
  msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE, GST_MESSAGE_TYPE(GST_MESSAGE_ERROR | GST_MESSAGE_EOS));

  /* Free resources */
  if (msg != NULL)
    gst_message_unref (msg);
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  return 0;
}
```


# 三、Walkthrough

这段代码展现 gstreamer 最基本的方式，source elements 产生数据并传递出去，经过中间 filter 的过滤，最终流入 pad elements。

## 3.1 元素的创建
```
/* Create the elements */
source = gst_element_factory_make ("videotestsrc", "source");
sink = gst_element_factory_make ("autovideosink", "sink");
```
正如在这段代码中所看到的，可以使用gst_element_factory_make()创建新的元素。第一个参数是要创建的元素的类型。第二个参数是我们要给这个特定实例的名称。如果您没有保存一个指针(以及用于更有意义的调试输出)，那么命名元素是很有用的。但是，如果您为这个名称传递NULL, GStreamer将为您提供一个惟一的名称。

## 3.2 创建管道

```
/* Create the empty pipeline */
pipeline = gst_pipeline_new ("test-pipeline");
```
GStreamer中的所有元素通常都必须包含在管道中，然后才能被使用，因为它负责一些clock和消息传递函数。(因为 bus 在 pipeline中吧)

```
/* Build the pipeline */
gst_bin_add_many (GST_BIN (pipeline), source, sink, NULL);
if (gst_element_link (source, sink) != TRUE) {
  g_printerr ("Elements could not be linked.\n");
  gst_object_unref (pipeline);
  return -1;
}
```

pipeline 是一种特定类型的 bin，它是用来包含其他元素的 元素。因此，所有适用于 bin 的方法也适用于 pipeline。在我们的例子中，我们调用gst_bin_add_many()将元素添加到 pipeline。该函数接受要添加的元素列表，以NULL结尾。可以使用gst_bin_add()添加单个元素。

然而，这些元素之间并没有联系。为此，我们需要使用gst_element_link()。它的第一个参数是源，第二个参数是目标。请记住，只有驻留在同一个 bin 中的元素才能链接在一起，所以请记住在链接它们之前将它们添加到 pipeline 中!

## 3.3 Properties

```
/* Modify the source's properties */
g_object_set (source, "pattern", 0, NULL);
```

大多数 GStreamer 元素都具有可定制的属性: 可以修改的命名属性来更改元素的行为(可写属性)，或者查询元素的内部状态(可读属性)。

通过g_object_get()读取属性，通过g_object_set()设置属性。

GStreamer元素都是一种特殊的GObject，它是提供属性设施的实体。

## 3.4 Error checking

```
/* Start playing */
ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
if (ret == GST_STATE_CHANGE_FAILURE) {
  g_printerr ("Unable to set the pipeline to the playing state.\n");
  gst_object_unref (pipeline);
  return -1;
}
```

我们调用gst_element_set_state()，但是这次我们检查它的返回值是否错误。改变状态是一个微妙的过程，具体参考 Basic tutorial 3: Dynamic pipelines.

```
/* Wait until error or EOS */
bus = gst_element_get_bus (pipeline);
msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE, GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

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
      break;
    case GST_MESSAGE_EOS:
      g_print ("End-Of-Stream reached.\n");
      break;
    default:
      /* We should not reach here because we only asked for ERRORs and EOS */
      g_printerr ("Unexpected message received.\n");
      break;
  }
  gst_message_unref (msg);
}
```
