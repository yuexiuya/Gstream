https://gstreamer.freedesktop.org/documentation/application-development/index.html

## 一、前言

GStreamer是一个非常强大和灵活的创建流媒体应用程序框架。许多的GStreamer框架的美德来自于它的模块化：GStreamer可以无缝地整合新的插件模块。但由于模块化和功耗往往要付出更大的复杂性，所以编写新的应用程序并不总是容易的。

本指南旨在帮助您了解Gstreamer框架使您可以基于它开发应用。第一章将集中在一个简单的音频播放器的发展，有了很大的努力去帮助你了解GStreamer的概念。后面的章节将讨论与媒体回放相关的更高级的主题，也将涉及其他形式的媒体处理（捕获、编辑等）。 

## 二、谁需要这个手册

这本书是关于从应用程序开发者的角度GStreamer；它描述了如何使用gstreamer库和工具写GStreamer应用程序。

*关于编写插件的说明，我们建议插件作者指南。*

## 三、gstream开发必要的一些技能
- 一定的c语言基础
- 了解 GObject 和 Glib 编程
- 尤其是这些机制

(

    GObject instantiation

    GObject properties (set/get)

    GObject casting

    GObject referecing/dereferencing

    glib memory management

    glib signals and callbacks

    glib main loop
)

## 四、结构

1. GStreamer给你程序的概述，它的设计原则和依据。 
2. 如何构建一个简单的Gstream
3. 基础教程：在先进的GStreamer的概念，我们将以先进的主题使GStreamer脱颖而出的竞争对手。我们将讨论使用动态参数和接口的应用程序管道交互，我们将讨论线程和线程管道、调度和时钟（以及同步）。大部分的主题不是只向你介绍他们的API，但主要是为了解决和理解其概念GStreamer应用编程问题提供一个更深入的了解。
4. 高级教程，对于GStreamer应用更高层次的接口，我们将进入更高层次的编程API GStreamer。你不需要知道所有的细节，从以前的部分来理解这一点，但你需要了解基本概念，然而GStreamer。We will, amongst others, discuss playbin and autopluggers.
5. 最后在附录中，你会发现在一些随机的信息整合与GNOME，KDE，OS X和Windows，一些调试的帮助和一般提示改进和简化程序的编程。