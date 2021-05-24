---
layout: post
title: 指定した文字列をランダムに表示するスクリーンセイバー(C++)
subtitle: スクリーンセイバー作成の基本
thumbnail-img: /assets/img/screensaver.png
tags: [C++,スクリーンセーバー,ツール]
comments: true
---

スクリーンセーバーを作成するためのサンプルプログラムです。
Visual C++ でスクリーンセーバーを作るには、下記のような手順で行えます。

![](/assets/img/screensaver.png){: .mx-auto.d-block :}

1. プロジェクトを Visual C++ Win32 プロジェクト（空のプロジェクト）でプロジェクトを新規作成する。
1. プロジェクトに C++ ファイルを追加し、ScreenSaverProc 関数と ScreenSaverConfigureDialog 関数と RegisterDialogClasses 関数を実装する。
1. プロジェクトにリソースを追加し、ダイアログを作成する。このときダイアログの ID が 2003 になるようにする。ここで追加されたダイアログがスクリーンセーバーの設定画面となります。
1. リソースに文字列を追加する。ID が 1 となるようにする。この文字列が Windows のスクリーンセーバー名として認識されます。

今回のプログラムは、スクリーンセーバーの設定画面で設定された文字をランダムで表示するスクリーンセーバーのサンプルプログラムです。マルチディスプレイにも対応するため、スクリーンセーバーの起動時に各ディスプレイの矩形を取得し、それぞれの面積の比率に応じてランダムで表示する割合を変えています。設定した情報をレジストリに保存し、スクリーンセーバーの実行時にレジストリから値を取得するようにしています。スクリーンセーバーのひな形として参考になれば幸いです。

