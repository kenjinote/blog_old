---
layout: post
title: SQLiteを使ったデータベース検索(C++)
subtitle: 英和単語辞典を作成します
thumbnail-img: /assets/img/englishworddictionary.png
tags: [C++,SQLite,データベース,検索,ツール,英和単語辞典]
comments: true
---

### データベースの種類について
データベースを利用する手法はいくつかある。Windows環境であれは下記の手法が主流となっていると思います。

- SQLiteを使う方法
- Access DB（MDB）を使う方法
- MySQLを使う方法
- Microsoft SQL Serverを使う方法

クライアント単独のアプリであれば、単独でデータベースを操作するためのライブラリが存在しているため、
上記の上2つがよいと思います。今回は、SQLiteを使った方法を紹介します。

### SQLiteの導入
SQLiteは下記のURLから必要なソースコードをダウンロードして、プロジェクトに組込めばそのまますぐに使えるようになります。

[SQLite Download Page](https://www.sqlite.org/download.html)

上記の`Source Code`から`sqlite-amalgamation-XXXXXXX.zip`（Xは数字）をダウンロード・解凍して、
`sqlite3.c`と`sqlite3.h`をプロジェクトに追加します。

後は、基本的な使い方は、以下のようになります。
{% highlight cpp linenos %}
// sqliteのインクルード
#include "sqlite3.h"

...

// データベースファイルを開く
sqlite3* db = NULL;
auto status = sqlite3_open_v2("ejdict.sqlite3", &db, SQLITE_OPEN_READONLY, nullptr);

// SQLの実行
char* errmsg;
status = sqlite3_exec(db, "select * from items where word like 'abc%'", callback, nullptr, &errmsg);

// データベースを閉じる
sqlite3_close(db);
{% endhighlight %}

### SQLiteを使ったサンプルプログラム
下記のDBを使って、英和辞書を作成します。

[英和辞書データ ダウンロード](https://kujirahand.com/web-tools/EJDictFreeDL.php)

上記からsqliteのデータベースをダウンロードします。
そのままでは、テーブル名や列名が分からないためsqliteのビューアで確認します。
（sqliteのビューアのお勧めは、[DB Browser for SQLite](https://sqlitebrowser.org/)です。）

![](/assets/img/dbbrowserforsqlite.png){: .mx-auto.d-block :}

テーブル名が`items`、英単語列が`word`、日本語訳が`mean`となっていることが確認できます。

これに基づいてサンプルプログラムを書いてみました。

![](/assets/img/englishworddictionary.png){: .mx-auto.d-block :}

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>
#include "sqlite3.h"

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

LPWSTR a2w(LPCSTR lpszText)
{
  const DWORD dwSize = MultiByteToWideChar(CP_UTF8, 0, lpszText, -1, 0, 0);
  LPWSTR lpszReturnText = (LPWSTR)GlobalAlloc(0, dwSize * sizeof(WCHAR));
  MultiByteToWideChar(CP_UTF8, 0, lpszText, -1, lpszReturnText, dwSize);
  return lpszReturnText;
}

LPSTR w2a(LPCWSTR lpszText)
{
  const DWORD dwSize = WideCharToMultiByte(CP_UTF8, 0, lpszText, -1, 0, 0, 0, 0);
  LPSTR lpszReturnText = (LPSTR)GlobalAlloc(0, dwSize * sizeof(char));
  WideCharToMultiByte(CP_UTF8, 0, lpszText, -1, lpszReturnText, dwSize, 0, 0);
  return lpszReturnText;
}

static int callback(void* hEdit2, int argc, char** argv, char** columnName)
{
  if (argc == 2)
  {
    if (argv[0] && argv[1])
    {
      LPWSTR lpszText1 = a2w(argv[0]);
      SendMessageW((HWND)hEdit2, EM_REPLACESEL, 0, (LPARAM)lpszText1);
      SendMessageW((HWND)hEdit2, EM_REPLACESEL, 0, (LPARAM)L" = ");
      GlobalFree(lpszText1);
      LPWSTR lpszText2 = a2w(argv[1]);
      SendMessageW((HWND)hEdit2, EM_REPLACESEL, 0, (LPARAM)lpszText2);
      SendMessageW((HWND)hEdit2, EM_REPLACESEL, 0, (LPARAM)L"\r\n");
      GlobalFree(lpszText2);
    }
  }
  return SQLITE_OK;
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hButton;
  static HWND hEdit1;
  static HWND hEdit2;
  static HFONT hFont;
  static UINT uDpiX = DEFAULT_DPI, uDpiY = DEFAULT_DPI;
  switch (msg)
  {
  case WM_CREATE:
    hEdit1 = CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("EDIT"), 0, WS_VISIBLE | WS_CHILD | WS_TABSTOP | ES_AUTOHSCROLL, 0, 0, 0, 0, hWnd, 0, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    hButton = CreateWindow(TEXT("BUTTON"), TEXT("検索"), WS_VISIBLE | WS_CHILD | WS_TABSTOP, 0, 0, 0, 0, hWnd, (HMENU)IDOK, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    hEdit2 = CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("EDIT"), 0, WS_VISIBLE | WS_CHILD | WS_HSCROLL | WS_VSCROLL | WS_TABSTOP | ES_READONLY | ES_MULTILINE | ES_AUTOHSCROLL | ES_AUTOVSCROLL, 0, 0, 0, 0, hWnd, 0, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    SendMessage(hWnd, WM_APP, 0, 0);
    break;
  case WM_SIZE:
    MoveWindow(hEdit1, POINT2PIXEL(10), POINT2PIXEL(10), POINT2PIXEL(256), POINT2PIXEL(20), TRUE);
    MoveWindow(hButton, POINT2PIXEL(256 + 20), POINT2PIXEL(10), POINT2PIXEL(100), POINT2PIXEL(20), TRUE);
    MoveWindow(hEdit2, POINT2PIXEL(10), POINT2PIXEL(40), LOWORD(lParam) - POINT2PIXEL(20), HIWORD(lParam) - POINT2PIXEL(50), TRUE);
    break;
  case WM_SETFOCUS:
    SetFocus(hEdit1);
    break;
  case WM_COMMAND:
    if (LOWORD(wParam) == IDOK)
    {
      sqlite3* db = NULL;
      auto status = sqlite3_open_v2("ejdict.sqlite3", &db, SQLITE_OPEN_READONLY, nullptr);
      if (status == SQLITE_OK)
      {
        const int nTextLength = GetWindowTextLength(hEdit1);
        LPWSTR lpszText = (LPWSTR)GlobalAlloc(0, sizeof(WCHAR)*(nTextLength + 1));
        GetWindowText(hEdit1, lpszText, nTextLength + 1);
        LPSTR lpszTextA = w2a(lpszText);
        GlobalFree(lpszText);
        const int nTextLengthA = (int)GlobalSize(lpszTextA);
        LPSTR lpszSQL = (LPSTR)GlobalAlloc(0, nTextLengthA + 100);
        lstrcpyA(lpszSQL, "select word, mean from items where word like '");
        lstrcatA(lpszSQL, lpszTextA);
        lstrcatA(lpszSQL, "%' limit 100");
        SetWindowText((HWND)hEdit2, 0);
        char* errmsg;
        status = sqlite3_exec(db, lpszSQL, callback, (void*)hEdit2, &errmsg);
        if (errmsg)
        {
          LPWSTR lpszText = a2w(errmsg);
          SendMessageW((HWND)hEdit2, EM_REPLACESEL, 0, (LPARAM)lpszText);
          GlobalFree(lpszText);
        }
        GlobalFree(lpszSQL);
        GlobalFree(lpszTextA);
        sqlite3_close(db);
      }
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
    SendMessage(hEdit1, WM_SETFONT, (WPARAM)hFont, 0);
    SendMessage(hEdit2, WM_SETFONT, (WPARAM)hFont, 0);
    break;
  case WM_CLOSE:
    DestroyWindow(hWnd);
    break;
  case WM_DESTROY:
    DeleteObject(hFont);
    PostQuitMessage(0);
    break;
  default:
    return DefDlgProc(hWnd, msg, wParam, lParam);
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
    DLGWINDOWEXTRA,
    hInstance,
    0,
    LoadCursor(0,IDC_ARROW),
    0,
    0,
    szClassName
  };
  RegisterClass(&wndclass);
  HWND hWnd = CreateWindow(
    szClassName,
    TEXT("英和辞書"),
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
    if (!IsDialogMessage(hWnd, &msg))
    {
      TranslateMessage(&msg);
      DispatchMessage(&msg);
    }
  }
  return (int)msg.wParam;
}
{% endhighlight %}

### 参考
- [ドシロウトがcodeanywhereを使ってみた](https://qiita.com/basictomonokai/items/0902058199c99b81fc63)

