## 一、Gstream的初始化
在gstreamer库可以使用，gst_init必须从主应用程序调用。这个调用将执行库的必要的初始化以及解析程序特定的命令行选项。

### 1.1 gstream初始化的第一个例子

- gst_init (&argc, &argv) ==》 初始化gst
- gst_version (&major, &minor, &micro, &nano) ==》获取gst的版本号
- 
```
#include <stdio.h>
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  const gchar *nano_str;
  guint major, minor, micro, nano;

  gst_init (&argc, &argv);

  gst_version (&major, &minor, &micro, &nano);

  if (nano == 1)
    nano_str = "(CVS)";
  else if (nano == 2)
    nano_str = "(Prerelease)";
  else
    nano_str = "";

  printf ("This program is linked against GStreamer %d.%d.%d %s\n",
          major, minor, micro, nano_str);

  return 0;
}

```
### 1.2 gstream初始化的第二个例子

使用GOption初始化的例子，GOption对于解析shell比较有帮助。后续可以参照这个demo来做

```
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  gboolean silent = FALSE;
  gchar *savefile = NULL;
  GOptionContext *ctx;
  GError *err = NULL;
  GOptionEntry entries[] = {
    { "silent", 's', 0, G_OPTION_ARG_NONE, &silent,
      "do not output status information", NULL },
    { "output", 'o', 0, G_OPTION_ARG_STRING, &savefile,
      "save xml representation of pipeline to FILE and exit", "FILE" },
    { NULL }
  };

  ctx = g_option_context_new ("- Your application");
  g_option_context_add_main_entries (ctx, entries, NULL);
  g_option_context_add_group (ctx, gst_init_get_option_group ());
  if (!g_option_context_parse (ctx, &argc, &argv, &err)) {
    g_print ("Failed to initialize: %s\n", err->message);
    g_clear_error (&err);
    g_option_context_free (ctx);
    return 1;
  }
  g_option_context_free (ctx);

  printf ("Run me with --help to see the Application options appended.\n");

  return 0;
}

```
## 二、Element

Element在gst的地位就类似于QObject在Qt里的地位吧。所有复杂的pipeline均是Element，比如说每个解码器，编码器，分流器，视频或音频输出实际上是一个gstelement。

### 2.1 source Element
源元素生成用于管道的数据，例如从磁盘读取或从声卡读取数据。只会生成数据，不会接受数据。

```
source --> XXX
```
### 2.2 Filters, convertors, demuxers, muxers and codecs
过滤器之类的元素，可以有任意个输入输出衬垫


```
//filter
sink --> src

//demuxer
sink --> auido
     --> video
```

### 2.3 Sink elements
接收器元素，是重点。只接受，不产生。

```
--> sink
```

### 2.4 第一个例子

gst_element_factory_make()是用来创建一个元素最简单的方法。

gst_object_unref() 可以用来释放某一个元素


```
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElement *element;

  /* init GStreamer */
  gst_init (&argc, &argv);

  /* create element */
  element = gst_element_factory_make ("fakesrc", "source");
  if (!element) {
    g_print ("Failed to create element of type 'fakesrc'\n");
    return -1;
  }

  gst_object_unref (GST_OBJECT (element));

  return 0;
}


```

其实，我们创建一个元素是依赖于元素工厂的。比如我们上面使用的 gst_element_factory_make ("fakesrc", "source")。首先我们应当确保fakesrc存在。

那么，我们可以使用一种更加安全和保险的形式来创建一个元素。



```
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElementFactory *factory;
  GstElement * element;

  /* init GStreamer */
  gst_init (&argc, &argv);

  /* create element, method #2 */
  factory = gst_element_factory_find ("fakesrc");
  if (!factory) {
    g_print ("Failed to find factory of type 'fakesrc'\n");
    return -1;
  }
  element = gst_element_factory_create (factory, "source");
  if (!element) {
    g_print ("Failed to create element, even though its factory exists!\n");
    return -1;
  }

  gst_object_unref (GST_OBJECT (element));
  gst_object_unref (GST_OBJECT (factory));

  return 0;
}


```
### 2.5 第二个例子

写第二个例子的用意在于介绍一下 GstElement这个结构体。

1. 虽然这个结构体里类型很多，但都是基于GObject属性来实现的，因此通常的GObject的查询方式这里都是支持的
2. 这里详细说明一下GstObject里面的name属性。也就是我们gst_element_factory_make()和gst_element_factory_create()里所给element定义的名字

```
struct _GstElement
{
  GstObject             object;

  /*< public >*/ /* with LOCK */
  GStaticRecMutex      *state_lock;

  /* element state */
  GCond                *state_cond;
  guint32               state_cookie;
  GstState              current_state;
  GstState              next_state;
  GstState              pending_state;
  GstStateChangeReturn  last_return;

  GstBus               *bus;

  /* allocated clock */
  GstClock             *clock;
  GstClockTimeDiff      base_time; /* NULL/READY: 0 - PAUSED: current time - PLAYING: difference to clock */

  /* element pads, these lists can only be iterated while holding
   * the LOCK or checking the cookie after each LOCK. */
  guint16               numpads;
  GList                *pads;
  guint16               numsrcpads;
  GList                *srcpads;
  guint16               numsinkpads;
  GList                *sinkpads;
  guint32               pads_cookie;

  /*< private >*/
  union {
    struct {
      /* state set by application */
      GstState              target_state;
      /* running time of the last PAUSED state */
      GstClockTime          start_time;
    } ABI;
    /* adding + 0 to mark ABI change to be undone later */
    gpointer _gst_reserved[GST_PADDING + 0];
  } abidata;
};
```
我们可以使用g_object_get()轻易的拿到这个element的名字

