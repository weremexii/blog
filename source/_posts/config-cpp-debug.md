---
title: macOS下配置VSCode的C++程序的编译和调试
date: 2020-12-26 21:29:45
updated: 2020-12-26 21:29:45
tags: 
- macOS
- vscode
- C++
categories:
- develop
desc: VSCode来写cpp文件并不是C++编程首选,但是这不妨碍我们折腾它的编译与调试
---

VSCode来写cpp文件并不是C++编程首选,但是这不妨碍我们折腾它的编译与调试

<!--more-->

相信来学习C++的你已经了解了C++的编译器是如何将一个或多个cpp文件变成一个可执行文件的,而我们在mac的VSCode的配置上要下功夫的,就是如何让VSCode告诉编译器如何编译

<div class="tip">
    截止到博主写这篇文章时,博主并不需要多文件编译,以下的内容或许会更新,但应该没有多文件编译的内容.其实也和单文件编译类似,只要知道告诉编译器做什么就好
</div>

### 一点背景

+ 关于编译器

    以下只是个人使用感受

    和许多类Unix系统类似,macOS已经内置了C++的编译器,用于编译可执行文件,不过这个编译器似乎并不自带什么方便的调试器

    但是macOS还自带了一个通用的调试器LLDB,似乎可以调试多种高级语言写出来的程序

    你会发现macOS无论是clang还是g++都可以执行,不过好像它们都被链接到同一个编译器

    综上,我们要解决如何调用调试器的问题,以及我们的编译命令主要用`g++`

+ 关于VSCode

    VSCode是为前端工作者设计的,不过只要安装一些插件就可以用来完成后端工作,可定制性很高

    对于编译调试,需要有配置文件来告诉VSCode如何编译调试,所以我们的整个工作是写一个正确的配置文件控制VSCode和编译器

    你可以把自己的一堆cpp文件放在一个文件夹下(这个文件夹的子文件夹也可以),之后我们会在文件夹下新建`.vscode`文件夹,这个文件夹将会用来存放重要的`tasks.json`和`.launch.json`文件.前者是编译的配置文件,后者是调试的配置文件,后者对前者依赖程度很高

### 配置

首先在VSCode的扩展页搜索`C/C++`,它主要来完成自动补全和生成默认的编译配置

然后安装`CodeLLDB`,它主要用来连接macOS上的调试器lldb

#### 编译调试

来到一个cpp文件下,按`Shift+Cmd+B`,第一次按下会跳出默认的编译选项,我们不做选择,而是点击'C/C++ : Build g++ active file'的那个小齿轮,这样会自动生成`tasks.json`并打开编辑它的标签页

我的配置如下

```json
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "shell",
			"label": "C/C++: g++ build active file",
			"command": "/usr/bin/g++",
			"args": [
				"-std=c++11",
				"-stdlib=libc++",
				"${file}",
				"-o",
				"${fileDirname}/${fileBasenameNoExtension}", 
			],
			"options": {
				"cwd": "/usr/bin"
			},
			"problemMatcher": [
				"$gcc"
			],
			"group": {
				"kind": "build",
				"isDefault": true
			}
		},
		{
			"type": "shell",
			"label": "C/C++: g++ build active file for debug",
			"command": "/usr/bin/g++",
			"args": [
				"-std=c++11", 
				"-stdlib=libc++",
				"${file}",
				"-o",
				"${workspaceFolder}/debug/${fileBasenameNoExtension}",
				"-g"  //生成可供调试的可执行文件,等价于--debug参数,会生成dSYM文件
			],
			"options": {
				"cwd": "/usr/bin"
			},
			"problemMatcher": [
				"$gcc"
			],
			"group": "build"
		}
	]
}
```

主要由两个部分组成,第一个是编译用的,只生成可执行文件;第二个是调试用的,还会生成dSYM调试用文件

第一个的`"group" :"isDefault": true`可以使它成为`Shift+Cmd+B`编译快捷键的默认操作;第二个则会在调试时被调用,并在debug文件夹下生成可执行文件和调试用文件,而后我们配置的调试命令会执行debug下的可执行文件(而不是像第一个那样执行与cpp文件同目录下的可执行文件)

同样在cpp文件标签页下,我们换到debug页,点击`创建launch.json`,然后选择LLDB

我的配置如下

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "LLDB Launch",
            "program": "${workspaceFolder}/debug/${fileBasenameNoExtension}",
            "args": [],
            "cwd": "${workspaceFolder}",
            "preLaunchTask": "C/C++: g++ build active file for debug"
        }
    ]
}
```

可以看到,`"preLaunchTask"`节点对于了我们之前的第二个编译配置

以上就是基本配置,你可以选择copy我的配置,或者看看这些配置这些参数代表了什么,然后自己在VSCode生成的配置基础上写出自己的一份配置

我也是这么过来的QAQ

#### 自动补全

有时候编译完会出现C++标准的警告

在VSCode的设置里搜索`Cpp standard`,设置为你需要的标准