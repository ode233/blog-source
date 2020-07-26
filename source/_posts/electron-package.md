---
title: electron打包生成windows安装包
date: 2020-07-27 04:04:58
tags: electron
categories: electron
---
## 工具
直接上用的人多的electron-builder, 配置也简单。

## 配置
win打包主要配置：
```json
"win": {
      "icon": "build/icons/icon.ico",
      "requestedExecutionLevel": "highestAvailable",
      "target": "nsis"
},
"nsis": {
    "oneClick": false,                                 // 是否使用一键安装
    "allowToChangeInstallationDirectory": true         // 是否允许用户选择安装目录
},
```

## 坑
主要是安装后的卸载程序也太蠢了，直接删除同级目录下的所有文件，如果安装时选了一个有其他文件的目录那就凉凉了（我测试的时候就中招了）。想了下解决方案最简单的就是不管用户选什么目录，安装的时候都自动在用户选择的目录下再创建一个新目录来作为实际安装路径。  
先在网上搜了半天还是没找到直接的操作方法，大概看了下electron-build的官方文档应该是需要自定义NSIS的脚本，于是又去看了NSIS的语法，两个都看完后发现要完全达到我之前的解决方案估计要重写NSIS(主要是要判断什么时候开始安装，然后在开始安装时修改用户的安装路径)。  
权衡了一下还是在electron-build的NSIS上调整好了，通过include来包含我自定义的NSIS脚本，脚本很简单：
```NSIS
# onVerifyInstDir是用户选择安装目录后的回调函数，也就是说在用户选择完安装目录后，自动在对该目录加上一个子目录作为安装目录
Function .onVerifyInstDir
    StrCpy $INSTDIR "$INSTDIR\<AppName>\"
FunctionEnd
```
虽然没有完全杜绝之前的问题（用户直接在目录输入框中修改的话就没办法），但还是一定层度上防止了误删的情况吧。

## 参考资料
[electron-build打包windows安装包配置](https://www.electron.build/configuration/nsis)  
[NSIS 用戶手冊](https://omega.idv.tw/nsis/Contents.html)
