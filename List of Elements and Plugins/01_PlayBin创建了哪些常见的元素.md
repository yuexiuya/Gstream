

```
0:00:00.098695439 18795      0x18a9cf0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "playbin"
Setting pipeline to PAUSED ...
0:00:00.099250463 18795      0x18a9cf0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "uridecodebin"
0:00:00.108512013 18795      0x18a9cf0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "filesrc" named "source"
0:00:00.108750528 18795      0x18a9cf0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "decodebin"
0:00:00.108868907 18795      0x18a9cf0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "typefind" named "typefind"
Pipeline is PREROLLING ...
0:00:00.216620971 18795      0x19f6280 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "qtdemux"
0:00:00.226509298 18795 0x7f9f680e7680 WARN                 qtdemux qtdemux_types.c:196:qtdemux_type_get: unknown QuickTime node type pasp
0:00:00.229697708 18795 0x7f9f680e7680 WARN                 qtdemux qtdemux.c:7979:qtdemux_parse_trak:<qtdemux0> unknown version 00000000
0:00:00.267910213 18795 0x7f9f680e7680 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "multiqueue"
0:00:00.318629652 18795 0x7f9f680e7680 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "h264parse"
0:00:00.319289384 18795 0x7f9f680e7680 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "capsfilter"
0:00:00.323073474 18795 0x7f9f680e7680 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "aacparse"
0:00:00.490009448 18795 0x7f9f5c0045e0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "pulsesink"
0:00:00.561659232 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "xvimagesink"
0:00:00.567899437 18795 0x7f9f5c0045e0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "faad"
0:00:00.765322537 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "avdec_h264"
Redistribute latency...
0:00:00.772828248 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "input-selector"
0:00:00.773122097 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:365:gst_element_factory_create: creating element "input-selector"
0:00:00.773361365 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "tee" named "audiotee"
0:00:00.773719606 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "bin" named "vbin"
0:00:00.773788611 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "queue" named "vqueue"
0:00:00.774140146 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "identity" named "identity"
0:00:00.787068989 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "videobalance" named "videobalance"
0:00:00.805181737 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "videoconvert" named "conv"
0:00:00.813867741 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "videoscale" named "scale"
0:00:00.817879413 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "bin" named "vdbin"
0:00:00.817956977 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "videoconvert" named "vdconv"
0:00:00.826414429 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "deinterlace" named "deinterlace"
0:00:00.828582306 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "bin" named "abin"
0:00:00.828651275 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "queue" named "aqueue"
0:00:00.828825855 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "identity" named "identity"
0:00:00.838508219 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "volume" named "volume"
0:00:00.839912076 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "audioconvert" named "conv"
0:00:00.844945685 18795 0x7f9f5c0042d0 INFO     GST_ELEMENT_FACTORY gstelementfactory.c:363:gst_element_factory_create: creating element "audioresample" named "resample"

```
