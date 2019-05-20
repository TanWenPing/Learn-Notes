<!--
 * @Description: 
 * @Author: twp
 * @LastEditors: twp
 * @Date: 2019-05-20 15:57:06
 * @LastEditTime: 2019-05-20 16:34:54
 -->

# 二维码识别之Android完整编译Zbar

二维码扫描比较常用的是ZXing和Zbar两个库。ZXing是纯Java代码实现的，适用于Android平台；Zbar是C实现的，可以供很多语言和平台使用，比如Java、iOS平台、Android平台，Python等等。很明显Zbar的识别率和速度都是明显快于ZXing的，但是无奈那时候不会编译Zbar，只好下载了ZXing，但是由于当时技术能力不足，对于ZXing自定义剪切框也做不出来，只好下载了别人编译好的Zbar，可能由于别人修改了代码或者编译的不是很完整，后期bug层出，废了好大劲才完善好。

后来一直没有机会学习二维码扫描，直到前几天需要给我们平台的APP加上了二维码扫描功能，我决定使用ZBar，于是我完整的编译了一次，今天把这个过程记录下来，希望可以帮助到需要的同学。

    比如微信使用的是ZXing，但是我肯定的说他们修改了不少源码，而且有很多地方应该改成了jni实现，所以微信的识别速率和准确率是相当高的，不过今天我编译后的封装也是秒秒钟就可以识别。

因为Zbar是基于LGPL-2.1开源的，因此我基于LGPL-2.1协议，我把一个完整的项目源码和sample放到Github上了，提供直接调用zbar的识别byte[]数据的功能和调用相机识别二维码的功能： 
<https://github.com/yanzhenjie/android-zbar-sdk>

    特别声明：本文已经修复了zbar识别中文乱码的问题！！！

## 编译Zbar

    在正式编译之前要注意：编译Zbar需要先编译libiconv，编译libiconv需要linux环境，需要用到gcc。如果你没有linux环境也没有关系，我已经提供了编译好的libiconv。

其实在Zbar的官网也可以下载到他们已经编译好的so和jar，但是so文件他们只提供了armeabi、armeabi-v7a、x86平台： 
<https://sourceforge.net/projects/zbar/files/?source=navbar>

所以我就抛弃了提供的编译包，自己编译了，下面是步骤。

首先在Zbar的开源主页下载Zbar源码： 
https://github.com/ZBar/ZBar

顺便在开源主页点开android文件夹，发现编译Zbar需要libiconv，接下来下载libiconv： 
<http://www.gnu.org/software/libiconv>

对于libiconv我是下载的在2017-02-02时发布的最新版1.15。

## 一、编译libiconv

如果你没有linux环境编译libiconv，那么你可以在这里下载我已经编译好的libiconv1.15： 
<http://download.csdn.net/detail/yanzhenjie1003/9833225>，下好好文件后，你就可以直接跳过这一节，看下面Zbar和libiconv一起编译了。

如果你有linux环境可以编译libiconv，那么继续往下看。 
下载好libiconv后，进入libiconv文件夹，如果报权限错误进不去的话执行sudo chmod 777 -R libiconv就可以了： 
libiconv

进来后先执行：./configure，如果提示没权限那么执行：sudo chmod 777 configure，然后重新执行/.configure即可。

等./configure执行完后再执行make命令即可完成编译

编译时可能遇到以下错误： 
1、configure: error: no acceptable C compiler found in $PATH 
这个是说你没有安装gcc，安装gcc后再次执行未完成命令即可。

二、Zbar和libiconv一起编译
libiconv编译完成了，接下来把Zbar和libiconv放到一起，编译出我们需要的so文件。

把刚才编译好的libiconv放入我们项目的jni文件夹。
解压刚才下载好的Zbar，首先把Zbar的头文件所在文件夹zbar/include放入我们项目的jni文件夹下。
把Zbar对java的接口文件zbarjni.c放入我们项目的jni文件夹，zbrjni.c在zbar/java文件夹下。
把Zbar的核心库文件所在的文件夹zbar/zbar放到我们项目的jni文件夹下。
把Zbar编译时需要的Android.mk、Applicaiton.mk、config.h从zbar\android\jni下拷贝到我们项目的jni文件夹下。
此时我们项目的jni文件夹是这样的： 
zbar

理论上现在可以开始编译了吧，但是呢因为我们改动了zbar的文件夹结构，所以我们要对Android.mk进行改动，主要改的是文件夹路径和文件路径，修改后的Android.mk的内容如下：

MY_LOCAL_PATH := $(call my-dir)

# libiconv
include $(CLEAR_VARS)
LOCAL_PATH := $(MY_LOCAL_PATH)
LOCAL_MODULE := libiconv
LOCAL_CFLAGS := \
    -Wno-multichar \
    -D_ANDROID \
    -DLIBDIR="c" \
    -DBUILDING_LIBICONV \
    -DBUILDING_LIBCHARSET \
    -DIN_LIBRARY

