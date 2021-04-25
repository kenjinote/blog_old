---
layout: post
title: DisableProcessWindowsGhosting 関数を使って応答なし表示に移行しないようにする(C++)
tags: [C++,応答なし]
comments: true
---

ウィンドウを表示するプログラムで、時間がかかるループの処理などを行いメッセージを処理しない状態が続くと、ウィンドウのタイトルバーに「応答なし」という文字が追加され、薄く白いウィンドウ表示になってしまいます。

この状態は、Ghost 状態と呼ばれる。この状態へ移行しないようにするための関数が用意されています。

<a href="https://docs.microsoft.com/ja-jp/windows/win32/api/winuser/nf-winuser-disableprocesswindowsghosting">DisableProcessWindowsGhosting</a> 関数

この関数を使えば、プログラムを Ghost 状態へ移行しないようにすることができます。
一度この関数を読んでしまうと再び Ghost 状態へ移行するようにすることはできないようです。

### 参考文献
- [DisableProcessWindowsGhosting function (winuser.h)](https://docs.microsoft.com/ja-jp/windows/win32/api/winuser/nf-winuser-disableprocesswindowsghosting)
