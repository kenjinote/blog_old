---
layout: post
title: 文字列のURLエンコード・デコード(C++)
subtitle: Web検索するときのURL生成などに
thumbnail-img: /assets/img/urlencode.png
tags: [C++,文字列変換,URLエンコード,ツール]
comments: true
---

URLエンコード・デコード（別名パーセントエンコーディング）を行うツールです。
`UTF-8`の文字コードで符号化したうえで、1文字ずつ変換を行っていきます。

![](/assets/img/urlencode.png){: .mx-auto.d-block :}


[プロジェクトのダウンロード](https://github.com/kenjinote/UrlEncode/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#pragma comment(lib, "shlwapi")

#include <windows.h>
#include <shlwapi.h>

////////////////////////////////////////////
// 入力されたUnicode文字列のURLエンコード行う
// lpszSrc: URLエンコード対象の文字列(Unicode)
// lpszDst: URLエンコード出力の文字列(Unicode)バッファ、NULL指定可
// 戻り値: 出力バッファの文字列(Unicode)長、NULLを含む
////////////////////////////////////////////
int UrlEncode(LPCWSTR lpszSrc, LPWSTR lpszDst)
{
  DWORD iDst = 0;
  const DWORD dwTextLengthA = WideCharToMultiByte(CP_UTF8, 0, lpszSrc, -1, 0, 0, 0, 0);
  LPSTR szUTF8TextA = (LPSTR)GlobalAlloc(GMEM_FIXED, dwTextLengthA); // NULL を含んだ文字列バッファを確保
  if (szUTF8TextA)
  {
    if (WideCharToMultiByte(CP_UTF8, 0, lpszSrc, -1, szUTF8TextA, dwTextLengthA, 0, 0))
    {
      for (DWORD iSrc = 0; iSrc < dwTextLengthA && szUTF8TextA[iSrc] != '\0'; ++iSrc)
      {
        LPCSTR lpszUnreservedCharacters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-._~";
        if (StrChrA(lpszUnreservedCharacters, szUTF8TextA[iSrc]))
        {
          if (lpszDst) lpszDst[iDst] = (WCHAR)szUTF8TextA[iSrc];
          ++iDst;
        }
        else if (szUTF8TextA[iSrc] == ' ')
        {
          if (lpszDst) lpszDst[iDst] = L'+';
          ++iDst;
        }
        else
        {
          if (lpszDst) wsprintfW(&lpszDst[iDst], L"%%%02X", szUTF8TextA[iSrc] & 0xFF);
          iDst += 3;
        }
      }
      if (lpszDst) lpszDst[iDst] = L'\0';
      ++iDst;
    }
    GlobalFree(szUTF8TextA);
  }
  return iDst; // NULL 文字を含む
}

////////////////////////////////////////////
// 入力されたUnicode文字列のURLデーコード行う
// lpszSrc: URLデコード対象の文字列(Unicode)
// lpszDst: URLデコード出力の文字列(Unicode)バッファ、NULL指定可
// 戻り値: 出力バッファの文字列(Unicode)長、NULLを含む
////////////////////////////////////////////
int UrlDecode(LPCWSTR lpszSrc, LPWSTR lpszDst)
{
  const int nSrcLength = lstrlenW(lpszSrc);
  int iSrc, iBuf;
  for (iSrc = 0, iBuf = 0; iSrc < nSrcLength; ++iSrc)
  {
    if (lpszSrc[iSrc] == L'%' && isxdigit(lpszSrc[iSrc + 1]))
    {
      iSrc += 2;
    }
    ++iBuf;
  }
  LPSTR szUTF8TextA = (LPSTR)GlobalAlloc(GMEM_FIXED, iBuf + 1); // NULLを含める
  for (iSrc = 0, iBuf = 0; iSrc < nSrcLength; ++iSrc)
  {
    if (lpszSrc[iSrc] == L'%' && isxdigit(lpszSrc[iSrc + 1]))
    {
      WCHAR num[3] = { lpszSrc[iSrc + 1], lpszSrc[iSrc + 2], L'\0' };
      szUTF8TextA[iBuf] = (CHAR)wcstol(num, 0, 16);
      iSrc += 2;
    }
    else if (lpszSrc[iSrc] == L'+')
    {
      szUTF8TextA[iBuf] = ' ';
    }
    else
    {
      szUTF8TextA[iBuf] = (CHAR)lpszSrc[iSrc];
    }
    ++iBuf;
  }
  szUTF8TextA[iBuf] = 0;
  const DWORD iDst = MultiByteToWideChar(CP_UTF8, 0, szUTF8TextA, -1, 0, 0); // NULL 文字を含む
  if (lpszDst)
  {
    MultiByteToWideChar(CP_UTF8, 0, szUTF8TextA, -1, lpszDst, iDst);
  }
  GlobalFree(szUTF8TextA);
  return iDst; // NULL 文字を含む
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hEdit1, hEdit2, hButton1, hButton2;
  static HFONT hFont;
  switch (msg)
  {
  case WM_CREATE:
    hFont = CreateFont(32, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, TEXT("Segoe UI"));
    hEdit1 = CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("EDIT"), 0, WS_CHILD | WS_VISIBLE | WS_VSCROLL | ES_MULTILINE, 10, 10, 512, 256, hWnd, 0, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    hEdit2 = CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("EDIT"), 0, WS_CHILD | WS_VISIBLE | WS_VSCROLL | ES_MULTILINE, 10, 316, 512, 256, hWnd, 0, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    hButton1 = CreateWindow(TEXT("BUTTON"), TEXT("↓エンコード"), WS_CHILD | WS_VISIBLE, 10, 276, 128, 30, hWnd, (HMENU)100, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    hButton2 = CreateWindow(TEXT("BUTTON"), TEXT("↑デコード"), WS_CHILD | WS_VISIBLE, 148, 276, 128, 30, hWnd, (HMENU)101, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    SendMessage(hEdit1, WM_SETFONT, (WPARAM)hFont, 0);
    SendMessage(hEdit2, WM_SETFONT, (WPARAM)hFont, 0);
    SendMessage(hButton1, WM_SETFONT, (WPARAM)hFont, 0);
    SendMessage(hButton2, WM_SETFONT, (WPARAM)hFont, 0);
    break;
  case WM_COMMAND:
    switch (LOWORD(wParam))
    {
    case 100:
    {
      const DWORD dwTextLength = GetWindowTextLengthW(hEdit1);
      if (dwTextLength)
      {
        LPWSTR lpszEditText = (LPWSTR)GlobalAlloc(GMEM_FIXED, sizeof(WCHAR)*(dwTextLength + 1));
        if (lpszEditText)
        {
          GetWindowTextW(hEdit1, lpszEditText, dwTextLength + 1);
          SetWindowText(hEdit2, 0);
          const int nSize = UrlEncode(lpszEditText, 0);
          if (nSize)
          {
            LPWSTR lpszEncodeText = (LPWSTR)GlobalAlloc(0, sizeof(WCHAR)*nSize);
            if (lpszEncodeText)
            {
              UrlEncode(lpszEditText, lpszEncodeText);
              SetWindowTextW(hEdit2, lpszEncodeText);
              GlobalFree(lpszEncodeText);
            }
          }
          GlobalFree(lpszEditText);
        }
      }
    }
    break;
    case 101:
    {
      const DWORD dwTextLength = GetWindowTextLengthW(hEdit2);
      if (dwTextLength)
      {
        LPWSTR lpszEditText = (LPWSTR)GlobalAlloc(GMEM_FIXED, sizeof(WCHAR)*(dwTextLength + 1));
        if (lpszEditText)
        {
          GetWindowTextW(hEdit2, lpszEditText, dwTextLength + 1);
          SetWindowText(hEdit1, 0);
          const int nSize = UrlDecode(lpszEditText, 0);
          if (nSize)
          {
            LPWSTR lpszDecodeText = (LPWSTR)GlobalAlloc(0, sizeof(WCHAR)*nSize);
            if (lpszDecodeText)
            {
              UrlDecode(lpszEditText, lpszDecodeText);
              SetWindowTextW(hEdit1, lpszDecodeText);
              GlobalFree(lpszDecodeText);
            }
          }
          GlobalFree(lpszEditText);
        }
      }
    }
    break;
    }
    break;
  case WM_DESTROY:
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
  TCHAR szClassName[] = TEXT("Window");
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
  RECT rect = { 0,0,532,582 };
  DWORD dwStyle = WS_CAPTION | WS_SYSMENU | WS_CLIPCHILDREN;
  AdjustWindowRect(&rect, dwStyle, FALSE);
  HWND hWnd = CreateWindow(
    szClassName,
    TEXT("Unicode 文字列の URL エンコード"),
    dwStyle,
    CW_USEDEFAULT,
    0,
    rect.right - rect.left,
    rect.bottom - rect.top,
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

### 参考文献
- [http://www.ietf.org/rfc/rfc3986.txt](http://www.ietf.org/rfc/rfc3986.txt)

Google のサイトで入力文字列の URL エンコードができるサイトがあります。（他にも Base64 や MD5 などへの変換もできるみたいですね）

[https://toolbox.googleapps.com/apps/encode_decode/](https://toolbox.googleapps.com/apps/encode_decode/)

ここでプログラムの出力が正しいかチェックできます。