https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer/html/

# 一、GStreamer

```
#include <gst/gst.h>
```

Gstreamer库应用使用gst_init()来进行初始化。你应该传递正确的主函数的argc & argv 变量指针以便于Gstreamer可以处理自己的命令行选项，如下所示。

```
int main (int argc, char *argv[])
{
  // initialize the GStreamer library
  gst_init (&argc, &argv);
  //...
}
```

## gst_init ()

```
void gst_init (int *argc,
          char **argv[]);
//初始化GStreamer库，设置内部路径列表，注册内建元素，并加载标准插件。
//Parameters
//(1)argc : pointer to application's argc.
//(2)argv : pointer to application's argv.
```
## gst_init_check()

```
gboolean
gst_init_check (int *argc,
                char **argv[],
                GError **err);
//和gst_init功能类似，多一个失败返回FLASE的功能。
//Parameters
//(1)argc : pointer to application's argc.
//(2)argv : pointer to application's argv.
//(3)err : 错误信息的指针(可以用来查看错误信)
//Return
//如果创建成功返回TRUE

```
## gst_init_get_option_group ()

```
GOptionGroup *
gst_init_get_option_group (void);
//和gst_init功能类似，也是完成gstreamer的初始化。使用这种方法会返回一个带有Gstreamer参数的GOptionGroup数组。这个组需要使用标准的GOption回调，因此在使用这个方法之前，需要确保GOption的所有参数被初始化了
//更多请参考(g_option_context_add_group())
//Parameters
//Return
//a pointer to GStreamer's option group.
```
## gst_is_initialized ()

```
gboolean
gst_is_initialized (void);
//gstreamer是否被初始化了
//Parameters
//Return
//被初始化返回true

```
## void	gst_deinit ()

```
void
gst_deinit (void);
//释放gst_init()申请的资源
//不过一般不需要使用，因为在程序结束的时候，这些资源会自动释放
```

## gst_version ()

```
void
gst_version (guint *major,
             guint *minor,
             guint *micro,
             guint *nano);
//得到Gstreamer的库版本
//Parameters
//major : pointer to a guint to store the major version number
//minor : pointer to a guint to store the minor version number.
//micro : pointer to a guint to store the micro version number.
//nano : pointer to a guint to store the nano version number.
```
## gst_version_string ()

```
gchar *
gst_version_string (void);
//得到Gstreamer的版本字符串
//Return
//gchar : 版本字符串
```
## gst_segtrap_is_enabled ()

```
gboolean
gst_segtrap_is_enabled (void);
//GStreamer核心中的一些函数可能会安装一个自定义的SIGSEGV处理程序，以便更好捕获并向应用程序报告错误。目前，该特性在加载插件时默认启用。
//你可以通过gst_segtrap_set_enabled()来禁用此行为。如果应用想要使用自己的处理机制的时候可以这么做。
//Return
//开启返回TRUE
```
## gst_segtrap_set_enabled ()

```
void
gst_segtrap_set_enabled (gboolean enabled);
//应用程序选择是否关闭SIGSEGV处理程序
//Parameters
//enabled : true 开启
```

## gst_registry_fork_is_enabled ()

```
gboolean
gst_registry_fork_is_enabled (void);
//在默认情况下，GStreamer将使用一个辅助子进程执行扫描和重建注册表文件。
//该函数可
//Returns
//TRUE if GStreamer will use the child helper process when rebuilding the registry.
```
## gst_registry_fork_set_enabled ()

```
void
gst_registry_fork_set_enabled (gboolean enabled);
//应用程序可能希望在重新构建注册表时禁用/启用子辅助进程的生成
//Returns
//Parameters
//enabled 禁用/启用
```

## gst_update_registry ()
```
gboolean
gst_update_registry (void);

//迫使GStreamer重新扫描其插件路径，并更新默认的插件注册表。
//应用程序将几乎从不需要调用这个函数,只有有用如果应用程序知道新的插件已经安装(或旧的删除)自启动的应用程序(或者更精确地说,第一次调用gst_init())和应用程序想要利用任何新任插件而不需要重新启动应用程序。
//应用程序应该假设注册表更新既不是原子的也不是线程安全的，因此不应该有任何动态的管道(包括playbin和解码元素)，并且在更新过程中也不应该创建任何元素或者访问GStreamer注册表。
//注意，这个函数可能会阻塞相当长的时间。
//Returns
//TRUE if the registry has been updated successfully (does not imply that there were changes), otherwise FALSE.
```

# 二、GstObject

