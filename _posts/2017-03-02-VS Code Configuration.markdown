---
layout: post
title:  "V$ Code 的基礎設定"
date:   2017-03-02 13:36:58
categories: Coding
tags: VSCode, C/C++
---
Vitual Studio太肥太難用，佔了一堆系統資源，加上不支援MacOS與Linux系統，所以果斷放棄

原本都是用[SublimeText](https://www.sublimetext.com/)，不過常常跳廣告就...(ry

直到看完這篇[“為什麼我從 Sublime Text 跳槽 Visual Studio Code？”](https://hungys.xyz/why-i-switched-from-sublime-to-vscode/)就怒跳坑了

以下是主要的幾個優點:
* 開源以及持續活躍的開發
* git整合(之後來試試看)
* 內建Debugger框架
* 超豐富的套件資源

# 下面來說說怎麼安裝VS Code以及建置C/C++編譯環境

先從Vitual Studio的[官網](https://code.visualstudio.com/)下載

* Mac 支援 .dbm包裝
* Windows支援Installer 與 .zip
* Linux 支援 .deb, .rpm, .tar.gz 三種

VS Code不是IDE，但是還是可以build程式碼

下面提供C/C++編譯設定方法

詳細設定可以看[官方文件](https://code.visualstudio.com/docs/languages/cpp)
1. 開啟VS Code(廢話...)
2. 在左邊的Sidebar找到Extension ![Extension](https://chiaoy.github.io/assets/VS Code/Extension.png)
3. 找到C/C++，安裝
4. 開啟你的程式碼，你會看到一堆小蚯蚓（像是在＃include <>那行）
5. 點擊左邊的小燈泡
6. 點擊“Add include path to settings”
7. VS Code會自動生成c_cpp_properties.json，然後就...別理他
8. 打開`Command Palette` (⇧⌘P).
9. 輸入`Tasks: Configure Task Runner`
10. 選擇 `Others`
11. 把下面的Code複製貼上
```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "0.1.0",
    "command": "g++",
    "isShellCommand": true,
    "showOutput": "always",
    "args": ["-g", "-o ${fileBasenameNoExtension}", "${file}"],
    "echoCommand": true
}
```
12. 切換到你的C/C++ code分頁
13. Build(⇧⌘B)
14. 就醬子（撒花）

