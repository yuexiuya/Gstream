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


## gst_pad_is_linked ()

```
gboolean		gst_pad_is_linked			(GstPad *pad);
//检测 pad 是否已经连接
//Parameters
//(1)pad : GstPadLinkReturn
//Return
//gboolean : true = 已经连接； false = 未连接
```

## gst_pad_query_caps ()

```
GstCaps *
gst_pad_query_caps (GstPad *pad,
                    GstCaps *filter);
//得到 pad 的能力。
//注意：
//(1)此方法并不能确保返回由 gst_event_new_caps()设置的caps；请使用gst_pad_get_current_caps()。
//(2)
//(3)此方法返回的GstCaps不可写。如果想要修改，请先使用gst_caps_make_writable().

//Parameters
//(1)pad : a GstPad (获得该 pad 的 caps)
//(2)filter ： suggested GstCaps, or NULL.
//Return
//GstCaps : the caps
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

## gst_element_factory_get_longname

```
//it is a macro
//用于获得 GstElementFactory 的名字
gst_element_factory_get_longname(GstElementFactory *)
//因为这是一个宏定义，常常这里只用于输出打印的字符串
    g_print ("Pad Templates for %s:\n", gst_element_factory_get_longname (factory));
```

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

## gst_element_factory_get_num_pad_templates ()

```
guint
gst_element_factory_get_num_pad_templates
                               (GstElementFactory *factory);
//得到 factory 中的 pad_templates 的数量

//Parameters
//(1) factory : ptr -> GstElementFactory
//Returns
//guint : pad_templates 的数量
```

## gst_element_factory_get_static_pad_templates ()

```
const GList *
gst_element_factory_get_static_pad_templates
                               (GstElementFactory *factory);
//得到 GstElementFactory* 中所有的 static pad template，并存储在 GList* 的容器中

//Parameters
//(1)factory : 元素工厂
//Return
//包含 static pad template 的 GList 容器
```


```
struct _GstStaticPadTemplate {
  const gchar     *name_template; // pad template 的名字
  GstPadDirection  direction;    // pad的方向
  GstPadPresence   presence;     // pad的存在属性(always,Sometimes, request)
  GstStaticCaps    static_caps;  // Caps of pad
};
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

## gst_bus_timed_pop_filtered ()

```
GstMessage *
gst_bus_timed_pop_filtered (GstBus *bus,
                            GstClockTime timeout,
                            GstMessageType types);
//从总线中获取信息，等待指定的超时(并丢弃任何与掩码不匹配的消息)。
//如果超时为0，则该函数的行为就像gst_bus_pop_filter()。
//如果超时是GST_CLOCK_TIME_NONE，此函数将永远阻塞，直到总线上发布匹配的消息。

//Parameters
//(1) bus : 总线
//(2) timeout : 超时
//(3) types ：消息类型

//Returns
//GstMessage * : 如果总线上没有找到匹配的消息返回NULL，否则在总线上找到与该过滤器匹配的GstMessage，直到超时过期。
//该消息是从总线中提取的，需要在使用后对gst_message_unref()进行未处理。
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


## gst_caps_is_any

```
gboolean
gst_caps_is_any (const GstCaps *caps);
//caps 是否代表任意的音频格式
//Parameters
//(1) caps : the GstCaps to test
//Returns
```

## gst_caps_is_empty

```
gboolean
gst_caps_is_any (const GstCaps *caps);
//caps 是否代表 空的音频格式
//Parameters
//(1) caps : the GstCaps to test
//Returns
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

## gst_caps_get_size ()

```
guint
gst_caps_get_size (const GstCaps *caps);
//返回 GstCaps 中存储的 structures 的数量

//Parameters
//(1) caps : a GstCaps
//Returns
//the number of structures that caps contains

```

## gst_caps_get_structure ()

```
GstStructure *
gst_caps_get_structure (const GstCaps *caps,
                        guint index);
//从 Caps 中 得到一条 GStructure
//警告：
//(1) caps 是 const，GstStructure * 是 non-const. 这是为了编程的方便。调用者必须明白 GstCaps 中 structures 不可以轻易被更改。但是，如果你知道caps 是可写的 (你 copy 过他们，或者使用过 gst_caps_make_writable()).
//(2) 你可以更改 GstStructure* 通过这种方式： gst_structure_set().
//(3) 你并不是需要 free 或者 unref 这个 structure，它属于 GstCaps。

//(1) caps : a GstCaps
//(2) index : index of the structure (一个GstCaps中可能含有多个GstStruture)
Returns
//a pointer to the GstStructure corresponding to index .
```

## gst_structure_get_name

