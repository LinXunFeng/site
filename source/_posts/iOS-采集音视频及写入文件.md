---
title: iOS - 采集音视频及写入文件
date: 2017-09-26 11:50:32
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

音视频采集包括两部分：视频采集和音频采集。在iOS中可以同步采集视频与音频，通过系统框架AVFoundation，可以帮助我们采集音频与视频，对于视频还可以进行切换前后摄像头，最终我们将录制好的视频写入沙盒中

+<!-- more -->

<The rest of contents | 余下全文>

> 音视频采集包括两部分：视频采集和音频采集。在iOS中可以同步采集视频与音频，通过系统框架AVFoundation，可以帮助我们采集音频与视频，对于视频还可以进行切换前后摄像头，最终我们将录制好的视频写入沙盒中

![目录](https://linxunfeng.github.io/images/2017/09/iOS-采集音视频及写入文件/1.gif)

## 音视频数据的采集与展示
### 一、初始化视频的输入与输出
``` swift
// 懒加载一个session，所有的操作都需要session来执行
fileprivate lazy var session: AVCaptureSession = AVCaptureSession()
// 保存视频输出
fileprivate var videoOutput: AVCaptureVideoDataOutput?
// 保存视频输入
fileprivate var videoInput: AVCaptureDeviceInput?
// 保存预览图层
fileprivate var previewLayer: AVCaptureVideoPreviewLayer?
```
设置视频输入源与输出源
```swift
// 设置视频输入源
guard let devices = AVCaptureDevice.devices() as? [AVCaptureDevice] else { return }
// 获取我们的前置摄像头(后置为.back)
guard let device = devices.filter({ $0.position == .front }).first else { return }
guard let input = try? AVCaptureDeviceInput(device: device) else { return }
self.videoInput = input

// 设置视频输出源
let output = AVCaptureVideoDataOutput()
let queue = DispatchQueue.global()
// 设置代理，并在代理中获取采集到的数据，需要遵守 AVCaptureVideoDataOutputSampleBufferDelegate
output.setSampleBufferDelegate(self, queue: queue)
self.videoOutput = output
```
设置音频的输入源与输出源
```swift
// 设置音频的输入源
guard let device = AVCaptureDevice.defaultDevice(withMediaType: AVMediaTypeAudio) else { return }
guard let input = try? AVCaptureDeviceInput(device: device) else {return}

// 设置音频输出源
let output = AVCaptureAudioDataOutput()
let queue = DispatchQueue.global()
// 需要遵守 AVCaptureAudioDataOutputSampleBufferDelegate
output.setSampleBufferDelegate(self, queue: queue)
```
添加音频与视频的输入与输出到session中，但是每次添加之前需要先判断是否可以添加

``` swift
// 添加输入与输出

// 注意：每次对session进行设置之前都需要调用session的【beginConfiguration】方法
// 来告诉系统你现在要开始进行配置，结束配置后再调用【commitConfiguration】方法来提交配置
session.beginConfiguration()
if session.canAddInput(input) {
    session.addInput(input)
}
if session.canAddOutput(output) {
    session.addOutput(output)
}
session.commitConfiguration()
```
### 二、实现音视频的采集代理
音视频虽然需要遵守的代理名称不一样，但是需要实现的方法是一致的，所以要拿到音频或者视频就得先进行判断，需要用到AVCaptureOutput的这个方法
```
// This convenience method returns the first AVCaptureConnection in the receiver's
// connections array that has an AVCaptureInputPort of the specified mediaType. If 
// no connection with the specified mediaType is found, nil is returned.

open func connection(withMediaType mediaType: String!) -> AVCaptureConnection!
```
```swift
extension ViewController: AVCaptureVideoDataOutputSampleBufferDelegate, AVCaptureAudioDataOutputSampleBufferDelegate {
    func captureOutput(_ captureOutput: AVCaptureOutput!, didOutputSampleBuffer sampleBuffer: CMSampleBuffer!, from connection: AVCaptureConnection!) {
        if videoOutput?.connection(withMediaType: AVMediaTypeVideo) == connection {
            print("视频数据")
        } else {
            print("音频数据")
        }
    }
}
```
### 三、初始化一个预览图层用来显示采集到的视频（非采集所必须的步骤）
```swift
// 创建预览图层
guard let previewLayer = AVCaptureVideoPreviewLayer(session: session) else {return}
previewLayer.frame = view.bounds

// 将图层添加到控制器的view的layer中
view.layer.insertSublayer(previewLayer, at: 0)
self.previewLayer = previewLayer
```

现在基本功能都有了，如果想要开始采集音视频只需要调用
```swift
// 开始录制
session.startRunning()
// 结束录制
session.stopRunning()
```

### 切换镜头
其实就是换掉当前的视频输入法制，这里的过程跟上面的设置输入源一样。
```swift
// 1.取出之前镜头的方向
guard let videoInput = videoInput else { return }
let position: AVCaptureDevicePosition = videoInput.device.position == .front ? .back : .front

guard let devices = AVCaptureDevice.devices() as? [AVCaptureDevice] else { return }
guard let device = devices.filter({ $0.position == position }).first else { return }
guard let newInput = try? AVCaptureDeviceInput(device: device) else { return }

// 2.移除之前的input，添加新的input
session.beginConfiguration()
session.removeInput(videoInput)
if session.canAddInput(newInput) {
    session.addInput(newInput)
}
session.commitConfiguration()

// 3.保存最新的input
self.videoInput = newInput
```

## 录制视频写入文件
```swift
fileprivate var movieOutput: AVCaptureMovieFileOutput?
```
> 在开始采集音视频的时候就要开始写入文件

``` swift
// 开始写入文件 

// 1、创建写入文件的输出
let fileOutput = AVCaptureMovieFileOutput()
self.movieOutput = fileOutput // 保存起来，用于停止写入文件

// 设置类型，不然报错(这两句很重要)
let connection = fileOutput.connection(withMediaType: AVMediaTypeVideo)
connection?.automaticallyAdjustsVideoMirroring = true

if session.canAddOutput(fileOutput) {
    session.addOutput(fileOutput)
}

// 2、直接开始写入文件
let filePath = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true).first! + "/abc.mp4"
let fileUrl = URL(fileURLWithPath: filePath)
fileOutput.startRecording(toOutputFileURL: fileUrl, recordingDelegate: self)
```
> 在停止采集音视频的时候停止写入文件

```
// 停止写入文件
movieOutput?.stopRecording()
```

详情请看 [DEMO](https://github.com/LinXunFeng/LXFRecordAndWriteMediaDemo)