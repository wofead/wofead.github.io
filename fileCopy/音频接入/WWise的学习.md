---
layout:     post
title:      WWise的学习
subtitle:   WWise
date:       2019-12-9
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - WWise

---

### 目录
1. Wwise 基本方法
2. WWise event
3. Game Object
4. Game Sync
5. Soundcaster（声音选角器）
6. 性能分析和故障排除
7. 理解SoundBank
8. Wwise 声音引擎
9. Listener

> Sometimes, you should know what you want, and what's your aim is. Then try your best to chase pursue it.

## Wwise 基本方法

WWise一共由5个组件构成：

1. 音频对象
2. Event（事件）
3. GameSync（游戏同步体）
4. Game Object
5. Listener
6. File Packager(文件打包器)

代表游戏单个声音的 Audio Object 完全由声音设计师在 Wwise 应用程序中创建和管理。而 Game Object 和 Listener（代表发送或接收音频的特定游戏元素）由程序员在游戏中创建和管理。最后两个部分（Event 和 Game Sync）用于驱动游戏中的音频。这两个组件在音频素材和游戏组件之间架起桥梁，因此是 Wwise 和游戏中都不可分割的一部分。

![](https://www.audiokinetic.com/images/2019.1.6_7110/?source=WwiseFundamentalApproach&id=approach_01.png)

## WWise event
Wwise 使用 Event（事件）驱动游戏中的音频。这些 Event 将 Action（动作）应用于工程层级结构中的不同声音对象或对象组。您选择的动作用于指定 Wwise 对象是否播放、停止、暂停等。

Event 分为两种不同的类型：
1. Action Event（动作事件）——这些 Event 使用一个或多个动作（例如播放、停止、暂停等）来驱动游戏中的声音、音乐和振动。
2. Dialogue Event（对白事件）——这些事件使用一种带 State 和 Switch 的决策树来动态地确定被播放的对象。

Action Event：Wwise 使用“动作”Event 驱动游戏中的声音、音乐和振动。这些 Event 将动作应用于工程层级结构中的不同结构。每个 Event 都可以包含一个或一系列动作。用户选择的动作指定 Wwise 对象是否将播放、暂停、停止等。为了处理声音、音乐或振动（motion）对象之间的过渡段，每个 Event 动作还拥有一组参数，您可以使用这些参数延迟或者淡入淡出来衔接前后的对象。

Dialogue Event：Wwise 使用 Dialogue Event 来驱动游戏中的动态对白。Dialogue Event 基本上是一组决定播放哪条对白的规则或条件。Dialogue Event 可用于重新创建游戏中存在的各种不同场景、条件或结果。为了确保用户能够覆盖所有情形，Wwise 还允许用户创建默认或备用条件。所有条件都用一系列 Stae 和 Switch 值来定义。这些 State 和 Switch 值组合成路径，用来定义游戏中的条件或者结果。每条路径再与 Wwise 中的特定声音对象相关联。在游戏中调用 Dialogue Event 时，游戏根据 Dialogue Event 中设定的条件对现有条件进行验证。满足游戏当前状况的条件（也就是 State／Switch 路径）决定播放哪条对白。

将 Event 集成到游戏中，在为游戏创建 Event 后，声音设计师可以将它们打包成 SoundBank（声音库）。然后将这些 SoundBank 加载到游戏中。在游戏中，游戏代码可以触发这些事件。例如，当玩家被杀后，通过触发相应事件来播放特殊的“Die”声音并停止播放“EnergyShield”声音。为了将这些事件集成到游戏中，程序员必须指定对哪个Game Object执行事件动作。这通过发送各个事件来完成。当您希望更改音频时，应由游戏代码来发送事件。您可以使用字符串或 ID 来发送事件。

## Game Object
Game Object（游戏对象）是 Wwise 中的核心概念，因为声音引擎中被触发的每个 Event（事件）都与一个 Game Object 相关联。Game Object 通常指游戏中能够发出声音的特定对象或元素，包括角色、武器、环境对象（比如火把）等。然而在某些情况下，您可以将 Game Object 指定给某个游戏元素的不同部分。例如，您可以将不同的 Game Object 指定给巨人角色的不同部分，以使巨人的脚步声和配音听起来好象来自 3D 声音空间中的不同位置。

Wwise 为每个 Game Object 存储了各种信息，Game Object 将使用这些信息来确定在游戏中如何播放每个声音。以下所有类型的信息均可与 Game Object 相关联：
* 与 Game Object 相关联的音频对象的属性偏置值，包括音量和音高。
* 3D 位置和朝向。
* Game Sync 信息，包括 State（状态）、Switch（切换开关）和 RTPC（实时参数控制）。
* 环境效果。
* 声障（Obstruction）和声笼（Occlusion）。

## Game Sync

在完成初步的游戏设计后，你可以开始考虑使用被称为Game Sync(游戏同步体）Wwise元素来串接处理互动音频中的变化和替换行为，这些行为也是游戏内容的一部分。你可以在五种不同的GameSync类型中指明你所需要的类型，以配合提升游戏画面的质量，取得最佳的效果。

* State（状态）——游戏中发生的变化，将将在全局范围内影响现有音效、音乐或振动（motion）的属性。
* Switch（切换开关）——为在不同条件下需要不同声音、音乐或振动的特定游戏元素而提供的替换项。
* RTPC（实时参数控制）——以某种方式映射到可变游戏参数值的属性，使得游戏参数值的改变将导致属性本身改变。
* Trigger（触发器）——对游戏中自发事件的响应，将启动插播乐句（Stinger）。插播乐句是叠加在当前正在播放的音乐上并与之混音的简短乐句。

在构建游戏项目时，您必须兼顾应对质量需求、内存使用限制以及您所面临的时间约束。使用 Game Sync 可以从战略上简化您的工作，节省存储空间，帮助创建真正沉浸式的游戏体验。

State：State 就如同“混音快照”，也即对游戏的音频和振动属性施加的全局偏置或调整，代表游戏中的物理和环境条件发生了变化。使用 State 可以简化设计音频和振动的方式，帮助您优化使用素材。

State 用作“混音快照”可以充分控制最终的声音输出，令其充满多层次的细节，并可与多种 State 组合，获得预期的结果。当一个对象上注册了多个 State 时，多个值的变化都可能会影响该对象的单个属性。在这种情况下，每个值的变化将累加在一起。

Switch：Switch（切换开关）代表游戏在不同条件下为特定 Game Objec t提供的条件替换项。声音、音乐和振动对象被组织起来并指派给各个 switch，从而当游戏中一个条件变为另一个条件时，都能够播放相应的音效或振动对象。指定给一个切换开关的各个 Wwise 对象被组合到一个 switch container（切换容器）中。当 Event（事件）发出改变信号时，切换容器确保开关被切换，并播放正确的声音、音乐或振动对象。

RTPC：实时参数控制（RTPC）用于根据游戏中发生的实时参数值变化，来实时编辑特定的对象属性。通过使用 RTPC，您可以将游戏参数映射到属性值，并“自动执行”属性更改，增强游戏的真实感。参数值以坐标图视图形式显示，其中，一条轴表示 Wwise 中的切换开关组或属性值，另一条轴表示游戏中的参数值。通过将属性值映射到游戏参数值，您将创建一条定义两个参数之间总体关系的 RTPC 曲线。您可以创建任意数量的曲线，为游戏玩家创造丰富的沉浸式体验。

Trigger：与所有 Game Sync 一样，Trigger（触发器）也是一个 Wwise 元素，游戏先对它进行调用，然后在 Wwise 中根据游戏中发生的事件来定义合适的反馈方式。更具体地说，在互动音乐中，触发器响应游戏中的自发事件，并启动 Stinger（插播乐句）。Stinger 是叠加在正在播放的音乐上并与之混音的简短乐句，也是以音乐的方式对游戏所做出的反应。例如，当一个忍者拔出武器时，您可以在已在播放中的动作音乐上插入强调类型的音乐效果，以增加场景的感染力。游戏将调用 Trigger，然后 Trigger 将启动插播乐句，于是在正在播放的配乐之外将播放一段音乐片段。

## 声音选角器

Soundcaster 可用于开发过程中的任一环节，可以使用工程中的任何 Wwise 对象和事件来创建音频和振动（motion）模拟。

* 原型设计和试验。
* 验证概念。
* 同时试听声音和音乐对象。
* 对游戏中的音频和振动进行性能分析。
* 对音频和振动进行混合和测试。

## 性能分析和故障排除

利用 Wwise 的 Game Profiler（游戏性能分析器）和 Game Object Profiler（Game Object 性能分析器）来测试音频和振动在每款平台上的性能表现。

Game Profiler 布局分为以下三个视图：
* Capture Log（捕获日志）——捕获并记录来自声音引擎的所有信息的日志。
* Performance Monitor（性能监视器）——声音引擎执行每项活动的性能图示，例如 CPU、内存和带宽。当从声音引擎中捕获到信息时，将实时显示该信息。
* Advanced Profiler（高级性能分析器）——一整套声音引擎衡量指标，可帮助您监控性能和排查故障。

Game Object Profiler 布局包含以下视图：
* Game Object Explorer（游戏对象浏览器）——Wwise Game Object 性能分析工具的控制中心，在此您可以选择要实时监视的Game Object 和 Listener。
* Game Object 3D Viewer（游戏对象3D 查看器）——Game Object 和 Listener 的三维视觉表示。
* Game Sync Monitor（游戏同步体监控器）——用于实时分析 RTPC（实时参数控制）值的工具。游戏运行期间将绘制 RTPC 值的曲线图，被监视的不同 Game Object 具有不同的 RTPC 值。

## 理解SoundBank

为了高效地管理游戏的音频和振动（motion）组件，Wwise 将游戏的所有音频和振动数据置于bank（库）中。bank 大体上是指包含游戏音频和振动数据、媒体或两者兼有的文件。这些 bank 在游戏的特定时间点加载到游戏平台内存中。通过仅加载必要的资源，可以针对每个平台优化音频和振动的内存占用。bank 是您所有工作成果的集合，其中包含构成游戏所需的最终音频和/或振动内容。

Wwise 中有两类 bank：
* 初始化 bank——包含工程所有一般信息的特殊 bank，包括有关总线层级结构的信息和有关state、switch、RTPC 和环境效果的信息。Wwise 在生成 SoundBank 时将自动创建初始化 bank。初始化 bank 通常在游戏开始时立即加载，因此在游戏期间可以轻松访问所有一般工程信息。在默认情况下，初始化 bank 被命名为“Init.bnk”。
* SoundBank——包含事件数据、声音和振动结构数据和/或媒体的文件。SoundBank 不同于初始化 bank，一般在游戏的不同时间点加载和卸载，这样可以提高平台内存利用率。事件与工程结构元数据也可以和媒体放到不同的 SoundBank 中，这样您可以只在需要时才加载媒体文件。由于所有平台各不相同，因此 Wwise 允许您轻松地为每款平台量身定制 SoundBank，并同时生成所有平台的 SoundBank。Wwise 还为您提供排查 SoundBank 相关故障的工具，确保您符合不同平台的要求。

为帮助您更加高效地工作，Wwise 中提供了 SoundBank 界面布局。此布局包括为您的项目创建、管理和生成 SoundBank 所需要的所有视图，包括 SoundBank Manager（声音库管理器）、SoundBank Editor（声音库编辑器）、Project Explorer（工程浏览器）和 Event Viewer（事件查看器）。

## 文件打包器

为 Wwise 工程生成的 SoundBank 和任何媒体流文件可使用 File Packager 这个独立的实用工具打包。文件包（file package）将文件系统抽象出来，封装成为独立单元，这意味着您可以避免平台具体文件系统的某些限制，包括文件名长度和实际文件数量。文件包还可以帮助您更好地管理游戏的多语言版本以及游戏发布后提供的可下载内容。

## Wwise 声音引擎

您可以使用 Wwise 创建漂亮的声音、音乐和振动结构，并将它们打包成 SoundBank（声音库），但是要呈现您设计的声音和振动（motion），您还需要强大、可靠的声音引擎。Wwise 声音引擎的最基本功能是实时管理和处理游戏音频和振动的各方面需求。它可以轻松地集成到游戏开发管线中，从而融入最终的游戏作品。

声音引擎还能够执行以下功能：
* 处理由声音设计师在创作工具中创建和定义的常见播放行为，包括随机和顺序播放。
* 可直接处理由声音设计师在创作工具中创建和细调过的淡变和交叉淡变。
* 使用声音设计师在创作工具中设定的播放限制、特定优先级设置和虚声部（virtual voice）管理声音和振动对象的优先级。
* 通过简单的 API 即可直接支持变化无穷的环境效果。另外，由于声音引擎动态创建和销毁指派给环境用的通路，它可以确保所有平台上具有一致的体验，并降低内存占用率和 CPU 工作负载。
* 当游戏中的元素部分或完全阻挡了声源时，引擎支持施加听起来很自然的声障和声笼条件效果。
* 支持游戏中多达 8 个不同的 Listener（听者）。
* 包含程序调试信息，在创作工具中可以实时显示这些信息，其结果是，声音设计师可以在游戏运行时实时进行性能分析，并根据分析结果采取正确的措施。

## Listener
Listener（听者）在游戏中代表话筒。Listener在游戏 3D 空间中拥有位置和朝向。在游戏期间，Listener 的坐标与 Game Objec t的位置进行比较，以便将与 Game Object 相关联的 3D 声音指定给相应的扬声器，来模模拟实的 3D 环境。程序员必须确保 Listener 的位置信息为最新状态；否则声音将会通过错误的扬声器来呈现。

如果多人同时在一个系统上玩游戏，或者同时显示多个视角，则每个视角需要属于其自己的 Listener，这样才能正确地为所有视角渲染音频。Wwise 声音引擎支持多达八个 Listener。在默认情况下，经过声明的每个 Game Object 只能指定给第一个 Listener。然而，程序员可以灵活地将任何 Game Object 动态地指定给任何 Listener，或取消这种指定。

