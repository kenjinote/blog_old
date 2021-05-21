---
layout: post
title: ビットマップボタン(C++)
thumbnail-img: /assets/img/bitmapbutton.png
tags: [C++,button,bitmap]
comments: true
---

任意のビットマップの背景を持つボタンのサンプルプログラムです。

実装方針は、普通のボタンを作成し、ボタンのウィンドウプロシージャを独自に書き換えます。その際に、
`WM_PRINTCLIENT`メッセージと`WM_ERASEBKGND`メッセージに対してビットマップを描画する処理を記述します。

![](/assets/img/bitmapbutton.png){: .mx-auto.d-block :}

[プロジェクトのダウンロード](https://github.com/kenjinote/BitmapButton/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>
#include "resource.h"

TCHAR szClassName[] = TEXT("Window");

class CBitmapButton
{
public:
  CBitmapButton() :
  m_bHover(0),
  m_bPush(0),
  m_bDisable(0),
  m_hBmpNorm(0),
  m_hBmpSel(0),
  m_hBmpHover(0),
  m_hBmpDis(0) {}
  ~CBitmapButton() {
    DestroyWindow(m_hWnd);
    DeleteObject(m_hBmpNorm);
    DeleteObject(m_hBmpSel);
    DeleteObject(m_hBmpHover);
    DeleteObject(m_hBmpDis);
  }
  HWND CreateButton(
    int x,               // ウィンドウの横方向の位置
    int y,               // ウィンドウの縦方向の位置
    int nWidth,          // ウィンドウの幅
    int nHeight,         // ウィンドウの高さ
    HWND hWndParent,     // 親ウィンドウまたはオーナーウィンドウのハンドル
    HMENU hMenu,         // メニューハンドルまたは子ウィンドウ ID
    HINSTANCE hInstance, // アプリケーションインスタンスのハンドル
    int nNorm,           // ノーマルのときのリソースビットマップID
    int nSel,            // 押したときのリソースビットマップID
    int nHover,          // ホバーのときのリソースビットマップID
    int nDis             // ディセーブルのときのリソースビットマップID
  ) {
    m_hBmpNorm = LoadBitmap(hInstance, MAKEINTRESOURCE(nNorm));
    m_hBmpSel = LoadBitmap(hInstance, MAKEINTRESOURCE(nSel));
    m_hBmpHover = LoadBitmap(hInstance, MAKEINTRESOURCE(nHover));
    m_hBmpDis = LoadBitmap(hInstance, MAKEINTRESOURCE(nDis));
    m_nWidth = nWidth;
    m_nHeight = nHeight;
    m_hWnd = CreateWindow(TEXT("BUTTON"), 0, WS_CHILD | WS_VISIBLE | BS_OWNERDRAW,
      x, y, nWidth, nHeight, hWndParent, hMenu, hInstance, 0);
    if (!m_hWnd) return NULL;
    SetClassLong(m_hWnd, GCL_STYLE, GetClassLong(m_hWnd, GCL_STYLE) & ~CS_DBLCLKS);
    SetWindowLongPtr(m_hWnd, GWLP_USERDATA, (LONG_PTR)this); // ウィンドウにインスタンスのポインタを保持しておく
    m_DefBtnProc = (WNDPROC)SetWindowLongPtr(m_hWnd, GWLP_WNDPROC, (LONG_PTR)GlobalButtonProc);
    return m_hWnd;
  }
private:
  HWND m_hWnd;
  int m_nWidth;      // ボタンの幅
  int m_nHeight;      // ボタンの高さ
  HBITMAP m_hBmpNorm;    // 通常のボタン画像
  HBITMAP m_hBmpSel;    // マウスダウン時のボタン画像
  HBITMAP m_hBmpHover;  // マウスオーバー時のボタン画像
  HBITMAP m_hBmpDis;    // 無効状態(ディセーブル）時のボタン画像
  WNDPROC m_DefBtnProc;  // ボタンのデフォルトウィンドウプロシージャを保持
  BOOL m_bHover;      // ホーバー状態かどうかのフラグ
  BOOL m_bPush;      // プッシュ状態かどうかのフラグ
  BOOL m_bDisable;    // 無効状態(ディセーブル）かどうかのフラグ

  virtual LRESULT CBitmapButton::LocalButtonProc(
    HWND hWnd,
    UINT msg,
    WPARAM wParam,
    LPARAM lParam
  ) {
    switch (msg) {
    case WM_PRINTCLIENT: // これを処理しないとAnimateWindow()が動作しない
    case WM_ERASEBKGND: // 背景を描画
    {
      HDC hdc = CreateCompatibleDC((HDC)wParam);
      if (m_bDisable) {
        SelectObject(hdc, m_hBmpDis);
      } else if (m_bPush) {
        SelectObject(hdc, m_hBmpSel);
      } else {
        if (m_bHover) {
          SelectObject(hdc, m_hBmpHover);
        } else {
          SelectObject(hdc, m_hBmpNorm);
        }
      }
      BitBlt((HDC)wParam, 0, 0, m_nWidth, m_nHeight, hdc, 0, 0, SRCCOPY);
      DeleteDC(hdc);
    }
    return 1;
    case WM_ENABLE: // 無効・有効状態の切り分け
      m_bDisable = !(BOOL)wParam;
      InvalidateRect(hWnd, 0, TRUE);
      break;
    case WM_LBUTTONDOWN:
      m_bPush = TRUE;
      m_bHover = FALSE;
      InvalidateRect(hWnd, 0, TRUE);
      break;
    case WM_LBUTTONUP:
      m_bPush = FALSE;
      InvalidateRect(hWnd, 0, TRUE);
      break;
    case WM_MOUSEMOVE:
      if (!m_bPush && !m_bHover) {
        m_bHover = TRUE;
        TRACKMOUSEEVENT  tme;
        tme.cbSize = sizeof(tme);
        tme.dwFlags = TME_LEAVE;
        tme.hwndTrack = hWnd;
        TrackMouseEvent(&tme);
        InvalidateRect(hWnd, 0, TRUE);
      }
      break;
    case WM_MOUSELEAVE:
      m_bHover = FALSE;
      InvalidateRect(hWnd, 0, TRUE);
      break;
    case WM_DESTROY:
      m_hWnd = NULL;
      break;
    }
    return CallWindowProc(m_DefBtnProc, hWnd, msg, wParam, lParam);
  }
  static LRESULT CALLBACK CBitmapButton::GlobalButtonProc(
    HWND hWnd,
    UINT msg,
    WPARAM wParam,
    LPARAM lParam
  ) {
    CBitmapButton *p = (CBitmapButton*)GetWindowLongPtr(hWnd, GWLP_USERDATA);
    if (p) {
      return p->LocalButtonProc(hWnd, msg, wParam, lParam);
    }
    return 0;
  }
};

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static CBitmapButton Btn1;
  switch (msg) {
  case WM_CREATE:
    Btn1.CreateButton(10, 10, 120, 34, hWnd, (HMENU)IDOK,
      ((LPCREATESTRUCT)(lParam))->hInstance,
      IDB_BITMAP1, IDB_BITMAP2, IDB_BITMAP3, IDB_BITMAP4);
    break;
  case WM_COMMAND:
    if (LOWORD(wParam) == IDOK) {
      MessageBox(hWnd, TEXT("ボタンが押されました。"), 0, 0);
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
  WNDCLASS wndclass = { CS_HREDRAW | CS_VREDRAW, WndProc, 0, 0, hInstance, 0,
    LoadCursor(0,IDC_ARROW), (HBRUSH)(COLOR_WINDOW + 1), 0, szClassName };
  RegisterClass(&wndclass);
  HWND hWnd = CreateWindow(szClassName, TEXT("Window"), WS_OVERLAPPEDWINDOW,
    CW_USEDEFAULT, 0, CW_USEDEFAULT, 0, 0, 0, hInstance, 0);
  ShowWindow(hWnd, SW_SHOWDEFAULT);
  UpdateWindow(hWnd);
  while (GetMessage(&msg, 0, 0, 0)) {
    TranslateMessage(&msg);
    DispatchMessage(&msg);
  }
  return (int)msg.wParam;
}
{% endhighlight %}