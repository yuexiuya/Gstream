# 一、Goal
  Pad Capabilities 是 GStreamer 中 最基础的 Element， 尽管大多数时候他们并不可见。
- 什么是 Pad Capabilities
- 如何 检索 Pad
- 什么时候去检索他们
- 你为什么要去了解他们

# 二、Introduction

## 2.1 Pads

正如前面所说到的那样，Pads 可以允许 metadata 流入或流出 element。 Caps 限制了可以流经 Pads 的种类，只允许特定的类型通过。如“RGB video 320*320 像素，30帧每秒” 或者 “每个样本音频16位，5.1频道每秒44100个样本”， 或者更复杂的格式如 mp3、h264

Pad 可以支持多种功能(例如，video sink 可以支持不同类型RGB或YUV格式的视频)， 而 Capabilities 可以指定为范围(例如，音频接收器每秒可以支持1到48000个样本的采样率)。 但是，从Pad到Pad的实际信息必须只有一个指定的类型。

<font color=#00ffff> 通过一个称为协商的过程，两个链接的pad在一个公共类型上达成一致，从而使该平台的功能变得固定(它们只有一个类型，并且不包含范围)。下面的示例代码的演练应该清楚地说明这一点。为了将两个元素链接在一起，它们必须共享一个共同的功能子集(否则它们不可能相互理解)。这是能力的主要目标 </font>

作为一个应用程序开发人员，您通常会通过将元素连接在一起来构建管道(如果使用像playbin这样的所有元素，则可以在较小程度上构建管道)。在这种情况下，您需要知道您的元素的Pad Caps，或者至少知道GStreamer拒绝将两个元素在协商错误的情况下链接。

## 2.2 Pads templates

Pad 是由 Pad 模板创建的，它显示了Pad可能拥有的所有可能的功能。模板对于创建几个类似的 pad 非常有用，而且还允许拒绝协商错误的元素连接。

如果它们的Pad模板的功能没有一个通用子集(它们的交集是空的)，就没有必要进行进一步的协商。

Pad模板可以被看作是谈判过程中的第一步。随着流程的发展，实际的pad将被实例化，并且它们的功能将被细化，直到它们被修复(或者协商失败)

## 2.3 Capabilities examples

这里说明一下如何 阅读 一个 Pad 的 Capabilities
- 1
```
SINK template: 'sink'
  Availability: Always  //always 表示这个 Pad 是长期存在的
  Capabilities:
    //支持两种音频格式
    //关于这里的 rate 和 channels 暂不明确用途
    //比特率=采样率X采样精度（位数）(*通道数)
    audio/x-raw
               format: S16LE  //signed 16字节 小端存储方式
                 rate: [ 1, 2147483647 ]
             channels: [ 1, 2 ]
    audio/x-raw
               format: U8   //unsigned 8-bit
                 rate: [ 1, 2147483647 ]
             channels: [ 1, 2 ]
```

- 2

```
SRC template: 'src'
  Availability: Always  //always 表示这个 Pad 是长期存在的

  Capabilities:
    video/x-raw   //表示该源pad输出原始视频
                width: [ 1, 2147483647 ]
               height: [ 1, 2147483647 ]
            framerate: [ 0/1, 2147483647/1 ]
               format: { I420, NV12, NV21, YV12, YUY2, Y42B, Y444, YUV9, YVU9, Y41B, Y800, Y8, GREY, Y16 , UYVY, YVYU, IYU1, v308, AYUV, A420 }
```
它支持广泛的维度和帧，以及一组YUV格式(花括号表示一个列表)。所有这些格式表明了不同的包装和对图像平面的次采样。

## 2.4 扩充说明

请记住，有些元素查询支持格式的底层硬件，并相应地提供它们的Caps(在进入就绪状态或更高时通常这样做)。因此，可以查询到的 Caps ，平台之间是有差异的。

# 三、A trivial Pad Capabilities Example

本教程实例化了两个元素(这次通过它们的工厂)，显示了它们的Pad模板，链接它们并设置了要播放的管道。在每个状态更改中，都显示了sink元素的Pad的功能，这样您就可以观察到协商是如何进行的，<font color="red">直到设置了Pad上限。</font>


