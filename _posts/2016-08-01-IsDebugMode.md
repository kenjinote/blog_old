---
layout: post
title: プログラムがデバッグ中かどうか判別する(C++)
subtitle: デバッグ時と処理を変えたい場合
thumbnail-img: /assets/img/isdebugmode.png
tags: [C++,デバッグ]
comments: true
---

実行中のプロセスから現在デバッグモードで起動しているかどうか判別する関数として、CheckRemoteDebuggerPresent関数があります。
使用例は下記の通りです。

![](/assets/img/isdebugmode.png){: .mx-auto.d-block :}

[プロジェクトのダウンロード](https://github.com/kenjinote/IsDebugMode/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
HANDLE hProcess = GetCurrentProcess();
BOOL bDebuggerPresent;
if (CheckRemoteDebuggerPresent(hProcess, &amp;bDebuggerPresent))
{
  if (bDebuggerPresent)
  {
    MessageBox(hWnd, TEXT("デバッグ中です。"), 0, 0);
  }
  else
  {
    MessageBox(hWnd, TEXT("デバッグ中ではないです。"), 0, 0);
  }
}
else
{
  MessageBox(hWnd, TEXT("CheckRemoteDebuggerPresent関数が失敗"), 0, 0);
}
{% endhighlight %}

### 参考文献
- [CheckRemoteDebuggerPresent function (debugapi.h)](https://docs.microsoft.com/en-us/windows/win32/api/debugapi/nf-debugapi-checkremotedebuggerpresent)
