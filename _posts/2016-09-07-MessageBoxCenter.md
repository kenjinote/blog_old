---
layout: post
title: メッセージボックスを親ウィンドウの中央に表示(C++)
thumbnail-img: /assets/img/messageboxcenter.png
tags: [C++,メッセージボックス]
comments: true
---

メッセージボックス（MessageBox）は標準でディスプレイの中央に表示される挙動ですが、メッセージボックスを親ウィンドウの中央に表示したい場合など任意の位置に表示させるには、フックを使うことでメッセージウィンドウの位置を調整することができます。

メッセージボックスのフックを行うにはフックプロシージャと言われる関数を作成し、メッセージボックスを表示する直前で SetWindowsHookEx 関数に引数としてフックプロシージャを渡します。

こうするとメッセージボックスが表示される直前で、フックプロシージャがシステムから呼び出されます（nCode = HCBT_ACTIVATEとして呼び出される）ので、そこでメッセージウィンドウの位置を調整します。ここで忘れてはいけないのがフックを解除（UnhookWindowsHookEx）することです。フックの解除しないと対象のメッセージボックス以外のウィンドウに対してもフックプロシージャが呼び出されてしまいます。

[プロジェクトのダウンロード](https://github.com/kenjinote/MessageBoxCenter/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>

TCHAR szClassName[] = TEXT("Window");
HHOOK g_hHook;

LRESULT CALLBACK CBTProc(int nCode, WPARAM wParam, LPARAM lParam)
{
  if (nCode == HCBT_ACTIVATE)
  {
    UnhookWindowsHookEx(g_hHook);
    const HWND hMessageBox = (HWND)wParam;
    const HWND hParentWnd = GetParent(hMessageBox);
    RECT rectMessageBox, rectParentWnd;
    GetWindowRect(hMessageBox, &rectMessageBox);
    GetWindowRect(hParentWnd, &rectParentWnd);
    SetWindowPos(
      hMessageBox,
      hParentWnd,
      (rectParentWnd.right + rectParentWnd.left - rectMessageBox.right + rectMessageBox.left) >> 1,
      (rectParentWnd.bottom + rectParentWnd.top - rectMessageBox.bottom + rectMessageBox.top) >> 1,
      0,
      0,
      SWP_NOSIZE | SWP_NOZORDER | SWP_NOACTIVATE);
  }
  return 0;
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hButton;
  switch (msg)
  {
  case WM_CREATE:
    hButton = CreateWindow(TEXT("BUTTON"), TEXT("メッセージボックスを表示"),
      WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)IDOK,
      ((LPCREATESTRUCT)lParam)->hInstance, 0);
    break;
  case WM_SIZE:
    MoveWindow(hButton, 10, 10, 256, 32, TRUE);
    break;
  case WM_COMMAND:
    if (LOWORD(wParam) == IDOK)
    {
      g_hHook = SetWindowsHookEx(WH_CBT, CBTProc, 0, GetCurrentThreadId());
      MessageBox(hWnd, TEXT("親ウィンドウの中央に表示"), TEXT("確認"), 0);
    }
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
    TEXT("メッセージボックスを親ウィンドウの中央に表示"),
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
