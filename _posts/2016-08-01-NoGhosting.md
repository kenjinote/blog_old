---
layout: post
title: プログラムが応答なし表示状態に移行しないようにする(C++)
thumbnail-img: /assets/img/noghosting.png
tags: [C++,応答なし]
comments: true
---

ウィンドウを表示するプログラムで、時間がかかるループの処理などを行いメッセージを処理しない状態が続くと、ウィンドウのタイトルバーに「応答なし」という文字が追加され、薄く白いウィンドウ表示になってしまいます。

![](/assets/img/noghosting.png){: .mx-auto.d-block :}

この状態は、Ghost状態と呼ばれます。この状態へ移行しないようにするための関数が用意されています。

<a href="https://docs.microsoft.com/ja-jp/windows/win32/api/winuser/nf-winuser-disableprocesswindowsghosting">DisableProcessWindowsGhosting</a> 関数

この関数を使えば、プログラムをGhost状態へ移行しないようにすることができます。
一度この関数を読んでしまうと再びGhost状態へ移行するようにすることはできないようです。

### 参考文献
- [DisableProcessWindowsGhosting function (winuser.h)](https://docs.microsoft.com/ja-jp/windows/win32/api/winuser/nf-winuser-disableprocesswindowsghosting)
