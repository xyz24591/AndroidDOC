# APK瘦身
## 1. 下表为Apk目录及文件说明：
* assets/	存放一些静态文件，可以通过AssertManager访问
* lib/	如果该目录存在，一般存放的是NDK编译出来的so
* META-INF/	保存着APK的签名信息
* res/	资源文件所在目录，包含drawable、layout等
* AndroidManifest.xml	程序全局配置文件
* classes.dex	Java Class，被DEX编译后可供Dalvik/ART虚拟机所理解的文件格式
* resources.arsc	编译后生成的二进制资源文件

## 2. 分析APK本身各部分所占大小，来确定好瘦身的主要方向
*　build -> Analyze APK 

一元会购： 分析版本 OneBuy-2.2-beta6.apk
1. res : 占 72.3% 13.1MB
2. classes.dex : 占 19.2% 3.5MB 
3. lib : 占 3.3% 615.2KB

主要的瘦身方向： 
1. 代码部分：冗余代码、无用功能、代码混淆、方法数缩减等；
2. 资源部分：冗余资源、资源混淆、图片处理等；
3. 对So文件的处理等。

## 4. 针对shareSDK 和 appkefu 冗余图片优化
* 两个第三方工程当中涉及的图片，名称不变，将图片内容变成 1 * 1的透明像素点图片

## 5. 删除未使用到xml和图片
如何知道哪些xml和图片未被使用到？使用Android Studio的Lint，步骤：点击菜单栏 Analyze -> Run Inspection by Name -> unused resources -> Moudule ‘app’ -> OK，这样会搜出来哪些未被使用到未使用到xml和图片

## 5. 使用webp格式
对于 res 文件夹，通常占空间最大的就是图片了。如果你的 Android Studio 为 2.3，并且项目的 minSdkVersion 为 18 或以上，应该使用 webp 而不是 png 图片。webp 图片有更小的体积，图片质量还没有什么损失。

我们可以选中 drawable 和 mipmap 文件夹，右键后选择 convert to webp，将图片转为 webp 格式。



## 5. 用使用TinyPng优化Android的资源图片
* 地址 https://tinypng.com/
* 需要注意的是：
1. tinypng支持png和jpg图片的压缩，并且也支持9图的压缩。
2. tinypng的缺点是在压缩某些带有过渡效果（带alpha值）的图片时，图片会失真。
3. tinypng提供了开放接口供开发者开发属于自己的压缩工具，不过这是付费服务，对于普通用户来说，tinypng为每个用户提供的每月图片免费压缩数量已经足够了。

## 6. 使用pngquant来压缩png资源缩小apk 
* 使用pngquant来压缩png资源缩小apk http://www.cnblogs.com/soaringEveryday/p/5148881.html  （待测试 ）
* Android Pngquant图片压缩 http://www.jianshu.com/p/0bf09aea6446
* 9.png的图片压缩不了，会有问题。Android studio会报错

## 7. 删除未使用到代码

* 同样使用Android Studio的Lint，步骤：点击菜单栏 Analyze -> Run Inspection by Name -> unused declaration -> Moudule ‘app’ -> OK
但清理效果不太明显

## 8. resConfig和lib
配置resConfigs

如果APP支持中文，可以配置resConfigs，只支持中文

	android {
  	...
    defaultConfig {
      ...
        resConfigs  "en","fr"

        ndk{
        //设置支持的SO库架构
        abiFilters 'armeabi','x86','armeabi-v7a','x86_64','arm64-v8a'
        }
    	}
    }

## 9. AndResGuard资源混淆 
* http://www.jianshu.com/p/7ffea26c9fd8

build.gradle 中 具体配置：

		apply plugin: 'AndResGuard'
		buildscript {
    		repositories {
        		jcenter()
    		}
    	dependencies {
        	classpath 'com.android.tools.build:gradle:2.1.0'
        	classpath 'com.tencent.mm:AndResGuard-gradle-plugin:1.2.3'
    		}
		}
	
    	andResGuard {
    	// mappingFile = file("./resource_mapping.txt")
    	//mappingFile用于增量更新，保持本次混淆与上次混淆结果一致；
    	mappingFile = null
    	//uss7zip为true时，useSign必须为true；
    	use7zip = true
    	//useSign为true时，需要配置signConfig；
    	useSign = true
   	 	//打开这个开关，会keep住所有资源的原始路径，只混淆资源的名字;
    	keepRoot = false


   		 //whiteList添加在代码内部需要动态获取的资源id，不混淆这部分；
    	whiteList = [
            // for your icon
            "R.drawable.icon",
            // for fabric
            "R.string.com.crashlytics.*",
            // for umeng update
            "R.string.umeng*",
            "R.string.UM*",
            "R.string.tb_*",
            "R.layout.umeng*",
            "R.layout.tb_*",
            "R.drawable.umeng*",
            "R.drawable.tb_*",
            "R.anim.umeng*",
            "R.color.umeng*",
            "R.color.tb_*",
            "R.style.*UM*",
            "R.style.umeng*",
            "R.id.umeng*",
            // umeng share for sina
            "R.drawable.sina*",
            // for google-services.json
            "R.string.google_app_id",
            "R.string.gcm_defaultSenderId",
            "R.string.default_web_client_id",
            "R.string.ga_trackingId",
            "R.string.firebase_database_url",
            "R.string.google_api_key",
            "R.string.google_crash_reporting_api_key",
            // umeng share for facebook
            "R.layout.*facebook*",
            "R.id.*facebook*",
            // umeng share for messager
            "R.layout.*messager*",
            "R.id.*messager*",
            // umeng share commond
            "R.id.progress_bar_parent",
            "R.id.webView",
	    // 将raw文件夹下的文件id放入白名单当中
            "R.raw.jhpaylibs_1_43",
            "R.raw.payass"
    	]
    	//用来指定文件重打包时是否压缩指定文件;
    	compressFilePattern = [
            "*.png",
            "*.jpg",
            "*.jpeg",
            "*.gif",
            "resources.arsc"
    	]
   		//sevenzip可使用artifacr或path，path指本地安装的7za（7zip命令行工具）。
    	sevenzip {
        	artifact = 'com.tencent.mm:SevenZip:1.2.0'
        	//path = "/usr/local/bin/7za"
    		}
		}	

## 10. 如何批量复制文件名称
https://zhidao.baidu.com/question/152591096.html



1.D盘下pdf文件夹下的一些文件，我想要将他们的文件名称整理到一个文本文件内，手动完成还是算了吧，太麻烦。依照下面的方法就可以了。<br>
2.首先我们要打开“命令提示符（管理员）”。如果你使用的是Win8.1系统，可以再开始按钮上“右击”的方法，选择“命令提示符（管理员）”。<br>
3.接着我们就要输入批量复制一个文件夹内的文件名称的命令了：
tree d:/pdf /f>d:pdf/pdf.txt<br>
4.上述命令中，“tree”是命令词，这个不变。“d:/pdf”是你要复制文件所在的文件夹，我的是D盘下的pdf文件夹。“/f”是“tree”命令的一个参数，也是不变的。“>”大于号相当于个“输出”的意思吧，输出到后面那个文本文件文件。“d:pdf/pdf.txt”是输出文件的路径与文件名。我的是在D盘的pdf文件夹下的pdf.txt文件。<br>
5.也就是说，上述命令要更改的只是你的输入文件夹与输出文本文件位置与名称。输入完成后回车，非常快速的，就会得到你的文本文件。<br>
6.打开这个文本文件pdf.txt，里面是我的“pdf”文件夹内文件的名称与格式。如果有子目录，其子目录内的文件名称同样会写入进来。<br>