```
#include <gst/gst.h>

/* Functions below print the Capabilities in a human-friendly format */
static gboolean print_field (GQuark field, const GValue * value, gpointer pfx) {
  gchar *str = gst_value_serialize (value);

  g_print ("%s  %15s: %s\n", (gchar *) pfx, g_quark_to_string (field), str);
  g_free (str);
  return TRUE;
}

static void print_caps (const GstCaps * caps, const gchar * pfx) {
  guint i;

  g_return_if_fail (caps != NULL);

  if (gst_caps_is_any (caps)) {
    g_print ("%sANY\n", pfx);
    return;
  }
  if (gst_caps_is_empty (caps)) {
    g_print ("%sEMPTY\n", pfx);
    return;
  }

  for (i = 0; i < gst_caps_get_size (caps); i++) {
    GstStructure *structure = gst_caps_get_structure (caps, i);

    g_print ("%s%s\n", pfx, gst_structure_get_name (structure));
    gst_structure_foreach (structure, print_field, (gpointer) pfx);
  }
}

/* Prints information about a Pad Template, including its Capabilities */
static void print_pad_templates_information (GstElementFactory * factory) {
  const GList *pads;
  GstStaticPadTemplate *padtemplate;

  g_print ("Pad Templates for %s:\n", gst_element_factory_get_longname (factory));
  if (!gst_element_factory_get_num_pad_templates (factory)) {
    g_print ("  none\n");
    return;
  }

  pads = gst_element_factory_get_static_pad_templates (factory);
  while (pads) {
    padtemplate = static_cast<GstStaticPadTemplate *>(pads->data);
    pads = g_list_next (pads);

    if (padtemplate->direction == GST_PAD_SRC)
      g_print ("  SRC template: '%s'\n", padtemplate->name_template);
    else if (padtemplate->direction == GST_PAD_SINK)
      g_print ("  SINK template: '%s'\n", padtemplate->name_template);
    else
      g_print ("  UNKNOWN!!! template: '%s'\n", padtemplate->name_template);

    if (padtemplate->presence == GST_PAD_ALWAYS)
      g_print ("    Availability: Always\n");
    else if (padtemplate->presence == GST_PAD_SOMETIMES)
      g_print ("    Availability: Sometimes\n");
    else if (padtemplate->presence == GST_PAD_REQUEST) {
      g_print ("    Availability: On request\n");
    } else
      g_print ("    Availability: UNKNOWN!!!\n");

    if (padtemplate->static_caps.string) {
      GstCaps *caps;
      g_print ("    Capabilities:\n");
      caps = gst_static_caps_get (&padtemplate->static_caps);
      print_caps (caps, "      ");
      gst_caps_unref (caps);

    }

    g_print ("\n");
  }
}

/* Shows the CURRENT capabilities of the requested pad in the given element */
static void print_pad_capabilities (GstElement *element, gchar *pad_name) {
  GstPad *pad = NULL;
  GstCaps *caps = NULL;

  /* Retrieve pad */
  pad = gst_element_get_static_pad (element, pad_name);
  if (!pad) {
    g_printerr ("Could not retrieve pad '%s'\n", pad_name);
    return;
  }

  /* Retrieve negotiated caps (or acceptable caps if negotiation is not finished yet) */
  caps = gst_pad_get_current_caps (pad);
  if (!caps)
    caps = gst_pad_query_caps (pad, NULL);

  /* Print and free */
  g_print ("Caps for the %s pad:\n", pad_name);
  print_caps (caps, "      ");
  gst_caps_unref (caps);
  gst_object_unref (pad);
}

int main(int argc, char *argv[]) {
  GstElement *pipeline, *source, *sink;
  GstElementFactory *source_factory, *sink_factory;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;
  gboolean terminate = FALSE;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the element factories */
  source_factory = gst_element_factory_find ("audiotestsrc");
  sink_factory = gst_element_factory_find ("autoaudiosink");
  if (!source_factory || !sink_factory) {
    g_printerr ("Not all element factories could be created.\n");
    return -1;
  }

  /* Print information about the pad templates of these factories */
  print_pad_templates_information (source_factory);
  print_pad_templates_information (sink_factory);

  /* Ask the factories to instantiate actual elements */
  source = gst_element_factory_create (source_factory, "source");
  sink = gst_element_factory_create (sink_factory, "sink");

  /* Create the empty pipeline */
  pipeline = gst_pipeline_new ("test-pipeline");

  if (!pipeline || !source || !sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Build the pipeline */
  gst_bin_add_many (GST_BIN (pipeline), source, sink, NULL);
  if (gst_element_link (source, sink) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }

  /* Print initial negotiated caps (in NULL state) */
  g_print ("In NULL state:\n");
  print_pad_capabilities (sink, "sink");

  /* Start playing */
  ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state (check the bus for error messages).\n");
  }

  /* Wait until error, EOS or State Change */
  bus = gst_element_get_bus (pipeline);
  do {
    msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE, GstMessageType (GST_MESSAGE_ERROR | GST_MESSAGE_EOS |
        GST_MESSAGE_STATE_CHANGED) );

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
          if (GST_MESSAGE_SRC (msg) == GST_OBJECT (pipeline)) {
            GstState old_state, new_state, pending_state;
            gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
            g_print ("\nPipeline state changed from %s to %s:\n",
                gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));
            /* Print the current capabilities of the sink element */
            print_pad_capabilities (sink, "sink");
          }
          break;
        default:
          /* We should not reach here because we only asked for ERRORs, EOS and STATE_CHANGED */
          g_printerr ("Unexpected message received.\n");
          break;
      }
      gst_message_unref (msg);
    }
  } while (!terminate);

  /* Free resources */
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  gst_object_unref (source_factory);
  gst_object_unref (sink_factory);
  return 0;
}

```


