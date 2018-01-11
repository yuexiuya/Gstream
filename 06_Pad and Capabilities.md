- Pad ：前面说到元素的连接，pad充当了连接的桥梁。
- Pad's Capabilities : 元素可以处理的多媒体类型都保存在Capabilities里面

## 一、Pad的简介

Pad的两个重要属性：方向 和 可用性。
- 方向：顾名思义是其方向性
- 可用性：总是、有时(动态)、请求。

### 1.1 Pads Always

这个没什么好说的，当这个元素被创建的时候，它的Pad属性也就被生成好了，在这个元素的生命周期里长期存在。

### 1.2 Dynamic (or sometimes) pads

顾名思义,动态随机Pads。这些Pads在元素被创建的时候，可能被隐藏，只有在特殊的时候才会被创建出来，使用完之后再次被释放。

gstream官网上推荐我们一种使用方法，就是为它的signal绑定一个方法去监听来得知pads被创建。


(1)学会使用gst-inspect

```
gst-inspect oggdemux
```

他会得到下面一段输出
- Factory Details : 生产元素的工厂名称
- Plugin Details ： 元素本身的描述
- Pad Templates: 包含一些Pads(sometimes)
- Element Flags:
- Element Implementation:
- Element Properties:
- Element Signals:

下面这些元素可以帮助我们进行编程
```
Factory Details:
  Long name:	Ogg demuxer
  Class:	Codec/Demuxer
  Description:	demux ogg streams (info about ogg: http://xiph.org)
  Author(s):	Wim Taymans <wim@fluendo.com>
  Rank:		primary (256)

Plugin Details:
  Name:			ogg
  Description:		ogg stream manipulation (info about ogg: http://xiph.org)
  Filename:		/usr/lib/x86_64-linux-gnu/gstreamer-0.10/libgstogg.so
  Version:		0.10.36
  License:		LGPL
  Source module:	gst-plugins-base
  Source release date:	2012-02-20
  Binary package:	GStreamer Base Plugins (Ubuntu)
  Origin URL:		https://launchpad.net/distros/ubuntu/+source/gst-plugins-base0.10

GObject
 +----GstObject
       +----GstElement
             +----GstOggDemux

Pad Templates:
  SRC template: 'src_%d'
    Availability: Sometimes
    Capabilities:
      ANY

  SINK template: 'sink'
    Availability: Always
    Capabilities:
      application/ogg
      application/x-annodex


Element Flags:
  no flags set

Element Implementation:
  Has change_state() function: 0x7fee06b68a20
  Has custom save_thyself() function: gst_element_save_thyself
  Has custom restore_thyself() function: gst_element_restore_thyself

Element has no clocking capabilities.
Element has no indexing capabilities.
Element has no URI handling capabilities.

Pads:
  SINK: 'sink'
    Implementation:
      Has chainfunc(): 0x7fee06b71a40
      Has custom eventfunc(): 0x7fee06b683e0
      Has custom queryfunc(): gst_pad_query_default
      Has custom iterintlinkfunc(): gst_pad_iterate_internal_links_default
      Has acceptcapsfunc(): gst_pad_acceptcaps_default
    Pad Template: 'sink'

Element Properties:
  name                : The name of the object
                        flags: 可读, 可写
                        String. Default: "oggdemux0"

Element Signals:
  "pad-added" :  void user_function (GstElement* object,
                                     GstPad* arg0,
                                     gpointer user_data);
  "pad-removed" :  void user_function (GstElement* object,
                                       GstPad* arg0,
                                       gpointer user_data);
  "no-more-pads" :  void user_function (GstElement* object,
                                        gpointer user_data);

```

(2) 一个例子
```

#include <gst/gst.h>

static void
cb_new_pad (GstElement *element,
        GstPad     *pad,
        gpointer    data)
{
  gchar *name;
  name = gst_pad_get_name (pad);
  g_print ("A new pad %s was created\n", name);
  g_free (name);

  /* here, you would setup a new pad link for the newly created pad */
//[..]

}

int
main (int   argc,
      char *argv[])
{
  GstElement *pipeline, *source, *demux;
  GMainLoop *loop;

  /* init */
  gst_init (&argc, &argv);

  /* create elements */
  pipeline = gst_pipeline_new ("my_pipeline");
  source = gst_element_factory_make ("filesrc", "source");
  g_object_set (source, "location", argv[1], NULL);
  demux = gst_element_factory_make ("oggdemux", "demuxer");

  /* you would normally check that the elements were created properly */

  /* put together a pipeline */
  gst_bin_add_many (GST_BIN (pipeline), source, demux, NULL);
  gst_element_link_pads (source, "src", demux, "sink");

  /* listen for newly created pads */
  g_signal_connect (demux, "pad-added", G_CALLBACK (cb_new_pad), NULL);

  /* start the pipeline */
  gst_element_set_state (GST_ELEMENT (pipeline), GST_STATE_PLAYING);
  loop = g_main_loop_new (NULL, FALSE);
  g_main_loop_run (loop);

//[..]

}
```

