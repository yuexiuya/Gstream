bin是一种容器元素，用来存放其他Element，但是其本身也是一个Element，可以像其他Element一样操作。

## 一、什么是bin？

容器允许将一组链接元素组合成一个逻辑元素。您不再处理单个元素，而是只处理一个元素bin。我们将看到，当您准备构建复杂的管道时，这是非常强大的，因为它允许您以较小的块分割管道。

bin还将管理其中包含的元素。它将对元素执行状态更改，并收集和转发总线消息。

管道:管理包含元素的同步和总线消息的通用容器。toplevel bin必须是一个管道，因此每个应用程序至少需要其中一个。


## 二、创建一个bin

bin 和 element 初始化的方式一样，使用元素工厂。也可以使用gst_bin_new() 和 gst_pipeline_new()。从bin中添加和删除元素可以使用gst_bin_add() 和 gst_bin_remove()。

如果你向bin中添加了一个element，那么这个bin将获取该element的所有权;如果你销毁了这个bin，那么这个元素也就被销毁了。如果你从bin中删除了一个元素，那么bin将自动销毁。


```
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElement *bin, *pipeline, *source, *sink;

  /* init */
  gst_init (&argc, &argv);

  /* create */
  pipeline = gst_pipeline_new ("my_pipeline");
  bin = gst_bin_new ("my_bin");
  source = gst_element_factory_make ("fakesrc", "source");
  sink = gst_element_factory_make ("fakesink", "sink");

  /* First add the elements to the bin */
  gst_bin_add_many (GST_BIN (bin), source, sink, NULL);
  /* add the bin to the pipeline */
  gst_bin_add (GST_BIN (pipeline), bin);

  /* link the elements */
  gst_element_link (source, sink);

[..]

}


```
在bin中有各种查找元素的函数。
- gst_bin_get_by_name()
- gst_bin_get_by_interface()
- gst_bin_iterate_elements() 遍历
