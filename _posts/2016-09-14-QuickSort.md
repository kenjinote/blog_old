---
layout: post
title: クイックソート(C++)
thumbnail-img: /assets/img/quicksort.png
tags: [C++,Windows,アルゴリズム]
comments: true
---

クイックソートは一般艇に最も多く使われているソートアルゴリズムです。

大まかな処理は、ソートする要素の中から「適当な値」を決め、その値より大きいものと小さいものに分け並べる、
分けられた 2 つのグループに対しても上記のソートを適用し、さらに細かく分けられたグループに対してソートしていく・・・
最終的にはすべての要素についてソートが完了しています。

[プロジェクトのダウンロード](https://github.com/kenjinote/QuickSort/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>

TCHAR szClassName[] = TEXT("Window");
#define N 10

void QuickSort(int start, int end, int *data)
{
  int lower, upper, div, temp;
  if (start >= end) {
    return;
  }
  // 先頭の値を「適当な値」とする
  div = data[start];
  for (lower = start, upper = end; lower < upper;) {
    while (lower <= upper && data[lower] <= div) {
      lower++;
    }
    while (lower <= upper&&data[upper] > div) {
      upper--;
    }
    if (lower < upper) {
      temp = data[lower];
      data[lower] = data[upper];
      data[upper] = temp;
    }
  }
  //最初に選択した値を中央に移動する
  temp = data[start];
  data[start] = data[upper];
  data[upper] = temp;
  QuickSort(start, upper - 1, data);
  QuickSort(upper + 1, end, data);
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static int sort[N];
  static HWND hButton;
  switch (msg)
  {
  case WM_CREATE:
    hButton = CreateWindow(TEXT("BUTTON"), TEXT("ソート"), WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)IDOK, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    for (int i = 0; i < N; i++)
    {
      sort[i] = rand() % N;
    }
    break;
  case WM_PAINT:
  {
    PAINTSTRUCT ps;
    HDC hdc = BeginPaint(hWnd, &ps);
    TCHAR szText[256];
    for (int i = 0; i < N; i++)
    {
      wsprintf(szText, TEXT("%d"), sort[i]);
      TextOut(hdc, 10, i * 32 + 50, szText, lstrlen(szText));
    }
    EndPaint(hWnd, &ps);
  }
  break;
  case WM_SIZE:
    MoveWindow(hButton, 10, 10, 256, 32, TRUE);
    break;
  case WM_COMMAND:
    if (LOWORD(wParam) == IDOK)
    {
      QuickSort(0, N-1, sort);
      InvalidateRect(hWnd, 0, 1);
      MessageBox(hWnd, TEXT("ソートしました"), TEXT("確認"), 0);
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
    TEXT("クイックソート"),
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

### 参考サイト
- [プログラミングの宝箱 アルゴリズムとデータ構造 第2版](http://amzn.to/2bNtoDH)

