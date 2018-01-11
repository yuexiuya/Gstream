- 流经管道的数据其实是 Buffers & Events 的组合。
- Buffers 里面存放了具体的媒体信息， Events 里面存放的是控制信息(比如seeking information 和 end-of-stream)。

# 一、 Buffers
缓冲区包含将流经管道的数据。源数据会创建一个新的缓冲区，并将它通过一个pad传递到链中的下一个元素。当你使用 Gstream 创建媒体管道时，您不必处理缓冲区，元素会为您做这些。

Buffer的组成
- 指向内存对象的指针。内存对象封装了内存中的一个区域。
- 缓冲区的时间戳。
- 一个refcount(引用计数)，指示有多少元素使用这个缓冲区。当没有元素引用时，这个refcount将用于销毁缓冲区。
- flag

举一个简单的例子

一个典型的视频或音频解码器的工作： 先创建一个缓冲区，分配内存，放入数据，并传递给下一个元素。该元素读取数据，最后再释放他。

不过，还有更复杂的场景。元素可以在不分配新的缓冲区的情况下修改缓冲区。元素还可以写入硬件内存(例如从视频捕获源)或从x服务器分配的内存(使用XShm)。缓冲区可以是只读的，等等。


## 二、 Event

```
static void
seek_to_time (GstElement *element,
          guint64     time_ns)
{
  GstEvent *event;

  event = gst_event_new_seek (1.0, GST_FORMAT_TIME,
                  GST_SEEK_FLAG_NONE,
                  GST_SEEK_METHOD_SET, time_ns,
                  GST_SEEK_TYPE_NONE, G_GUINT64_CONSTANT (0));
  gst_element_send_event (element, event);
}
```
