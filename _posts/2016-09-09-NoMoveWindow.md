---
layout: post
title: 移動できないウィンドウを作成(C++)
subtitle: 頑固なおひとだ
thumbnail-img: /assets/img/nomovewindow.png
tags: [C++,Window]
comments: true
---

タイトルバーのドラッグ等で動かすことができないウィンドウを作成するには、システムメニュー項目の移動`SC_MOVE`を削除することで実現できます。
この方法の応用で、閉じるボタン`SC_CLOSE`を無効にしたり、サイズ変更`SC_SIZE`を無効にしたりすることもできます。

[プロジェクトのダウンロード](https://github.com/kenjinote/NoMoveWindow/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>

TCHAR szClassName[] = TEXT("Window");

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  switch (msg)
  {
  case WM_CREATE:
    DeleteMenu(GetSystemMenu(hWnd, false), SC_MOVE, MF_BYCOMMAND);
    break;
  case WM_DESTROY:
    PostQuitMessage(0);
    break;
  default:
    return DefWindowProc(hWnd, msg, wParam, lParam);
  }
  return 0;
}

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPreInst, LPSTR pCmdLine, int nCmdShow)
{
  MSG msg;
  WNDCLASS wndclass = {
    CS_HREDRAW | CS_VREDRAW,
    WndProc,
    0,
    0,
    hInstance,
    0,
    LoadCursor(0,IDC_ARROW),
    (HBRUSH)(COLOR_WINDOW + 1),
    0,
    szClassName
  };
  RegisterClass(&wndclass);
  HWND hWnd = CreateWindow(
    szClassName,
    TEXT("Window"),
    WS_OVERLAPPEDWINDOW,
    CW_USEDEFAULT,
    0,
    CW_USEDEFAULT,
    0,
    0,
    0,
    hInstance,
    0
  );
  ShowWindow(hWnd, SW_SHOWDEFAULT);
  UpdateWindow(hWnd);
  while (GetMessage(&msg, 0, 0, 0))
  {
    TranslateMessage(&msg);
    DispatchMessage(&msg);
  }
  return (int)msg.wParam;
}
{% endhighlight %}

システムメニューの項目を削除する方法以外にも、`WM_SYSCOMMAND`メッセージの`SC_MOVE`に対して、デフォルトの処理を行わないことでウィンドウの移動をさせないようにできます。

### 参考文献
- [http://smdn.jp/programming/tips/disable_window_moving](http://smdn.jp/programming/tips/disable_window_moving)
