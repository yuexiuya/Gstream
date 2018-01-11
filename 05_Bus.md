- 总线是一个简单的系统，它负责从流线程中转发消息到它自己的线程上下文中的应用程序。总线最大的优点是应用程序不需要是线程敏感的(这里的总线机制非常像Qt里的事件循环)
- 总线是不需要创建的。在每个管道上都包含一个总线，应用程序不需要创建任何总线。应用程序只要做的就是在总线上设置一个消息处理程序——对象的信号处理程序。当主循环运行时，将周期性的检查总线以获取新信息，并触发回调。

## 一、如何使用总线

1. 运行一个GLib /Gtk +主循环(或者定期迭代默认的GLib主上下文)，并在总线上附加某种监视。这样，GLib主循环将检查总线以获取新消息，并在有消息时通知您。你可以使用gst_bus_add_watch() 和 gst_bus_add_signal_watch()要使用总线，请使用gst_bus_add_watch()将消息处理程序附加到管道总线上。当管道向总线发送消息时，将调用此处理程序。在这个处理程序中，检查信号类型(参见下一节)并做相应的事情。处理程序的返回值应该是正确的，以便将处理程序附加到总线上，返回FALSE以删除它。
2. 您可以自己检查总线上的消息。这可以通过使用gst_bus_peek()或gst_bus_poll()完成。


*add a watch*
```
#include <gst/gst.h>

static GMainLoop *loop;

static gboolean
my_bus_callback (GstBus     *bus,
         GstMessage *message,
         gpointer    data)
{
  g_print ("Got %s message\n", GST_MESSAGE_TYPE_NAME (message));

  switch (GST_MESSAGE_TYPE (message)) {
    case GST_MESSAGE_ERROR: {
      GError *err;
      gchar *debug;

      gst_message_parse_error (message, &err, &debug);
      g_print ("Error: %s\n", err->message);
      g_error_free (err);
      g_free (debug);

      g_main_loop_quit (loop);
      break;
    }
    case GST_MESSAGE_EOS:
      /* end-of-stream */
      g_main_loop_quit (loop);
      break;
    default:
      /* unhandled message */
      break;
  }

  /* we want to be notified again the next time there is a message
   * on the bus, so returning TRUE (FALSE means we want to stop watching
   * for messages on the bus and our callback should not be called again)
   */
  return TRUE;
}

gint
main (gint   argc,
      gchar *argv[])
{
  GstElement *pipeline;
  GstBus *bus;
  guint bus_watch_id;

  /* init */
  gst_init (&argc, &argv);

  /* create pipeline, add handler */
  pipeline = gst_pipeline_new ("my_pipeline");

  /* adds a watch for new message on our pipeline's message bus to
   * the default GLib main context, which is the main context that our
   * GLib main loop is attached to below
   */
  bus = gst_pipeline_get_bus (GST_PIPELINE (pipeline));
  bus_watch_id = gst_bus_add_watch (bus, my_bus_callback, NULL);
  gst_object_unref (bus);

  /* [...] */

  /* create a mainloop that runs/iterates the default GLib main context
   * (context NULL), in other words: makes the context check if anything
   * it watches for has happened. When a message has been posted on the
   * bus, the default main context will automatically call our
   * my_bus_callback() function to notify us of that message.
   * The main loop will be run until someone calls g_main_loop_quit()
   */
  loop = g_main_loop_new (NULL, FALSE);
  g_main_loop_run (loop);

  /* clean up */
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  g_source_remove (bus_watch_id);
  g_main_loop_unref (loop);

  return 0;
}


```
从上面的例子可以看出，管道 和 总线的交互总是异步的，所以它不应该被用来做一些实时性非常高的操作。这些实时性高的操作应该就在管道的上下文中实现，写GStreamer插件就是一种非常好的形式(插件形式未知).

- 如果你熟悉GLib MainLoop，你可以使用这样一种方式而不是add a watch
- 如果你使用GLib MainLoop循环，异步消息信号默认是不可用的。但是你可以安装一个自定义的处理机制(gst_bus_async_signal_func),具体参照文档吧