### 1.3 Request pads

element同样可以拥有request pads。这种pads不是自动被创建，而是根据请求被创建的。这在多路复用(multiplexers)类型的element中有很大的用处。例如 aggregators以及tee  element。

Aggregators  element可以把多个输入流合并成一个输出流; tee element件正好相反，它只有一个输入流，然后根据请求把数据流发送到不同的输出 pads。只要应用程序需要另一份数据流，它可以简单的从tee element请求到一个 输出pads。

- 下面一段代码演示了怎样在一个”tee”元件请求一个新的输出衬垫:


```
static void
some_function (GstElement *tee)
{
  GstPad * pad;
  gchar *name;

  pad = gst_element_get_request_pad (tee, "src%d");
  //这个gst_element_get_request_pad仅仅对request pad起作用，其他类型的pad会报错
  name = gst_pad_get_name (pad);
  g_print ("A new pad %s was created\n", name);
  g_free (name);

  /* here, you would link the pad */

  /* [..] */

  /* and, after doing that, free our reference */
  gst_object_unref (GST_OBJECT (pad));
}

```

- gst_element_get_request_pad() 可以用于检索一个 请求衬垫
- gst_element_get_compatible_pad () 可以用于检索一个 和目标衬垫想兼容的衬垫

下面的代码说明了如何请求一个相兼容的衬垫，并且把他们link起来。(如将从一个Ogg多路复用器请求一个兼容的pad从任何输入。)

```
static void
link_to_multiplexer (GstPad     *tolink_pad,
                     GstElement *mux)
{
  GstPad *pad;
  gchar *srcname, *sinkname;

  srcname = gst_pad_get_name (tolink_pad);
  pad = gst_element_get_compatible_pad (mux, tolink_pad, NULL);
  gst_pad_link (tolink_pad, pad);
  sinkname = gst_pad_get_name (pad);
  gst_object_unref (GST_OBJECT (pad));

  g_print ("A new pad %s was created and linked to %s\n", sinkname, srcname);
  g_free (sinkname);
  g_free (srcname);
}

```

## 二、Capabilities of a pad

在GstCaps对象中描述了pad的功能。在内部，GstCaps将包含一个或多个描述一个媒体类型的GstStructure。一个经过协商的pad将具有一个包含完全一个结构的功能集。而且，这个结构只包含固定值。这些约束不适用于未协商的pad或pad模板。

下面我们将剖析一下 gst-inspect-1.0 vorbisdec 所展现出来的信息

```
Pad Templates:
  SRC template: 'src'
    Availability: Always
    Capabilities:
      audio/x-raw
                 format: F32LE
                   rate: [ 1, 2147483647 ]
               channels: [ 1, 256 ]

  SINK template: 'sink'
    Availability: Always
    Capabilities:
      audio/x-vorbis
```

下面这一段话为原文翻译，仅做参考。目前我还不能明白其中属性的意义。

接收器将接受vorbis编码的音频数据，与媒体类型为“音频/x-vorbis”。源pad将使用原始音频媒体类型(在本例中为“audio/x-raw”)将原始(解码)音频样本发送到下一个元素。源pad还将包含音频samplerate的属性和通道数量，还有一些您现在不需要担心的内容。

### 2.1 属性和值

这里主要说明一下 gstream 编程常用的类型，包括 GLib 以及 GST 中的TYPE。

- 基础类型

G_TYPE_INT ：

G_TYPE_BOOLEAN ：TRUE \ FALSE

G_TYPE_FLOAT

G_TYPE_STRING : UTF-8 string

GST_TYPE_FRACTION : 分数形式

- 范围类型

GStreamer注册的GTypes，以表示可能的值范围。它们用于表示允许的音频采样值或支持的视频大小。

GST_TYPE_INT_RANGE ： 一系列可能存在的整数。例如，“vorbisdec”元素的速率属性可以在8000到50000之间。

GST_TYPE_FLOAT_RANGE： 该属性表示一系列可能的浮点值，具有较低的和较高的边界。

GST_TYPE_FRACTION_RANGE：该属性表示一系列可能的分数值，有一个较低的和一个上边界。

- 列表类型

GST_TYPE_LIST ： 该属性可以从列表中给出的基本值列表中获取任何值。

表示支持的大写字母为44100 Hz的采样率和48000 Hz的采样率，将使用一个整数值列表，其中一个值为44100，一个值为48000。

- 数组类型

GST_TYPE_ARRAY ： 属性是一个值数组。数组中的每个值本身也是一个完整的值。数组中的所有值都应该是相同的基本类型。这意味着一个数组可以包含任何整数的组合，整数的列表，整数的范围，浮点数和字符串的组合，但是它不能同时包含浮点数和整数。


