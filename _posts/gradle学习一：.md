

### gradlew学习一：

根目录下

build.gradle

setting.gradle



Setting.gradle告诉我们有几个模块，也就是moudle,对应Setting.java类,继承 PluginAware

build.gradle 对应project类，也继承PluginAware

```
void apply(Map<String, ?> options);
```

```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript { 
    repositories {           
        jcenter()            仓库地址
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.0'
        classpath 'com.qihoo360.replugin:replugin-plugin-gradle:2.2.4'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```

buildscript方法是定义了全局的相关属性，repositories定义了jcenter作为仓库。一个仓库代表着你的依赖包的来源，例如maven仓库。dependencies用来定义构建过程。这意味着你不应该在该方法体内定义子模块的依赖包，你仅仅需要定义默认的Android插件就可以了，因为该插件可以让你执行相关Android的tasks。 帮助你执行各种构建



repositories 和 dependencies来自ScriptHandler



allprojects方法可以用来定义各个模块的默认属性，你可以不仅仅局限于默认的配置，未来你可以自己创造tasks在allprojects方法体内，这些tasks将会在所有模块中可见。 



```
task clean(type: Delete) {
    delete rootProject.buildDir
}
```



初始化阶段，setting include 

配置阶段 分析project依赖关系，下载依赖文件，分析task依赖   dependsOn执行

执行阶段 先后执行task                   dofirst,dolast



几类task

android 

build

help

install

other

verification

task哪里来的    通过gradle依赖，通过apply插件

dependencies {
        classpath 'com.android.tools.build:gradle:2.2.0'
        classpath 'com.qihoo360.replugin:replugin-plugin-gradle:2.2.4'
    }

apply plugin: 'com.android.application'







app下

build.gradle

from是一个拷贝任务，代表引入 public AbstractCopyTask from(Object... sourcePaths) 

```
apply from: "config.gradle"              
```

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 25                 //编译版本, 在你本地的sdk路径下的platform下
    buildToolsVersion "25.0.2"          //3.0以上可以不用指定，里面包含一些列工具aapt,aidl,dx等
    defaultConfig {                        //定义app的核心属性
        applicationId "com.xx.xxx"          //包名
        minSdkVersion 16                  //程序适配的最小版本，低于这个版本安装不上
        targetSdkVersion 25               //程序安装在手机上的版本，不会随着手机系统升级程序就变成了新的版本，除非你在一次升级后改了这个（要做大量的测试兼容）
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
		multiDexEnabled  true           64位
		javaCompileOptions {
            annotationProcessorOptions {
                includeCompileClasspath = true
                arguments = [moduleName: project.getName()]
                includeCompileClasspath false
            }
        }    
        ndk {                              //arm过滤
            abiFilters "armeabi", "armeabi-v7a", "x86","arm64-v8a"
        }
    }
    buildTypes {
        release {    
            minifyEnabled false                      //是否开启混淆
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            buildConfigField "boolean", "IS_DEBUG", "true"
        }
    }
    packagingOptions {          //去除重复
           
    }
    compileOptions{           //编译配置
          
    }
    repositories {             //资源位置
    	flatDir {
        dirs 'libs'
    }
	}
	 lintOptions {           //忽略lint
        abortOnError false
    }
}
```

该文件的第一行是Android应用插件，该插件我们在上一篇博客已经介绍过，其是google的Android开发团队编写的插件，能够提供所有关于Android应用和依赖库的构建，打包和测试。 

仅依赖aar

```
implementation(name: 'xx', ext: 'aar')
    api 'com.android.support:recyclerview-v7:27.1.1'        //recyclerview
    api 'com.android.support:appcompat-v7:27.1.1'		
    implementation project(':xx')
```



## AppExtension

```
DefaultDomainObjectSet<ApplicationVariant> applicationVariantList
```



```
android.applicationVariants.all { variant ->
    variant.outputs.each { output ->
        if (variant.buildType.name.equals('release')) {
            def file = output.outputFile
            output.outputFile = new File(file.parent, "baoming_" + rootProject.ext.android.versionName + versionCode + ".apk")
        }
    }
}
```

buildToolsVersion  3.0以上会自动指定合适的，可以不用指定，然后3.0each替换为all，也不能new File了





defaultconfig的属性

```
   multiDexEnabled true 分包
   javaCompileOptions
   
   
   其他
   buildTypes
   packagingOptions
   repositories
   
   compileOptions
   aaptOptions.cruncherEnabled = false
   dexOptions {
        javaMaxHeapSize "4G"
   }
   lintOptions {//编译过程先不进行lint检查
        abortOnError false
   }
```





Task的使用

##### Android tasks 

androidDependencies - Displays the Android dependencies of the project.
signingReport - Displays the signing info for each variant.
sourceSets - Prints out all the source sets defined in this project.





##### Build tasks



assemble - Assembles all variants of all applications and secondary packages.
assembleAndroidTest - Assembles all the Test applications.
assembleDebug - Assembles all Debug builds.
assembleRelease - Assembles all Release builds.
build - Assembles and tests this project.
buildDependents - Assembles and tests this project and all projects that depend on it.
buildNeeded - Assembles and tests this project and all projects it depends on.
clean - Deletes the build directory.
compileDebugAndroidTestSources
compileDebugSources
compileDebugUnitTestSources
compileReleaseSources
compileReleaseUnitTestSources
extractDebugAnnotations - Extracts Android annotations for the debug variant into the archive file
extractReleaseAnnotations - Extracts Android annotations for the release variant into the archive file
mockableAndroidJar - Creates a version of android.jar that's suitable for unit tests.





##### Build Setup tasks

init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]





##### Help tasks

buildEnvironment - Displays all buildscript dependencies declared in root project 'PhoneFC'.
components - Displays the components produced by root project 'PhoneFC'. [incubating]
dependencies - Displays all dependencies declared in root project 'PhoneFC'.
dependencyInsight - Displays the insight into a specific dependency in root project 'PhoneFC'.
help - Displays a help message.
model - Displays the configuration model of root project 'PhoneFC'. [incubating]
projects - Displays the sub-projects of root project 'PhoneFC'.
properties - Displays the properties of root project 'PhoneFC'.
tasks - Displays the tasks runnable from root project 'PhoneFC' (some of the displayed tasks may belong to subprojects).





等等。



gradle 一些特殊配置

```
useLibrary 'org.apache.http.legacy'   

compileOptions {
        targetCompatibility JavaVersion.VERSION_1_8
        sourceCompatibility JavaVersion.VERSION_1_8
   }
testInstrumentationRunner rootProject.ext.dependencies["androidJUnitRunner"]
minifyEnabled false     混淆开启
shrinkResources true    删除无用资源，配合混淆开启
zipAlignEnabled true 	混淆后zipalign优化
lintOptions {
        disable 'InvalidPackage'
        disable "ResourceType"
        abortOnError false
    }
```