## gst_object_unref ()

```
void
gst_object_unref (gpointer object);
//减少对象的引用计数。如果引用计数=0，则销毁对象。
//这个函数不接受对象的锁，所以不要在LOCK的位置调用这个api，可能会导致死锁。
//Parameters
//(1)object : 一个object的对象。若该对象非gpoint类型，可以通过GST_OBJECT 宏转换。但是这个宏对于可转换的对象应该有限制，具体可以阅读一下这个宏相关的信息
```

## gst_object_set_name ()

```
gboolean
gst_object_set_name (GstObject *object,
                     const gchar *name);
                     //搜索指定名称的工厂
//Parameters
//设置Element的名称
//(1)object : GstObject 对象的指针
//(2)name : 设定的name
//Returns
//gboolean ： 成功\失败 -> true\flase
```
## gst_object_get_name ()

```
gchar *
gst_object_get_name (GstObject *object);
//获得Element的名称
//Parameters
//(1)object : GstObject 对象的指针
//Returns
//gchar * ： 返回element的name
```


# 三、GstElement

GObject

->GInitiallyUnowned

    ->GstObject

        ->GstElement

            ->GstBin

                ->GstPipeline

## gst_element_get_name()


```
#define gst_element_get_name(elem) gst_object_get_name(GST_OBJECT_CAST(elem))
gchar*	gst_object_get_name(GstObject *object);
//Parameters
//(1)elem 元素
//Return
//gchar* 元素的名字

```


## gst_element_link_pads ()

```
gboolean
gst_element_link_pads (GstElement *src,
                       const gchar *srcpadname,
                       GstElement *dest,
                       const gchar *destpadname);
//连接 src 和 dest 这两个 pad；副作用是如果其中一个 pad 没有父对象，那么他就会成为另一个 pad 的子对象。
//如果他们都没有父对象，则会连接失败。
//Parameters
//(1)src : 源衬垫
//(2)srcpadname : 源衬垫的名字，或者为NULL可以适用任意 pad 。（这里我不是很清楚，例子中似乎这里直接命名，而不是获取）
//(3)dest : 目标衬垫
//(4)destpadname : 目标衬垫的名字，或者为NULL可以适用任意 pad 。（这里我不是很清楚，例子中似乎这里直接命名，而不是获取）
//Return
//如果可以连接返回TRUE
```

## gst_element_link_filtered()

```
gboolean
gst_element_link_filtered (GstElement *src,
                           GstElement *dest,
                           GstCaps *filter);
//连接两个元素,filter作为过滤器
//link方向必须是从 src -> dest， 反方向并不会被尝试；
//这个方法会尝试去寻找没有连接的Pad，必要的时候会请求新的Pad

//Parameters
//(1) src : GstElement - 包含 source pad
//(2) dest ：GstElement - 包含 dest pad
//(3) filter : the GstCaps to filter the link, or NULL for no filter.

//Returns
//gboolean : TRUE if the pads could be linked, FALSE otherwise.
```

## gst_element_get_request_pad()

```
GstPad *
gst_element_get_request_pad (GstElement *element,
                             const gchar *name);
//用于检索一个 请求衬垫, by name (e.g. "src_%d")
//该GstPad应当调用gst_element_release_request_pad()释放
//Parameters
//(1) element : pad 所在的元素
//(2) name : 类似于 src_%d 的参数
//Return
//找到返回GstPad*,否则返回NULL
```

- src_%d 这个参数的获取方式，即通过gst-inspect检索 请求衬垫获取的参数；
- GstPad* 里面保存着Pad的属性

## gst_element_get_compatible_pad ()

```
GstPad *
gst_element_get_compatible_pad (GstElement *element,
                                GstPad *pad,
                                GstCaps *caps);
//在element中找寻一个还未被使用的，并且可以和 pad 兼容的 pad
//Parameters
//(1)element : 目标衬垫所在的元素
//(2)pad : 与目标衬垫兼容的衬垫
//(3)caps : 过滤器(可以为NULL)
//日常使用
//pad = gst_element_get_compatible_pad (mux, tolink_pad, NULL);
```

## gst_element_get_static_pad ()

```
GstPad *
gst_element_get_static_pad (GstElement *element,
                            const gchar *name);
//根据 element 的 name 检索一个 已经存在的 pad。

//Parameters
//(1) element : 元素
//(2) name ： 名字

//Rerturn
//如果有返回GstPad，否则返回NULL
```

## gst_ghost_pad_new ()

