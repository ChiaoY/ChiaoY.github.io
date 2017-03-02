---
layout: post
title:  "V$ Code 的基礎設定"
date:   2017-03-02 13:36:58
categories: Hotels
tags: VSCode, C/C++
---

先從Vitual Studio的[官網](https://code.visualstudio.com/)下載
Mac 支援 .dbm包裝
Windows支援Installer 與 .zip
Linux 支援 .deb, .rpm, .tar.gz 三種

VS Code不是IDE，但是還是可以build程式碼
下面提供C/C++編譯設定方法
詳細設定可以看[官方文件](https://code.visualstudio.com/docs/languages/cpp)
1. 開啟VS Code(廢話...)
2. 在左邊的Sidebar找到Extension ![Extension](https://chiaoy.github.io/assets/VS Code/Extension.png)
3. 找到C/C++，安裝
4. 開啟你的程式碼，你會看到一堆小蚯蚓（像是在＃include <>那行）
5. 點擊左邊的小燈泡
6. 點擊“Add include path to settings”
7. 

    segment test
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

# hello world

you can write text [with links](http://example.com) inline or [link references][1].

* one _thing_ has *em*phasis
* two __things__ are **bold**

[1]: http://example.com

---

hello world
===========

<this_is inline="xml"></this_is>

> markdown is so cool

    so are code segments

1. one thing (yeah!)
2. two thing `i can write code`, and `more` wipee!

