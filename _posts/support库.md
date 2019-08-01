一个Android工程通常包括主工程和依赖包，依赖包又有两种形式：

- 一种是单独的工程或者aar：在主工程的配置文件中指明主工程和依赖包的依赖关系之后，就可以在主工程中正常使用依赖包的类和接口了，这种适合于依赖包中有图片资源、so等不能打包到jar包中或者依赖包需要频繁改动的情况，比如[Nine Old Androids](https://link.jianshu.com?t=https://github.com/JakeWharton/NineOldAndroids)、[PullToRefresh](https://link.jianshu.com?t=https://github.com/chrisbanes/Android-PullToRefresh)、[FancyCoverFlow](https://link.jianshu.com?t=https://github.com/davidschreiber/FancyCoverFlow)等；
- 另一种是jar包：放在主工程的libs文件夹下，这种通常是依赖包中只有代码，比如[Fastjson.jar](https://link.jianshu.com?t=https://github.com/alibaba/fastjson)、[Volley.jar](https://link.jianshu.com?t=https://android.googlesource.com/platform/frameworks/volley)、[Gson.jar](https://link.jianshu.com?t=https://github.com/google/gson)等。

为了程序能够编译通过并在设备中正常运行，主工程除了依赖第三方的工程和jar包之外，还需要依赖安卓系统本身的代码，也就是我们在sdk的每个版本中看到的android.jar，这里面集成了android的所有API，随着android sdk的升级，高版本的sdk中会增加很多新的API，比如ActionBar、Fragment、RecyclerView等，如果在低版本的sdk中需要使用高版本新增的API怎么办？不可能去更新移动设备中的android.jar吧，因为硬件设备集成的sdk版本是固定的，android.jar也是固定的，所以最好的方式是将新增的API以依赖包的形式集成到需要使用高版本API的应用程序中。



 许多情况下，某项功能可能对应用开发者很有用，但是添加到 Android 框架却并不合适。例如，某个应用可能仅需要用于特定用例的某项功能，如在不同版本的 Android 系统之间顺畅切换。 

支持库提供一系列不同的功能：

- [向后兼容](https://developer.android.google.cn/topic/libraries/support-library/#backward)版本的框架组件。   v4,v7
- 用于实现建议的 Android [布局模式](https://developer.android.google.cn/topic/libraries/support-library/#layout-patterns)的 UI 元素。 design
- 支持不同的[设备类型](https://developer.android.google.cn/topic/libraries/support-library/#form-factors)。
- 其他[实用程序](https://developer.android.google.cn/topic/libraries/support-library/#utils)功能。











 我们一个个来分析

 这个包是为Android 2.3(API版本为9)及以上的版本设计的(Support V4首次发布是在2011年，它支持的最低版本是Android 1.6即API Level 4，V4的名字也是根据其支持的最低API版本来的，随着系统的迭代Android 1.6的设备已经很少了，官方在Support Library 24.2.0版本的时候移除了对Android 2.2(API Level 8)及以下版本的支持，所以从Android Support Library 24.2.0开始，V4包支持的最低版本是Android 2.3即API Level 9)，它包含大部分高版本中有而低版本中没有的API，包括application components、user interface features、accessibility、data handling、network connectivity、programming utilities，下面是对V4中的一些关键API的介绍：



 

 在Android Support Library 24.2.0及之后的版本中，为了增强效率和减小APK的大小起见，Android将V4包从一个独立的依赖包拆分成v4 compat library、v4 core-utils library、v4 core-ui library、v4 media-compat library和v4 fragment library这5个包，考虑到V4的向后兼容，你在工程中依赖V4这个依赖包时默认是包含拆分后的5个包的，但为了节省APK大小，建议在开发过程中根据实际情况依赖对应的V4包，移除不必要的V4包。

 

####  v4 compat library

兼容一些 Framework API，如 Context.getDrawable() 和 View.performAccessibilityAction()，大小为 602k，在AS中的依赖方式如下：

```
    compile 'com.android.support:support-compat:24.2.1'
```

#### v4 core-utils library

提供一系列核心的工具类，如 AsyncTaskLoader 和 PermissionChecker，大小为 90k，在AS中的依赖方式如下：

```
    compile 'com.android.support:support-core-utils:24.2.1'
```

#### v4 core-ui library

提供一系列核心的 UI，如 ViewPager、 NestedScrollView，大小为 240k，在AS中的依赖方式如下：

```
    compile 'com.android.support:support-core-ui:24.2.1'
```

#### v4 media-compat library

android.media 兼容库，包括 MediaBrowser 和 MediaSession，大小为 248k，在AS中的依赖方式如下：

```
    compile 'com.android.support:support-media-compat:24.2.1'
```

#### v4 fragment library

跟fragment相关部分，大小为 136k。V4这个子库依赖了其他4个子库，所以我们一旦依赖这个库就会自动导入其他4个子库，这跟直接依赖整个support-v4效果类似，在AS中的依赖方式如下：

```
compile 'com.android.support:support-fragment:24.2.1'
```

拆包并不一定代表能够真的解决问题，V4各子包的依赖关系如下，可见即使拆包之后，要用到V4中的某个API时，依赖包并没有减小多少：



 ![support_2](D:\githubdata\pic\support_2.png)

 

 当运行到类库中的类时，会做校验，比如在高版本机器上，用到fragment内容时就会调用框架自带的内容，而非支持库的内容，这些判断通常以...Compat结尾

```
static {
    if (Build.VERSION.SDK_INT >= 18) { // JellyBean MR2
        IMPL = new AccessibilityServiceInfoJellyBeanMr2Impl();
    } else if (Build.VERSION.SDK_INT >= 16) { // JB
        IMPL = new AccessibilityServiceInfoJellyBeanImpl();
    } else if (Build.VERSION.SDK_INT >= 14) { // ICS
        IMPL = new AccessibilityServiceInfoIcsImpl();
    } else {
        IMPL = new AccessibilityServiceInfoStubImpl();
    }
}
```

 



#### Dalvik 可执行文件分包支持库

 com.android.support:multidex:1.0.0  

 

 

V7和V4一样，同样包含多个依赖包，但和V4不同的是，V7下的多个子包并不是后面拆分开来的，而是最初发布时就以各个独立库的形式发布的。它是针对Android 2.3(API Level 9)及以上的版本谷歌提供了一系列的support包（和V4包的命名一样，V7最初支持的最低版本是Android 2.1即API Level 7，所以称其为V7，同样在Android Support Library 24.2.0将V7支持的最低版本改为Android 2.3即API Level 9了），这些support包各自对应着特定的功能，每一个都可以单独地被引用。

#### v7 appcompat library

这个包支持对Action Bar接口的设计模式、Material Design接口的实现等，核心类有ActionBar、AppCompatActivity、AppCompatDialog、ShareActionProvider等，在AS中的依赖方式如下：

```
    compile 'com.android.support:appcompat-v7:24.2.1'
```

**注意：**这个包需要依赖android-support-v4。

 

####  v7 cardview library

支持cardview控件，使用Material Design语言设计，卡片式的信息展示，在电视App中有广泛的使用，在AS中的依赖方式如下：

```
    compile 'com.android.support:cardview-v7:24.2.1'
```

 

####  v7 gridlayout library

一个支持GridLayout布局的support包，在AS中的依赖方式如下：

```
    com.android.support:gridlayout-v7:24.2.1
```

#### v7 mediarouter library

一个用于设备间音频、视频交换显示的support包，在AS中的依赖方式如下：

```
    com.android.support:mediarouter-v7:24.2.1
```

####  v7 palette library

该库提供了palette类，使用这个类可以很方便提取出图片中主题色。比如在音乐App中，从音乐专辑封面图片中提取出专辑封面图片的主题色，然后将播放界面的背景色设置为封面的主题色，随着播放音乐的改变，播放界面的背景色也会巧妙的跟着改变，从而提供更好的用户体验。，在AS中的依赖方式如下：

```
    com.android.support:palette-v7:24.2.1
```

#### v7 recyclerview library

核心类是RecyclerView，用于替换ListView、GridView，具体可以查阅RecyclerView方面的资料，在AS中的依赖方式如下：

```
    com.android.support:recyclerview-v7:24.2.1
```

#### v7 Preference Support Library

一个用于支持各种控件存储配置数据的support包，比如CheckBoxPreference和ListPreference，在AS中的依赖方式如下：

```
    com.android.support:preference-v7:24.2.1
```

 



v8支持库，还有v13,v14等等

 

####  Annotations Support Library

一个支持注解的support包，在AS中的依赖方式如下：

```
    compile 'com.android.support:support-annotations:24.2.1'
```

 

#### Design Support Library

一个用于支持Design Patterns的support包，它提供了Material Desgin设计风格的控件，在AS中的依赖方式如下：

```
    com.android.support:design:24.2.1
```

 

 

 

 

 一些大更新

6.0

​	运行时权限

 	取消支持 Apache HTTP 客户端

5.0 

​	Material Design 样式 

 

 4.4 

  	添加全屏沉浸模式

 

 support库版本号如何配置合理

 



![suport_3](D:\githubdata\pic\suport_3.png)

 

 

为什么要这么配置呢，这个是为了解决冲突，如果版本不一致，一些依赖会有很多子依赖，如果使用了不同的版本，有可能导致冲突。

 app-task-help-dependencies 查看项目里的依赖关系

依赖关系如图



![support4](D:\githubdata\pic\support4.png)



可以看到rxandroid1.1.0内部用了rxjava1.1.0,但由于同一种依赖可升级，（都是compile)，所以被升级成了adapter-rxjava2.0.2内部的rxjava1.1.1,同理okhttp也是这样







compileSdkVersion       : 27,   代码编译版本

buildToolsVersion       : "27.0.3",

minSdkVersion           : 14,

targetSdkVersion        : 27,

 

compileSdkVersion     编译时版本   最理想的情况是把编译版本提高，这样可以为新的API做好准备   和support大版本要一致。

buildToolVersion   高版本去构建低版本SDK     路径android -sdk tool -sdk build tool看最高版本

minSdkVersion   程序运行的最低版本，会对代码引入的一些包做检验，lint 

targetSdkVersion    

具体点就是随着 Android 系统的升级，某个系统的 API 或者模块的行为可能会发生改变，但是为了保证老 APK 的行为还是和以前兼容。只要 APK 的 targetSdkVersion 不变，即使这个 APK 安装在新 Android 系统上，其行为还是保持老的系统上的行为，这样就保证了系统对老应用的前向兼容性。 

安装的时候是多少，即使系统升级还是多少版本号。如何要适配新版本就要发布新的target包

原则上

 minSdkVersion <= targetSdkVersion <= compileSdkVersion 

 

 

 