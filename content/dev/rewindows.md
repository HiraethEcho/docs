---
title: Windows重装指南
date: 2025-01-18
summary: 重装和初始化windows
tags:
  - windows
  - hack
categories: handbook
---

# Windows重装指南

记录一下重装或者第一次安装windows的一些注意事项。

## 重装

按<kbd>windows</kbd>然后按住<kbd>shift</kbd>点击重启，这样重启后会进入恢复模式（之类的），这里可以选择重装系统。或者在设置中的系统恢复部分。  
重装系统只重装C盘，其他盘是不动的。但是应用安装在其他盘的时候会有问题。

## 安装

window11有个莫名其妙的设定，不能在没有网络的时候安装，必须登陆微软账户，很神经，但是可以绕开。  
在安装界面按<kbd>Shift+F10</kbd>调出命令行，然后输入

```
oobe\bypassnro
```

之后会重新启动安装程序，可以选择无网络连接。

## 设置

### 用户文件夹

在用户目录`C:\Users\_username_`下有一些用户目录，移动这些目录需要通过右键菜单的`属性`

### 设置

### 其他

电源管理

pagefiles

## 软件源

首先从microsoft store安装，其次用scoop，最后用msi安装。

### 应用商店

### scoop

权限：

```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

更改环境变量以修改`scoop`安装位置

```
$env:SCOOP='D:\Applications\Scoop'
[Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')
```

安装`scoop `

```
Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
```

或者

```
iwr -useb get.scoop.sh | iex
```

## 软件

## 清理

### win SxS

### 应用

清理应用商店，卸载预装软件

```
Get-AppxPackage -All 　　　　　　　　　　　　　　　　　　　/* 获取Win10以上系统所有预装软件 */
Get-AppxPackage -All {预装软件全名} | Remove-AppxPackage　　/* 管道方式卸载Win10以上系统预装软件 */
Remove-AppxPackage {预装软件全名} 　　　　　　　　　　　　/* 常规卸载Win10以上系统预装软件 */
Get-AppxPackage -All | Select-Object Name, PackageFullName　　　/*（推荐）获取Win10以上系统所有预装软件 */
Remove-AppxPackage -Package "PackageFullName"　　　　　　/*（推荐）常规卸载Win10以上系统预装软件 */
Get-AppxPackage *BingWeather* | Remove-AppxPackage　　　　/*卸载单个软件
Get-AppXPackage | Remove-AppxPackage　　　　　　　　　　/*卸载所有软件命令
```

图形化选择

```
Get-AppxPackage | Out-GridView -Passthru | Remove-AppXPackage -AllUsers
```

删除某些

```
Get-AppxProvisionedPackage -Online | where-object {$_.packagename -like "xbox"} | Remove-AppxProvisionedPackage -Online
```

去除某个分区的`WindowsApps`文件夹

```
Remove-AppxVolume -Volume X:\
```
