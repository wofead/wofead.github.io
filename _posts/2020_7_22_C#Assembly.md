# C# Assembly

[toc]

Assembly,配件或程序集，以示和组件加以区别。一个配件有时候指一个EXE或者DLL文件，其实是一个应用程序（就是指带有主程序入口点的模块）或者一个库文件。但是配件实际上可以是由一个或者多个文件组成（dlls，exes，html等等），代表一组资源，以及类型的定义和实现的集合。一个配件可以包含对其他配件的引用。所有这些资源、类型和引用都在一个列表manifest中描述。manifest也是配件的的一部分，所以配件是一个自我描述的，不需要其他附加的部件。

​	描述配件的另一个重要的特性是，它是.Net环境下类型表示的一部分，也可以说是基本单位。因为，区分一个类型的表示就是包含这个类型的配件名字加上类型名本身。

​	举个例子，配件A定义了类型T，配件B中一样定义了类型B，但是.Net把这两个类型认为是不同的类型。注意，不要把**配件(assembly)**和**命名空间(namespace)**混淆起来。其实命名空间仅仅是用来把类型名用树的形式组织起来的手段。对于运行环境来将，类型名即使类型名，和命名空间一点关系都没有。总之，记住配件名加上类型名唯一标识一个运行时类型。另外，配件也是.Net框架用于安全策略的基本单位。

## 怎么生成一个Assembly

介绍一下c#中的**csc**命令：

1. csc File.cs  --->产生File.exe
2. csc /target:library FIle.cs ---->File.dll
3. csc /out:My.exe File.cs --->My.exe

使用.Net编译器，然后定义一个类。编译生成一个exe可执行文件。或者使用/target:library编译生成库文件。

## 私有配件和共享配件

私有配件通常只被一个应用程序使用,一般它被保存在应用程序目录,或者其子目录下面.。而共享配件通常保存在全局的配件catch缓冲区中, 它是一个由.Net运行时环境维护的配件仓库。

