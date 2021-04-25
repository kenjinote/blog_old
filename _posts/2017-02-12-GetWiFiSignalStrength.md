---
layout: post
title: 現在接続している Wi-Fi のシグナル強度を取得する(C++)
thumbnail-img: /assets/img/getwifisignalstrength.png
tags: [C++,Wi-Fi,ツール]
comments: true
---

![](/assets/img/getwifisignalstrength.png){: .mx-auto.d-block :}


[プロジェクトのダウンロード](https://github.com/kenjinote/GetWiFiSignalStrength/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#pragma comment(lib, "wlanapi.lib")

#include <windows.h>
#include <wlanapi.h>
#include "resource.h"

TCHAR szClassName[] = TEXT("Window");

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hList;
  switch (msg)
  {
  case WM_CREATE:
    hList = CreateWindowEx(WS_EX_CLIENTEDGE, TEXT("LISTBOX"), 0, WS_VISIBLE | WS_CHILD | LBS_NOINTEGRALHEIGHT, 0, 0, 0, 0, hWnd, (HMENU)IDOK, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    SendMessage(hWnd, WM_APP, 0, 0);
    SetTimer(hWnd, 0x1234, 1000, 0);
    break;
  case WM_TIMER:
    KillTimer(hWnd, 0x1234);
    SendMessage(hWnd, WM_APP, 0, 0);
    SetTimer(hWnd, 0x1234, 1000, 0);
    break;
  case WM_SIZE:
    MoveWindow(hList, 10, 10, LOWORD(lParam) - 20, HIWORD(lParam) - 20, TRUE);
    break;
  case WM_APP:
    {
      SendMessage(hList, LB_RESETCONTENT, 0, 0);
      DWORD dwCurVersion = 0;
      DWORD dwResult = 0;
      WCHAR GuidString[39] = { 0 };
      HANDLE hClient = NULL;
      dwResult = WlanOpenHandle(2, NULL, &dwCurVersion, &hClient);
      if (dwResult == ERROR_SUCCESS)
      {
        PWLAN_INTERFACE_INFO_LIST pIfList = NULL;
        dwResult = WlanEnumInterfaces(hClient, NULL, &pIfList);
        if (dwResult == ERROR_SUCCESS)
        {
          for (int i = 0; i < (int)pIfList->dwNumberOfItems; ++i)
          {
            PWLAN_INTERFACE_INFO pIfInfo = (WLAN_INTERFACE_INFO *)&pIfList->InterfaceInfo[i];
            StringFromGUID2(pIfInfo->InterfaceGuid, (LPOLESTR)&GuidString, sizeof(GuidString) / sizeof(*GuidString));
            if (pIfInfo->isState == wlan_interface_state_connected)
            {
              PWLAN_AVAILABLE_NETWORK_LIST pBssList = NULL;
              dwResult = WlanGetAvailableNetworkList(hClient, &pIfInfo->InterfaceGuid, 0, NULL, &pBssList);
              if (dwResult == ERROR_SUCCESS)
              {
                for (int j = 0; j < (int)pBssList->dwNumberOfItems; ++j)
                {
                  PWLAN_AVAILABLE_NETWORK pBssEntry = (WLAN_AVAILABLE_NETWORK *)& pBssList->Network[j];
                  if (pBssEntry->dwFlags & WLAN_AVAILABLE_NETWORK_CONNECTED)
                  {
                    TCHAR szText[1024];
                    wsprintf(szText, TEXT("%ws, %u%%"), pBssEntry->strProfileName, pBssEntry->wlanSignalQuality);
                    SendMessage(hList, LB_ADDSTRING, 0, (LPARAM)szText);
                  }
                }
              }
              if (pBssList != NULL)
              {
                WlanFreeMemory(pBssList);
                pBssList = NULL;
              }
            }
          }
          if (pIfList != NULL)
          {
            WlanFreeMemory(pIfList);
            pIfList = NULL;
          }
        }
        WlanCloseHandle(hClient, NULL);
      }
    }
    break;
  case WM_DESTROY:
    KillTimer(hWnd, 0x1234);
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
    LoadIcon(hInstance, MAKEINTRESOURCE(IDI_ICON1)),
    LoadCursor(0,IDC_ARROW),
    (HBRUSH)(COLOR_WINDOW + 1),
    0,
    szClassName
  };
  RegisterClass(&wndclass);
  HWND hWnd = CreateWindow(
    szClassName,
    TEXT("接続中のWi-Fiのシグナルの強度を表示"),
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

### 参考文献
- [WlanGetAvailableNetworkList function (wlanapi.h)](https://docs.microsoft.com/en-us/windows/win32/api/wlanapi/nf-wlanapi-wlangetavailablenetworklist)