### 2.2 Capabilities 的作用
Capabilities(Caps) 描述在两个垫子之间传输的数据类型，或者一个pad(模板)支持的数据类型。这使它们在各种用途上非常有用:
1. 自动跟踪(Autoplugging):根据其功能自动寻找到连接到pad的元素。所有的自动插件都使用这种方法。
2. 兼容性检测(Compatibility detection)： 当两个pad连接在一起，Gstream会认证这两个pad是否支持相同的媒体类型。
3. 元数据(Metadata): 阅读 Capabilities，可以了解元素中数据的信息；
4. 过滤(Filtering): 应用程序可以使用功能来限制可能的媒体类型，这些媒体类型可以在两个pad之间传输到它们支持的流类型的特定子集。



#### 2.2.1 关于 metadata 的一个例子

没仔细看，第二遍来研究一下
```
static void
read_video_props (GstCaps *caps)
{
  gint width, height;
  const GstStructure *str;

  g_return_if_fail (gst_caps_is_fixed (caps));

  str = gst_caps_get_structure (caps, 0);
  if (!gst_structure_get_int (str, "width", &width) ||
      !gst_structure_get_int (str, "height", &height)) {
    g_print ("No width/height available\n");
    return;
  }

  g_print ("The video size of this set of capabilities is %dx%d\n",
       width, height);
}


```

#### 2.2.2 关于 Filtering 的例子

虽然 Capabilities 主要是用于描述pad的媒体类型，但是我们必须对其有充分的了解，以便于与插件进行交互。这里我们将会演示一下，如何对元素中的数据类型进行过滤。

- 利用 gst_caps_new_simple 创建了一个GstCap
- 利用 gst_element_link_filtered() link两个元素，并进行过滤

```

static gboolean
link_elements_with_filter (GstElement *element1, GstElement *element2)
{
  gboolean link_ok;
  GstCaps *caps;

  caps = gst_caps_new_simple ("video/x-raw",
          "format", G_TYPE_STRING, "I420",
          "width", G_TYPE_INT, 384,
          "height", G_TYPE_INT, 288,
          "framerate", GST_TYPE_FRACTION, 25, 1,
          NULL);

  link_ok = gst_element_link_filtered (element1, element2, caps);
  gst_caps_unref (caps);

  if (!link_ok) {
    g_warning ("Failed to link element1 and element2!");
  }

  return link_ok;
}

```

- 当需要过滤的媒体类型(video/x-raw)不仅仅只有一个时候，可以有一种更加方便，更加格式化的方式设置过滤器

```
static gboolean
link_elements_with_filter (GstElement *element1, GstElement *element2)
{
  gboolean link_ok;
  GstCaps *caps;

  caps = gst_caps_new_full (
      gst_structure_new ("video/x-raw",
             "width", G_TYPE_INT, 384,
             "height", G_TYPE_INT, 288,
             "framerate", GST_TYPE_FRACTION, 25, 1,
             NULL),
      gst_structure_new ("video/x-bayer",
             "width", G_TYPE_INT, 384,
             "height", G_TYPE_INT, 288,
             "framerate", GST_TYPE_FRACTION, 25, 1,
             NULL),
      NULL);

  link_ok = gst_element_link_filtered (element1, element2, caps);
  gst_caps_unref (caps);

  if (!link_ok) {
    g_warning ("Failed to link element1 and element2!");
  }

  return link_ok;
}
```


### 2.3 Ghost pads

幽灵衬垫。

之前我们一直在讨论 pads of element，但是有没有一种可能 element 中没有 pads ？？？ 假设此时我们创建了 element1， element2，并将他们放入一个 bin 中，那么 bin 还会把 pad 暴露给我们吗？显然这是不可能的。这便是 Ghost pads的意义！！！

![image](https://github.com/yuexiuya/Gstream/blob/master/image/GhostPadA.png?raw=true)

当有了 Ghost Pads ， 我们的 bin 就有了一个暴露给外部的 pad。对于外部来说， element1 的 pad 现在也是 Bin 的 pad 了。

![image](https://github.com/yuexiuya/Gstream/blob/master/image/GstPadB.png?raw=true)

下面我们来演示一下，如何创建 Ghost Pad。

- gst_ghost_pad_new ()

```
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElement *bin, *sink;
  GstPad *pad;

  /* init */
  gst_init (&argc, &argv);

  /* create element, add to bin */
  sink = gst_element_factory_make ("fakesink", "sink");
  bin = gst_bin_new ("mybin");
  gst_bin_add (GST_BIN (bin), sink);

  /* add ghostpad */
  pad = gst_element_get_static_pad (sink, "sink");
  gst_element_add_pad (bin, gst_ghost_pad_new ("sink", pad));
  gst_object_unref (GST_OBJECT (pad));

[..]

}
```
