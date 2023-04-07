## 编译打包总体流程

在Android开发过程中，我们通过Gradle命令，启动一个构建任务，最终会生成产物“APK”文件。常规APK的构建流程如下

![](	https://blog-1256965811.cos.ap-guangzhou.myqcloud.com/img/Android应用模块构建流程.png)

经典的Android应用构建流程如图所示，主要分为以下几步：

* 编译器将源代码转换成DEX文件（Dalvik虚拟机可执行文件，其中包括在Android设备上运行的字节码），并编译所有的资源文件，生成资源表和R文件。
* APK打包器将DEX文件和已编译资源合成APK或AAB（取决于所选的build任务）
* 打包器使用调试或发布密钥库为APK或AAB签名
* 在生成最终的APK之前，打包器会使用zipalign工具对应用进行对齐优化，以减少其在设备上运行时所占用的内存



## 编译打包详细流程

下面是一个详细的打包流程图
![](https://blog-1256965811.cos.ap-guangzhou.myqcloud.com/img/详细的构建流程.png)

### 编译资源

apk的资源包括：

* 工程中res目录下的所有文件
* assets目录下的文件
* AndroidManifest.xml

使用aapt2工具对资源进行编译，输出资源二进制文件压缩包（内部是.flat结尾的文件）

```shell
aapt2 compile -o  build/res.zip --dir res
```
**aapt2**

aapt2(Android Asset Packaging Tool2)是一种构建工具，Android Studio和Android Gradle插件使用它来编译和打包应用的资源，它是aapt优化后的工具。老版本的Android默认使用AAPT编译器进行资源编译，从Android Studio 3.0开始，AS默认启用了AAPT2作为资源编译的编译器。

与aapt不同，aapt2把资源编译打包过程拆分为两部分：

* 编译：将资源文件编译为二进制文件（flat）
  * 把所有Android的资源文件进行解析，生成扩展名为.flat的二进制文件。比如使png图片，那么就会被压缩处理，采用.png.flat的扩展名。可以在/build/intermediates/merged_res/文件下查看生成的中间产物
* 链接：合并所有已编译的文件并将它们打包到一个软件包中
  * 这一步会生成辅助文件，R.java和resources.arsc文件，R文件是一个资源索引文件，其值是资源ID。而resources.arsc文件是编译阶段生成的所有二进制资源文件的索引集合，通过R文件中的资源ID，即可在resources.arsc找到对应的资源文件真实路径。

>通过把资源编译拆分为两部分，aapt2能够很好的提升资源编译的性能。例如，之前一个资源文件发生变动，aapt需要做全量编译，appt2只需要重新编译改变的文件，然后和其他未发生改变的文件进行链接即可。

为什么需要进行资源编译：aapt2编译时把资源文件编译为FLAT文件，flat文件中部分数据是原始的资源内容，一部分是文件的相关信息。通过编译，生成的中间文件包含的信息比较全面，可用于增量编译。且二进制资源体积更小，加载更快。

### 链接资源

通过aapt2工具将资源整合，输出资源文件索引集合包resource.arsc和索引文件R文件

```shell
aapt2 link build/res.zip -I $ANDROID_HOME/platforms/android-29/android.jar -- java build --manifest AndroidManifest.xml -o build/app-debug.apk  
```

**resource.arsc**

我们都知道，引用R文件内的资源id，其实是一串数字，比如 2131099964 ，将其转为十六进制：0x7f06013c，这时的资源索引就是0x7f06013c，资源索引具有固定的格式：**0xPPTTEEEE**

* PackageId（2位）+ TypeId（2位）+ EntryId（4位）
* PP：PackageId，包的命名空间，取值范围为[0x01,0x7f]，第三方应用均为7f
* TT：资源类型，有anim、layout、mipmap、string、style等资源类型
* EEEE：代表某一类资源在偏移数组中的值

0x7f06013c中，PackageId=0x7f、TypeId=0x06、EntryId=0x013c

我们可以讲arsc函数想像成一个含有多个Pair数组的文件，且每个资源类型（TypeId）对应一个```Pair[]```（或多个，为了便于理解先只认为是一个）。因此在arsc中查找0x7f06013c元素的值，就是去设法找到TypeId=0x06所对应的数组```Paid[]```，然后找到其中第0x013c号元素```Pair[0x013c]```.这个元素恰好就是Pair("img", "./res/drawable-xxhdpi/img.png")，左边是资源名称img，右边是资源的文件路径"./res/drawable-xxhdpi/img.png"，有了文件路径，程序便可以访问到对应的资源文件了。[一文读懂resource.arsc文件结构](https://juejin.cn/post/6880043571010813959)

### AIDL文件编译

AIDL是安卓上方便使用Binder进行进程间通信（IPC），而定义的一种接口定义语言，最终还是会被解析为java文件（Binder IPC方式的模版代码）。

通过sdk/build-tools目录下的aidl工具，也可以手动进行aidl文件的解析

### Java、Kotlin文件编译

* 通过Java Compiler编译项目中所有的Java代码，包括R.java、.aidl文件生成的Java文件、Java源文件，生成.class文件。
  
```shell
javac -d build -cp $ANDROID_HOME/platforms/android_28/android.jar com/*/.java
```

* 通过Kotlin Compiler编译项目中所有的Kotlin代码，生成.class文件

```shell
kotlinc hello.kt -d hello.jar
```

注解处理器（APT、KAPT）生成代码也是在这个阶段生成的。当注解的生命周期被设置为CLASS的时候，就代表该注解会在编译class文件的时候生效，并且生成java源文件和class字节码文件。

### 打包dex文件

将上一步生成的.class文件打包生成dex文件。使用工具：dx或者d8

```shell
d8 --output build/ --lib $ANDROID_HOME/platforms/android_28/android.jar build/com/example/application/*.class   
```

在安卓中，不管是Dalvik还是Art，与JVM有很大的区别。Android系统并不直接使用Class文件，而是将所有的Class文件聚合打包成DEX文件，dex文件相比当个Class文件更加紧凑，可以直接在Android Runtime下执行。

DEX文件大致结构：

* header：DEX文件头，记录一些当前文件的信息以及其他数据结构在文件中的偏移量
* string_ids:字符串的偏移量
* type_ids:类型信息的偏移量
* proto_ids:方法声明的偏移量
* field_ids:字段信息的偏移量
* method_ids:方法信息（所在类，方法声明以及方法名）的偏移量
* class_def:类信息的偏移量
* data:数据区
* link_data:静态链接数据区

从header到data之间都是偏移量数组，并不存储真实数据，所有数据都存在data数据区，根据其偏移量区查找。[Android逆向笔记 —— DEX 文件格式解析](https://zhuanlan.zhihu.com/p/66800634)

**D8编译器与R8工具**

在AGP 3.X以后，google分别引入D8编译器和R8工具作为默认的DEX编辑器和混淆压缩工具。

* 在AGP3.0.1之后，D8编译器取代了Dx，用于将class文件打包成DEX，D8编译器编译更快、时间更短；DEX编译时占用内容更小；生成的dex文件大小更小；同时拥有相同或者是更好的运行时性能。
* 在AGP3.4.0之后，默认开启R8，R8是ProGuard的替代工具，用于代码的压缩（shrinking）和混淆（obfuscation）

在AGP3.4.0版本中，R8把desugaring、shrinking、obfuscating、optimizing和dexing都合并到一部执行。在AGP3.4.0版本以前比那一流程如下：

![](https://blog-1256965811.cos.ap-guangzhou.myqcloud.com/img/AGP3.4.0之前编译流程.webp)

在AGP3.4.0之后编译流程如下：

![](https://blog-1256965811.cos.ap-guangzhou.myqcloud.com/img/AGP3.4.0之后编译流程.webp)

### 生成APK包

在资源文件与代码文件都编译完成后，通过zipflinger命令将manifest、resources、dexassets等文件打包成一个压缩包，就是apk文件。

### zipalign 对齐处理

zipalign是一种归档对齐工具，可对Android应用（APK）文件提供重要的优化

zipalign会对apk中未压缩的数据进行4字节对齐，对齐的主要过程是将apk保重所有的资源文件距文件起始偏移为4字节的整数倍，对齐后就可以使用mmap函数读取文件，可以像读取内存一样为普通文件进行操作。如果没有4字节对齐，就必须显式的读取，这样比较缓慢并且会耗费额外的内存。

### 对apk进行签名

生成APK文件后，必须要对该APK文件进行签名，否则无法安装。比较熟知的工具是：jarsigner和apksigner

* jarsigner：由JDK提供的签名工具，只能进行v1签名，并且必须在为APK文件签名之后使用zipalign执行对齐操作
* apksigner：由google为Android提供的签名和签证工具。可以进行v1、v2、v3、v4签名；必须在为apk文件签名之前使用zipalign，如果在使用apksigner为APK签名之后做出进一步更改，签名便会失效

### 编译打包过程中的Task

```gradle
//aidl 转换aidl文件为java文件
> Task :app:compileDebugAidl

//生成BuildConfig文件
> Task :app:generateDebugBuildConfig

//获取gradle中配置的资源文件
> Task :app:generateDebugResValues

// merge资源文件，AAPT2 编译阶段
> Task :app:mergeDebugResources

// merge assets文件
> Task :app:mergeDebugAssets
> Task :app:compressDebugAssets

// merge所有的manifest文件
> Task :app:processDebugManifest

//生成R文件 AAPT2 链接阶段
> Task :app:processDebugResources

//编译kotlin文件
> Task :app:compileDebugKotlin

//javac 编译java文件
> Task :app:compileDebugJavaWithJavac

//转换class文件为dex文件
> Task :app:dexBuilderDebug

//打包成apk并签名
> Task :app:packageDebug
```

## 参考

[从构建工具看 Android APK 编译打包流程](https://mp.weixin.qq.com/s/6Cb6MqV9GQG60hBltss61A)