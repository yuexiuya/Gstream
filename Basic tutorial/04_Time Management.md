# 一、目标
  本章 教我们如何使用 GStreamer 时间相关的 功能。
- 如何查询管道中的 stream position 或 duration
- 如何 seek 到一个不同的时间点

# 二、介绍

  GstQuery 类型，从中可以查询到一些关于 element 或者 pad 的信息。在下面这个例子中，当 seek 操作发生时，我们查询这个 pipeline。(一些源，如 live stream， 并不允许 seeking)

  在上一章中，一旦我们把 pipeline 组件并启动，下面所需要做的就是等待 bus 上的 ERROR 或者 EOS。在这一章，我们将定期的调用这个方法，以查询 pipeline 里面的 stream position，并打印在屏幕上。

# 三、Seeking example

```
#include <gst/gst.h>

/* Structure to contain all our information, so we can pass it around */
typedef struct _CustomData {
  GstElement *playbin;  /* Our one and only element */
  gboolean playing;      /* Are we in the PLAYING state? */
  gboolean terminate;    /* Should we terminate execution? */
  gboolean seek_enabled; /* Is seeking enabled for this media? */
  gboolean seek_done;    /* Have we performed the seek already? */
  gint64 duration;       /* How long does this media last, in nanoseconds */
} CustomData;

/* Forward definition of the message processing function */
static void handle_message (CustomData *data, GstMessage *msg);

int main(int argc, char *argv[]) {
  CustomData data;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;

  data.playing = FALSE;
  data.terminate = FALSE;
  data.seek_enabled = FALSE;
  data.seek_done = FALSE;
  data.duration = GST_CLOCK_TIME_NONE;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  data.playbin = gst_element_factory_make ("playbin", "playbin");

  if (!data.playbin) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Set the URI to play */
  g_object_set (data.playbin, "uri", "https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);

  /* Start playing */
  ret = gst_element_set_state (data.playbin, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (data.playbin);
    return -1;
  }

  /* Listen to the bus */
  bus = gst_element_get_bus (data.playbin);
  do {
    msg = gst_bus_timed_pop_filtered (bus, 100 * GST_MSECOND,
        GstMessageType( GST_MESSAGE_STATE_CHANGED | GST_MESSAGE_ERROR | GST_MESSAGE_EOS | GST_MESSAGE_DURATION) );

    /* Parse message */
    if (msg != NULL) {
      handle_message (&data, msg);
    } else {
      /* We got no message, this means the timeout expired */
      if (data.playing) {
        gint64 current = -1;

        /* Query the current position of the stream */
        if (!gst_element_query_position (data.playbin, GST_FORMAT_TIME, &current)) {
          g_printerr ("Could not query current position.\n");
        }

        /* If we didn't know it yet, query the stream duration */
        if (!GST_CLOCK_TIME_IS_VALID (data.duration)) {
          if (!gst_element_query_duration (data.playbin, GST_FORMAT_TIME, &data.duration)) {
            g_printerr ("Could not query current duration.\n");
          }
        }

        /* Print current position and total duration */
        g_print ("Position %" GST_TIME_FORMAT " / %" GST_TIME_FORMAT "\r",
            GST_TIME_ARGS (current), GST_TIME_ARGS (data.duration));

        /* If seeking is enabled, we have not done it yet, and the time is right, seek */
        if (data.seek_enabled && !data.seek_done && current > 10 * GST_SECOND) {
          g_print ("\nReached 10s, performing seek...\n");
          gst_element_seek_simple (data.playbin, GST_FORMAT_TIME,
              GstSeekFlags( GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_KEY_UNIT ), 30 * GST_SECOND);
          data.seek_done = TRUE;
        }
      }
    }
  } while (!data.terminate);

  /* Free resources */
  gst_object_unref (bus);
  gst_element_set_state (data.playbin, GST_STATE_NULL);
  gst_object_unref (data.playbin);
  return 0;
}

static void handle_message (CustomData *data, GstMessage *msg) {
  GError *err;
  gchar *debug_info;

  switch (GST_MESSAGE_TYPE (msg)) {
    case GST_MESSAGE_ERROR:
      gst_message_parse_error (msg, &err, &debug_info);
      g_printerr ("Error received from element %s: %s\n", GST_OBJECT_NAME (msg->src), err->message);
      g_printerr ("Debugging information: %s\n", debug_info ? debug_info : "none");
      g_clear_error (&err);
      g_free (debug_info);
      data->terminate = TRUE;
      break;
    case GST_MESSAGE_EOS:
      g_print ("End-Of-Stream reached.\n");
      data->terminate = TRUE;
      break;
    case GST_MESSAGE_DURATION:
      /* The duration has changed, mark the current one as invalid */
      data->duration = GST_CLOCK_TIME_NONE;
      break;
    case GST_MESSAGE_STATE_CHANGED: {
      GstState old_state, new_state, pending_state;
      gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
      if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data->playbin)) {
        g_print ("Pipeline state changed from %s to %s:\n",
            gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));

        /* Remember whether we are in the PLAYING state or not */
        data->playing = (new_state == GST_STATE_PLAYING);

        if (data->playing) {
          /* We just moved to PLAYING. Check if seeking is possible */
          GstQuery *query;
          gint64 start, end;
          query = gst_query_new_seeking (GST_FORMAT_TIME);
          if (gst_element_query (data->playbin, query)) {
            gst_query_parse_seeking (query, NULL, &data->seek_enabled, &start, &end);
            if (data->seek_enabled) {
              g_print ("Seeking is ENABLED from %" GST_TIME_FORMAT " to %" GST_TIME_FORMAT "\n",
                  GST_TIME_ARGS (start), GST_TIME_ARGS (end));
            } else {
              g_print ("Seeking is DISABLED for this stream.\n");
            }
          }
          else {
            g_printerr ("Seeking query failed.");
          }
          gst_query_unref (query);
        }
      }
    } break;
    default:
      /* We should not reach here */
      g_printerr ("Unexpected message received.\n");
      break;
  }
  gst_message_unref (msg);
}

```

