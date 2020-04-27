# 前言

Java 虚拟机动态加载,链接,和初始化类和接口.加载是查找具有特定名称的类或接口类型的二进制表示形式并从该二进制表示形式创建类或接口的过程.链接是 获取类或接口并将其组合到Java虚拟机的运行时状态(the run-time state of JVM)以便能够被执行的过程.类或接口的初始化包括执行类或接口的初始化方法.
	
这里描述了JVM 如何从类或接口的二进制表示形式派生符号引用(Symbolic References).说明了加载,链接和初始化的过程是如何被JVM首次执行的.指明类和接口的二进制表示形式如何被类加载器加载的和类和接口是如何被创建的.描述了链接和初始化的细节.介绍了绑定本地方法(native method)的概念.最后介绍了JVM何时退出


## 1. 运行时常量池(The Run-Time Constant Pool)

Java虚拟机为每个类和接口维护了一个运行时常量池.此数据结构的实现方式是符号表,满足了常规编程语言中符号表的目的.类或接口的二进制表示形式中的` CONSTANT_pool table`用于创建类或接口时构造运行时常量池.
运行时常量池中有两种条目(entry):符号引用(可能在之后被解析)和静态常量(无需进一步处理).
运行时常量池中的符号引用是从`CONSTANT_pool table`中按照每个条目的结构派生的.

- 从`CONSTANT_Class_Info`结构派生对类或接口的符号引用.这样的引用通过以下形式给出类或接口的名称:

- 对于非数组类或接口,名称是类或接口的二进制名称.

- 对于n维的数组类,名称以n次出现的ASCII字符 `[` 开头,后面跟元素类型的表示形式:

	- 如果元素类型是原始类型,则由相应的字段描述符表示.
	- 否则如果元素是引用类型,则用ASCII字符 `~`开头 ,后面是带有ASCII字符 `;` 的元素类型的二进制名称表示.  

> 我们在这篇文章中说的 类或接口的名称,都应该理解为上面的形式
>
> 这也是调用`Class.getName`方法得到的返回值. 

- 从`CONSTANT_Fieldref_Info` 结构派生对类或接口的字段的符号引用.这样的引用给出了字段的描述符和名称,以及对能够在其中找到该字段的类或接口.

- 从 `CONSTANT_Methodref_Info` 结构派生对类方法的符号引用.这样的引用给出了方法的名称和描述符,以及对能够在其中找到该方法的类.

- 从 `CONSTANT_InterfaceMethodref_Info` 结构派生出接口方法的符号引用.这样的引用给出了接口方法名称和描述符,以及能够在其中找到该方法的接口.

- 从 `CONSTANT_MethodHandle_Info` 结构派生出对方法句柄的符号引用.这样的引用根据方法句柄的类型给出了对类或接口的字段,类的方法或接口的方法给出符号引用.

- 从 `CONSTANT_MethodType_Info` 结构派生出对方法类型的符号引用.这样的引用给出了方法的描述符.

- 从` CONSTANT_Dynamic_info` 结构派生出对 `dynamically-computed` 常量的引用,这样的引用给出了:

	- 对方法句柄的引用,该方法句柄将被调用以计算常量的值.
	- 符号引用和静态常量的序列,在调用方法句柄时将用作静态参数.
	- an unqualified name and a field descriptor.

- 从 `CONSTANT_InvokeDynamic_info` 结构派生出对 `dynamically-computed call site` 的引用,这样的引用给出了:

	- 一个指向 在 `invokedynamic` 指令执行过程中被调用来计算一个`java.lang.invoke.CallSite` 实例的方法句柄的符号引用.
	- 符号引用和静态常量的序列,在调用静态方法句柄时将用作静态参数
	- an unqualified name and a method descriptor

运行时常量池中的静态常量也根据每个条目的结构从 `CONSTANT_pool table` 中的条目派生而来.

- 字符串常量是String类实例的引用,它是从`CONSTANT_String_info`结构派生的.为了派生一个字符串常量,JVM将检查由`CONSTANT_String_info`结构给出的代码点序列:

- - 如果先前已经在一个 String 类的实例上调用了`String.intern`方法,并且该实例包含与 `CONSTANT_String_info` 结构给出的Unicode代码点序列相同的序列,那么字符串常量是对那个 String 类实例的引用.
  - 否则,将创建一个新的 String 类实例,其中包含` CONSTANT_String_info` 结构给出的Unicode代码点序列.字符串常量是对新实例的引用.最后,在新实例上调用`String.intern`方法.

- 数值常量是从 `CONSTANT_Integer_info`,`CONSTANT_Float_info`,`CONSTANT_Long_info`,和 `CONSTANT_Double_info` 结构派生的.

> 请注意,`CONSTANT_Float_info` 结构代表`IEEE 754` 单格式的值,而 `CONSTANT_Double_info` 结构代表 `IEEE 754` 双格式的值.因此,从这两个结构派生的数值常量必须是可以分别使用` IEEE 754` 单格式和双格式表示的值.

> `CONSTANT_pool table` 中的其余结构-描述性结构 `CONSTANT_NameAndType_info`, `CONSTANT_Module_info` and `CONSTANT_Package_info` 以及基础结构 `CONSTANT_Utf8_info` 仅在构造运行时常量池时间接使用.运行时常量池中没有任何条目直接对应这些结构.

运行时常量池中的某些条目是可加载的,这意味着:

- 他们可能会被LDC系列指令推入堆栈
- 他们可能用于动态计算常量和调用点(call site)的引导程序方法(bootstrap)的静态参数

如果运行时常量池中的条目是从`CONSTANT_pool table` 中的可加载条目中派生的,则它是可加载的.因此可以加载运行时常量池中的以下条目:

- 对类和接口的符号引用
- 对方法句柄的符号引用
- 对方法类型的符号引用
- 对动态计算的常量的符号引用
- 静态常量

## 2. Java 虚拟机启动

JVM 通过使用 `bootstrap class loader`  或用户自定义的类加载器创建初始类或接口来启动.然后,JVM会链接初始类或接口,对其进行初始化,然后调用公共静态方法 `void main(String[])`.对此方法的调用将推动所有更进一步的执行.