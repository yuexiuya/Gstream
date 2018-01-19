# 一、目标
本章阐述一个新的 GStream 的概念，允许在运行中构建一个 pipeline，而不是在一开始定义一个完整的管道。

本章重点：

1. 如何在连接元素时获得更好的控制；
2. 如何订阅你感兴趣的事件；
3. element的各种状态；

# 二、介绍

正如你所将要看到的那样，本章的 pipeline 在播放之前 都没有被完整的创建，也就是本章的核心——Dynamic pipeline。这是可行的，如果这样做，pipeline 将会在 data 流到末尾时产生一个 Error 并且 stop。

在这个例子中，我们将打开一个 multiplexed (or muxed) 文件，混流(muxed) 意味这 audio 和 video 是存储在同一个文件里的。负责打开这种文件的 element叫做 demuxers，这种容器的格式( 存储audio & video 的容器)有 Matroska(MKV), Quick Time(QT.MOV), Ogg, or Advanced Systems Format(ASF, WMV,WMA)

如果一个 container 中嵌入 multiple streams (如 one video and two audio tracks, for exmaple), demuxers 将会分离它到不同的输出通道去。在这种方式中，管道中将创建不同的分支去处理不同类型的 data。

Element 中的输入输出口叫做 GstPad。如 source elements 只有 source pads，只负责输出；sink elements 只有 sink pads，只负责接受； 而 filter elements 两者都有。

![image](https://github.com/yuexiuya/Gstream/blob/master/image/Basic_tutorial3.png?raw=true)
