---
layout: post
title: バブルソート
subtitle: 実行時間が膨らむ泡のような並べ替え
thumbnail-img: /assets/img/bubblesort.png
tags: [C++,ソート,アルゴリズム]
comments: true
---

複数の要素を大きさの順に並び替えるアルゴリズムの一つにバブルソートがあります。

ソートの手順は次のようなものです。  
例えば、２，４、３、１という数字の並びを小さい順に並べるとき、

操作①　第1の数字２と第2の数字４とを比較して小さい順なのでそのままにする。  
操作②　第2の数字４と第3の数字３とを比較して小さい順ではないので３と４を入れ替える。  
操作③　第3の数字１と第4の数字１とを比較して小さい順ではないので１と４を入れ替える。  
操作④　初めに戻り、第1の数字２と第2の数字３とを比較して小さい順なのでそのままにする。  
操作⑤　第2の数字３と第3の数字１とを比較して小さい順ではないので１と３を入れ替える。  
操作⑥　初めに戻り、第1の数字２と第2の数字１とを比較して小さい順ではないので１と２を入れ替える。  
上記の操作でソートが完了します。  

並び替えるアルゴリズムの中で最も実装がシンプルですが、要素の数が増えたときに並び替えにかかる時間の増加が大きいので多数の要素の場合の並び替えには向きません。
上記の例では4つの数字の場合（3+2+1）の6回の走査が必要で、100つの数字の場合は(99+98+...+1)の4950回の走査が必要になります。一般にNつの数字の場合はN*(N-1)/2回の走査が必要です。

以下バブルソートを行うプログラムを作成しました。

[プロジェクトのダウンロード](https://github.com/kenjinote/BubbleSort/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>

TCHAR szClassName[] = TEXT("Window");
#define N 10

void BubleSort(int* data, int size)
{
  BOOL bFlag;
  int k = 0;
  do {
    bFlag = FALSE;
    for (int i = 0; i < size - 1 - k; i++)
    {
      if (data[i] > data[i + 1])
      {
        bFlag = TRUE;
        const int nTmp = data[i];
        data[i] = data[i + 1];
        data[i + 1] = nTmp;
      }
    }
    k++;
  } while (bFlag);
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static int data[N];
  static HWND hButton;
  switch (msg)
  {
  case WM_CREATE:
    hButton = CreateWindow(TEXT("BUTTON"), TEXT("ソート"), WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)IDOK, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    for (int i = 0; i < N; i++)
    {
      data[i] = rand() % N;
    }
    break;
  case WM_PAINT:
    {
      PAINTSTRUCT ps;
      HDC hdc = BeginPaint(hWnd, &ps);
      TCHAR szText[256];
      for (int i = 0; i < N; i++)
      {
        wsprintf(szText, TEXT("%d"), data[i]);
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
      BubleSort(data, N);
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
    TEXT("バブルソート"),
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

###   参考:
- [プログラミングの宝箱 アルゴリズムとデータ構造 第2版](http://amzn.to/2bNtoDH)
