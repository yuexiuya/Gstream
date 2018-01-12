在这一部分中，我们将介绍GStreamer更高级的特性。通过前面学到的基础知识，您应该能够创建一个简单的应用程序。然而，GStreamer提供的糖果远远超过了回放音频文件的基础。在本章中，您将了解GStreamer的底层特性和内部结构。

这部分的部分内容主要是解释GStreamer内部是如何工作的;它们对于实际开发并不是必须需要了解的。这包括一些章节，包括日程安排，自动跟踪和同步。但是，其他章节讨论了更高级的管道-应用程序交互的方法，并且可能对某些应用程序非常有用。这包括关于元数据、查询和事件、接口、动态参数和管道数据操作的章节。

# 一、Position tracking and seeking

到目前为止，我们已经研究了如何创建管道来进行媒体处理以及如何使其运行。大多数应用程序开发人员都有兴趣向用户提供关于媒体进度的反馈。例如，媒体播放器将希望显示一个显示歌曲进展的滑块，通常也会显示一条指示流长度的标签。代码转换应用程序将希望显示一个进度条，说明完成任务的百分比。GStreamer有内置的支持，可以使用称为查询的概念来完成所有这些工作。既然寻找是非常相似的，它也将在这里讨论。Seeking 是用事件的概念来完成的。

# 二、Querying: getting the position or length of a stream

这包括获取流的长度(如果可用)或获得当前位置。这些流属性可以以各种格式检索，例如时间、音频示例、视频帧或字节。这里最常用的函数是gst_element_query()，不过也提供了一些方便的包装器(比如gst_element_query_position()和gst_element_query_duration())。通常可以直接查询管道，它会为您找到内部细节，比如查询哪个元素。

- position or length of a stream， 一般都在demux元素中查询；
- GST_TIME_ARGS(time), 一个很好用的宏，可以转换成常用的时间；

```

#include <gst/gst.h>

static gboolean
cb_print_position (GstElement *pipeline)
{
  gint64 pos, len;

  if (gst_element_query_position (pipeline, GST_FORMAT_TIME, &pos)
    && gst_element_query_duration (pipeline, GST_FORMAT_TIME, &len)) {
    g_print ("Time: %" GST_TIME_FORMAT " / %" GST_TIME_FORMAT "\r",
         GST_TIME_ARGS (pos), GST_TIME_ARGS (len));
  }

  /* call me again */
  return TRUE;
}

gint
main (gint   argc,
      gchar *argv[])
{
  GstElement *pipeline;

[..]

  /* run pipeline */
  g_timeout_add (200, (GSourceFunc) cb_print_position, pipeline);
  g_main_loop_run (loop);

[..]

}
```


# 三、Events: seeking (and more)
事件的工作方式和查询类似。Applictaions和元素交互的事件是非常多的，但这里我们只关注seeking。

```
static void
seek_to_time (GstElement *pipeline,
          gint64      time_nanoseconds)
{
  if (!gst_element_seek (pipeline, 1.0, GST_FORMAT_TIME, GST_SEEK_FLAG_FLUSH,
                         GST_SEEK_TYPE_SET, time_nanoseconds,
                         GST_SEEK_TYPE_NONE, GST_CLOCK_TIME_NONE)) {
    g_print ("Seek failed!\n");
  }
}
```

- 当管道处于(PAUSE\PLAY)状态的时候，Seek 应该使用 GST_SEEK_FLAG_FLUSH 进行操作。此时管道的状态会变成 PREROLL, 直到 Seek 成功，状态会恢复成(PAUSE\PLAY)。当请求执行时，你可以使用 gst_element_get_state() 阻塞；或者 处理 bus 上的 ASYNC_DONE。

- 不使用 GST_SEEK_FLAG_FLUSH 的话，只能是在 state = play 的状态操作；

- gst_element_seek 会立即返回，但是并不会立刻执行；

- 可以在短的时间间隔内进行多次搜索，例如对滑块移动的直接响应。在寻找，内部，管道将被暂停(如果它正在播放)，这个位置将会在内部重新设置，demuxers和解码器将从新的位置开始解码，这将继续下去，直到所有的接收器再次拥有数据。如果它本来是在播放，它也将被设置为再次播放。由于新位置在视频输出中立即可用，所以您将看到新的帧，即使您的管道不是处于播放状态。
