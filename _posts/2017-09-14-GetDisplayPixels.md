---
layout: post
title: マルチディスプレイ環境で各ディスプレイの解像度を取得する(C++)
subtitle: トレーダーの環境で実行してみたい
thumbnail-img: /assets/img/getdisplaypixels.png
tags: [C++,マルチディスプレイ,解像度,ツール]
comments: true
---

マルチディスプレイ環境で各ディスプレイの解像度を取得するには、関数を使用して渡ってきたモニターハンドル`HMONITOR`に対して
`GetMonitorInfo`関数および`EnumDisplaySettings`関数を使って解像度を取得できます。

![](/assets/img/getdisplaypixels.png){: .mx-auto.d-block :}

[プロジェクトのダウンロード](https://github.com/kenjinote/GetDisplayPixels/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>

TCHAR szClassName[] = TEXT("Window");

BOOL CALLBACK MonitorEnumProc(HMONITOR hMonitor, HDC hdcMonitor, LPRECT lprcMonitor, LPARAM dwData)
{
  MONITORINFOEX MonitorInfoEx;
  MonitorInfoEx.cbSize = sizeof(MonitorInfoEx);
  if (GetMonitorInfo(hMonitor, &MonitorInfoEx) != 0)
  {
    DEVMODE dm = { 0 };
    dm.dmSize = sizeof(DEVMODE);
    if (EnumDisplaySettings(MonitorInfoEx.szDevice, ENUM_CURRENT_SETTINGS, &dm) != 0)
    {
      TCHAR szText[1024];
      wsprintf(szText, TEXT("%s, %dx%d"), MonitorInfoEx.szDevice, dm.dmPelsWidth, dm.dmPelsHeight);
      SendMessage((HWND)dwData, LB_ADDSTRING, 0, (LPARAM)szText);
    }
  }
  return TRUE;
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hButton;
  static HWND hList;
  switch (msg)
  {
  case WM_CREATE:
    hButton = CreateWindow(TEXT("BUTTON"), TEXT("取得"), WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)IDOK, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    hList = CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("LISTBOX"), 0, WS_VISIBLE | WS_CHILD | LBS_NOINTEGRALHEIGHT, 0, 0, 0, 0, hWnd, 0, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    break;
  case WM_SIZE:
    MoveWindow(hButton, 10, 10, 256, 32, TRUE);
    MoveWindow(hList, 10, 50, LOWORD(lParam) - 20, HIWORD(lParam) - 50 - 10, TRUE);
    break;
  case WM_COMMAND:
    if (LOWORD(wParam) == IDOK)
    {
      SendMessage(hList, LB_RESETCONTENT, 0, 0);
      EnumDisplayMonitors(NULL, NULL, MonitorEnumProc, (LPARAM)hList);
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
    TEXT("各ディスプレイの解像度（ピクセル数）を取得"),
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

###   参考
- [EnumDisplayMonitors function (winuser.h)](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-enumdisplaymonitors)