[プロジェクトのダウンロード](https://github.com/kenjinote/Screensaver/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#pragma comment(lib, "scrnsavw")
#pragma comment(lib, "comctl32")

#include <windows.h> 
#include <commctrl.h> 
#include <scrnsave.h>
#include <vector>
#include "resource.h"

#define ID_TIMER 2657

int GetArea(const LPRECT lpRect)
{
  return (lpRect->right - lpRect->left) * (lpRect->bottom - lpRect->top);
}

int GetTotalArea(const std::vector<RECT> *pMonitorList)
{
  int nTotalArea = 0;
  for (auto r : *pMonitorList)
  {
    nTotalArea += GetArea(&r);
  }
  return nTotalArea;
}

RECT GetRandomArea(const std::vector<RECT> *pMonitorList)
{
  RECT rect = { 0 };
  int nRandom = rand() % GetTotalArea(pMonitorList);
  for (auto r : *pMonitorList)
  {
    const int nArea = GetArea(&r);
    if (nRandom < nArea)
    {
      rect = r;
      break;
    }
    nRandom -= nArea;
  }
  return rect;
}

#define REG_KEY TEXT("Software\\Screensaver\\Setting")
class Setting {
  TCHAR m_szText[256];
  TCHAR m_szFont[256];
  DWORD m_nFontSize;
  DWORD m_nCountLimit;
  DWORD m_nSpeed;
public:
  Setting() : m_nFontSize(20), m_nCountLimit(1000), m_nSpeed(1000){
    lstrcpy(m_szText, TEXT("★"));
    lstrcpy(m_szFont, TEXT("メイリオ"));
  }
  void Load() {
    HKEY hKey;
    DWORD dwPosition;
    if (ERROR_SUCCESS == RegCreateKeyEx(HKEY_CURRENT_USER, REG_KEY, 0, 0, 0, KEY_READ, 0, &hKey, &dwPosition)) {
      DWORD dwType;
      DWORD dwByte;
      dwType = REG_SZ;
      dwByte = sizeof(m_szText);
      RegQueryValueEx(hKey, TEXT("Text"), NULL, &dwType, (BYTE *)m_szText, &dwByte);
      dwType = REG_SZ;
      dwByte = sizeof(m_szFont);
      RegQueryValueEx(hKey, TEXT("Font"), NULL, &dwType, (BYTE *)m_szFont, &dwByte);
      dwType = REG_DWORD;
      dwByte = sizeof(DWORD);
      RegQueryValueEx(hKey, TEXT("FontSize"), NULL, &dwType, (BYTE *)&m_nFontSize, &dwByte);
      dwType = REG_DWORD;
      dwByte = sizeof(DWORD);
      RegQueryValueEx(hKey, TEXT("CountLimit"), NULL, &dwType, (BYTE *)&m_nCountLimit, &dwByte);
      dwType = REG_DWORD;
      dwByte = sizeof(DWORD);
      RegQueryValueEx(hKey, TEXT("Speed"), NULL, &dwType, (BYTE *)&m_nSpeed, &dwByte);
      RegCloseKey(hKey);
    }
  }
  void Save() {
    HKEY hKey;
    DWORD dwPosition;
    if (ERROR_SUCCESS == RegCreateKeyEx(HKEY_CURRENT_USER, REG_KEY, 0, 0, 0, KEY_WRITE, 0, &hKey, &dwPosition)) {
      RegSetValueEx(hKey, TEXT("Text"), 0, REG_SZ, (CONST BYTE *)m_szText, sizeof(TCHAR) * (lstrlen(m_szText) + 1));
      RegSetValueEx(hKey, TEXT("Font"), 0, REG_SZ, (CONST BYTE *)m_szFont, sizeof(TCHAR) * (lstrlen(m_szFont) + 1));
      RegSetValueEx(hKey, TEXT("FontSize"), 0, REG_DWORD, (CONST BYTE *)&m_nFontSize, sizeof(DWORD));
      RegSetValueEx(hKey, TEXT("CountLimit"), 0, REG_DWORD, (CONST BYTE *)&m_nCountLimit, sizeof(DWORD));
      RegSetValueEx(hKey, TEXT("Speed"), 0, REG_DWORD, (CONST BYTE *)&m_nSpeed, sizeof(DWORD));
      RegCloseKey(hKey);
    }
  }
  DWORD GetFontSize() { return m_nFontSize; }
  void SetFontSize(int nFontSize) { m_nFontSize = nFontSize; }
  LPTSTR GetFont() { return m_szFont; }
  void SetFont(LPCTSTR lpszFont) { lstrcpy(m_szFont, lpszFont); }
  LPTSTR GetText() { return m_szText; }
  void SetText(LPCTSTR lpszText) { lstrcpy(m_szText, lpszText); }
  DWORD GetCountLimit() { return m_nCountLimit; }
  void SetCountLimit(int nCountLimit) { m_nCountLimit = nCountLimit; }
  DWORD GetSpeed() { return m_nSpeed; }
  void SetSpeed(DWORD nSpeed) { m_nSpeed = (nSpeed ? nSpeed : 1); }
};

BOOL CALLBACK MonitorEnumProc(HMONITOR hMonitor, HDC hdcMonitor, LPRECT lprcMonitor, LPARAM dwData)
{
  MONITORINFOEX MonitorInfoEx;
  MonitorInfoEx.cbSize = sizeof(MonitorInfoEx);
  if (GetMonitorInfo(hMonitor, &MonitorInfoEx) != 0)
  {
    DEVMODE dm = { 0 };
    dm.dmSize = sizeof(DEVMODE);
    if (EnumDisplaySettings(MonitorInfoEx.szDevice, ENUM_CURRENT_SETTINGS, &dm) != 0)
    {
      const int nMonitorWidth = dm.dmPelsWidth;
      const int nMonitorHeight = dm.dmPelsHeight;
      const int nPrimaryPosX = GetSystemMetrics(SM_XVIRTUALSCREEN);
      const int nPrimaryPosY = GetSystemMetrics(SM_YVIRTUALSCREEN);
      const int nMonitorPosX = dm.dmPosition.x - nPrimaryPosX;
      const int nMonitorPosY = dm.dmPosition.y - nPrimaryPosY;
      RECT rect = { nMonitorPosX, nMonitorPosY, nMonitorPosX + nMonitorWidth, nMonitorPosY + nMonitorHeight };
      ((std::vector<RECT>*)dwData)->push_back(rect);
    }
  }
  return TRUE;
}

LRESULT WINAPI ScreenSaverProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static std::vector<RECT> MonitorList;
  static Setting setting;
  static DWORD nCount;
  static HFONT hFont;
  static SIZE size;
  switch (msg)
  {
  case WM_CREATE:
    setting.Load();
    {
      HDC hdc = GetDC(hWnd);
      hFont = CreateFont(-MulDiv(setting.GetFontSize(), GetDeviceCaps(hdc, LOGPIXELSY), 72), 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, setting.GetFont());
      HFONT hOldFont = (HFONT)SelectObject(hdc, hFont);
      LPCTSTR lpszText = setting.GetText();
      GetTextExtentPoint32(hdc, lpszText, lstrlen(lpszText), &size);
      SelectObject(hdc, hOldFont);
      ReleaseDC(hWnd, hdc);
    }
    EnumDisplayMonitors(NULL, NULL, MonitorEnumProc, (LPARAM)&MonitorList);
    SetTimer(hWnd, ID_TIMER, 1000 / setting.GetSpeed(), NULL);
    break;
  case WM_TIMER:
    {
      if (wParam != ID_TIMER) break;
      KillTimer(hWnd, ID_TIMER);
      const HDC hdc = GetDC(hWnd);
      SetBkMode(hdc, TRANSPARENT);
      RECT rect = GetRandomArea(&MonitorList);
      SetTextColor(hdc, RGB(rand() % 256, rand() % 256, rand() % 256));
      HFONT hOldFont = (HFONT)SelectObject(hdc, hFont);
      LPCTSTR lpszText = setting.GetText();
      TextOutW(hdc, rect.left + rand() % (rect.right - rect.left - size.cx), rect.top + rand() % (rect.bottom - rect.top - size.cy), lpszText, lstrlen(lpszText));
      SelectObject(hdc, hOldFont);
      ReleaseDC(hWnd, hdc);
      if (nCount >= setting.GetCountLimit()) {
        InvalidateRect(hWnd, 0, 1);
        UpdateWindow(hWnd);
        nCount = 0;
      } else {
        ++nCount;
      }
      SetTimer(hWnd, ID_TIMER, 1000 / setting.GetSpeed(), NULL);
    }
    break;
  case WM_DESTROY:
    KillTimer(hWnd, ID_TIMER);
    DeleteObject(hFont);
    PostQuitMessage(0);
    break;
  default:
    break;
  }
  return DefScreenSaverProc(hWnd, msg, wParam, lParam);
}

