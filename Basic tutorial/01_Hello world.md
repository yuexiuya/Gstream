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
The elements are GStreamer's basic construction blocks. They process the data as it flows downstream from the source elements (data producers) to the sink elements (data consumers), passing through filter elements.
```

```