```
GstPad *
gst_ghost_pad_new (const gchar *name,
                   GstPad *target);

//Parameters
//(1) name : name of new pad; or null to assign to a default name
//(2) target : ghost pad 需要连接的 pad
//Rerturn
// a new GstPad, errror Return error !
```


## gst_element_link ()

```
gboolean
gst_element_link (GstElement *src,
                  GstElement *dest);
//Link src -> dest, 我觉得足够明确了
//Parameters
//(1)src
//(2)dest
//Returns
//true -> link成功， false -> link失败

```

# gst_element_link_many()

```
void
gst_bin_add_many (GstBin *bin,
                  GstElement *element_1,
                  ...);
//连接一系列的元素 ，不知道实用性如何，具体可参考上面的 gst_bin_add_many ()。感觉没有gst_element_link好用，毕竟需要指定src & dest
```

# gst_element_set_state ()

```
GstStateChangeReturn
gst_element_set_state (GstElement *element,
                       GstState state);
//设置元素的状态
//调用这个函数会遍历容器内所有元素并设置他们的状态
//如果该元素在另一个线程中异步执行状态的更改，这个函数将返回 GST_STATE_CHANGE_ASYNC。应用程序可以使用 gst_element_get_state ()等待它异步的状态更改，也可以等待总线上的GST_MESSAGE_ASYNC_DONE或GST_MESSAGE_STATE_CHANGED。
//State changes to GST_STATE_READY or GST_STATE_NULL never return GST_STATE_CHANGE_ASYNC.
//Parameters
//(1)element : 元素
//(2)state : gst状态
//Returns
//GstStateChangeReturn
```

# gst_element_get_state ()

```
GstStateChangeReturn
gst_element_get_state (GstElement *element,
                       GstState *state,
                       GstState *pending,
                       GstClockTime timeout);
//获取元素的状态
//针对异步使用 gst_element_set_state 的函数，这个方法会阻塞直到状态返回或超时
//没有异步使用 gst_element_set_state 的函数，会立刻返回；
//返回 GST_STATE_CHANGE_NO_PREROLL，等同于状态改变成功；但是虽然状态改变，但是无法产生数据(PAUSE操作)


//Parameters
//(1)element : 元素
//(2)state : gst状态
//Returns
//GstStateChangeReturn
```


```
typedef enum {
  GST_STATE_CHANGE_FAILURE             = 0,
  GST_STATE_CHANGE_SUCCESS             = 1,
  GST_STATE_CHANGE_ASYNC               = 2,
  GST_STATE_CHANGE_NO_PREROLL          = 3
} GstStateChangeReturn;
```

## gst_element_query_position ()

```
gboolean
gst_element_query_position (GstElement *element,
                            GstFormat format,
                            gint64 *cur);

//查询stream的位置信息，纳秒级的。
//Parameters
//(1) element : 需要查询时间的元素(一般是顶层元素)
//(2) GstFormat ： 输出格式
//(3) cur : 查询的时间
//Rerturn
//TRUE if the query could be performed.
```

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

## gst_element_query_duration ()

```
gboolean
gst_element_query_duration (GstElement *element,
                            GstFormat format,
                            gint64 *duration);
//查询stream的长度信息，纳秒级的。
//如果在播放的过程中， duration 改变了，我们可以从 bus 上得到一条DURATION_CHANGED 的信息
//Parameters
//(1) element : 需要查询时间的元素(一般是顶层元素)
//(2) format ： 输出格式
//(3) duration : 保存 查询的长度

//Return
// TRUE if the query could be performed.

```

# 四、GstBin


GObject

->GInitiallyUnowned

    ->GstObject

        ->GstElement

            ->GstBin

                ->GstPipeline

## gst_bin_new ()

```
GstElement *
gst_bin_new (const gchar *name);
//用给定 name 创建一个 容器(实际也是元素)

//Parameters
//(1) name : the name of the new bin.

//Return
//a new GstBin.
```


## gst_bin_add ()

```
gboolean
gst_bin_add (GstBin *bin,
             GstElement *element);
//向一个父元素中添加子组件
//Parameters
//(1)bin : 类型是GstBin,实际用法中GST_BIN (pipeline)，说明需要对一个父元素进行类型转换
//(2)element : 子组件
//Returns
//true -> 添加成功； false -> 添加失败

//一个例子

// create element, add to bin
sink = gst_element_factory_make ("fakesink", "sink");
bin = gst_bin_new ("mybin");
gst_bin_add (GST_BIN (bin), sink);

```

## gst_bin_add_many ()