```
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElement *element;
  gchar *name;

  /* init GStreamer */
  gst_init (&argc, &argv);

  /* create element */
  element = gst_element_factory_make ("fakesrc", "source");

  /* get name */
  g_object_get (G_OBJECT (element), "name", &name, NULL);
  g_print ("The name of the element is '%s'.\n", name);
  g_free (name);

  gst_object_unref (GST_OBJECT (element));

  return 0;
}


```


## 三、关于Element Factory

从上面的例子中，我们第一次使用了元素工厂去创建一个元素。其实元素工厂内部注册的元素是非常多的，可以用它来自动创建很多元素。

简单看下，这个例子没有深究

```
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElementFactory *factory;

  /* init GStreamer */
  gst_init (&argc, &argv);

  /* get factory */
  factory = gst_element_factory_find ("fakesrc");
  if (!factory) {
    g_print ("You don't have the 'fakesrc' element installed!\n");
    return -1;
  }

  /* display information */
  g_print ("The '%s' element is a member of the category %s.\n"
           "Description: %s\n",
           gst_plugin_feature_get_name (GST_PLUGIN_FEATURE (factory)),
           gst_element_factory_get_metadata (factory, GST_ELEMENT_METADATA_KLASS),
           gst_element_factory_get_metadata (factory, GST_ELEMENT_METADATA_DESCRIPTION));

  gst_object_unref (GST_OBJECT (factory));

  return 0;
}


```

也许元素工厂最强大的特性是，它包含了元素可以生成的PAD的完整描述，以及这些缓冲区的功能（通俗地说：什么类型的媒体可以流过这些缓冲区），而实际上不需要将这些插件加载到内存中。这可以被用来提供一个编码器，解码器选择列表，也可以用于autoplugging用媒体播放器。目前所有的基于GStreamer媒体播放器和autopluggers这样工作。我们会仔细看这些特点为我们了解在下一章gstpad和gstcaps：垫和能力

## 四、link elements
source   -->  filter  --> sink
    src -> sink    src -> sink
    
我们尝试把散落的元素搭建在一起，变成一个简单的工具链。


1. gst_bin_add_many : 把子元素添加到父元素里
2. gst_element_link_many ： 把各子元素连接起来

```
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElement *pipeline;
  GstElement *source, *filter, *sink;

  /* init */
  gst_init (&argc, &argv);

  /* create pipeline */
  pipeline = gst_pipeline_new ("my-pipeline");

  /* create elements */
  source = gst_element_factory_make ("fakesrc", "source");
  filter = gst_element_factory_make ("identity", "filter");
  sink = gst_element_factory_make ("fakesink", "sink");

  /* must add elements to pipeline before linking them */
  gst_bin_add_many (GST_BIN (pipeline), source, filter, sink, NULL);

  /* link */
  if (!gst_element_link_many (source, filter, sink, NULL)) {
    g_warning ("Failed to link elements!");
  }

[..]

}


```

## 五、Element States

创建后，元素实际上不会执行任何操作。你需要改变元素状态使它做某事。

GStreamer中有四种状态：

1. GST_STATE_NULL:这是默认状态。在该状态中没有分配资源，因此转换到它将释放所有资源。元素必须在这个状态时，其引用计数达才能为0，进行释放。

2. GST_STATE_READY:在就绪状态下，一个元素已经分配了它所有的全局资源，即可以在流中保存的资源。您可以考虑打开设备、分配缓冲区等。但是，在这种状态下没有打开流，因此流位置自动为零。如果先前打开了一个流，它应该在这种状态下关闭，位置、属性等应该重置。

3. GST_STATE_PAUSED:在这种状态下，一个元素打开了该流，但没有主动处理它。允许一个元素修改一个流的位置，读取和处理数据，并在状态改变为播放时准备播放，但不允许播放使时钟运行的数据。总之，暂停与播放相同，但没有运行时钟。进入暂停状态的元素应该准备好尽快移动到播放状态。例如，视频或音频输出将等待数据到达并排队，以便在状态改变后立即播放。此外，视频接收器已经可以播放第一帧（因为这还不影响时钟）。autopluggers可以使用同样的状态过渡到已经堵塞管道。大多数其他元素，如编解码器或过滤器，不需要在这种状态下显式地做任何事情。

4. GST_STATE_PLAY：与PAUSE一致，只不过可以使用始终


如果你设置状态的改变:

- NULL -> PLAYING， 它会在之间设置好 READY 和 PAUSE 状态；

- 当state = GST_STATE_PLAYING,管道会自动处理数据。GStreamer 会新起一个线程去处理他们。同时GStreamer会使用GstBus传递消息给应用主进程

- 当你为bin或者pipeline设置状态的时候，它通常会自动将状态变化传递到其中的所有元素。因此只需要为顶级管道设置状态就可以。
- 如果你想要动态的追加一个元素，请务必保证这个元素的状态和组件的状态一致