# 一、Metadata

GStreamer明确区分了两种类型的元数据，并支持这两种类型的元数据。
- stream tag ： 例子包括一首歌的作者，这首歌的名字或者是专辑的一部分。
- stream-info ： 它是对流属性的某种技术描述。这可以包括视频大小，音频采样，使用codecs等等。

# 二、Metadata reading

- Stream 信息从 GstPad 中获取是最方便的
- Tag 信息从 GStreamer 的 bus 里面获取(通过监听 GST_MESSAGE_TAG)

这里将说明一些小细节： GST_MESSAGE_TAG消息可能会在管道中多次触发。应用程序的职责是将所有这些标记放在一起，以一种友好、一致的方式将它们显示给用户。这时候可以尝试使用 gst_tag_list_merge()。加载新歌时一定要清空缓存，或者每隔几分钟听一次网络广播。另外，确保使用GST_TAG_MERGE_PREPEND作为合并模式，这样一个新的标题(稍后出现)会优先于显示的旧标题。