```
const gchar *
gst_structure_get_name (const GstStructure *structure);
//获得 structure 的名字

//Parameters
//(1) structure : a GstStructure
//Rerturn
//gchar * :name of structure
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

## gst_element_seek_simple ()

```
gboolean
gst_element_seek_simple (GstElement *element,
                         GstFormat format,
                         GstSeekFlags seek_flags,
                         gint64 seek_pos);
//最简单的seek API，意味着 seek 到 seek_pos。如果想要更复杂的操作，如播放速率，相对移动，请参考 gst_element_seek().
//当 pipeline 的状态为 preolled PAUSED or PLAYING ，seeking 会起效果(除非是live stream)
//一些 elements 也许在 READY 状态下 seeking，在这种场景下，他保存 seek event 并且 execute 它。

//Parameters
//(1) element : a GstElement to seek on
//(2) format : a GstFormat to execute the seek in, such as GST_FORMAT_TIME
//(3) seek_flags : seek options; playback applications will usually want to use GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_KEY_UNIT here
//(4) seek_pos : position to seek to (relative to the start); if you are doing a seek in GST_FORMAT_TIME this value is in nanoseconds
//multiply with GST_SECOND to convert seconds to nanoseconds or with GST_MSECOND to convert milliseconds to nanoseconds.
//Returns
//TRUE if the seek operation succeeded. Flushing seeks will trigger a preroll, which will emit GST_MESSAGE_ASYNC_DONE.
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
//与g_filename_to_uri()类似，但也尝试处理相对的文件路径。
//在将filename转换为URI之前，如果它是相对路径，它将被当前工作目录前缀，然后路径将被规范化，这样它就不包含任何内容。/”或“. ./ '段。
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


## gst_query_new_seeking ()

```
GstQuery *
gst_query_new_seeking (GstFormat format);
//创建一个 GstQuery，便于查询 stream 的 seeking 属性；
//Parameters
//(1)format: GstFormat
//Returns
//GstQuery * :
```


## gst_registry_get()

```
GstRegistry *
gst_registry_get (void);
//生成检索单例 plugin 注册表，这个调用者不会拥有注册表的指针，这个注册表的生命周期和GStreamer自身管理

//Return
//GstRegistry * : 注册表
```

## gst_registry_find_feature ()

```
GstPluginFeature *
gst_registry_find_feature (GstRegistry *registry,
                           const gchar *name,
                           GType type);
//通过 name 和 type 找到 pluginfeature
//Parameters
//(1)registry ：注册表
//(2)name ：
//(3)type :
//Returns
//GstPluginFeature ->

```

## gst_plugin_feature_set_rank ()

```
void
gst_plugin_feature_set_rank (GstPluginFeature *feature,
                             guint rank);
//为插件特性指定一个级别，以便自动跟踪使用最合适的特性。
//Parameter
//(1)feature :
//(2)rank : 优先级

static void enable_factory (const gchar *name, gboolean enable) {
    GstRegistry *registry = NULL;
    GstElementFactory *factory = NULL;

    registry = gst_registry_get_default ();
    if (!registry) return;

    factory = gst_element_factory_find (name);
    if (!factory) return;

    if (enable) {
        gst_plugin_feature_set_rank (GST_PLUGIN_FEATURE (factory), GST_RANK_PRIMARY + 1);
    }
    else {
        gst_plugin_feature_set_rank (GST_PLUGIN_FEATURE (factory), GST_RANK_NONE);
    }

    gst_registry_add_feature (registry, GST_PLUGIN_FEATURE (factory));
    return;
}

```

# GstBuffer

1. Object Hierarchy
Object --> GstBuffer

2. includes
```
#include <gst/gst.h>
```

3. Description

Buffers 是 GStreamer 中数据传输的基本单元。它们包含时间和偏移量，以及与 GstMemory blocks 相关的二进制 metadata。

我们使用 gst_buffer_new() 创建一个 buffers。下面的示例创建了一个缓冲区，它可以使用给定的宽度、高度和每个平面的比特来保存给定的视频帧。

```
GstBuffer *buffer;
GstMemory *memory;
gint size, width, height, bpp;
...
size = width * height * bpp;
buffer = gst_buffer_new ();
memory = gst_allocator_alloc (NULL, size, NULL);
gst_buffer_insert_memory (buffer, -1, memory);
...
```

使用上面这个步骤可能有些繁琐，可以直接使用 gst_buffer_new_allocate() 去创建一个 给定大小 的 buffer。

在 Buffer 中包含一系列的 GstMemory 对象。可以使用 gst_buffer_n_memory() 查询对象的个数， 可以使用 gst_buffer_peek_memory() 去获取内存的指针。