## 3.1 如何获取 Pad Template 的 信息，包括他的 Capabilities

- gst_element_factory_find = 获取元素工厂的指针
- gst_element_factory_get_longname = 获取元素工厂的名字
- gst_element_factory_get_static_pad_templates = 根据工厂获取元素模板
- 理解 GstStaticPadTemplate 结构体里的属性
- GList 的应用
- 如何打印出 caps 里面的内容(因为里面结构比较复杂，可以看一下。先不深究了，反正后续可以使用该类模板)

```
#include <gst/gst.h>
/* Functions below print the Capabilities in a human-friendly format */
static gboolean print_field (GQuark field, const GValue * value, gpointer pfx) {
  gchar *str = gst_value_serialize (value);

  g_print ("%s  %15s: %s\n", (gchar *) pfx, g_quark_to_string (field), str);
  g_free (str);
  return TRUE;
}


static void print_caps (const GstCaps * caps, const gchar * pfx) {
  guint i;

  g_return_if_fail (caps != NULL);

  if (gst_caps_is_any (caps)) {
    g_print ("%sANY\n", pfx);
    return;
  }
  if (gst_caps_is_empty (caps)) {
    g_print ("%sEMPTY\n", pfx);
    return;
  }

  for (i = 0; i < gst_caps_get_size (caps); i++) {
    GstStructure *structure = gst_caps_get_structure (caps, i);

    g_print ("%s%s\n", pfx, gst_structure_get_name (structure));
    gst_structure_foreach (structure, print_field, (gpointer) pfx);
  }
}

int main(int argc, char *argv[]) {
    GstElementFactory *source_factory;

    gst_init(&argc, &argv);

    source_factory = gst_element_factory_find ("audiotestsrc");
    if(!source_factory)
    {
        g_printerr ("Not all element factories could be created.\n");
         return -1;
    }
    const GList *pads;       // what is this ?????
    GstStaticPadTemplate *padtemplate;
    g_print ("Pad Templates for %s: , number  = %d \n", gst_element_factory_get_longname (source_factory), gst_element_factory_get_num_pad_templates (source_factory));
    if (!gst_element_factory_get_num_pad_templates (source_factory)) {
      g_print ("  none\n");
      return -1;
    }
    pads = gst_element_factory_get_static_pad_templates(source_factory);
    while(pads) {
        padtemplate = static_cast<GstStaticPadTemplate *>(pads->data);
        pads = g_list_next(pads);

        if (padtemplate->direction == GST_PAD_SRC)
           g_print ("  SRC template: '%s'\n", padtemplate->name_template);
         else if (padtemplate->direction == GST_PAD_SINK)
           g_print ("  SINK template: '%s'\n", padtemplate->name_template);
         else
           g_print ("  UNKNOWN!!! template: '%s'\n", padtemplate->name_template);

        if (padtemplate->presence == GST_PAD_ALWAYS)
             g_print ("    Availability: Always\n");
           else if (padtemplate->presence == GST_PAD_SOMETIMES)
             g_print ("    Availability: Sometimes\n");
           else if (padtemplate->presence == GST_PAD_REQUEST) {
             g_print ("    Availability: On request\n");
           } else
             g_print ("    Availability: UNKNOWN!!!\n");

        if (padtemplate->static_caps.string) {
          GstCaps *caps;
          g_print ("    Capabilities:\n");
          caps = gst_static_caps_get (&padtemplate->static_caps);
          print_caps (caps, "      ");
          gst_caps_unref (caps);

        }

        g_print ("\n");
    }
    return 0;
}

```
