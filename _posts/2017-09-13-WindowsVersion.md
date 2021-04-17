---
layout: post
title: Windowsのバージョンを取得(C++)
thumbnail-img: /assets/img/windowsversion.png
tags: [C++,Windows]
comments: true
---

Windowsの内部バージョンを取得するには、GetVersion 関数が使えるが、この関数はWindows 8 以降マニフェストで対応OSを明記していないと正しい値を返さない。

代わりに、ntdll.dllに定義されている RtlGetVersion 関数を使うことでマニフェストで明記しなくても正しくOSのバージョンを取得できるようです。
以下のプログラムでは、GetVersion 関数を使ってOSのバージョンを取得し、もし RtlGetVersion 関数が使える場合は OS のバージョンを上書きして画面上に表示するサンプルです。

[プロジェクトのダウンロード](https://github.com/kenjinote/GetWinVer/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>

void GetWindowsVersion(HWND hEdit)
{
  DWORD dwVersion = GetVersion();
  DWORD dwMajorVersion = (DWORD)(LOBYTE(LOWORD(dwVersion)));
  DWORD dwMinorVersion = (DWORD)(HIBYTE(LOWORD(dwVersion)));
  DWORD dwBuild = 0;

  if (dwVersion < 0x80000000)
    dwBuild = (DWORD)(HIWORD(dwVersion));

  // RtlGetVersion 関数が使える場合は使う
  const HMODULE hModule = GetModuleHandle(TEXT("ntdll.dll"));
  if (hModule)
  {
    typedef void (WINAPI*fnRtlGetVersion)(OSVERSIONINFOEXW*);
    fnRtlGetVersion RtlGetVersion = (fnRtlGetVersion)GetProcAddress(hModule, "RtlGetVersion");
    if (RtlGetVersion)
    {
      OSVERSIONINFOEXW osw = { sizeof(OSVERSIONINFOEXW) };
      RtlGetVersion(&osw);
      dwMajorVersion = osw.dwMajorVersion;
      dwMinorVersion = osw.dwMinorVersion;
      dwBuild = osw.dwBuildNumber;
    }
  }

  // レジストリに ReleaseID や UBR の値が存在する場合は取得する
  TCHAR szReleaseID[32] = { 0 };
  TCHAR szUBR[32] = { 0 };
  {
    HKEY hKey;
    if (ERROR_SUCCESS == RegOpenKeyEx(HKEY_LOCAL_MACHINE, TEXT("SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion"), 0, KEY_READ | KEY_WOW64_64KEY, &hKey))
    {
      DWORD dwType = REG_SZ;
      DWORD dwByte = sizeof(szReleaseID);
      if (ERROR_SUCCESS == RegQueryValueEx(hKey, TEXT("ReleaseId"), 0, &dwType, (LPBYTE)&szReleaseID, &dwByte))
      {
        lstrcat(szReleaseID, TEXT(" "));
      }
      dwType = REG_DWORD;
      dwByte = sizeof(DWORD);
      DWORD dwUBR = 0;
      if (ERROR_SUCCESS == RegQueryValueEx(hKey, TEXT("UBR"), 0, &dwType, (LPBYTE)&dwUBR, &dwByte))
      {
        wsprintf(szUBR, TEXT(".%d"), dwUBR);
      }
      RegCloseKey(hKey);
    }
  }

  TCHAR szText[1024];
  wsprintf(szText, TEXT("Windows %d.%d %s(OS ビルド %d%s)"), dwMajorVersion, dwMinorVersion, szReleaseID, dwBuild, szUBR);
  SetWindowText(hEdit, szText);
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hEdit;
  switch (msg)
  {
  case WM_CREATE:
    hEdit = CreateWindow(TEXT("EDIT"), 0, WS_VISIBLE | WS_CHILD |
      ES_READONLY | ES_AUTOHSCROLL | ES_AUTOVSCROLL | ES_MULTILINE,
      0, 0, 0, 0, hWnd, 0, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    GetWindowsVersion(hEdit);
    break;
  case WM_SETFOCUS:
    SetFocus(hEdit);
    SendMessage(hEdit, EM_SETSEL, 0, -1);
    break;
  case WM_SIZE:
    MoveWindow(hEdit, 10, 10, LOWORD(lParam) - 20, HIWORD(lParam) - 20, TRUE);
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
    TEXT("Windows の バージョンを取得"),
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

追記: Windows 10 においては ReleaseID と呼ばれる数値（1607 とか 1703）やビルド番号の後ろにドット区切りで数値が表記されることがあります。その数値がレジストリから取得できた場合は、バージョン情報やビルド番号に追記するようにしました。(2017/09/28)

### 参考サイト
- [https://msdn.microsoft.com/ja-jp/library/windows/desktop/ms724439.aspx](https://msdn.microsoft.com/ja-jp/library/windows/desktop/ms724439.aspx)  
- [https://msdn.microsoft.com/en-us/library/windows/hardware/ff561910(v=vs.85).aspx](https://msdn.microsoft.com/en-us/library/windows/hardware/ff561910(v=vs.85).aspx)  
- [http://www.atmarkit.co.jp/ait/articles/1704/04/news008_2.html](http://www.atmarkit.co.jp/ait/articles/1704/04/news008_2.html) 

