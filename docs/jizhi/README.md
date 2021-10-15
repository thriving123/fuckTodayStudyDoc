# 事件机制

## 事件机制概念
QAP-SDK 支持 3 种类型的事件机制：千牛系统级事件、插件级事件、页面级事件

### 千牛系统级事件
千牛系统级事件，是指作为范围为整个千牛 App 的事件，用于监听客户端的变化以做出响应。千牛系统级事件的名称都必须以 Global. 作为前缀。**千牛系统级事件只能监听，不能触发！**

如：

```
QN.on('Global.DidEnterBackground', data => {
    console.log('千牛应用进入了系统后台');
});
```

### 插件级事件
插件级事件，是指作用范围限定在当前插件内所有的页面范围内的事件，通常用于实现页面间通信。插件级事件的监听和触发，事件名称都必须以 App. 作为前缀，来限定作用范围。

如：

```
// 页面 A (注册事件)
QN.on('App.hello', data => {
    console.log(data.msg); // 'I am Page B'
});

// 页面 B （触发事件）
QN.emit('App.hello', {
    msg: 'I am Page B'
});
```

### 页面级事件
页面级事件，是指作用范围限定在当前页面范围内的事件，通常用于实现模块间通信。页面级事件的监听和触发，事件名称都必须以 Page. 作为前缀，来限定作用范围。

如：

```
// 模块 A(注册事件)
QN.on('Page.hello', data => {
    console.log(data.msg); // 'I am mobule B'
});

// 模块 B （触发事件）
QN.emit('Page.hello', {
    msg: 'I am mobule B'
});
```

> Tips: 如果事件不含上述的前缀，则默认为页面级别事件，前缀为 Page.。即事件 hello 和 Page.hello 等同。



## 如何使用事件

### 监听事件
绑定、监听一个事件，参考API：QN.on
[API地址链接](http://open.taobao.com/doc.htm?spm=a219a.7386653.0.0.3c5e669aVlkzpd#?treeId=428&docId=107273&docType=1&platformId=61)

调用示例

```
QN.on('Page.hello', (data) => {
    console.log(data);
});

// 如果需要关注事件注册的结果是成功还是失败
QN.on('App.hello', (data) => {
    console.log(data);
})
.then(result => {
    console.log('注册成功');
})
.catch(error => {
    console.log('注册失败');
});
```
### 触发事件
触发一个事件，参考API: QN.emit
[API地址链接](http://open.taobao.com/doc.htm?spm=a219a.7386653.0.0.3c5e669aVlkzpd#?treeId=428&docId=107267&docType=1&platformId=61)

调用示例

```
let data = {msg: 'msg from Page.hello'};
// 仅触发事件
QN.emit('Page.hello');

// 触发事件，并发送数据
QN.emit('Page.hello', data);

// 触发粘性事件
QN.emit('Page.hello', {sticky: true});

// 触发粘性事件，并发送数据
QN.emit('Page.hello', data, {sticky: true});

// 触发事件，并希望得知触发是否成功
QN.emit('Page.hello')
.then(result => {
    console.log('触发成功');
})
.catch(error => {
    console.log('触发失败');
});
```

### 取消监听事件
解绑、注销一个事件，参考API: QN.off
[API地址链接](http://open.taobao.com/doc.htm?spm=a219a.7386653.0.0.3c5e669aVlkzpd#?treeId=428&docId=107274&docType=1&platformId=61)

调用示例

```
// 注销事件时传递对应的回调函数。此时只会注销该回调函数，其他的事件回调函数不受影响。
QN.off('Page.hello', (data) => {
    console.log(data);
});
// 注销事件时不传递事件回调函数，此时会注销所有该事件下的回调函数。
QN.off('Page.hello');

// 如果需要关注注销事件的结果是成功还是失败
QN.off('App.hello', (data) => {
    console.log(data);
})
.then(result => {
    console.log('注销成功');
})
.catch(error => {
    console.log('注销失败');
});
```


## 千牛系统级事件

千牛系统级事件是指作为范围为整个千牛 App 的事件，用于监听客户端的变化以做出响应，比如利用这些事件，您可以对键盘事件、应用进入前/后台事件进行监听处理。

千牛系统级事件的名称都以 Global. 作为前缀。 千牛系统级事件只能监听，不能触发！，

如：

```
QN.on('Global.DidEnterBackground', data => {
    console.log('千牛应用进入了系统后台');
});
```

目前，千牛提供了以下系统级事件：

事件名称 | 	所属模块 | 	含义 | 备注
:--- | :---: | :---: | :---
Global.LowMemory | 	千牛应用 | 	千牛进入低内存状态	
Global.DidBecomeActive | 	千牛应用	 | 千牛应用进入前台，处于活跃状态	
Global.DidEnterBackground | 	千牛应用 | 	千牛应用进入后台	
Global.KeyboardWillShow	 | 系统键盘 | 	键盘将要显示	
Global.KeyboardDidShow | 	系统键盘	 | 键盘已经显示	
Global.KeyboardWillHide	 | 系统键盘 | 	键盘将要隐藏	
Global.KeyboardDidHide	 | 系统键盘	 | 键盘已经隐藏	
Global.KeyboardWillChangeFrame | 	系统键盘 | 键盘外框架大小将要变化更新 | 键盘的显示和隐藏都会影响键盘外框架大小变化，因此会同时触发此事件
Global.KeyboardDidChageFrame | 	系统键盘 | 键盘外框架大小已经变化更新 | 键盘的显示和隐藏都会影响键盘外框架大小变化，因此会同时触发此事件


## 插件级事件

插件级事件，是指作用范围限定在当前插件内所有的页面范围内的事件，通常用于实现页面间通信。
插件级事件的监听和触发，事件名称都必须以 App. 作为前缀，来限定作用范围。

如：

```
// 页面 A
QN.on('App.hello', data => {
    console.log(data.msg); // 'I am Page B'
});

// 页面 B
QN.emit('App.hello', {
    msg: 'I am Page B'
});
```

## 页面级事件

页面级事件，是指作用范围限定在当前页面范围内的事件，通常用于实现模块间通信。

页面级事件的监听和触发，事件名称都必须以 Page. 作为前缀，来限定作用范围。

如：

```
// 模块 A
QN.on('Page.hello', data => {
    console.log(data.msg); // 'I am mobule B'
});

// 模块 B
QN.emit('Page.hello', {
    msg: 'I am mobule B'
});

```

千牛默认内置了以下页面级事件：

为了便于开发者响应千牛应用、插件应用、页面的变化，千牛客户端提供了一系列内置事件。

事件名称 | 所属模块 | 	含义 | 备注
:--- | :---: | :---: | :---
Page.back | 	页面导航栏	 | 返回按钮点击事件	
Page.close	 | 页面导航栏 | 	关闭按钮点击事件	
Page.reload | 	页面导航栏	 | 刷新按钮点击事件	
Page.WillAppear | 	页面	 | 页面即将可见	
Page.DidAppear	 | 页面 | 	页面已经可见	
Page.WillDisappear | 	页面 | 	页面即将不可见 | 可能的情况：被千牛的其他页面遮挡，如扫码；被其他应用遮挡，如拍照；用户按了Home千牛将被最小化。
Page.DidDisappear | 	页面 | 	页面已经不可见