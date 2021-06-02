---
layout: post
title: Universal CRTがインストールされているかどうかチェックする(C++)
subtitle: コアとなるdllがロードできるか
thumbnail-img: /assets/img/isuniversalcrtinstalled.png
tags: [C++,Universal CRT,dll]
comments: true
---

Windows 10 SDK を使って開発されたアプリケーションを動作させるためには、システムに Universal CRT (Windows 10 Universal C Runtime) がインストールされている必要があります。

そこで Universal CRT がインストールされているかどうか確認するための2とおりの方法を確認するプログラムを書いてみました。

- 確認方法１　レジストリに特定の文字列が含まれているか確認する方法
- 確認方法２　`ucrtbase.dll`がロードできるかどうかで確認する方法

![](/assets/img/isuniversalcrtinstalled.png){: .mx-auto.d-block :}

[プロジェクトのダウンロード](https://github.com/kenjinote/IsUniversalCRTInstalled/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>
#include <vector>
#include <string>

TCHAR szClassName[] = TEXT("Window");

// レジストリを参照し、Universal CRT がインストールされているかチェックする
BOOL IsUniversalCRTInstalled1()
{
  LPCWSTR lpszSearchPath = L"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\SideBySide\\Winners";
  std::vector<std::wstring>keylist;
  WCHAR szTemp[1024];
  HKEY hKey;
  if (RegOpenKeyExW(HKEY_LOCAL_MACHINE, lpszSearchPath, 0, KEY_READ | KEY_WOW64_64KEY, &hKey) == ERROR_SUCCESS)
  {
    DWORD cSubKeys = 0;
    if (RegQueryInfoKeyW(hKey, 0, 0, 0, &cSubKeys, 0, 0, 0, 0, 0, 0, 0) == ERROR_SUCCESS && cSubKeys)
    {
      for (DWORD i = 0; i < cSubKeys; ++i)
      {
        DWORD cbName = _countof(szTemp);
        if (RegEnumKeyExW(hKey, i, szTemp, &cbName, 0, 0, 0, 0) == ERROR_SUCCESS)
        {
          for (auto architecture : { L"x86", L"amd64", L"wow64" })
          {
            if (szTemp[0] != architecture[0]) continue;
            WCHAR szKey[1024];
            lstrcpyW(szKey, architecture);
            lstrcatW(szKey, L"_microsoft-windows-ucrt_31bf3856ad364e35");
            const int nKeyLength = lstrlenW(szKey);
            if (CompareStringW(LOCALE_SYSTEM_DEFAULT, NORM_IGNORECASE, szKey, nKeyLength, szTemp, nKeyLength) == CSTR_EQUAL)
            {
              keylist.push_back(szTemp);
            }
          }
        }
      }
    }
    RegCloseKey(hKey);
  }
  if (keylist.size() == 0)
  {
    return FALSE;
  }
  for (auto key : keylist)
  {
    lstrcpyW(szTemp, lpszSearchPath);
    lstrcatW(szTemp, L"\\");
    lstrcatW(szTemp, key.c_str());
    if (RegOpenKeyExW(HKEY_LOCAL_MACHINE, szTemp, 0, KEY_READ | KEY_WOW64_64KEY, &hKey) == ERROR_SUCCESS)
    {
      DWORD dwType = REG_SZ;
      DWORD cbName = _countof(szTemp);
      if (RegQueryValueEx(hKey, 0, 0, &dwType, (LPBYTE)szTemp, &cbName) == ERROR_SUCCESS)
      {
        if (lstrlenW(szTemp) > 0)
        {
          RegCloseKey(hKey);
          return TRUE;
        }
      }
      RegCloseKey(hKey);
    }
  }
  return FALSE;
}

// Universal CRT のモジュール（ucrtbase.dll）がロードできるかどうかチェックする
BOOL IsUniversalCRTInstalled2()
{
  BOOL bIsInstalled = FALSE;
  HMODULE hModule = LoadLibrary(TEXT("ucrtbase.dll"));
  if (hModule)
  {
    bIsInstalled = TRUE;
    FreeLibrary(hModule);
  }
  return bIsInstalled;
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hButton1;
  static HWND hButton2;
  switch (msg)
  {
  case WM_CREATE:
    hButton1 = CreateWindow(TEXT("BUTTON"), TEXT("Universal CRT がインストールされているかどうか判定 その①"), WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)1000, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    hButton2 = CreateWindow(TEXT("BUTTON"), TEXT("Universal CRT がインストールされているかどうか判定 その②"), WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)1001, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    break;
  case WM_SIZE:
    MoveWindow(hButton1, 10, 10, 512, 32, TRUE);
    MoveWindow(hButton2, 10, 50, 512, 32, TRUE);
    break;
  case WM_COMMAND:
    if (LOWORD(wParam) == 1000)
    {
      MessageBox(hWnd, IsUniversalCRTInstalled1() ? TEXT("Universal CRT はインストールされています。") : TEXT("Universl CRT はインストールされていません。"), TEXT("確認"), 0);
    }
    else if (LOWORD(wParam) == 1001)
    {
      MessageBox(hWnd, IsUniversalCRTInstalled2() ? TEXT("Universal CRT はインストールされています。") : TEXT("Universl CRT はインストールされていません。"), TEXT("確認"), 0);
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
    TEXT("Universal CRT がインストールされているかどうか判定"),
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

Universal CRT のインストールは、Windows Update (KB2999226, KB3118401など)をあてるか、Visual Studio 2015移行をインストールすることでインストールされるようです。

###   参考
- [Windows での汎用の C ランタイムの更新プログラム](https://support.microsoft.com/ja-jp/topic/windows-%E3%81%A7%E3%81%AE%E6%B1%8E%E7%94%A8%E3%81%AE-c-%E3%83%A9%E3%83%B3%E3%82%BF%E3%82%A4%E3%83%A0%E3%81%AE%E6%9B%B4%E6%96%B0%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%A0-c0514201-7fe6-95a3-b0a5-287930f3560c)
