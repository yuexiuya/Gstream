# 一、Goal

<font color="red" size=5> 硬解码加速 </font>

随着低功率设备越来越普遍，硬件加速视频解码已迅速成为一种必然。

本教程(实际上是一节课)介绍了硬件加速的一些背景知识，并解释了GStreamer如何从中受益。

如果设置正确，你不需要做任何特别的事情来激活硬件加速;GStreamer会自动利用它。


# 二、Introduction

视频解码可以是一项非常占用cpu的任务，特别是对于像1080p高清电视这样的高分辨率。幸运的是，配备了可编程gpu的现代显卡能够处理这一工作，让CPU专注于其他任务。对于低功耗的cpu来说，拥有专用的硬件是必不可少的。

<font color="0xFFFF">在目前的情况下(2016年6月)，每个GPU制造商提供了一种不同的方法来访问他们的硬件(一个不同的API)，并且一个强大的行业标准还没有出现。</font>

截至2016年6月，至少有8个不同的视频解码加速api。

- VAAPI(Video Acceleration API):最初是由Intel在2007年设计的，目标是基于unix的操作系统的X Window系统，现在是开源的。现在它也通过dmabuf支持Wayland。它目前并不局限于Intel gpu，因为其他制造商可以自由使用这个API，例如，想象技术或S3图形。通过 gstreamer-vaapi 包访问GStreamer

- VDPAU(Video Decode and Presentation API for UNIX):最初由NVidia在2008年设计，针对基于unix的操作系统的X Window系统，现在是开源的。虽然它也是一个开源的图书馆，但是除了NVidia之外，没有其他的制造商在使用它。可以通过插件中的vdpau元素访问GStreamer。

- OpenMAX(Open Media Acceleration):由非营利技术联盟Khronos Group管理，它是一个“免版主、跨平台的c语言编程接口集，为日常工作提供了抽象，特别对音频、视频和静态图像非常有用”。通过gst-omx插件可以访问GStreamer。

- OVD(Open Video Decode):另一个来自AMD图形的API，设计为一个平台不可知论的方法，用于软件开发人员在AMD Radeon显卡中利用通用的视频解码(UVD)硬件。目前无法使用GStreamer!!!

- DCE(Distributed Codec Engine):一个开源软件库(“libdce”)和API规范的德州仪器，针对Linux系统和ARM平台。通过gstreamer-ducati插件可以访问GStreamer.

- Android MediaCodec:这是Android的API，可以访问设备的硬件解码器和编码器。这可以通过在gst-plugin -bad中的androidmedia插件访问。这包括编码和解码

- Apple VideoTool Box Framework:苹果的API访问h是通过applemedia插件提供的，该插件包括通过vtenc元素进行编码，并通过vtdec元素进行解码。

- Video4Linux:最近的Linux内核有一个以标准方式公开硬件编解码器的内核API，现在由gst-plugin -good的v4l2插件支持。这可以根据平台支持解码和编码

# 三、硬件加速视频解码插件的内部工作

这些api通常提供许多功能，如视频解码、后处理或解码帧的表示。

例如，gstreamer-vaapi插件提供了vaapidecode、vaapipostproc和vaapisink元素，这些元素允许通过VAAPI进行硬件加速解码，将原始视频帧上载到GPU内存，分别下载GPU框架到系统内存和GPU框架的表示。

重要的是要区分传统的GStreamer框架，哪些是驻留在系统内存中，哪些是由硬件加速api生成的框架。后者存储在GPU的内存里，Gstreamer是无法访问的。他们也可以下载到系统内存里，并且用常规的GStreamer框架处理。但是，显然在GPU中的处理效果要高得多。

GStreamer需要跟踪这些“硬件缓冲区”的位置，因此常规缓冲区仍然从元素到元素。它们看起来像普通的缓冲区，但是映射它们的内容要慢得多，因为必须从硬件加速元素所使用的特殊内存中检索。这种特殊的内存类型是使用分配查询机制来协商的。

这一切意味着，如果系统中存在一个特定的硬件加速API，并且相应的GStreamer插件也可用，那么像playbin这样的自动插入元素可以自由地使用硬件加速来构建它们的管道;应用程序不需要做任何特殊的事情来启用它。

当playbin必须在不同的同样有效的元素中进行选择时，比如传统的软件解码(例如通过vp8dec)或硬件加速解码(例如通过vaapidecode)，它使用他们的rank来决定。rank是表示其优先级的每个元素的属性;playbin将简单地选择能够构建完整管道并具有最高级别的元素。

因此，playbin是否会使用硬件加速，取决于所有能够处理该媒体类型的元素的相对级别。因此，确保硬件加速的最简单方法是通过改变关联元素的秩，如本代码所示:

```
static void enable_factory (const gchar *name, gboolean enable) {
    GstRegistry *registry = NULL;
    GstElementFactory *factory = NULL;

    registry = gst_registry_get();
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

关键方法是 gst_plugin_feature_set_rank()，它将请求的元素工厂的级别设置为所需的级别。为了方便起见，等级划分为无级、边缘级、次级和主级，但任何数字都可以。在启用元素时，我们将其设置为PRIMARY+1，因此它的级别比通常具有一级级别的其他元素要高。将元素的级别设置为 NONE 会使自动插入机制永远不会选择它。
