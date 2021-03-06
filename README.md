# FU Live Demo PC

FULivePC 是 Faceunity 的面部跟踪和虚拟道具功能在PC中的集成，作为一款集成示例。
## 目录
[新特性](#sdk-v52-更新)
[SDK内容](#运行环境)
[集成说明](#集成方法)
[道具失效等问题](#faq)

## SDK v6.0.0 更新内容

更新内容
- 优化人脸检测，提高检测率，提高性能。
- 新增质感美颜功能（注：道具支持SDK v6.0.0以上版本）。
- 人脸融合(海报换脸)效果优化（注：道具支持 SDK v6.0.0以上版本）。
- 背景分割分割精度优化（注：此版本背景分割、手势识别道具只支持 SDK v6.0.0以上版本）。
- 舌头跟踪trackface逻辑支持，Getfaceinfo支持。
- 新增Avatar捏脸功能，需FUEditor 6.0.0以上版本。
- 美颜滤镜优化（注：原有滤镜整合，重命名归类及效果新增， 道具支持SDK v5.5.0以上版本）。
- 修复mebedtls符号冲突问题。
- 注：美发、Animoji道具支持FUEditor v5.6.0以上制作版本，其余道具在任意SDK皆可兼容

更新文档
- [美颜道具参数说明_完整版](docs/美颜道具参数说明_完整版.pdf)
- [美妆bundle参数说明](docs/美妆bundle参数说明.pdf)
- [质感美颜参数说明](docs/质感美颜参数说明.pdf)

## 运行环境

本SDK目前我们提供了32位和64位双平台的库文件

- [SDK]内部的API执行前要保证初始化好OpenGL环境，【确保】fuSetup等API调用时OpenGL context是可用的。

- [Demo]为了演示SDK功能我们制作了Demo程序，界面绘制使用imgui。如果需要使用QT，可切换到dev_qt分支进行参考集成。
注：SDK仅包含下述文件列表内容里的\*.dll,\*.lib和\*.h文件，它是不依赖任何界面库的，如需更换其他界面库如MFC或者不具备编译环境等，可选择不编译此Demo，直接参考代码将SDK的dll等[集成](#jump2)到您的新工程。
## 文件说明

### 一、库文件

  - funama.h 函数调用接口头文件
  - Win32/Win64 库文件、
  - assets 数据文件

### 二、数据文件

- v3.bundle 初始化必须的数据模型

- face_beautification.bundle 我司美颜道具

- anim_model.bundle 表情优化数据模型

- ardata_ex.bundle 高精度数据模型

- tongue.bundle 舌头驱动数据模型

- fxaa.bundle 3D道具去锯齿道具

- items文件夹下的 *.bundle 文件是我司制作的特效贴纸文件，自定义特效贴纸制作的文档和工具请联系我司获取。

  注：这些数据文件都是二进制数据，与扩展名无关。实际在app中使用时，打包在程序内或者从网络接口下载这些数据都是可行的，只要在相应的函数接口传入正确的文件路径即可。

## SDK接入指引

### 初始化

#### 导入证书

您需要拥有我司颁发的证书才能使用我们的SDK的功能，获取证书方法：

  - 1、拨打电话 **0571-89774660**
  - 2、发送邮件至 **marketing@faceunity.com** 进行咨询。

PC端发放的证书为authpack.h文件，如果您已经获取到鉴权证书，将证书文件覆盖工程中include下的authpack.h文件即可。根据应用需求，鉴权数据也可以在运行时提供(如网络下载)，不过要注意证书泄露风险，防止证书被滥用。

#### 初始化SDK

首先把nama.lib链接到工程中，并确保nama.dll在运行时可以正确加载。包含funama.h之后就可以开始调用我们提供的接口函数。

首先初始化OpenGL环境。

接着对 faceunity 环境进行初始化。其中 v3data 是人脸跟踪数据的指针，指向 v3.bundle 文件中的内容； g_auth_package 为密钥数组，该密钥定义在我司提供的证书文件authpack.h中，没有该密钥会导致初始化失败。

```C
fuSetup(v3data, v3Size, nullptr, g_auth_package, sizeof(g_auth_package));
```

### 道具创建、销毁与切换

#### 道具创建

之后进行道具加载，函数返回的道具 handle 要保存在当前程序中，之后绘制道具时需要提供该 handle 作为道具的标识。其中 data 为指向道具文件内容的指针，size 为该文件的字节大小。

```C
static int mItemsArray[2] = {0, 0};
...
if (!mItemsArray[0]){
  mItemsArray[0] = fuCreateItemFromPackage(data, size);
}
```

#### 道具销毁

##### 销毁单个道具

```c
void fuDestroyItem(int item);
```

参数说明：

`item ` 要销毁的道具句柄

该接口将释放传入的句柄所对应的资源。示例如下：

```c
if (mItemsArray[0] > 0)
    fuDestroyItem(mItemsArray[0]);
```

##### 销毁全部道具

```c
void fuDestroyAllItems();
```

该接口可以销毁全部道具句柄所对应的资源,同样在执行完该接口后请将所有句柄都置为0。示例如下：

```java
fuDestroyAllItems();
```

#### 道具切换
如果需要切换句柄数组中某一位的句柄时，需要先创建一个新的道具句柄，并将该句柄替换到句柄数组中需要被替换的位置上，最后再把被替换的句柄销毁掉。下面以替换句柄数组的第ITEM_ARRAYS_EFFECT位为例进行说明：

```c
int newEffectItem = loadItem(path);
if (mItemsArray[0] > 0) {
    fuDestroyItem(mItemsArray[0]);
}
mItemsArray[0] = newEffectItem;
```
### 视频处理

之后就可以调用绘制函数将已经加载的道具绘制到图像数据中，如下例所示，提供图像数据的指针、宽度m_frameWidth、高度m_frameHeight、当前运行帧号、道具 handle 列表、以及绘制道具数量等信息后，完成绘制操作。其中，输入和输出图像数据是布局为BGRA、类型为unsigned char的数据。

```C
fuRenderItems(0, img, m_frameWidth, m_frameHeight, g_frame_id++, mItemsArray, 1);
```

## 视频美颜
美颜功能实现步骤与道具类似，首先加载美颜道具，并将fuCreateItemFromPackage返回的美颜道具handle保存下来:

```C
mItemsArray[1] = fuCreateItemFromPackage(g_res_zip, (int)g_res_size);
```

之后，将该handle和其他需要绘制的道具一起传入绘制接口即可。注意 fuRenderItems() 最后一个参数为所绘制的道具数量，这里以一个普通道具和一个美颜道具一起绘制为例。加载美颜道具后不需设置任何参数，即可启用默认设置的美颜的效果。

```C
fuRenderItems(0, img, m_frameWidth, m_frameHeight, g_frame_id, mItemsArray, 2);
```

### 参数设置
美颜道具主要包含七个模块的内容：滤镜、美白、红润、磨皮、亮眼、美牙、美型。每个模块都有默认效果，它们可以调节的参数如下。新版参数可参考 ./docs/美颜道具参数说明_完整版.pdf

### 一、滤镜

目前版本中提供以下滤镜：

普通滤镜：

```C
"origin", "delta", "electric", "slowlived", "tokyo", "warm"
```

美颜滤镜：

```C
"ziran", "danya", "fennen", "qingxin", "hongrun"
```

其中 "origin" 为原图滤镜，其他滤镜属于风格化滤镜及美颜滤镜，美颜滤镜具有一定美颜、增白、亮唇等功能。滤镜由参数 filter_name 指定。切换滤镜时，通过 fuItemSetParams 设置美颜道具的参数，如下：

```C
fuItemSetParamd(m_beautyHandles, "filter_name", "origin");
```

另外滤镜开放了滤镜强度接口，可以通过参数 filter_level 来控制当前滤镜程度。该参数的取值范围为[0, 1]，0为无效果，1.0为默认效果。客户端需要针对每个滤镜记录用户的选择的filter_level，当切换滤镜时，设置该参数。

```C
fuItemSetParamd(m_beautyHandles, "filter_level", mFaceBeautyFilterLevel);
```

### 二、美白和红润

#### 美白

通过参数 color_level 来控制美白程度。该参数的推荐取值范围为0~1，0为无效果，0.5为默认效果，大于1为继续增强效果。

设置参数的例子代码如下：

```C
fuItemSetParamd(m_beautyHandles, "color_level", mFaceBeautyColorLevel);
```

#### 红润

通过参数 red_level 来控制红润程度。该参数的推荐取值范围为0~1，0为无效果，0.5为默认效果，大于1为继续增强效果。

```C
fuItemSetParamd(m_beautyHandles, "red_level", mFaceBeautyRedLevel);
```

注: 新增的美颜滤镜如 “shaonv”滤镜本身能够美白肤色，提亮红唇，开启该滤镜时，适当减弱独立的美白红润功能。

### 三、磨皮

新版美颜中，控制磨皮的参数有五个：blur_level，skin_detect，nonshin_blur_scale，heavy_blur，blur_blend_ratio。

`blur_level` 指定磨皮程度。该参数的推荐取值范围为0.0~6.0，0.0为无效果，原则上不建议参数值大于6.0，不过如果超过6.0也将会继续加大磨皮效果。

`skin_detect`  指定是否开启皮肤检测，开启后，将自动检测是否皮肤，是皮肤的区域将直接根据blur_level指定的磨皮程度进行磨皮，非皮肤区域将减轻磨皮导致模糊的效果。该参数的推荐取值为0-1，0为无效果，1为开启皮肤检测，默认不开启。

`nonshin_blur_scale` 指定开启皮肤检测后，非皮肤区域减轻磨皮导致模糊的程度。该参数范围是[0.0,1.0]，0表示不磨皮，1表示完全磨皮，默认值为0.45。调整该参数需要先开启 skin_detect。

__新增朦胧美肤:__

`heavy_blur` 指定是否开启朦胧美肤功能。大于1开启朦胧美肤功能。

`blur_blend_ratio` 指定磨皮结果和原图融合率。该参数的推荐取值范围为0-1。

注意：朦胧美肤使用了比较强的模糊算法，优点是会把皮肤磨得更加光滑，瑕疵更少，而且性能比老版磨皮更好，缺点是会降低一些清晰度。另外开启朦胧美肤后blur_level，skin_detect两个参数继续有效，而 nonshin_blur_scale 参数对朦胧美肤无效

设置参数的例子代码如下：

```C
fuItemSetParamd(m_beautyHandles, "skin_detect", mFaceBeautyALLBlurLevel);
fuItemSetParamd(m_beautyHandles, "heavy_blur", mFaceBeautyType);
fuItemSetParamd(m_beautyHandles, "blur_level", 6 * mFaceBeautyBlurLevel);
fuItemSetParamd(m_beautyHandles, "blur_blend_ratio", 0.5);
fuItemSetParamd(m_beautyHandles, "nonshin_blur_scale", 0.45);
```

### 四、亮眼

使眼睛区域的纹理变得更加清晰，眼眸更加明亮。可通过参数 eye_bright 来控制亮眼程度。该参数的推荐取值范围为0～1，0为关闭该功能，0到1效果逐渐增强。

设置参数的例子代码如下：

```C
fuItemSetParamd(m_beautyHandles, "eye_bright", mBrightEyesLevel);
```

### 五、美牙

使牙齿区域变得更亮更白。可通过参数 tooth_whiten 来控制美牙程度。该参数的推荐取值范围为0～1，0为关闭该功能，0到1效果逐渐增强。

设置参数的例子代码如下：

```C
fuItemSetParamd(m_beautyHandles, "tooth_whiten", mBeautyTeethLevel);
```

### 六、美型

#### 1、基本美型

美型支持四种基本美型：女神、网红、自然、默认，一种高级美型：自定义。由参数 face_shape 指定：默认（3）、女神（0）、网红（1）、自然（2）、自定义（4）。

```C
//  Set item parameters - shaping
fuItemSetParamd(mItemsArray[1], "face_shape", 3);
```
在上述四种基本美型及一种高级美型的基础上，我们提供了以下三个参数：face_shape_level、eye_enlarging、cheek_thinning。

参数 face_shape_level 用以控制变化到指定基础脸型的程度。该参数的取值范围为[0, 1]。0为无效果，即关闭美型，1为指定脸型。

若要关闭美型，可将 face_shape_level 设置为0。

```C
//  Set item parameters - shaping level
fuItemSetParamd(mItemsArray[1], "face_shape_level", 1.0);
```
参数 eye_enlarging 用以控制眼睛大小。此参数受参数 face_shape_level 影响。该参数的推荐取值范围为[0, 1]。大于1为继续增强效果。
```C
//  Set item parameters - shaping
fuItemSetParamd(mItemsArray[1], "eye_enlarging", 1.0);
```
参数 cheek_thinning 用以控制脸大小。此参数受参数 face_shape_level 影响。该参数的推荐取值范围为[0, 1]。大于1为继续增强效果。
```C
//  Set item parameters - shaping
fuItemSetParamd(mItemsArray[1], "cheek_thinning", 1.0);
```



#### #### 2、高级美型

##### 精细脸型调整功能

新增优化瘦脸、大眼的效果，增加额头调整、下巴调整、瘦鼻、嘴型调整4项美颜变形，将 face_shape 设为4即可开启精细脸型调整功能，FULiveDemo中可以在脸型中选择自定义来开启精细脸型调整功能

__使用方法__：

- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型

##### 瘦脸

优化瘦脸变形效果，比之前更加自然

__使用方法__：

- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  cheek_thinning: 0.0,   // 使用了原有参数cheek_thinning控制瘦脸 ，范围0 - 1

##### 大眼

优化大眼变形效果，比之前更加自然

__使用方法__：

- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  eye_enlarging: 0.0,   // 使用了原有参数eye_enlarging控制大眼，范围0 - 1

##### 额头调整

新增加的一款美颜变形，可以调整额头大小

__使用方法__：

- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  intensity_forehead: 0.5,   // 大于0.5 变大，小于0.5变小

##### 下巴调整

新增加的一款美颜变形，可以调整下巴大小

__使用方法__：

- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  intensity_chin: 0.5,   // 大于0.5 变大，小于0.5变小

##### 瘦鼻

新增加的一款美颜变形，可以进行瘦鼻操作

__使用方法__：

- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  intensity_nose: 0.0,   // 0为正常大小，大于0开始瘦鼻，范围0 - 1

##### 嘴型调整

新增加的一款美颜变形，可以调整嘴型大小

__使用方法__：

- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  intensity_mouth: 0.5,   // 大于0.5变大，小于0.5变小

### 七、美颜美型突变过渡效果

使美颜变形过度的更自然，避免突变效果，可通过参数 change_frames 来控制渐变所需要的帧数，0 渐变关闭 ，大于0开启渐变，值为渐变所需要的帧数。

设置参数的例子代码如下：

```
fuItemSetParamd(mItemsArray[1], "change_frames", 10);
```
## 手势识别
目前我们的手势识别功能也是以道具的形式进行加载的。一个手势识别的道具中包含了要识别的手势、识别到该手势时触发的动效、及控制脚本。加载该道具的过程和加载普通道具、美颜道具的方法一致。

线上例子中 heart_v2.bundle 为爱心手势演示道具。将其作为道具加载进行绘制即可启用手势识别功能。手势识别道具可以和普通道具及美颜共存，类似美颜将 mItemsArray 扩展为三个并在最后加载手势道具即可。

自定义手势道具的流程和2D道具制作一致，具体打包的细节可以联系我司技术支持。
## 3D绘制抗锯齿功能

高效全屏抗锯齿，使得3D绘制效果更加平滑。

__使用方法__：

- 加载fxaa.bundle，随新版本SDK提供
- 绘制时将fxaa.bundle放在道具数组最后一个

```
mItemsArray[ITEM_ARRAYS_EFFECT_ANIMOJI_3D] = fuCreateItemFromPackage(g_res_zip, (int)g_res_size);
```

## 照片驱动功能

针对照片进行精确的人脸重建，然后支持实时表情驱动，预置表情播放。可以用于实时应用，也可以用于生成表情包等。

该功能的资源有两种方式生成方式：

- 使用FUEditor v4.3.0以上版本离线制作道具
- 利用相芯提供的云服务在线上传照片生成道具
  在线云服务的方式请联系技术支持获取更多细节。

__使用方法__：

- 直接加载对应的道具
- 需要带有照片驱动权限的证书

## 人脸夸张变形功能

新增了5款夸张变形。

__使用方法__：

- 直接加载对应的道具
- 需要带有照片驱动权限的证书

## 音乐滤镜

音乐滤镜是使用播放音乐的时间戳进行驱动的，在每次处理图像前，将音乐的播放进度传入音乐滤镜道具即可，方式如下：

```c
fuItemSetParam(mItemsArray[0], "music_time", time); 
```

如果没有音乐则可以模拟音乐播放进度，demo中提供的道具对应的音乐时长为28s，换算成ms为28000ms，在没有音乐的情况下，可以从加载音乐滤镜开始计时，每次处理图像前获取一下当前时间与开始加载音乐滤镜的时间差，转换成ms传入音乐滤镜即可，当时间差超过28000ms时归0重新开始计时即可。效果详见FULiveDemo，道具可以通过FUEditor进行制作（v4.2.1及以上）。

## 优化表情校准功能

增加被动校准模式，将之前的表情校准定义为主动校准模式。

- 被动校准：该种模式下会在整个用户使用过程中逐渐进行表情校准，用户对该过程没有明显感觉。该种校准的强度相比主动校准较弱。
- 主动校准：老版本的表情校准模式。该种模式下系统会进行快速集中的表情校准，一般为初次识别到人脸之后的2-3秒钟。在该段时间内，需要用户尽量保持无表情状态，该过程结束后再开始使用。该过程的开始和结束可以通过 ```fuGetFaceInfo``` 接口获取参数 ```is_calibrating```。

__使用方法__：

- 调用 ```fuSetExpressionCalibration``` 接口控制表情校准功能的开关及不同模式，参数为0时关闭表情校准，1为主动校准，2为被动校准。

  注：优化后的SDK只支持被动校准功能，即fuSetExpressionCalibration接口只支持0（关闭）或2（被动校准）这两个数字，设置为1时将不再有效果。

## 动漫滤镜+Animoji的AR模式

**动漫滤镜：**一款动漫风格的滤镜，使用方式与普通道具一致，只需要加载并保存在道具具柄数组中传入视频处理接口即可。Demo使用的场景是配合Animoji道具的AR模式一起使用，不过动漫滤镜并不局限于和Animoji的AR模式一起使用，你可以单独使用，也可以配合其他任何道具一起使用。需要注意的是，客户端需要根据当前OpenGL ES版本，设置动漫滤镜的"glVer"属性。当使用OpenGL ES 3.0版本才能较好的使用动漫滤镜效果，如果使用3.0以下版本的GL环境，效果会稍微差一些。设置方法如下：

```c
fuItemSetParam(itemHandle, "glVer", 3); // 2代表兼容模式，3代表正常模式。
```

**Animoji的AR模式：**与Animoji普通模式不同，AR模式会显示真实的场景，且Animoji形象会跟随人脸移动，有了更多的互动性。如果配合动漫滤镜使用Animoji的AR模式，可以使Animoji形象与真实场景的风格更加一致，使融合更加自然。Animoji开启AR模式的方式如下：

```java
// 设置 Animoji 跟随人脸
fuItemSetParam(itemHandle, "{\"thing\":\"<global>\",\"param\":\"follow\"}", 1); // value为1代表开启，value为0代表关闭。
```

## 舌头驱动

Nama SDK 从5.6.0开始支持舌头驱动功能，使用具有舌头特效的道具时，需要先加载驱动舌头的数据模型，加载方式如下：

```c
fuLoadTongueModel(tongueData,tongueDataSize);
```

## 海报换脸

Nama SDK 从5.8.0开始支持新版海报换脸功能，该功能可实现将用户的脸完美的融合到海报中模特的脸上，实现换脸功能。

首先需要加载change_face.bundle道具，然后参考参考[海报换脸接口文档](docs/海报换脸接口文档.md)，同时也可以参考我们Demo中的实际使用方法。

## 人脸美妆

Nama SDK 从5.8.0开始支持新版的人脸美妆功能，该功能可实现口红、腮红、眉毛、眼影、眼线、睫毛、美瞳等功能，同时支持精细调整。

首先需要加载face_makeup.bundle道具，然后参考[美妆bundle参数说明](docs/美妆bundle参数说明.pdf)进行开发，同时也可以参考我们Demo中的实际使用方法。

## 美发功能

Nama SDK 从5.8.0开始支持美发功能，该功能可以改变人物头发颜色，目前支持8种普通发色及5种渐变色，同时调节美发程度。

### 普通发色

首先加载 hair_color.bundle，然后使用参数 Index 来切换发色，该参数的推荐取值范围为0~7。通过参数 Strength 可以调节发色的强度，该参数的推荐取值范围为0~1。示例：

```c
/**设置美发参数**/
fuItemSetParam(mItemsArray[ITEM_ARRAYS_EFFECT_HAIR_NORMAL_INDEX], "Index", mHairColorIndex);
                    fuItemSetParam(mItemsArray[ITEM_ARRAYS_EFFECT_HAIR_NORMAL_INDEX], "Strength", mHairColorStrength);
```

### 渐变色

首先加载 hair_gradient.bundle，然后使用参数 Index 来切换发色，该参数的推荐取值范围为0~4。通过参数 Strength 可以调节发色的强度，该参数的推荐取值范围为0~1。示例：

```java
/**设置美发参数**/
cfuItemSetParam(mItemsArray[ITEM_ARRAYS_EFFECT_HAIR_GRADIENT_INDEX], "Index", mHairColorIndex);
                    fuItemSetParam(mItemsArray[ITEM_ARRAYS_EFFECT_HAIR_GRADIENT_INDEX], "Strength", mHairColorStrength);
```
## 新增功能介绍

### 质感美颜功能

#### 功能概述

质感美颜方案是一套更为精致高效的美颜解决方案，包含磨皮、美型、滤镜、美妆4大模块，提供60+套丰富素材库，支持客户任意切换风格与效果变化。 

#### 接口说明

#### 备注

### Avatar捏脸功能

#### 功能概述

虚拟角色支持捏脸功能，通过使用FUEditor 6.0.0模板facepup生成的bundle，在客户端调节对应参数，可以实现用户对脸型，发型，唇色等部位自定义，生成和保存用户想要的虚拟形象的功能。且该虚拟形象可以通过引擎实时根据人脸驱动，做出和人脸同步的表情。

#### 接口说明

加载制作好的捏脸道具后，可以设置以下参数进行捏脸。

##### 进入捏脸状态

```c
//进入捏脸状态
fuItemSetParamd(1,"enter_facepup",1)
```

##### 清除全部捏脸参数

```c
//清除全部捏脸参数
fuItemSetParamd(1,"clear_facepup",1)
```

##### 捏脸

```c
//设置第一个捏脸属性“大眼”的权值，范围[0-1]。这里param对应的就是第几个捏脸属性，从1开始。
fuItemSetParamd(1,"\{\"name\":\"facepup\",\"param\":\"1\"\}",1);

//设置第二个捏脸属性“尖下巴”的权值，范围[0-1]。这里param对应的就是第几个捏脸属性，从1开始。
fuItemSetParamd(1,"\{\"name\":\"facepup\",\"param\":\"2\"\}",0.5);
```

##### 保存或退出捏脸

二选一即可

```c
//1. 直接退出捏脸状态，不保存当前捏脸状态，进入跟踪状态。使用上一次捏脸，进行人脸表情跟踪。
fuItemSetParamd(1,"quit_facepup",1)
//2. 触发保存捏脸，并退出捏脸状态，进入跟踪状态。耗时操作，必要时设置。
fuItemSetParamd(1,"need_recompute_facepup",1);
```

#### 备注

具体资源制作要求，详询技术支持。

## 鉴权

我们的系统通过标准TLS证书进行鉴权。客户在使用时先从发证机构申请证书，之后将证书数据写在客户端代码中，客户端运行时发回我司服务器进行验证。在证书有效期内，可以正常使用库函数所提供的各种功能。没有证书或者证书失效等鉴权失败的情况会限制库函数的功能，在开始运行一段时间后自动终止。

证书类型分为**两种**，分别为**发证机构证书**和**终端用户证书**。

#### - 发证机构证书
**适用对象**：此类证书适合需批量生成终端证书的机构或公司，比如软件代理商，大客户等。

发证机构的二级CA证书必须由我司颁发，具体流程如下。

1. 机构生成私钥
机构调用以下命令在本地生成私钥 CERT_NAME.key ，其中 CERT_NAME 为机构名称。
```
openssl ecparam -name prime256v1 -genkey -out CERT_NAME.key
```

2. 机构根据私钥生成证书签发请求
机构根据本地生成的私钥，调用以下命令生成证书签发请求 CERT_NAME.csr 。在生成证书签发请求的过程中注意在 Common Name 字段中填写机构的正式名称。
```
openssl req -new -sha256 -key CERT_NAME.key -out CERT_NAME.csr
```

3. 将证书签发请求发回我司颁发机构证书

之后发证机构就可以独立进行终端用户的证书发行工作，不再需要我司的配合。

如果需要在终端用户证书有效期内终止证书，可以由机构自行用OpenSSL吊销，然后生成pem格式的吊销列表文件发给我们。例如如果要吊销先前误发的 "bad_client.crt"，可以如下操作：
```
openssl ca -config ca.conf -revoke bad_client.crt -keyfile CERT_NAME.key -cert CERT_NAME.crt
openssl ca -config ca.conf -gencrl -keyfile CERT_NAME.key -cert CERT_NAME.crt -out CERT_NAME.crl.pem
```
然后将生成的 CERT_NAME.crl.pem 发回给我司。

#### - 终端用户证书
**适用对象**：直接的终端证书使用者。比如，直接客户或个人等。

终端用户由我司或者其他发证机构颁发证书，并通过我司的证书工具生成一个代码头文件交给用户。该文件中是一个常量数组，内容是加密之后的证书数据，形式如下。
```
static char g_auth_package[]={ ... }
```
## FAQ

## 为什么过了一段时间人脸识别失效了？

检查证书。如证书是否正确使用，是否过期。您需要拥有我司颁发的证书才能使用我们的SDK的功能，获取证书方法：1、拨打电话 **0571-88069272** 2、发送邮件至 **marketing@com** 进行咨询。

用户在库环境初始化时，需要提供该数组进行鉴权，具体参考 fuSetup 接口。没有证书、证书失效、网络连接失败等情况下，会造成鉴权失败，在控制台或者Android平台的log里面打出 "not authenticated" 信息，并在运行一段时间后停止渲染道具。

任何其他关于授权问题，请email：support@com