int CALLBACK EnumFontsProc(LOGFONT *lplf, TEXTMETRIC *lptm, DWORD dwType, LPARAM lpData)
{
  if (lplf->lfFaceName[0] != L'@') {
    SendMessage((HWND)lpData, CB_ADDSTRING, 0, (LPARAM)lplf->lfFaceName);
  }
  return TRUE;
}

BOOL WINAPI ScreenSaverConfigureDialog(HWND hDlg, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static Setting setting;
  switch (msg)
  {
  case WM_INITDIALOG:
    InitCommonControls();
    SendDlgItemMessage(hDlg, IDC_SPIN1, UDM_SETRANGE, 0, MAKELPARAM(1000, 1));
    SendDlgItemMessage(hDlg, IDC_SPIN2, UDM_SETRANGE, 0, MAKELPARAM(1000, 1));
    SendDlgItemMessage(hDlg, IDC_SPIN3, UDM_SETRANGE, 0, MAKELPARAM(1000, 1));
    SendDlgItemMessage(hDlg, IDC_SPIN1, UDM_SETBUDDY, (WPARAM)GetDlgItem(hDlg, IDC_EDIT2), 0);
    SendDlgItemMessage(hDlg, IDC_SPIN2, UDM_SETBUDDY, (WPARAM)GetDlgItem(hDlg, IDC_EDIT3), 0);
    SendDlgItemMessage(hDlg, IDC_SPIN3, UDM_SETBUDDY, (WPARAM)GetDlgItem(hDlg, IDC_EDIT4), 0);
    {
      HDC hdc = GetDC(hDlg);
      EnumFontFamilies(hdc, NULL, (FONTENUMPROC)EnumFontsProc, (LPARAM)GetDlgItem(hDlg,IDC_COMBO1));
      ReleaseDC(hDlg, hdc);
    }
    setting.Load();
    SetDlgItemText(hDlg, IDC_EDIT1, setting.GetText());
    SendDlgItemMessage(hDlg, IDC_COMBO1, CB_SELECTSTRING, -1, (LPARAM)setting.GetFont());
    SetDlgItemInt(hDlg, IDC_EDIT2, setting.GetFontSize(), FALSE);
    SetDlgItemInt(hDlg, IDC_EDIT3, setting.GetCountLimit(), FALSE);
    SetDlgItemInt(hDlg, IDC_EDIT4, setting.GetSpeed(), FALSE);
    return TRUE;
  case WM_COMMAND:
    switch (LOWORD(wParam))
    {
    case IDOK:
      {
        TCHAR szTemp[256];
        GetDlgItemText(hDlg, IDC_EDIT1, szTemp, _countof(szTemp));
        setting.SetText(szTemp);
        GetDlgItemText(hDlg, IDC_COMBO1, szTemp, _countof(szTemp));
        setting.SetFont(szTemp);
        setting.SetFontSize(GetDlgItemInt(hDlg, IDC_EDIT2, 0, FALSE));
        setting.SetCountLimit(GetDlgItemInt(hDlg, IDC_EDIT3, 0, FALSE));
        setting.SetSpeed(GetDlgItemInt(hDlg, IDC_EDIT4, 0, FALSE));
        setting.Save();
        EndDialog(hDlg, IDOK);
      }
      return TRUE;
    case IDCANCEL:
      EndDialog(hDlg, IDCANCEL);
      return TRUE;
    }
    return FALSE;
  }
  return FALSE;
}

BOOL WINAPI RegisterDialogClasses(HANDLE hInst)
{
  return TRUE;
}
{% endhighlight %}

### 参考文献
- [ScreenSaverProc function (scrnsave.h)](https://docs.microsoft.com/ja-jp/windows/win32/api/scrnsave/nf-scrnsave-screensaverproc)