## 介绍

 在传统的Windows应用程序开发中，[动态连接库](https://baike.baidu.com/item/动态连接库/10577642)(DLL)为软件提供了一种重要的可重用机制。同样[组件对象模型](https://baike.baidu.com/item/组件对象模型/3351546)(COM)也通过DLLs和EXEs的形式提供了组件重用机制。在.NET的世界里， 则由assembly提供了类似的可重用的代码绑定机制。Assembly中包含了可以在[CLR](https://baike.baidu.com/item/CLR/10567215)(Common Language Runtime)中执行的代码。所有的.NET应用程序都是由一个或多个assembly组成的，不论你在创建一个[Console](https://baike.baidu.com/item/Console/10509035), WinForms，WebForms应用程序或者一个类库时，实际上你都是在创建assembly。甚至.NET本身也是通过assembly来实现其功能。

一个assembly可以由一个或者多个文件组成，简单来说，你可以把assembly理解成一个逻辑上的DLL。每个assembly必须有一个单独的执行入口如[DllMain](https://baike.baidu.com/item/DllMain/763193), WinMain, Main等。Assembly也有一套配置（Deploying）和[版本控制](https://baike.baidu.com/item/版本控制/3311252)（Versioning）的机制。它可以避免一些诸如DLL兼容性问题，并且大大简化配置上存在的问题。

## 静态和动态的Assembly

通常我们可以用Visual Studio.NET或命令行编译器(.NET SDK中带的)来生成assembly。

如果你正确的编译你的代码，assembly就会以DLL或者EXE的形式保存在磁盘上。像这样保存在物理磁盘上的assembly被称为静态assembly。

.NET也允许你通过Reflection APIs来动态生成assembly。（Reflection指获得assembly信息以及assembly类型信息的功能，类型信息指assembly中的class, interface, member, method等内容。Reflection APIs在System.Reflection名称空间内）。像这样的驻留在内存里的assembly被称作动态assembly。如果需要，动态assembly也可以保存在磁盘中。

## 私有和共享的Assembly

启动一个.Net应用程序的时候，程序首先要检查自己的安装目录中是否有需要的Assembly，如果几个程序运行，那么每个都要在自己的安装目录中查找自己需要的Assembly。也就是每个程序都要在自己安装目录中查找自己需要的Assembly，这样的Assembly称为私有的Assembly。

还有一种就是所欲的程序共享Assembly，而不是使用他们自己的，对于这种可以在全局Assembly缓存。这样的Assembly在全局过程中有效，可以被机器内的所有程序共享，称为共享Assembly。

## Assembly优点

### 可以把你从DLL中解救出来

dll在版本方面很折磨人，典型的COM组件应用通常是把一个单独版本的组件放在执行的机器上，这样带来的问题就是开发人员在更新或者维护组件时常常因为组件版本的向后兼容性的限制而碰钉子。

.Net中很好解决这个问题，建立一个私用的Assembly，它将有能力管理同一组件的不同版本，Assembly保留其不同版本的copy，如果不同的应用程序需要同一组件的不同版本，那么通过调用组件不同的copy就行了。

### 支持并行执行

意思中同一个Assembly的不同版本可以在同一个机器同时执行。不同的应用程序可以同时使用同一Assembly的不同版本。共享式Assembly支持这种并行执行。

### 自描述的

COM组件需要把一些细节信息保存在系统注册表或者类型库里，当使用COM组件的程序运行时，他们要去注册表里手机细节信息然后才能调用。

而Assembly是自描述的，他们不需要把信息保存在注册表里，所有的信息都存在于Assembly自己的元数据(Metadata)里了。

### 配置简化

Assembly是自描述的，它不依赖于注册表保存信息，因此完全可以使用XCopy之类方法来配置它。

### 卸载容易

直接删除就可以。

## Assembly的结构

**Assembly Manifest： 包含Assembly的数据结构的细节**

**类型元数据：包含Assembly中允许的类型数据（class，interface，member，property ...**

**Microsoft intermediate language code(MSIL)微软中间语言**

> MSIL [反汇编程序](https://baike.baidu.com/item/反汇编程序/7871134)是 MSIL [汇编程序](https://baike.baidu.com/item/汇编程序/298210)（Ilasm.exe） 的伙伴工具。 Ildasm.exe 采用包含 [Microsoft](https://baike.baidu.com/item/Microsoft)中间语言 （MSIL） 代码的可迁移可执行 （PE） 文件，并创建相应的文本文件作为 Ilasm.[exe](https://baike.baidu.com/item/exe) 的输入。
>
> 类似[Java](https://baike.baidu.com/item/Java/85979)字节码的语言，也是为了能在不同[平台](https://baike.baidu.com/item/平台)移植所生成的中间代码。MSIL是将[.NET](https://baike.baidu.com/item/.NET)代码转化为[机器语言](https://baike.baidu.com/item/机器语言)的一个中间过程。它是一种介于高级语言和基于Intel的[汇编语言](https://baike.baidu.com/item/汇编语言/61826)的伪汇编语言。当用户编译一个.NET程序时，[编译器](https://baike.baidu.com/item/编译器)将源代码翻译成一组可以有效地转换为本机代码且独立于[CPU](https://baike.baidu.com/item/CPU)的指令。当执行这些指令时，实时（[JIT](https://baike.baidu.com/item/JIT)）编译器将它们转化为CPU特定的代码。由于公共语言运行库支持多种实时编译器，因此同一段msil代码可以被不同的编译器实时编译并运行在不同的结构上。从理论上来说，MSIL将消除多年以来业界中不同语言之间的纷争。在.NET的世界中可能出现下面的情况一部分代码可以用EFFIL实现，另一部分代码使用C#或VB.NET完成的，但是最后这些代码都将被转换为中间语言。
>
> 编译为托管代码时，编译器将源代码翻译为[Microsoft](https://baike.baidu.com/item/Microsoft)中间语言 (MSIL)，这是一组可以有效地转换为本机代码且独立于 CPU 的指令。MSIL 包括用于加载、存储和初始化对象以及对对象调用方法的指令，还包括用于算术和逻辑运算、控制流、直接内存访问、异常处理和其他操作的指令。要使代码可运行，必须先将 MSIL 转换为特定于 CPU 的代码，这通常是通过实时 (JIT) 编译器来完成的。由于[公共语言运行库](https://baike.baidu.com/item/公共语言运行库)为它支持的每种计算机结构都提供了一种或多种 JIT 编译器，因此同一组 MSIL 可以在所支持的任何结构上 JIT 编译和运行。
>
> 当编译器产生MSIL时，它也产生[元数据](https://baike.baidu.com/item/元数据)。元数据描述代码中的类型，包括每种类型的定义、每种类型的成员的签名、代码引用的成员和运行库在执行时使用的其他数据。MSIL 和元数据包含在一个可移植可执行 (PE) 文件中，此文件基于并扩展过去用于可执行内容的已公布的 Microsoft PE 和公共对象文件格式 (COFF)。这种文件格式包含 MSIL 或本机代码以及元数据，使得操作系统能够识别公共语言运行库映像。文件中的元数据以及 MSIL 的存在使代码能够描述自身，这意味着不再需要类型库或接口定义语言 (IDL)。运行库在执行过程中根据需要从该文件中查找并提取元数据。



## 单文件和多文件Assembly

上面提到的assembly结构中包含的东西可以被绑定到一个单独的文件里。这样的assembly叫单文件assembly。另外，所有的MSIL代码和相关的[元数据](https://baike.baidu.com/item/元数据/1946090)也可以被分到多个文件中，这些文件中每一个单独的文件称为一个.NET Module(模块)，.NET module中也可以包括其他一些文件如图像文件或资源文件。

下面我们了解一下assembly manifest的更详细的信息。Assembly manifest保存了assembly细节的数据结构。对多文件assembly来说，assembly manifest好像一个“绑定器”把多个文件绑定到一个assembly中。请注意Manifest和Metadata并不相同，Metadata保存的是在assembly和module里用到的数据类型(如class, interface, method等)的相应信息，而Manifest是用来描述assembly本身结构的细节信息的。

 对单文件Assembly来说，Manifest嵌在DLL或EXE文件内，对多文件assembly, Manifest可以内嵌在每个文件中也可以存在于一个委托（constituent）文件里。后面将会有详细说明。

下面列出了Manifest中的主要信息：

- Assembly名字
- 版本号
- Assembly运行的机器的操作系统和处理器
- Assembly中包含的文件列表
- 所有Assembly依赖的信息
- Strong Name信息

## Metadata

 Metadata数据是对assembly中数据的定义。每个EXE或DLL包含自己的详细的类型信息，这种数据叫Metadata。主要包括以下信息：

- Assembly的名字和版本
- Assembly暴露出的类型信息
- 基本的类和接口信息细节
- 安全访问细节
- 属性细节

## modules

前面提过Assembly可以有一个或多个[Modules](https://baike.baidu.com/item/Modules/5864907)组成。Module可以看作是一系列可管理的功能模块。它们转化为[MSIL](https://baike.baidu.com/item/MSIL/9049893)，一旦代码在runtime中运行，它们就可以被加入assembly。请注意module本身并不能执行，要利用它们首先要把它们加到assembly里。当然一个module可以被加到多个assembly中；配置一个assembly的同时也必须配置所用的modules。



## 实例

## 创建单文件的Assembly

我们创建一个Employee类，然后使用csc将其编译成一个dll，会得到一个Employee.dll文件，这个就是一个单文件的assembly。

### 创建多文件Assembly

我们创建三个，一个是**Clerk** ，一个是**Manager**，一个是**CompanyStaff**，分别`csc /t:module clerk.cs`和`csc /t:module manager.cs`模块和`csc /t:library /addmodule:clerk.dll /addmodule:manager.dll companystaff.cs`程序集。

然后`csc simpleclient.cs /r:companystaff.dll`来调用。



