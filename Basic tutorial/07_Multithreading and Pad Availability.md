# 一、Goal

GStreamer可以自动处理多线程，但是，在某些情况下，您可能需要手动解耦线程。

本教程展示了如何做到这一点，此外，还完成了关于Pad可用性的说明。更准确地说，这份文件解释道

- 如何为管道的某些部分创建新的执行线程?
- Pad的可用性是什么?
- 如何复制流

# 二、介绍

## 2.1 Multithreading
GStreamer是一个多线程框架。它可以根据自己的需求创建或者销毁线程。

在此基础上，当构建管道时，应用程序可以显式地指定分支(管道的一部分)在不同的线程上运行(例如，让音频和视频解码器同时执行)

这是使用队列元素完成的，它的工作方式如下。sink pad只是对数据进行排队并返回控制。在另一个线程中，数据被排出队列并被推到下游。

## 2.2 The example pipeline

这里谈到了一个 queue 的 element。此时还不太理解，但是非常重要！！！

![image](https://github.com/yuexiuya/Gstream/blob/master/image/Basic_thread.png?raw=true)

源是一个合成的音频信号(一个连续的音调)，它使用一个tee元素(它通过它的源板发送通过它的接收器的所有东西)来分裂。然后，一个分支将信号发送到声卡，另一个则呈现波形的视频并将其发送到屏幕。

如图所示，队列创建一个新线程，因此这条管道运行在3个线程中。具有多个接收器的管道通常需要是多线程的，因为要保持同步，接收器通常会阻塞执行，直到所有其他的接收器都准备好了，如果只有一个线程被第一个接收器阻塞，它们就无法准备就绪。


## 2.3 Request Pads

- Always Pads ： 始终存在的 Pad
- Sometimes Pads ： 当数据开始流动时而创建的 Pad
- Request Pads ： 基于需求创建的 Pad

# 三、Simple multithreaded example

大致浏览了一下，主要说明了

- queue 这个 Element，我这里理解通过这个 queue 可以创建一个新的线程。
- request pad 的连接

```
#include <gst/gst.h>

int main(int argc, char *argv[]) {
  GstElement *pipeline, *audio_source, *tee, *audio_queue, *audio_convert, *audio_resample, *audio_sink;
  GstElement *video_queue, *visual, *video_convert, *video_sink;
  GstBus *bus;
  GstMessage *msg;
  GstPad *tee_audio_pad, *tee_video_pad;
  GstPad *queue_audio_pad, *queue_video_pad;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  audio_source = gst_element_factory_make ("audiotestsrc", "audio_source");
  tee = gst_element_factory_make ("tee", "tee");
  audio_queue = gst_element_factory_make ("queue", "audio_queue");
  audio_convert = gst_element_factory_make ("audioconvert", "audio_convert");
  audio_resample = gst_element_factory_make ("audioresample", "audio_resample");
  audio_sink = gst_element_factory_make ("autoaudiosink", "audio_sink");
  video_queue = gst_element_factory_make ("queue", "video_queue");
  visual = gst_element_factory_make ("wavescope", "visual");
  video_convert = gst_element_factory_make ("videoconvert", "csp");
  video_sink = gst_element_factory_make ("autovideosink", "video_sink");

  /* Create the empty pipeline */
  pipeline = gst_pipeline_new ("test-pipeline");

  if (!pipeline || !audio_source || !tee || !audio_queue || !audio_convert || !audio_resample || !audio_sink ||
      !video_queue || !visual || !video_convert || !video_sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Configure elements */
  //(1)创建声音和波形
  g_object_set (audio_source, "freq", 215.0f, NULL);
  g_object_set (visual, "shader", 0, "style", 1, NULL);

  //(2) link 所有的 Always 属性的 Pad
  /* Link all elements that can be automatically linked because they have "Always" pads */
  gst_bin_add_many (GST_BIN (pipeline), audio_source, tee, audio_queue, audio_convert, audio_resample, audio_sink,
      video_queue, visual, video_convert, video_sink, NULL);
  if (gst_element_link_many (audio_source, tee, NULL) != TRUE ||
      gst_element_link_many (audio_queue, audio_convert, audio_resample, audio_sink, NULL) != TRUE ||
      gst_element_link_many (video_queue, visual, video_convert, video_sink, NULL) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }

  /* Manually link the Tee, which has "Request" pads */
  tee_audio_pad = gst_element_get_request_pad (tee, "src_%u");
  g_print ("Obtained request pad %s for audio branch.\n", gst_pad_get_name (tee_audio_pad));
  queue_audio_pad = gst_element_get_static_pad (audio_queue, "sink");
  tee_video_pad = gst_element_get_request_pad (tee, "src_%u");
  g_print ("Obtained request pad %s for video branch.\n", gst_pad_get_name (tee_video_pad));
  queue_video_pad = gst_element_get_static_pad (video_queue, "sink");
  if (gst_pad_link (tee_audio_pad, queue_audio_pad) != GST_PAD_LINK_OK ||
      gst_pad_link (tee_video_pad, queue_video_pad) != GST_PAD_LINK_OK) {
    g_printerr ("Tee could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }
  gst_object_unref (queue_audio_pad);
  gst_object_unref (queue_video_pad);

  /* Start playing the pipeline */
  gst_element_set_state (pipeline, GST_STATE_PLAYING);

  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE, (GstMessageType)(GST_MESSAGE_ERROR | GST_MESSAGE_EOS));

  /* Release the request pads from the Tee, and unref them */
  gst_element_release_request_pad (tee, tee_audio_pad);
  gst_element_release_request_pad (tee, tee_video_pad);
  gst_object_unref (tee_audio_pad);
  gst_object_unref (tee_video_pad);

  /* Free resources */
  if (msg != NULL)
    gst_message_unref (msg);
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);

  gst_object_unref (pipeline);
  return 0;
}

```

## 如何创建 request pad 并且连接他们

```
//Request pad 需要手动的创建和释放
//创建
gst_element_get_request_pad
//释放
gst_element_release_request_pad
```
