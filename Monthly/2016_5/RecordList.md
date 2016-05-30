# 2016.5月 备忘
---



## Git

### 1. 同时操作不同的远程仓库的问题

场景：公司内网`git`上要干活，同时自己的`github`上也要干活；那么push代码的时候，用户名和邮箱要分别设置过；否则`github`上的记录会不正确；

可以直接参照我之前的博客文章[github上Contributions丢失的问题](http://agger0207.github.io/blog/2016/01/01/githubshang-contributionsdiu-shi-de-wen-ti/)

核心:

	$ git config --local user.name "yourusername"
	$ git config --local user.email "username@users.noreply.github.com"
	$ git config --list

也可以参照[Git: Using Different User Emails for Different Repositories](https://orrsella.com/2013/08/10/git-using-different-user-emails-for-different-repositories/)中的做法；这里面的做法主要是考虑到每次需要对local repo重新设置，也就是每次clone下代码后都要设置一次，比较容易忘记，所以将global的user.email设置为空；然后每次提交的时候都通过一个脚本去做检查，如果local的user.email没有设置过，那么就报错；这样就会强行提示作者给local设置一次. 原文: 

	I don’t have user.email configured globally, but on a per-repository basis. And in-case I forget to set it up for a new repo, I’m not able to commit to it and a friendly message reminds me to configure it and how to do it.

## CocoaPods

### 1. [The format of .xcodeproj file is modified after running `pod install` via CocoaPods 0.39](https://github.com/CocoaPods/CocoaPods/issues/5416)

给CocoaPods report了一个issue, 核心是，CocoaPods 0.39安装后，会错误的改掉工程文件的格式，从old style plist format改回XML格式，这样如果依赖一些第三方工程文件解析库写Xcode插件，就会出问题；比如[alunny/node-xcode](https://github.com/alunny/node-xcode)不会按照XML来解析Xcode的工程文件;

扩展:

1. CocoaPods 0.39 相比 0.38.2的改动:   
	不支持递归搜索Pod库中的头文件;  其中，`RestKit`就因为这个改动所以老版本不支持CocoaPods 0.39, 后面有相应改动可以参见[Use framework-style import statements](https://github.com/RestKit/RestKit/pull/2310/commits); 
2. CocoaPods 1.0 相比 0.39的改动:   
	暂时没有仔细研究，不过我们自己的一些私有库的`podspec`是会报错的; 当前我自己唯一确定的是需要在`PodFile`中指定target: 
	
	```ruby
	platform :ios, '7.0'

	target 'TestPods' do
	
		pod 'AFNetworking'
		pod 'RestKit'

	end
	```
3. 关于[alunny/node-xcode](https://github.com/alunny/node-xcode); 这个是一个用`node.js`来解析与修改Xcode工程文件的库，不过对`group`的支持很差，而且向工程中添加/删除 资源文件或者库文件也存在问题；但是工程解析是可以复用的。可以学习一下，或者在这个基础上自己重新写一个操作库。可以参照`CocoaPods`的API来写，同样都是操作Xcode Project file, 他们的区别无非是一个是用ruby写的，一个是用node.js写的.
4. 类似的，也有一个Objective-C封装的对Xcode进行编辑的库，没有仔细研究过: [appsquickly/XcodeEditor](https://github.com/appsquickly/XcodeEditor); 有兴趣写Xcode插件来操作Xcode工程的可以参考下.

### 2. 多个CocoaPods版本共存问题
如果你经常遇到我上面提到的问题，那么很可能就需要在本地维护多个CocoaPods版本了，那么多个版本共存主要需要知道如下命令:

1. 安装指定版本的CocoaPods
		
		$sudo gem install cocoapods -v 0.38.2

2. 使用指定版本的CocoaPods
		
		$ pod _0.38.2_ install
		$ pod _0.39.9_ update
		$ pod _0.38.2_ install --verbose --no-repo-update
		$ pod _0.39.9_ update --verbose --no-repo-update

3. 查看当前安装的最新版本

		$ pod --version
		
4. 查看所有安装的CocoaPods版本
		
		$ gem list cocoaPods
		
参考: [Cocoapods多版本使用技巧](http://www.jianshu.com/p/226d47bde47d)

## AFNetworking

1. AFNetworking对IPv6的支持问题(TODO)