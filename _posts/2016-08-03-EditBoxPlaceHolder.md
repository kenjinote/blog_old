---
layout: post
title: EditBoxにヒント（プレースフォルダー）を表示する(C++)
thumbnail-img: /assets/img/editboxplaceholder.png
tags: [C++,Editbox]
comments: true
---

## 自前で入力ヒントを描画する方法（サブクラス化）

![](/assets/img/editboxplaceholder.png){: .mx-auto.d-block :}

エディットボックスの初期表示(エディットボックスに何も入力されていないとき）に何を入力すべきかユーザーに示すために薄くヒントとなる文字列（プレースフォルダー）を表示するサンプルです。
エディットボックスのウィンドウプロシージャを独自に処理して、入力ヘルプを描画する方法です。

[プロジェクトのダウンロード](https://github.com/kenjinote/EditBoxPlaceHolder/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>
#include <tchar.h>

TCHAR szClassName[] = TEXT("Window");

class EditBoxEx
{
  HWND m_hWnd;
  WNDPROC EditWndProc;
  LPTSTR m_lpszPlaceholder;
  static LRESULT CALLBACK EditProc1(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
  {
    EditBoxEx* _this = (EditBoxEx*)GetWindowLongPtr(hWnd, GWLP_USERDATA);
    if (_this)
    {
      switch (msg)
      {
      case WM_PAINT:
        if (GetWindowTextLength(hWnd) == 0)
        {
          PAINTSTRUCT ps;
          const HDC hdc = BeginPaint(hWnd, &ps);
          SetTextColor(hdc, RGB(169, 169, 169));
          RECT rect;
          GetClientRect(hWnd, &rect);
          const HFONT hOldFont = (HFONT)SelectObject(hdc, (HFONT)SendMessage(hWnd, WM_GETFONT, 0, 0));
          DrawText(hdc, _this->m_lpszPlaceholder, -1, &rect, DT_SINGLELINE | DT_VCENTER);
          SelectObject(hdc, hOldFont);
          EndPaint(hWnd, &ps);
          return 0;
        }
        break;
      default:
        break;
      }
      return CallWindowProc(_this->EditWndProc, hWnd, msg, wParam, lParam);
    }
    return 0;
  }
public:
  EditBoxEx() : m_hWnd(NULL), EditWndProc(NULL), m_lpszPlaceholder(NULL){}
  ~EditBoxEx() { DestroyWindow(m_hWnd); GlobalFree(m_lpszPlaceholder); }
  HWND Create(HWND hParent, int x, int y, int width, int height, LPCTSTR lpszPlaceholder)
  {
    if (lpszPlaceholder && !m_lpszPlaceholder)
    {
      m_lpszPlaceholder = (LPTSTR)GlobalAlloc(0, sizeof(TCHAR) * (lstrlen(lpszPlaceholder) + 1));
      lstrcpy(m_lpszPlaceholder, lpszPlaceholder);
    }
    m_hWnd = CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("EDIT"), 0, WS_VISIBLE | WS_CHILD, x, y, width, height, hParent, 0, GetModuleHandle(0), 0);
    SetWindowLongPtr(m_hWnd, GWLP_USERDATA, (LONG_PTR)this);
    EditWndProc = (WNDPROC)SetWindowLongPtr(m_hWnd, GWLP_WNDPROC, (LONG_PTR)EditProc1);
    return m_hWnd;
  }
};

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HFONT hFont;
  static EditBoxEx* pEdit1;
  static EditBoxEx* pEdit2;
  switch (msg)
  {
  case WM_CREATE:
    hFont = CreateFont(26, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, TEXT("メイリオ"));
    pEdit1 = new EditBoxEx();
    if (!pEdit1) return -1;
    {
      const HWND hEdit1 = pEdit1->Create(hWnd, 10, 10, 256, 32, TEXT("名前"));
      SendMessage(hEdit1, WM_SETFONT, (LPARAM)hFont, 1);
    }
    pEdit2 = new EditBoxEx();
    if (!pEdit2) return -1;
    {
      const HWND hEdit2 = pEdit2->Create(hWnd, 10, 50, 256, 32, TEXT("メールアドレス"));
      SendMessage(hEdit2, WM_SETFONT, (LPARAM)hFont, 1);
    }
    break;
  case WM_DESTROY:
    delete pEdit1;
    delete pEdit2;
    DeleteObject(hFont);
    PostQuitMessage(0);
    break;
  default:
    return DefWindowProc(hWnd, msg, wParam, lParam);
  }
  return 0;
}

int WINAPI _tWinMain(HINSTANCE hInstance, HINSTANCE hPreInst, LPTSTR pCmdLine, int nCmdShow)
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

## APIで実現する方法

{: .box-note}
**注意** この方法は、Comclt32.dllバージョン6.0が必要となります。

![](/assets/img/editboxbanner.png){: .mx-auto.d-block :}

EditBoxのメッセージ`EM_SETCUEBANNER`を使うと、グレーの入力ヘルプを簡単に表示させることができます。

（ただ、文字色の指定が出来ないので、色を独自に変えるには上記で紹介したエディットボックスのサブクラス化を行う必要があると思います。）

{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#pragma comment(lib, "comctl32")

#include <windows.h>
#include <commctrl.h>

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hEdit;
  static HWND hButton1;
  static HWND hButton2;
  switch (msg)
  {
  case WM_CREATE:
    hEdit = CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("EDIT"), 0, WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, 0, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    hButton1 = CreateWindow(TEXT("BUTTON"), TEXT("入力補助テキストを設定"), WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)IDOK, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    hButton2 = CreateWindow(TEXT("BUTTON"), TEXT("入力補助テキストを解除"), WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)IDCANCEL, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    break;
  case WM_SIZE:
    MoveWindow(hEdit, 10, 10, 512, 32, TRUE);
    MoveWindow(hButton1, 10, 50, 256, 32, TRUE);
    MoveWindow(hButton2, 10, 90, 256, 32, TRUE);
    break;
  case WM_COMMAND:
    if (LOWORD(wParam) == IDOK)
    {
      SendMessage(hEdit, EM_SETCUEBANNER, TRUE, (LPARAM)TEXT("メールアドレスを入力してください"));
    }
    else if (LOWORD(wParam) == IDCANCEL)
    {
      SendMessage(hEdit, EM_SETCUEBANNER, TRUE, (LPARAM)TEXT(""));
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
  TCHAR szClassName[] = TEXT("Window");
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
    TEXT("エディットボックスでユーザーに情報の入力を促すためのヒントテキスト（グレー）を表示する"),
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
  MSG msg;
  while (GetMessage(&msg, 0, 0, 0))
  {
    TranslateMessage(&msg);
    DispatchMessage(&msg);
  }
  return (int)msg.wParam;
}
{% endhighlight %}

### 参考文献
- [EM _ setcuebanner メッセージ](https://docs.microsoft.com/ja-jp/windows/win32/controls/em-setcuebanner)
- [視覚スタイルを有効にする](https://docs.microsoft.com/ja-jp/windows/win32/controls/cookbook-overview)