Buffer 通常具有时间戳和持续时间，但它们都不能保证(它们可以设置为GST_CLOCK_TIME_NONE)。只要有一个有意义的值，就应该设置它们。时间戳和持续时间以纳秒为单位(它们是GstClockTime值) -- 不太理解

缓冲区 DTS 是指当缓冲区被解码时，通常是单调递增的时间戳。缓冲 PTS 是指当缓冲内容呈现给用户时的时间戳，并且不总是单调递增。-- DTS 总时长，PTS 当前播放位置

Buffer 也可以有一个或两个开始和结束偏移。这些都是媒体类型特定的。对于视频缓冲区，起始偏移量通常是帧数。对于音频缓冲区，它将是到目前为止所产生的样本数量。对于压缩数据，它可以是源文件或目标文件中的字节偏移量。同样，结束偏移将是缓冲区结束的偏移量。如果您知道缓冲区的媒体类型(前面的CAPS事件)，那么它们只能被有意义地解释。可以将两者都设置为GST_BUFFER_OFFSET_NONE。 -- 不太理解

使用gst_buffer_ref()来增加缓冲区的refcount。当您想要在将一个句柄推到下一个元素之后，就必须这样做。缓冲区refcount确定缓冲区的可写性，当refcount恰好为1时，缓冲区是可写的，即当调用者仅引用缓冲区时。

为了有效地从现有的缓冲区中创建一个较小的缓冲区，您可以使用gst_buffer_copy_region()。该方法尝试在两个缓冲区之间共享内存对象。
## gst_buffer_new ()

```
GstBuffer *
gst_buffer_new (void);
//创建一个空的 buffer
// MT safe

//Return
//the new GstBuffer.
```

## gst_allocator_alloc ()

```
GstMemory *
gst_allocator_alloc (GstAllocator *allocator,
                     gsize size,
                     GstAllocationParams *params);
//分配一个至少为 size 大小的内存块
//可选的params可以指定内存的前缀和填充，params = NULL 时什么也不指定；如果 flags 包含GST_MEMORY_FLAG_ZERO_PREFIXED和gst_memory_flag_zero_padd，则前缀/填充将填满0
//当 allocator 为空时，将使用默认的分配器。

//Parameters
//(1) allocator : 分配器
//(2) size : 最小的大小
//(3)params : flags

//Returns
//GstMemory * : a new GstMemory
```

## gst_buffer_insert_memory ()

```
void
gst_buffer_insert_memory (GstBuffer *buffer,
                          gint idx,
                          GstMemory *mem);
//插入 mem 到 buffer 中的 idx 位置，该函数获取了 mem 的所有权，因此不会增加它的引用计数
//只能将gst_buffer_get_max_memory()添加到缓冲区中。如果添加了更多的内存，将自动合并现有的内存块，为新内存腾出空间 - 不懂它在说啥

//Parameters
//(1) buffer : a GstBuffer
//(2) id : the index to add the memory at, or -1 to append it to the end
//(3) mem : a GstMemory
```

## gst_buffer_new_allocate()

```
GstBuffer *
gst_buffer_new_allocate (GstAllocator *allocator,
                         gsize size,
                         GstAllocationParams *params);
//类似于 gst_buffer_new && gst_allocator_alloc
//尝试使用给定大小的数据和分配器的额外参数创建一个新分配的 buffer 。如果无法分配请求的内存数量，则返回NULL。分配的缓冲区内存没有清除
//请注意，当size == 0时，缓冲区将没有与之关联的内存。

//Parameter
//(1) allocator : 分配器
//(2) size : the size in bytes of the new buffer's data.
//(3) params : optional Parameters

//Return
// a new GstBuffer or null
```

## gst_buffer_n_memory ()

```
guint
gst_buffer_n_memory (GstBuffer *buffer);
//获取该缓冲区所拥有的内存块的数量。这个值永远不会大于gst_buffer_get_max_memory()返回的值
//Parameters
//(1) buffer : a GstBuffer.
//Return
//guint : the number of memory blocks this buffer is made of.
```

## gst_buffer_peek_memory ()

```
GstMemory *
gst_buffer_peek_memory (GstBuffer *buffer,
                        guint idx);
//获得 buffer 中 idx 位置 的 GstMemory 的指针
//内存块保持有效，直到缓冲区中的内存块被删除、替换或合并，通常使用任何修改缓冲区中的内存的调用
//Parameters
//(1) buffer : a Gstbuffer
//(2) idx : 索引
//Returns
//GstMemory : point -> GstMemory
```

## gst_buffer_copy_region ()

```
GstBuffer *
gst_buffer_copy_region (GstBuffer *parent,
                        GstBufferCopyFlags flags,
                        gsize offset,
                        gsize size);
```
