---
layout: post
title: Hello World
subtitle: WindowsのGUIでHello Worldを表示する
thumbnail-img: /assets/img/helloworld.png
tags: [C++,Windows,描画,HelloWorld]
---

![helloworld.png](/assets/img/helloworld.png){: .mx-auto.d-block :}

Windows の GUI プログラムで Hello World を表示するサンプルプログラムの紹介です。
Windows で GUI プログラムを作成するうえで、最もシンプルなものとなります。

プログラムが実行されると、まずエントリーポイントの関数である `WinMainCRTStartup` が呼び出されます。その関数内で、
プログラムで必要となるメインウィンドウを作成します。ウィンドウの表示が不要なプログラムであれば作成する必要はありません。

作成したウィンドウで様々なメッセージを独自で処理するために、ウィンドウプロシージャ関数というものが必要となります。
ここでは `WndProc` という名前で関数を書いていますが、エントリーポイントの関数とは異なり任意の関数名を命名することができます。

`WndProc` ウィンドウプロシージャ関数はウィンドウにメッセージが送られたタイミングで呼び出され、第二引数にメッセージの種類と
第三引数、第四引数にはメッセージの保続情報が渡されてきます。この情報を元に様々なメッセージに対してウィンドウでどう処理するか
を記述し、ウィンドウプログラムの作成を行っていきます。

ここでは、ウィンドウのクライアント領域の描画メッセージ `WM_PAINT` の処理を記述することで、ウィンドウのクライアント領域に
`Hello World!`という文字列を表示させるプログラムを作成しました。

[C++コード]
{% highlight cpp linenos %}
#define UNICODE
#pragma comment(linker,"/opt:nowin98")
#include<windows.h>

TCHAR szClassName[]=TEXT("Window");

LRESULT CALLBACK WndProc(HWND hWnd,UINT msg,WPARAM wParam,LPARAM lParam)
{
  switch(msg)
  {
  case WM_PAINT:
    {
      PAINTSTRUCT ps;
      const HDC hdc=BeginPaint(hWnd,&ps);
      RECT rect;
      GetClientRect(hWnd,&rect);
      DrawText(hdc,TEXT("Hello World!"),-1,&rect,DT_CENTER|DT_SINGLELINE|DT_VCENTER);
      EndPaint(hWnd,&ps);
    }
    break;
  case WM_DESTROY:
    PostQuitMessage(0);
    break;
  default:
    return DefWindowProc(hWnd,msg,wParam,lParam);
  }
  return 0;
}

EXTERN_C void __cdecl WinMainCRTStartup()
{
  MSG msg;
  const HINSTANCE hInstance=GetModuleHandle(0);
  const WNDCLASS wndclass={
    CS_HREDRAW|CS_VREDRAW,
    WndProc,
    0,
    0,
    hInstance,
    0,
    LoadCursor(0,IDC_ARROW),
    (HBRUSH)(COLOR_WINDOW+1),
    0,
    szClassName
  };
  RegisterClass(&wndclass);
  const HWND hWnd=CreateWindow(
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
  ShowWindow(hWnd,SW_SHOWDEFAULT);
  UpdateWindow(hWnd);
  while(GetMessage(&msg,0,0,0))
  {
    TranslateMessage(&msg);
    DispatchMessage(&msg);
  }
  ExitProcess(msg.wParam);
}

#if _DEBUG
void main(){}
#endif
{% endhighlight %}