```
void
gst_bin_add_many (GstBin *bin,
                  GstElement *element_1,
                  ...);
//向一个父元素中添加很多很多的子组件，具体参考上面的gst_bin_add()，就是将需要一次又一次调用具体参考上面的gst_bin_add()，简化为gst_bin_add_many()这个api。
//NULL-terminated 这个文档中出现了这个词组，我觉得有必要熟记这个词。->空终止字符串（计算机术语），说明这个列表的结尾，必须已NULL为结尾。可能是告知api这个list结束了吧
//gst_bin_add_many (GST_BIN (pipeline), source, filter, sink, NULL);
```


# 五、GstPipeline

## gst_pipeline_new ()

```
GstElement *
gst_pipeline_new (const gchar *name);
//创建一个pipeline
//Parameters
//(1)name -> pipe name
//Returns
//ptr -> new pipe
```

## gst_pipeline_get_bus ()

```
GstBus *
gst_pipeline_get_bus (GstPipeline *pipeline);
//得到管道上的总线指针，这个总线会监听来自GstMessage的消息
//Parameters
//(1)pipeline -> pipe name
//Returns
//GstBus，得到这个bus之后，会减少一个引用计数
//小贴士：ref增加引用计数；unref减少引用计数；这里不禁可以猜测bus是有限的，每次获取一个bus，就会减少一个引用计数
```

# 六、GstElementFactory

## gst_element_factory_make ()

```
GstElement *
gst_element_factory_make (const gchar *factoryname,
                          const gchar *name);
//用给定的工厂创建一个 GstElement 对象。
//Parameters
//(1)factoryname : 一个给定的已经实例化的工厂名称；
//(2)name : 指定的新组件名称，如果name = NULL，会自动分配一个唯一的名称，不过一般应该大家都会指定的。
//Return
//成功->返回新的组件的指针；失败 -> 返回NULL

```
## gst_element_factory_create ()

```
GstElement *
gst_element_factory_create (GstElementFactory *factory,
                            const gchar *name);
//用给定的工厂创建一个 GstElement 对象。
//Parameters
//(1)factory : 一个已经实例化的工厂指针；
//(2)name : 指定的新组件名称，如果name = NULL，会自动分配一个唯一的名称，不过一般应该大家都会指定的。
//Return
//成功->返回新的组件的指针；失败 -> 返回NULL

```

## gst_element_factory_find ()

```
GstElementFactory *
gst_element_factory_find (const gchar *name);
//搜索指定名称的工厂
//Parameters
//(1)name : 工厂名称
//Returns
//搜索到返回 GstElementFactory ， 没有查询到返回 NULL
```

# 七、GstBus

异步消息总线

## gst_bus_add_watch ()

```
guint
gst_bus_add_watch (GstBus *bus,
                   GstBusFunc func,
                   gpointer user_data);
//向总线表上增加一个监测func，这个func将遵从默认的优先级(G_PRIORITY_DEFAULT)
//这个func是将会异步的接受main loop里的消息，每条bus只能被一个func监听；如果你想要使用一个新的funcB，需要先把当前func删除
//The bus watch will only work if a GLib main loop is being run.
//这个监听func可以用gst_bus_remove_watch()删除或者当func 返回FALSE的时候也可被删除。如果func被加入了默认的main context中去，也可以用g_source_remove()删除。

```

# 八、GstPluginfeature
  GstPlugin的基本类

## gst_plugin_feature_get_name()


```
#define gst_plugin_feature_get_name(feature)      GST_OBJECT_NAME(feature)
//原来在gstpluginFeature是一个宏，以后个人还是最好使用这种宏的形式；
//获得组件的名称
//Parameters
//(1)feature GstPluginFeature
//Return
//gchar* 返回插件的名称

```

# 九、GstCaps

## gst_caps_is_fixed ()

```
gboolean
gst_caps_is_fixed (const GstCaps *caps);
//判断 GstCaps 是否为固定的(即只含有一个structure，并且在struture中的每一条只描述一个类型)
//非固定类型的为 GST_TYPE_INT_RANGE 或 GST_TYPE_LIST
//Parameters
//(1) the GstCaps to test
//Return
// ture = 固定； false = 非固定
```


## gst_caps_new_simple ()

