---
layout: post
title: ウィンドウの背景色を変更する(C++)
subtitle: 色はセンスを問われる
thumbnail-img: /assets/img/windowbackgroundcolor.png
tags: [C++,ウィンドウ,Window,色,背景色]
comments: true
---

ウィンドウの背景色を変更するプログラムを紹介します。背景色を変える方法が幾つかあります。

1. ウィンドウクラスのスタイルのブラシに設定する
1. `WM_ERASEBKGND`で独自で塗り潰して、`TRUE`を返す 
1. ダイアログの場合は、`WM_CTLCOLORDLG`でブラシを返す

今回は、一番実装が簡単な、1の方法をご紹介します。

サンプルプログラムでは、プログラム起動時に`RegisterClass`関数でウィンドウのクラス登録を行う際に、作成したブラシを設定しています。また、ウィンドウ内のボタンを押した際に動的に背景色を変えるように'SetClassLongPtr'関数を使って、動的に作成したブラシをウインドウクラスに設定しています。


![](/assets/img/windowbackgroundcolor.png){: .mx-auto.d-block :}

[プロジェクトのダウンロード](https://github.com/kenjinote/WindowBackgroundColor/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>

#define DEFAULT_DPI 96
#define SCALEX(X) MulDiv(X, uDpiX, DEFAULT_DPI)
#define SCALEY(Y) MulDiv(Y, uDpiY, DEFAULT_DPI)
#define POINT2PIXEL(PT) MulDiv(PT, uDpiY, 72)

TCHAR szClassName[] = TEXT("Window");

BOOL GetScaling(HWND hWnd, UINT* pnX, UINT* pnY)
{
  BOOL bSetScaling = FALSE;
  const HMONITOR hMonitor = MonitorFromWindow(hWnd, MONITOR_DEFAULTTONEAREST);
  if (hMonitor)
  {
    HMODULE hShcore = LoadLibrary(TEXT("SHCORE"));
    if (hShcore)
    {
      typedef HRESULT __stdcall GetDpiForMonitor(HMONITOR, int, UINT*, UINT*);
      GetDpiForMonitor* fnGetDpiForMonitor = reinterpret_cast<GetDpiForMonitor*>(GetProcAddress(hShcore, "GetDpiForMonitor"));
      if (fnGetDpiForMonitor)
      {
        UINT uDpiX, uDpiY;
        if (SUCCEEDED(fnGetDpiForMonitor(hMonitor, 0, &uDpiX, &uDpiY)) && uDpiX > 0 && uDpiY > 0)
        {
          *pnX = uDpiX;
          *pnY = uDpiY;
          bSetScaling = TRUE;
        }
      }
      FreeLibrary(hShcore);
    }
  }
  if (!bSetScaling)
  {
    HDC hdc = GetDC(NULL);
    if (hdc)
    {
      *pnX = GetDeviceCaps(hdc, LOGPIXELSX);
      *pnY = GetDeviceCaps(hdc, LOGPIXELSY);
      ReleaseDC(NULL, hdc);
      bSetScaling = TRUE;
    }
  }
  if (!bSetScaling)
  {
    *pnX = DEFAULT_DPI;
    *pnY = DEFAULT_DPI;
    bSetScaling = TRUE;
  }
  return bSetScaling;
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hButton;
  static HFONT hFont;
  static UINT uDpiX = DEFAULT_DPI, uDpiY = DEFAULT_DPI;
  static HBRUSH hBrush;
  switch (msg)
  {
  case WM_CREATE:
    hButton = CreateWindow(TEXT("BUTTON"), TEXT("背景色をランダムに変える"), WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)IDOK, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    SendMessage(hWnd, WM_APP, 0, 0);
    break;
  case WM_SIZE:
    MoveWindow(hButton, POINT2PIXEL(10), POINT2PIXEL(10), POINT2PIXEL(256), POINT2PIXEL(32), TRUE);
    break;
  case WM_COMMAND:
    if (LOWORD(wParam) == IDOK)
    {
      // ウィンドウの背景色をランダムに変える
      if (hBrush) DeleteObject(hBrush);
      hBrush = CreateSolidBrush(RGB(rand() % 256, rand() % 256, rand() % 256));
      SetClassLongPtr(hWnd, GCLP_HBRBACKGROUND, (LONG_PTR)hBrush);
      InvalidateRect(hWnd, 0, TRUE);
    }
    break;
  case WM_NCCREATE:
    {
      const HMODULE hModUser32 = GetModuleHandle(TEXT("user32.dll"));
      if (hModUser32)
      {
        typedef BOOL(WINAPI*fnTypeEnableNCScaling)(HWND);
        const fnTypeEnableNCScaling fnEnableNCScaling = (fnTypeEnableNCScaling)GetProcAddress(hModUser32, "EnableNonClientDpiScaling");
        if (fnEnableNCScaling)
        {
          fnEnableNCScaling(hWnd);
        }
      }
    }
    return DefWindowProc(hWnd, msg, wParam, lParam);
  case WM_DPICHANGED:
    SendMessage(hWnd, WM_APP, 0, 0);
    break;
  case WM_APP:
    GetScaling(hWnd, &uDpiX, &uDpiY);
    DeleteObject(hFont);
    hFont = CreateFontW(-POINT2PIXEL(10), 0, 0, 0, FW_NORMAL, 0, 0, 0, SHIFTJIS_CHARSET, 0, 0, 0, 0, L"MS Shell Dlg");
    SendMessage(hButton, WM_SETFONT, (WPARAM)hFont, 0);
    break;
  case WM_DESTROY:
    if (hBrush) DeleteObject(hBrush);
    DeleteObject(hFont);
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
  HBRUSH hBrush = CreateSolidBrush(RGB(0,0,255)); // ウィンドウの背景色初期設定
  WNDCLASS wndclass = {
    CS_HREDRAW | CS_VREDRAW,
    WndProc,
    0,
    0,
    hInstance,
    0,
    LoadCursor(0,IDC_ARROW),
    hBrush, // 既定の色はシステムのウィンドウ色設定 → (HBRUSH)(COLOR_WINDOW + 1),
    0,
    szClassName
  };
  RegisterClass(&wndclass);
  HWND hWnd = CreateWindow(
    szClassName,
    TEXT("Window"),
    WS_OVERLAPPEDWINDOW | WS_CLIPCHILDREN,
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
  DeleteObject(hBrush);
  return (int)msg.wParam;
}
{% endhighlight %}

### 参考文献
- [ウィンドウクラスの概要 クラスの背景ブラシ](https://docs.microsoft.com/ja-jp/windows/win32/winmsg/about-window-classes#class-background-brush)
- [WM _ ERASEBKGND メッセージ](https://docs.microsoft.com/ja-jp/windows/win32/winmsg/wm-erasebkgnd)