```
GstBus *bus;

[..]

bus = gst_pipeline_get_bus (GST_PIPELINE (pipeline);
gst_bus_add_signal_watch (bus);
g_signal_connect (bus, "message::error", G_CALLBACK (cb_message_error), NULL);
g_signal_connect (bus, "message::eos", G_CALLBACK (cb_message_eos), NULL);

[..]
```
## 二、消息类型

- 根据上面的代码里，我们已经简单了解 GST_MESSAGE_ERROR 和 GST_MESSAGE_EOF 两种类型
- GStreamer 有一些预定义的消息类型，并且这些消息是可扩展的

如下，特别多的类型
```
typedef enum
{
  GST_MESSAGE_UNKNOWN           = 0,
  GST_MESSAGE_EOS               = (1 << 0),
  GST_MESSAGE_ERROR             = (1 << 1),
  GST_MESSAGE_WARNING           = (1 << 2),
  GST_MESSAGE_INFO              = (1 << 3),
  GST_MESSAGE_TAG               = (1 << 4),
  GST_MESSAGE_BUFFERING         = (1 << 5),
  GST_MESSAGE_STATE_CHANGED     = (1 << 6),
  GST_MESSAGE_STATE_DIRTY       = (1 << 7),
  GST_MESSAGE_STEP_DONE         = (1 << 8),
  GST_MESSAGE_CLOCK_PROVIDE     = (1 << 9),
  GST_MESSAGE_CLOCK_LOST        = (1 << 10),
  GST_MESSAGE_NEW_CLOCK         = (1 << 11),
  GST_MESSAGE_STRUCTURE_CHANGE  = (1 << 12),
  GST_MESSAGE_STREAM_STATUS     = (1 << 13),
  GST_MESSAGE_APPLICATION       = (1 << 14),
  GST_MESSAGE_ELEMENT           = (1 << 15),
  GST_MESSAGE_SEGMENT_START     = (1 << 16),
  GST_MESSAGE_SEGMENT_DONE      = (1 << 17),
  GST_MESSAGE_DURATION_CHANGED  = (1 << 18),
  GST_MESSAGE_LATENCY           = (1 << 19),
  GST_MESSAGE_ASYNC_START       = (1 << 20),
  GST_MESSAGE_ASYNC_DONE        = (1 << 21),
  GST_MESSAGE_REQUEST_STATE     = (1 << 22),
  GST_MESSAGE_STEP_START        = (1 << 23),
  GST_MESSAGE_QOS               = (1 << 24),
  GST_MESSAGE_PROGRESS          = (1 << 25),
  GST_MESSAGE_TOC               = (1 << 26),
  GST_MESSAGE_RESET_TIME        = (1 << 27),
  GST_MESSAGE_STREAM_START      = (1 << 28),
  GST_MESSAGE_NEED_CONTEXT      = (1 << 29),
  GST_MESSAGE_HAVE_CONTEXT      = (1 << 30),
  GST_MESSAGE_ANY               = ~0
} GstMessageType;
```

1. Error、Warning和Info：

所有的这些错误包含一个GError的字符串，可以使用gst_message_parse_error()， gst_parse_warning() 和 gst_parse_info()来调试。

2. End-of-stream:

在流结束时发出此消息。管道的状态不会改变，但进一步的媒体处理将停止。应用程序可以使用这个跳转到他们的播放列表中的下一首歌曲。在流结束后，也可以在小溪中寻找回来。回放将继续自动进行。此消息没有具体参数。

3. Tag

当流中发现元数据(如歌手，歌曲名称，采样率，比特率等等)，可以使用gst_message_tag()来解析(需要使用gst_tag_list_unref())释放

4. State-changes

成功的状态改变后发出的。gst_message_parse_state_changed（）可以用来解析这个过渡的旧的和新的状态 

5.Buffering

在网络流缓存期间发出的。一个可以手动提取进度（百分比）从消息从结构中提取“缓冲百分比”属性返回的gst_message_get_structure()。参见缓冲

还有很多，具体日后参照吧。