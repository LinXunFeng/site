---
title: iOS - Swift UITableView的scrollToRow的坑
date: 2017-09-12 09:16:39
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

今天鄙人使用SnapKit来布局cell，然后用scrollToRow来滚到底部就遇到了一个很奇葩的现象。
我设置了在键盘弹出后聊天消息列表会自动滚到底部。
1.随便输入一条消息，点发送后，在聊天消息列表中并没有滚到最新消息那一行。
2.退出键盘不做任何操作再打开键盘也是滚到刚才那里(即最新消息的上一条所在位置)
3.只有在退出键盘后把聊天消息列表的消息向上拉一点距离露出最新消息所在的cell之后，再点击才有用

+<!-- more -->

<The rest of contents | 余下全文>

## 简介
在tableView中，我们一般会用到scrollToRow这个来控制tableView滚到指定的某一行。一般写法如下所示
```swift
// MARK: 滚到底部
func scrollToBottom(animated: Bool = false) {
    if dataArr.count > 0 {
        tableView.scrollToRow(at: IndexPath(row: dataArr.count - 1, section: 0), at: .bottom, animated: animated)
    }
}
```
## 情况
今天鄙人使用SnapKit来布局cell，然后用scrollToRow来滚到底部就遇到了一个很奇葩的现象。
我设置了在键盘弹出后聊天消息列表会自动滚到底部。
1.随便输入一条消息，点发送后，在聊天消息列表中并没有滚到最新消息那一行。
2.退出键盘不做任何操作再打开键盘也是滚到刚才那里(即最新消息的上一条所在位置)
3.只有在退出键盘后把聊天消息列表的消息向上拉一点距离露出最新消息所在的cell之后，再点击才有用
![](/images/2017/09/iOS - Swift UITableView的scrollToRow的"坑"/1.gif)

## 分析
在无奈之下，经过了一步步的探索，终于发现了问题的所在
首先我们要了解一下scrollToRow执行后会调用哪些函数及顺序
**会调用这两个方法**
```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell
```
```swift
func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat
```
### 步骤一
我在 heightForRow 中写了具体的数据，也就是把高度写死，不再是动态获取。接着执行程序得到如下结果
比如我原本有10条数据，现在加入了一条后执行了scrollToRow，它会
1.先调用 heightForRow 11次，**即包括最新加入的那一条**
2.然后再调用 cellForRow
3.最后在调一次 heightForRow
后面的2和3是针对最新消息的
### 步骤二
我在 heightForRow 中不再写死高度，而是从模型数据中动态获取高度(高度是在cell布局后获取的，再赋值到模型数据中的cellHeight变量)
```
执行程序得到这个结果：调用 heightForRow 11次，然后就没了
```
好吧，问题就出现在对heightForRow的第11次调用，前10次都有返回具体的高度，而最后一次是0~。
### 结论
**现在清楚了，要想在调用 scrollToRow 到指定的那一行，前提条件是那一行的高度不能为0。**
所以在上面的情况中，发送完消息后，最新消息的cell的确是插入到了tableView，也有显示出来(后面我自己测的)，但就是无法滚到最新消息那一行，就是因为 heightForRow 返回的高度为0
在上面的情况中，向上拉一点距离露出cell后scrollToRow才有效就是因为此时heightForRow返回的高度不再为0
## 解决方案
按本人自身的情况来说，有两种解决方法
### 第一种
在传入的模型数据中给予明确计算出来的数值就好。
### 第二种
我使用SnapKit来自动布局cell的位置然后再来获取高度，这做法主要就是为了避免运算。所以我不选用第一种解决方法
好了，方法如下：
```swift
// dataArr是用来存放模型的数组
let indexPath = IndexPath(row: dataArr.count - 1, section: 0)
// 调用tableView的数据源办法
_ = self.tableView(tableView, cellForRowAt: indexPath)
```
在插入最新消息后，调用tableView的数据源方法来让它先对cell进行布局，这样就获取到了cell的高度，然后再执行 scrollToRow 就好了。
![完美](/images/2017/09/iOS - Swift UITableView的scrollToRow的"坑"/2.gif)