```
GstCaps *
gst_caps_new_simple (const char *media_type,
                     const char *fieldname,
                     ...);
//顾名思义，用一种简单的方式创建caps
//创建一个GstCaps(包含Pad的能力)，它包含一个GstStructure的结构体。
//这个 GstStructure 的格式和用 gst_structure_new()创建的一致。
//调用者有必要释放这个GstCaps

//Parameters
//(1)media_type : 媒体类型
//(2)fieldname :
//...
//(...)NULL
//Return
//创建成功返回GstCaps*， 失败返回NULL

//一个例子

caps = gst_caps_new_simple ("video/x-raw",
        "format", G_TYPE_STRING, "I420",
        "width", G_TYPE_INT, 384,
        "height", G_TYPE_INT, 288,
        "framerate", GST_TYPE_FRACTION, 25, 1,
        NULL);
```

## gst_caps_new_full()

```
GstCaps *
gst_caps_new_full (GstStructure *struct1,
                   ...);
//配合gst_structure_new()创建过滤器

//Parameters
//(1) struct1 ： 过滤器的属性
//(..)
//(NULL)

//Return
// success -> GstCaps*, 否则返回NULL

//一个例子

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
```


## 十、 Event

# gst_event_new_seek ()

```
GstEvent *
gst_event_new_seek (gdouble rate,
                    GstFormat format,
                    GstSeekFlags flags,
                    GstSeekType start_type,
                    gint64 start,
                    GstSeekType stop_type,
                    gint64 stop);
//用给定的参数分配一个新的 Seek 事件。
//Seek Event 包含一个 playback， 以给定的速率反复的从 start -> stop
//1.0的速率意味着正常的回放速率，2.0意味着双倍速度。负值意味着向后播放。不允许使用0.0的速率，而应该通过暂停管道来完成。
//管道有一个默认的播放段，配置为起始位置为0，停止位置为-1，速率为1.0。当前配置的回放段可以用 GST_QUERY_SEGMENT 查询。
//start_type和stop_type指定如何在回放段中调整当前配置的启动和停止字段。可以对最后配置的值进行相对或绝对的调整。类型的GST_SEEK_TYPE_NONE意味着该位置不应该被更新。
//当速率为正并且开始被更新时，重放将从新配置的开始位置开始。
//对于负速率，重放将从新配置的停止位置(如果有的话)开始。如果停止位置更新，则它必须与-1 (GST_CLOCK_TIME_NONE)不同，以得到负值。
//不可能找到与当前播放位置相关的内容，这样做，暂停管道，使用GST_QUERY_POSITION查询当前播放位置，并使用GST_SEEK_TYPE_SET更新播放段当前位置，以达到所需的位置。

//Parameters
//(1) element : 需要 seek 的元素
//(2) rate : 快进速率
//(3) flags : 具体参照 GstSeekType， 常规使用 GST_SEEK_FLAG_FLUSH
//(4) start_type : The type and flags for the new start position
//(5) start : The value of the new start position
//(6) stop_type : The type and flags for the new stop position
//(7) stop : The value of the new stop position

//参数太多，实际应用中注意下面这几个参数就可以
gst_element_seek (pipeline, 1.0, GST_FORMAT_TIME, GST_SEEK_FLAG_FLUSH,
                           GST_SEEK_TYPE_SET, 60 * GST_MSECOND*1000,
                           GST_SEEK_TYPE_NONE, GST_CLOCK_TIME_NONE)
```

## gst_uri_is_valid ()

```
gboolean
gst_uri_is_valid (const gchar *uri);
//检测 uri 是否可用的，尝试着使用了一下%的路径，是可以检测通过的。估计这个API的检测和网络uri有关。
//Parameters
//(1) uri : a uri
//Return
//gboolean : 如果可用返回true； 不可用 返回false
```

## gst_filename_to_uri ()

```
gchar *
gst_filename_to_uri (const gchar *filename,
                     GError **error);
//与g_filename_to_uri()类似，但也尝试处理相对的文件路径。在将filename转换为URI之前，如果它是相对路径，它将被当前工作目录前缀，然后路径将被规范化，这样它就不包含任何内容。/”或“. ./ '段。
//Parameters
//(1) filename : 绝对路径 或者 文件名
//(2) error ： 错误码
//Returns
//gchar * ： a uri 字符串； 或者 NULL on error。必须使用 g_free() 释放。
```


## gst_parese_launch ()

```
GstElement *
gst_parse_launch (const gchar *pipeline_description,
                  GError **error);
//创建一个基于命令行语法的新管道。请注意，即使设置了错误，您可能会得到一个非NULL的返回值。

//Parameters
//(1) pipeline_description : 用来描述管道的属性(基于命令行的形式)
//(2) error : 设置失败时返回的错误码

//Returns
// GstElement* : 管道
```
