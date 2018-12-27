---
title: iOS - 视频采集详解
date: 2017-10-16 23:26:38
categories: "iOS"
tags:
  - iOS
  - Objective-C
---

<Excerpt in index | 首页摘要> 

[苹果官方文档-AVFoundation](https://developer.apple.com/library/content/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/04_MediaCapture.html#//apple_ref/doc/uid/TP40010188-CH5-SW2)

为了管理从相机或者麦克风等这样的设备捕获到的信息，我们需要输入对象(input)和输出对象(output)，并且使用一个会话(AVCaptureSession)来管理 input 和 output 之前的数据流

+<!-- more -->
<The rest of contents | 余下全文>

[苹果官方文档-AVFoundation](https://developer.apple.com/library/content/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/04_MediaCapture.html#//apple_ref/doc/uid/TP40010188-CH5-SW2)

为了管理从相机或者麦克风等这样的设备捕获到的信息，我们需要输入对象(input)和输出对象(output)，并且使用一个会话(AVCaptureSession)来管理 input 和 output 之前的数据流：

| 类名                         | 简介                        |
| -------------------------- | :------------------------ |
| AVCaptureDevice            | 输入设备，例如 摄像头 麦克风           |
| AVCaptureInput             | 输入端口 [使用其子类]              |
| AVCaptureOutput            | 设备输出 [使用其子类]，输出视频文件或者静态图像 |
| AVCaptureSession           | 管理输入到输出的数据流               |
| AVCaptureVideoPreviewLayer | 展示采集 预览View               |

如图，通过单个 session，也可以管理多个 input 和 output 对象之间的数据流，从而得到视频、静态图像和预览视图
![多个输入输出设备](linxunfeng.github.io/images/2017/10/iOS-视频采集详解/1.png)

如图，input 可以有一个或多个输入端口，output 也可以有一个或多个数据来源（如：一个 [AVCaptureMovieFileOutput](https://developer.apple.com/documentation/avfoundation/avcapturemoviefileoutput) 对象可以接收视频数据和音频数据）

当添加 input 和 output 到 session 中时，session 会自动建立起一个连接(AVCaptureConnection)。我们可以使用这个 connection 来设置从 input 或者 从 output 得到的数据的有效性，也可以用来监控在音频信道中功率的平均值和峰值。

![AVCaptureConnection](linxunfeng.github.io/images/2017/10/iOS-视频采集详解/2.png)

## 使用 Session 来管理数据流
创建一个 session 用来管理捕获到的数据，需要先将 inputs 和 outputs 添加到 session 中，当 session 执行 [startRunning] 方法后就会开始将数据流发送至 session，通过执行[stopRunning] 方法来结束数据流的发送。
```objc
AVCaptureSession *captureSession = [[AVCaptureSession alloc] init];

// 添加 inputs 和 outputs

[session startRunning];
```
在 [session startRunning] 之前我们需要进行一些基本的配置 (如：设备分辨率，添加输入输出对象等)
### 设置分辨率
```objc
// 设置分辨率 720P 标清
if ([captureSession canSetSessionPreset:AVCaptureSessionPreset1280x720]) {
    captureSession.sessionPreset = AVCaptureSessionPreset1280x720;
}
```
附苹果官方文档中可供配置的分辨率列表

![分辨率列表](linxunfeng.github.io/images/2017/10/iOS-视频采集详解/3.png)

其中高分辨率(AVCaptureSessionPresetHigh) 为默认值，会根据当前设备进行自适应，但是这样之后导出来的文件就会很大，一般情况下设置为标清(AVCaptureSessionPreset1280x720) 就可以了

### 输入对象

```objective-c
// 直接使用后置摄像头
AVCaptureDevice *videoDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
```

```objective-c
// 在这个方法中的 mediaType 有三个选项供我们使用
// AVMediaTypeVideo 视频
// AVMediaTypeAudio 音频
// AVMediaTypeMuxed 混合(视频 + 音频)
+ (nullable AVCaptureDevice *)defaultDeviceWithMediaType:(AVMediaType)mediaType;
```
但是这种方式只能获取到后置摄像头，如果想要获取前置摄像头，可使用
```objective-c
AVCaptureDevice *videoDevice;
NSArray *devices = [AVCaptureDevice devices];
for (AVCaptureDevice *device in devices) {
   if(device.position == AVCaptureDevicePositionFront) {
        // 前置摄像头
        videoDevice = device;
   }
}
```
```objective-c
// 通过设备获取输入对象
AVCaptureDeviceInput *videoInput = [AVCaptureDeviceInput deviceInputWithDevice:videoDevice error:nil];
// 给会话添加输入
if([captureSession canAddInput:videoInput]) {
    [captureSession addInput:videoInput];
}
```

### 输出对象



```objective-c
// 视频输出：设置视频原数据格式：YUV, RGB 
// 苹果不支持YUV的渲染，只支持RGB渲染，这意味着： YUV => RGB
AVCaptureVideoDataOutput *videoOutput = [[AVCaptureVideoDataOutput alloc] init];

// videoSettings: 设置视频原数据格式 YUV FULL
videoOutput.videoSettings = @{(NSString *)kCVPixelBufferPixelFormatTypeKey:@(kCVPixelFormatType_420YpCbCr8BiPlanarFullRange)};

// 设置代理：获取帧数据
// 队列：串行/并行，这里使用串行，保证数据顺序 
dispatch_queue_t queue = dispatch_queue_create("LinXunFengSerialQueue", DISPATCH_QUEUE_SERIAL);
[videoOutput setSampleBufferDelegate:self queue:queue];

// 给会话添加输出对象
if([captureSession canAddOutput:videoOutput]) {
    // 给会话添加输入输出就会自动建立起连接
    [captureSession addOutput:videoOutput];
}
```


在这里，输出对象可以设置帧率



```objective-c
// 帧率：1秒10帧就差不多比较流畅了
videoOutput.minFrameDuration = CMTimeMake(1, 10);
```
输出对象在设置视频原数据格式时使用 videoSettings 属性，需要赋值的类型是字典
格式有两种，一种是YUV，另一种是RGB（一般我们都使用YUV，因为体积比RGB小）



```objective-c
// key
kCVPixelBufferPixelFormatTypeKey 指定解码后的图像格式

// value
kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange  : YUV420 用于标清视频[420v]
kCVPixelFormatType_420YpCbCr8BiPlanarFullRange   : YUV422 用于高清视频[420f] 
kCVPixelFormatType_32BGRA : 输出的是BGRA的格式，适用于OpenGL和CoreImage

区别：
1、前两种是相机输出YUV格式，然后转成RGBA，最后一种是直接输出BGRA，然后转成RGBA;
2、420v 输出的视频格式为NV12；范围： (luma=[16,235] chroma=[16,240])
3、420f 输出的视频格式为NV12；范围： (luma=[0,255] chroma=[1,255])
```
### 预览图层
```objective-c
AVCaptureVideoPreviewLayer *previewLayer = [AVCaptureVideoPreviewLayer layerWithSession:captureSession];
previewLayer.frame = self.view.bounds;
[self.view.layer  addSublayer:previewLayer];
```
实时显示摄像头捕获到的图像，但不适用于滤镜渲染

### 代理方法
```objective-c
#pragma mark - AVCaptureVideoDataOutputSampleBufferDelegate
/*
 CMSampleBufferRef: 帧缓存数据，描述当前帧信息
 CMSampleBufferGetXXX : 获取帧缓存信息
 CMSampleBufferGetDuration : 获取当前帧播放时间
 CMSampleBufferGetImageBuffer : 获取当前帧图片信息
 */
// 获取帧数据
- (void)captureOutput:(AVCaptureOutput *)output didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer fromConnection:(AVCaptureConnection *)connection {
    // captureSession 会话如果没有强引用，这里不会得到执行
    
    NSLog(@"----- sampleBuffer ----- %@", sampleBuffer);
}
```
```objc
// 获取帧播放时间
CMTime duration = CMSampleBufferGetDuration(sampleBuffer);
```
在代理方法中，可以把 sampleBuffer 数据渲染出来去显示画面。适用于滤镜渲染
```objective-c
// 获取图片帧数据
CVImageBufferRef imageBuffer = CMSampleBufferGetImageBuffer(sampleBuffer);
CIImage *ciImage = [CIImage imageWithCVImageBuffer:imageBuffer];
UIImage *image = [UIImage imageWithCIImage:ciImage];

dispatch_async(dispatch_get_main_queue(), ^{
    self.imageView.image = image;
});
```
需要注意的是：代理方法中的所有动作所在队列都是在异步串行队列中，所以更新UI的操作需要回到主队列中进行！！

但是此时会发现，画面是向左旋转了90度，因为默认采集的视频是横屏的，需要我们进一步做调整。以下步骤添加在[session startRunning];之前即可，但是一定要在添加了 input 和 output之后～
```objective-c
// 获取输入与输出之间的连接
AVCaptureConnection *connection = [videoOutput connectionWithMediaType:AVMediaTypeVideo];
// 设置采集数据的方向
connection.videoOrientation = AVCaptureVideoOrientationPortrait;
// 设置镜像效果镜像
connection.videoMirrored = YES;
```

## Demo
[LXFAudioVideo](https://github.com/LinXunFeng/LXFAudioVideo)