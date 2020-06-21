debugout.js
===========

(debug output) 会从您的日志中生成一个文本文件，可以对其进行搜索，添加时间戳，进行下载等。 [请参见下面的一些示例。](#outputting).

Debugout's `log()` 接受任何类型的对象，包括函数。 Debugout 不是一个猴子补丁，而是一个单独的日志记录类，您可以使用它来代替 `console`.

debugout 的一些要点:

- 获取整个日志，或者在运行时或任何时间获取日志的最后几行
- 搜索并切片日志
- 通过可选的时间戳更好地了解使用模式
- 在一处切换实时日志记录（console.log）
- （可选）将输出存储在其中，window.localStorage并在每个会话中连续添加到同一日志中
- （可选）将日志限制为最近的X行以限制内存消耗

## [Try the demo](http://inorganik.github.io/debugout.js/)

### 安装

只需将debugout.js文件包括在您的项目中，或通过包管理器进行安装：

npm: `npm install debugout.js`

bower: `bower install debugout.js`

### 用法

在脚本顶部的全局名称空间中创建一个新的debugout对象，并将所有控制台日志方法替换为debugout的log方法：

```js
var bugout = new debugout();

// 代替 console.log('some object or string')
bugout.log('some object or string');
```
无论您记录什么，都将保存并在新行中将其添加到日志文件中。

### 方法

- `log()` - 像 `console.log()` 一样, 但已保存!
- `getLog()` - 返回整个日志。
- `tail(numLines)` - 返回日志的最后X行，其中X是您传递的数字。默认为100。
- `search(string)` - 返回编号行，其中找到与您的搜索相匹配的行。传递一个字符串。
- `getSlice(start, numLines)` - 获取日志的“切片”。传递起始线以及您想要的后几行
- `downloadLog()` - 下载日志的.txt文件。 [请参见下面的示例](#outputting).
- `clear()` - 清除日志。
- `determineType()` - 为你提供更方便的 `typeof` 方法

### 选项 <a name="options"></a>

在 debugout 的定义中, 您可以编辑选项：

```js
// 实时打印 (转发到console.log)
self.realTimeLoggingOn = true; 
// 在每个日志的前面插入一个时间戳
self.useTimestamps = false; 
// 使用window.localStorage()存储输出，并在每次会话中不断添加到相同的日志中
self.useLocalStorage = false; 
// 在完成调试后设置为false，以避免日志占用内存
self.recordLogs = true; 
// 避免日志可能耗尽无限的内存
self.autoTrim = true; 
// 如果autoTrim为true，那么将保存这最多最近的行
self.maxLines = 2500; 
// tail()将检索多少行
self.tailNumLines = 100; 
// 调用downloadLog()方法后, 下载日志的文件名
self.logFilename = 'log.txt';
// 日志对象的最大递归深度
self.maxDepth = 25;
```

### 输出示例 <a name="outputting"></a>

这是您可以使用该日志的几个示例。每个示例均假定您已建立`debugout`对象并使用该对象进行记录：

```js
var bugout = new debugout();
bugout.log('something');
bugout.log(somethingElse);
bugout.log('etc');
```

##### 示例1：将日志下载为.txt文件的按钮

只需调用`debugout`的`downloadLog()`方法。您可以通过编辑更改文件名`self.logFilename`。

```html
<input type="button" value="Download log" onClick="bugout.downloadLog()">
````

##### 示例2：将日志附加到电子邮件的PhoneGap应用

所示示例使用 [Email Composer插件](https://github.com/inorganik/cordova-emailComposerWithAttachments) 还需要File插件: `cordova plugin add org.apache.cordova.file`.

```js
function sendLog() {
	var logFile = bugout.getLog();

	// save the file locally, so it can be retrieved from emailComposer
	window.requestFileSystem(LocalFileSystem.PERSISTENT, 0, function(fileSystem) {
		// create the file if it doesn't exist
		fileSystem.root.getFile('log.txt', {create: true, exclusive: false}, function(file) {
			// create writer
			file.createWriter(function(writer) {
		        // write
	    		writer.write(logFile);
	    		// when done writing, call up email composer
				writer.onwriteend = function(evt) {
		            // params: subject,body,toRecipients,ccRecipients,bccRecipients,bIsHTML,attachments,filename
		            var subject = 'Log from myApp';
		            var body = 'Attached is a log from my recent testing session.';
					window.plugins.emailComposer.showEmailComposer(subject,body,[],[],[],false,['log.txt'], ['myApp log']);
		        }
			}, fileSystemError);
		}, fileSystemError);
	}, fileSystemError);
}
function fileSystemError(error) {
    bugout.log('Error getting file system: '+error.code);
}
```
### 更多...

- 如果发生错误或其他事件，通过ajax请求将日志发送到服务器。
- 允许用户下载提交的表单的副本。
- 生成收据供用户下载。
- 记录调查答案，知道用户回答每个问题花了多长时间。