LOCAL_SRC_FILES := \
    libiconv-1.15/lib/iconv.c \
    libiconv-1.15/libcharset/lib/localcharset.c \
    libiconv-1.15/lib/relocatable.c

LOCAL_C_INCLUDES := \
    $(LOCAL_PATH)/libiconv-1.15/include \
    $(LOCAL_PATH)/libiconv-1.15/libcharset \
    $(LOCAL_PATH)/libiconv-1.15/libcharset/include

include $(BUILD_SHARED_LIBRARY)

LOCAL_LDLIBS := -llog -lcharset

# -----------------------------------------------------

# libzbar
include $(CLEAR_VARS)
LOCAL_PATH := $(MY_LOCAL_PATH)
LOCAL_MODULE := zbar
LOCAL_SRC_FILES := \
            zbarjni.c \
            zbar/img_scanner.c \
            zbar/decoder.c \
            zbar/image.c \
            zbar/symbol.c \
            zbar/convert.c \
            zbar/config.c \
            zbar/scanner.c \
            zbar/error.c \
            zbar/refcnt.c \
            zbar/video.c \
            zbar/video/null.c \
            zbar/decoder/code128.c \
            zbar/decoder/code39.c \
            zbar/decoder/code93.c \
            zbar/decoder/codabar.c \
            zbar/decoder/databar.c \
            zbar/decoder/ean.c \
            zbar/decoder/i25.c \
            zbar/decoder/qr_finder.c \
            zbar/qrcode/bch15_5.c \
            zbar/qrcode/binarize.c \
            zbar/qrcode/isaac.c \
            zbar/qrcode/qrdec.c \
            zbar/qrcode/qrdectxt.c \
            zbar/qrcode/rs.c \
            zbar/qrcode/util.c

LOCAL_C_INCLUDES := \
            $(LOCAL_PATH)/include \
            $(LOCAL_PATH)/zbar \
            $(LOCAL_PATH)/libiconv-1.15/include

LOCAL_SHARED_LIBRARIES := libiconv

include $(BUILD_SHARED_LIBRARY)
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
然后在Application.mk中填写你要编译的平台，如果想全部编译：

APP_ABI := all
1
如果要指定编译某几个平台，把平台名称依次空格隔开写上即可：

APP_ABI := armeabi armeabi-v7a x86 x86_64 mips mips_64 arm64_v8a
1
此时我们用命令行进入项目的jni文件夹的父母路，比如一般jni情况下jni文件夹位于ProjectName/ModuleName/src/main/jni，那么我们就进入这个main，然后此时执行ndk-build进行编译。

如果提示没有ndk-build这个命令，那么你需要从<http://developer.android.com>下载ndk并且在电脑上配置PATH。

等ndk-build执行完后会在libs下生成所有平台的so文件夹，文件夹里面是需要的libiconv和zbar的so文件。

编译Zbar和libiconv时遇到的错误解决
编译过程中可能发现如下错误，请按照修改方案修改即可。

1、libiconv-1.15/jni/libcharset/lib/localcharset.c:51:24: error: langinfo.h: No such file or directory 
打开libiconv-1.15/libcharset/config.h文件，搜索#define HAVE_LANGINFO_CODESET，大概在14行，把这行注释了即可：

/* #define HAVE_LANGINFO_CODESET 1 */
1
2、…c undeclaired… 
打开libiconv-1.15/libcharset/lib/localcharset.c，搜索到函数get_charset_aliases()，大概在124行。

大概在195行左右，有一个int c;（没有的话你可以搜索int c;），把这个一行代码移动到get_charset_aliases()开头： 
移动之前： 
移动之前

移动之后： 
移动之后

zbar的jar包
现在so文件有了，剩下的就是怎么调用so中的函数来识别条码/二维码了，首先把zbar/java下在net.sourceforge.zbar包和里边的java文件拷贝到你的项目的java目录下，大概结构如下： 
这里写图片描述

当然你也像这样使用源码，也可以把这几个类打包成jar包。

调用Zbar识别二维码
现在全部都编译好了，jar文件也有了，我们可以调用jar中封装的方法来识别二维码了：

byte[] imageData = ...;

Image barcode = new Image(size.width, size.height, "Y800");
barcode.setData(imageData);
// 指定二维码在图片中的区域，也可以不指定，识别全图。
// barcode.setCrop(startX, startY, width, height);

String qrCodeString = null;

int result = mImageScanner.scanImage(barcode);
if (result != 0) {
    SymbolSet symSet = mImageScanner.getResults();
    for (Symbol sym : symSet)
        qrCodeString = sym.getData();
}

if (!TextUtils.isEmpty(qrCodeString)) {
    // 成功识别二维码，qrCodeString就是数据。
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
如何和相机结合使用等复杂操作这里不再说了，一个完整的项目我放到Github上了： 
https://github.com/yanzhenjie/android-zbar-sdk

山高水远，江湖再见！