## 3.1 用户界面刷新

本章 讲述的关于 时间的查询 和 seeking 动作 最后都是服务于 界面的刷新

```
/* We got no message, this means the timeout expired */
if (data.playing) {
```
像上面这样做，可以避免无效的查询。如果管道处于播放状态，则是刷新屏幕的时候了。如果我们不在状态，我们不想做任何事情，因为大多数查询都会失败。

我们这里大约每秒10次，对我们的UI来说是一个足够好的刷新率。我们将在屏幕上打印当前的媒体位置，我们可以学习如何查询管道。这涉及到将在下一小节中显示的几个步骤，但是，由于位置和持续时间都是常见的查询，GstElement提供了更简单、现成的替代方案。


## 3.2 gst_element_query_position()

查询当前播放位置

```
/* Query the current position of the stream */
if (!gst_element_query_position (data.pipeline, GST_FORMAT_TIME, &current)) {
  g_printerr ("Could not query current position.\n");
}
```

## 3.2 gst_element_query_duration()

查询总时长

```
/* If we didn't know it yet, query the stream duration */
if (!GST_CLOCK_TIME_IS_VALID (data.duration)) {
  if (!gst_element_query_duration (data.pipeline, GST_FORMAT_TIME, &data.duration)) {
     g_printerr ("Could not query current duration.\n");
  }
}
```

## 3.3 如何打印 gst 查询的时间

占位符使用  %" GST_TIME_FORMAT "
实际参数使用 GST_TIME_ARGS (current)
```
g_print ("Position %" GST_TIME_FORMAT " / %" GST_TIME_FORMAT "\r",
    GST_TIME_ARGS (current), GST_TIME_ARGS (data.duration));
```

## 3.4 关于能否进行 seek 以及 如何 seek

- gst_query_parse_seeking
- gst_element_seek_simple

```
/* If seeking is enabled, we have not done it yet, and the time is right, seek */
if (data.seek_enabled && !data.seek_done && current > 10 * GST_SECOND) {
  g_print ("\nReached 10s, performing seek...\n");
  gst_element_seek_simple (data.pipeline, GST_FORMAT_TIME,
      GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_KEY_UNIT, 30 * GST_SECOND);
  data.seek_done = TRUE;
}
```

- 梳理一下 GstFormat

GST_FORMAT_TIME: 指示我们在时间单元中指定目标。其他的seek格式使用不同的单位。这里是不是就可猜测一下其他 Flags 的意义，其实就是单位时间内步进的单位。
```
/**
 * GstFormat:
 * @GST_FORMAT_UNDEFINED: undefined format
 * @GST_FORMAT_DEFAULT: the default format of the pad/element. This can be
 *    samples for raw audio, frames/fields for raw video (some, but not all,
 *    elements support this; use @GST_FORMAT_TIME if you don't have a good
 *    reason to query for samples/frames)
 * @GST_FORMAT_BYTES: bytes
 * @GST_FORMAT_TIME: time in nanoseconds
 * @GST_FORMAT_BUFFERS: buffers (few, if any, elements implement this as of
 *     May 2009)
 * @GST_FORMAT_PERCENT: percentage of stream (few, if any, elements implement
 *     this as of May 2009)
 *
 * Standard predefined formats
 */
/* NOTE: don't forget to update the table in gstformat.c when changing
 * this enum */
typedef enum {
  GST_FORMAT_UNDEFINED  =  0, /* must be first in list */
  GST_FORMAT_DEFAULT    =  1,
  GST_FORMAT_BYTES      =  2,
  GST_FORMAT_TIME       =  3,
  GST_FORMAT_BUFFERS    =  4,
  GST_FORMAT_PERCENT    =  5
} GstFormat;
```

- 梳理一下 GstSeekFlags

GST_SEEK_FLAG_FLUSH ：在进行查找之前，这将丢弃管道中的所有数据。

