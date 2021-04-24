---
layout: post
title: GDI+で複数の色を同時に塗り替える(C++)
thumbnail-img: /assets/img/fillmultiplecolor.png
tags: [C++,GDI+,塗り潰し,画像,色]
comments: true
---

ビットマップの色を塗り替えるときに、複数同時に塗り替えられると便利な場合があります。

例えば、赤色と青色が半分半分の画像があり、赤色の部分を青色に、青色の部分を赤色に変えたい場合です。
次のような順序で色を塗り替えるとうまくいきません。
1. 赤色の部分を青色に塗りつぶす
1. 青色の部分を赤色に塗りつぶす
なぜなら、２の時点ですべてが青色になっているからです。

そんな場合は、GDI+のImageAttributesにSetRemapTable関数を使って複数の色の対応を入力してやると、
複数の色を同時に塗り替えることができます。

![fillmultiplecolor.png](/assets/img/fillmultiplecolor.png){: .mx-auto.d-block :}
左の4色をランダムな色に塗り分けるサンプル

[プロジェクトのダウンロード](https://github.com/kenjinote/GdiplusChangeColorsSetRemapTable/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")
#pragma comment(lib,"gdiplus.lib")

#include <windows.h>
#include <gdiplus.h>

using namespace Gdiplus;

TCHAR szClassName[] = TEXT("Window");

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hButton;
  static Bitmap*pBitmapOriginal;
  static Bitmap*pBitmapCurrent;
  switch (msg)
  {
  case WM_CREATE:
    pBitmapOriginal = new Bitmap(256, 256, PixelFormat24bppRGB);
    {
      Graphics g(pBitmapOriginal);
      g.FillRectangle(&SolidBrush(Color(255, 0, 0)), 0, 0, 128, 128);
      g.FillRectangle(&SolidBrush(Color(0, 255, 0)), 128, 0, 128, 128);
      g.FillRectangle(&SolidBrush(Color(0, 0, 255)), 0, 128, 128, 128);
      g.FillRectangle(&SolidBrush(Color(255, 0, 255)), 128, 128, 128, 128);
    }
    pBitmapCurrent = new Bitmap(256, 256, PixelFormat24bppRGB);
    hButton = CreateWindow(TEXT("BUTTON"), TEXT("変換"), WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)IDOK, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    break;
  case WM_PAINT:
    {
      PAINTSTRUCT ps;
      HDC hdc = BeginPaint(hWnd, &ps);
      {
        Graphics g(hdc);
        g.DrawImage(pBitmapOriginal, Rect(10, 10, 256, 256));
        g.DrawImage(pBitmapCurrent, Rect(256 + 20, 10, 256, 256));
      }
      EndPaint(hWnd, &ps);
    }
    break;
  case WM_SIZE:
    MoveWindow(hButton, 10, 256+20, 256, 32, TRUE);
    break;
  case WM_COMMAND:
    if (LOWORD(wParam) == IDOK)
    {
      Gdiplus::ImageAttributes attr;
      Gdiplus::ColorMap *pColorMap = new Gdiplus::ColorMap[4];
      pColorMap[0].oldColor = Color(255, 0, 0);
      pColorMap[0].newColor = Color(rand() % 256, rand() % 256, rand() % 256);
      pColorMap[1].oldColor = Color(0, 255, 0);
      pColorMap[1].newColor = Color(rand() % 256, rand() % 256, rand() % 256);
      pColorMap[2].oldColor = Color(0, 0, 255);
      pColorMap[2].newColor = Color(rand() % 256, rand() % 256, rand() % 256);
      pColorMap[3].oldColor = Color(255, 0, 255);
      pColorMap[3].newColor = Color(rand() % 256, rand() % 256, rand() % 256);
      attr.SetRemapTable(4, pColorMap);
      Graphics g(pBitmapCurrent);
      Gdiplus::RectF rect(0, 0, 256, 256);
      g.DrawImage(pBitmapOriginal, rect, 0, 0, (Gdiplus::REAL)pBitmapOriginal->GetWidth(), (Gdiplus::REAL)pBitmapOriginal->GetHeight(), Gdiplus::UnitPixel, &attr, 0, 0);
      delete[]pColorMap;
      InvalidateRect(hWnd, 0, 1);
    }
    break;
  case WM_DESTROY:
    delete pBitmapOriginal;
    delete pBitmapCurrent;
    PostQuitMessage(0);
    break;
  default:
    return DefWindowProc(hWnd, msg, wParam, lParam);
  }
  return 0;
}

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPreInst, LPSTR pCmdLine, int nCmdShow)
{
  ULONG_PTR gdiToken;
  GdiplusStartupInput gdiSI;
  GdiplusStartup(&gdiToken, &gdiSI, NULL);
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
  GdiplusShutdown(gdiToken);
  return (int)msg.wParam;
}
{% endhighlight %}

### 参考文献
- [ImageAttributes.SetRemapTable メソッド](https://docs.microsoft.com/ja-jp/dotnet/api/system.drawing.imaging.imageattributes.setremaptable)
