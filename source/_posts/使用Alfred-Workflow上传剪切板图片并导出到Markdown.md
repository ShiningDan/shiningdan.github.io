---
title: 使用Alfred Workflow上传剪切板图片并导出到Markdown
date: 2017-01-15 13:32:37
categories: coding
tags:
  - Alfred
  - 七牛
  - Markdown
---

感谢[K.Chen 的博客](http://kchen.cc/2016/03/19/alfred-workflow-for-markdown-image-on-qiniu/)为我提供了实现的方法。

## 应用场景

在我们使用 Markdown 做笔记的时候，经常遇到的一个很麻烦的问题就是如何将图片插入到 Markdown 笔记中。原有的 Markdown 笔记不支持将本地图片上传到笔记中，即使是马克飞象等编辑器支持将图片剪贴到 Markdown 笔记中进行预览，但是其图片链接是一个本地链接，若是将笔记本上传到网络中，图片则无法显示。所以，我在向 Markdown 笔记插入图片的时候，使用的是七牛云存储相关的图片，然后导出外链到本地笔记，这样，本地可以通过外链预览图片，即使是笔记上传到了网上也可以通过外链预览图片。

但是，在本地剪切图片，上传到七牛云并且导出外链到 Markdown 笔记中是一个很复杂的过程。下面是一般情况下将图片插入 Markdown 笔记需要进行的步骤：

1. 使用截屏工具将截屏并且将截屏文件保存在本地桌面（Mac）
2. 为获得的截屏文件修改一个合适的名字
3. 打开七牛云存储，上传本体图片（可以设置前缀）
4. 退出上传页面，在网页中找到刚刚上传的图片，点击复制外链，获得图片的外链
5. 在 Markdown 笔记中使用 `![]()` 添加外链，可以实现图片预览


由于其步骤繁琐并且消耗的时间很多，经常会打断我们的写作思路，所以我们需要一种快捷的方法将剪切板中的笔记上传到七牛云并导出链接。下面是本教程的实现目标：

1. 使用截屏工具截屏并保存到剪贴板
2. 使用 Alfred Workflow 将剪贴板内容上传到七牛云，并将该图片的外链自动保存到剪贴板
3. 使用`![]()`在 Markdown 笔记中插入图片。

<!--more-->

## 准备工作

### 七牛云准备工作

首先需要在七牛开发者平台申请，并获得 AccessKey 和 SecretKey，用来支持上传本地文件到自己的存储空间。步骤如下：

1. 开通[七牛开发者账号](https://portal.qiniu.com/create)
2. 登录七牛开发者平台，查看[Access Key 和 Secret Key](https://portal.qiniu.com/user/key)

[七牛开发者工具](http://developer.qiniu.com/resource/official.html#tool)提供了很多的工具，其中可以实现上传本地文件的工具有两个，一个是[qshell](http://developer.qiniu.com/code/v6/tool/qshell.html)，另一个是[qrsbox](http://developer.qiniu.com/code/v6/tool/qrsbox.html)。他们的区别是 qshell 是使用命令行上传图片，除了上传图片以外，qshell 还支持很多其他的功能（虽然这些功能在本教程中不会使用），qshell 需要使用命令行来触发上传的动作。qrsbox 是一个带有图形界面的自动化上传工具，可以实现后台自动监听某个文件夹内部文件的增减，并且自动将本地文件同步到七牛云中。

本文使用的是 qshell 来上传文件，因为我觉得这样灵活，当然使用 qrsbox 也是一个很好的选择，自动化同步本地文件夹也十分方便。

### qshell 配置

可以在[七牛云 qshell ](http://developer.qiniu.com/code/v6/tool/qshell.html)工具中下载对应平台的 qshell 可执行文件，因为我使用的是 Mac 平台，所以是在 Mac 下进行配置。

首先创建文件夹保存 qshell 执行文件和需要同步的图片和文件，我创建的目录为`/Users/yuchenzhang/Documents/hexo/qshell`，其中有两个文件夹：

1. cli文件夹：用来放置 qshell 执行文件
2. data文件夹：用来放置需要同步的图片和文件

在创建完文件夹并且将 qshell 拷贝到 cli 以后，我们要配置 qshell API。

首先要使用七牛的 API，必须先设置 AccessKey 和 SecretKey 。命令如下：
```
qshell account ELUs327kxVPJrGCXqWae9yioc0xYZyrIpbM6Wh6o LVzZY2SqOQ_I_kM1n00ygACVBArDvOWtiLkDtKi_
```
上面的 `ELUs327kxVPJrGCXqWae9yioc0xYZyrIpbM6Wh6o` 是您的 AccessKey，而 `LVzZY2SqOQ_I_kM1n00ygACVBArDvOWtiLkDtKi_` 是您的 SecretKey。如果您想查看当前 AccessKey 和 SecretKey 设置，使用命令：
```
qshell account
```
上面的命令会输出当前您设置好的 AccessKey 和 SecretKey。接下来，我们就可以放心地使用七牛的 API 功能了。

我们需要使用 qshell 的 qupload 命令来同步本地文件。在该 cli 文件夹下新建数据同步的配置文件`conf.json`，配置文件中的内容可以参考[qshell qupload 参考文档](https://github.com/qiniu/qshell/wiki/qupload)

针对我的情况，我使用的配置文件 `conf.json` 中的内容为：

```
{
	"src_dir"       :   "/Users/yuchenzhang/Documents/hexo/qshell/data", //需要同步的文件夹
    "access_key"    :   "<AccessKey>", //我的 AccessKey
    "secret_key"    :   "<SecretKey>", //我的 SecretKey
    "bucket"        :   "myweb",  //需要同步到哪个存储空间中
    "zone"			:	"nb", //该存储空间在哪个片区（华东）
    "rescan_local"	:	true //默认情况下，本地新增的文件不会被同步，需要手动设置为true才会去检测新增文件。
}
```

在配置完成以后，就可以使用`./qshell qupload conf.json` 在 cli 文件夹路径下运行命令，将 data 文件夹中的数据同步到七牛云上。至此，qshell 配置的部分完成。本人测试的时候，成功将 data 文件夹下的 test.png 图片同步到七牛云上

![](http://ojt6zsxg2.bkt.clouddn.com/test1.png)

### Alfred Workflow 准备工作

首先创建一个空白的 Alfred Workflow，如下：

![](http://ojt6zsxg2.bkt.clouddn.com/2017:01:15:15:52:34.png)

然后为这个空白的工作流创建一个引导，如何触发这个工作流，我选择的方法是使用 Keyword，也可以按照[K.chen](http://kchen.cc/2016/03/19/alfred-workflow-for-markdown-image-on-qiniu/)的方法，使用一个快捷键进行上传。

![](http://ojt6zsxg2.bkt.clouddn.com/a23ace591c850f4a5efdab45e2cf1b93.png)

然后添加一个 action，使用/usr/bin/osascript(AS)作为脚本语言，如下：

![](http://ojt6zsxg2.bkt.clouddn.com/cdaa76e314749cb1d73676908d20cc3f.png)

其中的内容是：

```
property fileTypes : {¬
	{«class PNGf», ".png"}, ¬
	{JPEG picture, ".jpg"}}
on getType()
	repeat with aType in fileTypes
		repeat with theInfo in (clipboard info)
			if (first item of theInfo) is equal to (first item of aType) then return aType
		end repeat
	end repeat
	return missing value
end getType
set theType to getType()
if theType is not missing value then
	set filePath to "/Users/yuchenzhang/Documents/hexo/qshell/data/" --这里换成你自己放置图片的路径
	set fileName to do shell script "date \"+%Y%m%d%H%M%S\" | md5" --用当前时间的md5值做文件名
	if fileName does not end with (second item of theType) then set fileName to (fileName & second item of theType as text)
	set markdownUrl to "![](http://ojt6zsxg2.bkt.clouddn.com/" & fileName & ")" --这里是你的七牛域名
	set filePath to filePath & fileName
	
	try
		set imageFile to (open for access filePath with write permission)
		set eof imageFile to 0
		write (the clipboard as (first item of theType)) to imageFile
		close access imageFile
		set the clipboard to markdownUrl
		try
			tell application "System Events"
				keystroke "v" using command down
			end tell
		end try
		do shell script "/Users/yuchenzhang/Documents/hexo/qshell/cli/qshell qupload /Users/yuchenzhang/Documents/hexo/qshell/cli/conf.json" --此处是你的qshell脚本目录和命令
	on error
		try
			close access imageFile
		end try
		return ""
	end try
else
	return ""
end if
```

代码中需要替换的地方有三个，分别是：

选择需要同步的路径。

```
set filePath to "/Users/yuchenzhang/Documents/hexo/qshell/data/" --这里换成你自己放置图片的路径
```

替换为自己的存储空间的域名，自己的域名可以在七牛云存储，空间的这个地方找到：

![](http://ojt6zsxg2.bkt.clouddn.com/298592f04f11443522b8d9643b21594d.png)

```
set markdownUrl to "![](http://ojt6zsxg2.bkt.clouddn.com/" & fileName & ")" --这里是你的七牛域名
```

最后要设置执行 qshell 的命令路径：

```
do shell script "/Users/yuchenzhang/Documents/hexo/qshell/cli/qshell qupload /Users/yuchenzhang/Documents/hexo/qshell/cli/conf.json" --此处是你的qshell脚本目录和命令
```

在设置完脚本以后，可以设置一个提醒，在完成该 Alfred Workflow 动作以后，会收到什么样的提醒。我设置的是一个 Post Notification，如下：

![](http://ojt6zsxg2.bkt.clouddn.com/c76dff59aef025e6e16277773c7577b1.png)

最后，用线将上面三个部分连接起来。

![](http://ojt6zsxg2.bkt.clouddn.com/7d8f735f08285050327401139d4134a7.png)

此时，使用截屏软件截屏，图片保存在剪贴板里的时候，可以使用 Alfred Workflow 输入 qupload 上传剪贴板里的图片.

![](http://ojt6zsxg2.bkt.clouddn.com/4160588386f9cae292e97f440ac8aea4.png)

完成！