GST_SEEK_FLAG_KEY_UNIT： 寻求关键帧的位置

GST_SEEK_FLAG_ACCURATE：寻求精确的位置(非常耗时)

```
/**
 * GstSeekFlags:
 * @GST_SEEK_FLAG_NONE: no flag
 * @GST_SEEK_FLAG_FLUSH: flush pipeline
 * @GST_SEEK_FLAG_ACCURATE: accurate position is requested, this might
 *                     be considerably slower for some formats.
 * @GST_SEEK_FLAG_KEY_UNIT: seek to the nearest keyframe. This might be
 *                     faster but less accurate.
 * @GST_SEEK_FLAG_SEGMENT: perform a segment seek.
 * @GST_SEEK_FLAG_SKIP: when doing fast foward or fast reverse playback, allow
 *                     elements to skip frames instead of generating all
 *                     frames.
 * @GST_SEEK_FLAG_SNAP_BEFORE: go to a location before the requested position,
 *                     if KEY_UNIT this means the keyframe at or before the
 *                     requested position the one at or before the seek target.
 * @GST_SEEK_FLAG_SNAP_AFTER: go to a location after the requested position,
 *                     if KEY_UNIT this means the keyframe at of after the
 *                     requested position.
 * @GST_SEEK_FLAG_SNAP_NEAREST: go to a position near the requested position,
 *                     if KEY_UNIT this means the keyframe closest to the
 *                     requested position, if both keyframes are at an equal
 *                     distance, behaves like SNAP_BEFORE.
 *
 * Flags to be used with gst_element_seek() or gst_event_new_seek(). All flags
 * can be used together.
 *
 * A non flushing seek might take some time to perform as the currently
 * playing data in the pipeline will not be cleared.
 *
 * An accurate seek might be slower for formats that don't have any indexes
 * or timestamp markers in the stream. Specifying this flag might require a
 * complete scan of the file in those cases.
 *
 * When performing a segment seek: after the playback of the segment completes,
 * no EOS will be emmited by the element that performed the seek, but a
 * #GST_MESSAGE_SEGMENT_DONE message will be posted on the bus by the element.
 * When this message is posted, it is possible to send a new seek event to
 * continue playback. With this seek method it is possible to perform seamless
 * looping or simple linear editing.
 *
 * When doing fast forward (rate > 1.0) or fast reverse (rate < -1.0) trickmode
 * playback, the @GST_SEEK_FLAG_SKIP flag can be used to instruct decoders
 * and demuxers to adjust the playback rate by skipping frames. This can improve
 * performance and decrease CPU usage because not all frames need to be decoded.
 *
 * The @GST_SEEK_FLAG_SNAP_BEFORE flag can be used to snap to the previous
 * relevant location, and the @GST_SEEK_FLAG_SNAP_AFTER flag can be used to
 * select the next relevant location. If KEY_UNIT is specified, the relevant
 * location is a keyframe. If both flags are specified, the nearest of these
 * locations will be selected. If none are specified, the implementation is
 * free to select whichever it wants.
 * The before and after here are in running time, so when playing backwards,
 * the next location refers to the one that will played in next, and not the
 * one that is located after in the actual source stream.
 *
 * Also see part-seeking.txt in the GStreamer design documentation for more
 * details on the meaning of these flags and the behaviour expected of
 * elements that handle them.
 */
typedef enum {
  GST_SEEK_FLAG_NONE            = 0,
  GST_SEEK_FLAG_FLUSH           = (1 << 0),
  GST_SEEK_FLAG_ACCURATE        = (1 << 1),
  GST_SEEK_FLAG_KEY_UNIT        = (1 << 2),
  GST_SEEK_FLAG_SEGMENT         = (1 << 3),
  GST_SEEK_FLAG_SKIP            = (1 << 4),
  GST_SEEK_FLAG_SNAP_BEFORE     = (1 << 5),
  GST_SEEK_FLAG_SNAP_AFTER      = (1 << 6),
  GST_SEEK_FLAG_SNAP_NEAREST    = GST_SEEK_FLAG_SNAP_BEFORE | GST_SEEK_FLAG_SNAP_AFTER,
  /* Careful to restart next flag with 1<<7 here */
} GstSeekFlags;
```

## 3.5 如何查询 bus 上的消息

```
case GST_MESSAGE_DURATION:
  /* The duration has changed, mark the current one as invalid */
  data->duration = GST_CLOCK_TIME_NONE;
  break;
```
当流的持续时间发生变化时，该消息会被发布到总线上。这里我们只是将当前的持续时间标记为无效，因此稍后会重新查询

- 监听状态的变化

```
case GST_MESSAGE_STATE_CHANGED: {
  GstState old_state, new_state, pending_state;
  gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
  if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data->pipeline)) {
    g_print ("Pipeline state changed from %s to %s:\n",
        gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));

    /* Remember whether we are in the PLAYING state or not */
    data->playing = (new_state == GST_STATE_PLAYING);
```
