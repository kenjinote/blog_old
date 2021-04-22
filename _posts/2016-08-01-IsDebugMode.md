---
layout: post
title: CheckRemoteDebuggerPresent 関数を使って動的にデバッグ中かどうか確認する(C++)
tags: [C++,デバッグ]
comments: true
---

実行中のプロセスから現在デバッグモードで起動しているかどうか判別する関数に CheckRemoteDebuggerPresent 関数というものがある。
使用例は下記の通りです。

[プロジェクトのダウンロード](https://github.com/kenjinote/GetWiFiSignalStrength/archive/master.zip)

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
