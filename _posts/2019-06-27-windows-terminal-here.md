---
title: 'Windows Terminal - 在当前目录打开'
layout: post
subtitle: ''
tags:
  - Windows Terminal
category: 
---
Windows Terminal 已经在微软商店上架，虽然还是早期版本，但也能用了。

我原来通过注册表，在目录的右键菜单里添加了“ 在当前位置打开命令行 ” 的命令项，准备改成 Windows Terminal 版本的。

经过一堆查资料加折腾，终于搞定了。

打开 regedit.exe ，在 计算机\HKEY_CLASSES_ROOT\Directory 下手工编辑，或者把下面的内容保存到 wt.reg 文件内，然后双击

	Windows Registry Editor Version 5.00
	
	[HKEY_CLASSES_ROOT\Directory\Background\shell\cmd_here]
	@="在此打开命令提示符[&C]"
	"Icon"="C:\\Windows\\System32\\cmd.exe"
	
	[HKEY_CLASSES_ROOT\Directory\Background\shell\cmd_here\command]
	@="cmd /c pushd \"%V\" & start wt.exe"
	
	[HKEY_CLASSES_ROOT\Directory\shell\cmd_here]
	@="在此打开命令提示符[&C]"
	"Icon"="C:\\Windows\\System32\\cmd.exe"
	
	[HKEY_CLASSES_ROOT\Directory\shell\cmd_here\command]
	@="cmd /c pushd \"%V\" & start wt.exe"
	

稍有瑕疵，就算会闪一下 cmd 的黑窗口，不过只是小问题。希望未来 wt 能支持参数启动吧。