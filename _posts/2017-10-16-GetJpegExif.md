---
layout: post
title: GDI+ を使って JPEG の Exif 情報を取得する(C++)
thumbnail-img: /assets/img/getjpegexif.png
tags: [C++,Jpeg,画像,ツール]
comments: true
---

デジタルカメラなどで写真を撮ると、撮った画像ファイルの中のにメタ情報として撮影日時やカメラの機種、絞り値、画素数、ISO 感度など様々な情報が書き込まれます。

Windows では GDI+ を使って JPEG の Exif 情報を取得する事ができるので、Exift 情報を取得するサンプルプログラムを作りました。

[プロジェクトのダウンロード](https://github.com/kenjinote/GetJpegExif)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#pragma comment(lib, "gdiplus")

#include <windows.h>
#include <gdiplus.h>
#include <strsafe.h>

using namespace Gdiplus;

HRESULT PropertyTypeFromWORD(WORD index, WCHAR* string, UINT maxChars)
{
  HRESULT hr = E_FAIL;

  WCHAR* propertyTypes[] = {
    L"Nothing",                   // 0
    L"PropertyTagTypeByte",       // 1
    L"PropertyTagTypeASCII",      // 2
    L"PropertyTagTypeShort",      // 3
    L"PropertyTagTypeLong",       // 4
    L"PropertyTagTypeRational",   // 5
    L"Nothing",                   // 6
    L"PropertyTagTypeUndefined",  // 7
    L"Nothing",                   // 8
    L"PropertyTagTypeSLONG",      // 9
    L"PropertyTagTypeSRational" }; // 10

  hr = StringCchCopyW(string, maxChars, propertyTypes[index]);
  return hr;
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  static HWND hEdit;
  switch (msg)
  {
  case WM_CREATE:
    hEdit = CreateWindow(TEXT("EDIT"), TEXT("JPEG ファイルをドラッグ＆ドロップしてください"), WS_VISIBLE | WS_CHILD | WS_VSCROLL | ES_READONLY | ES_MULTILINE | ES_AUTOHSCROLL | ES_AUTOVSCROLL, 0, 0, 0, 0, hWnd, (HMENU)IDOK, ((LPCREATESTRUCT)lParam)->hInstance, 0);
    DragAcceptFiles(hWnd, TRUE);
    break;
  case WM_SIZE:
    MoveWindow(hEdit, 0, 0, LOWORD(lParam), HIWORD(lParam), TRUE);
    break;
  case WM_DROPFILES:
    {
      SetWindowText(hEdit, 0);
      const HDROP hDrop = (HDROP)wParam;
      const DWORD nFiles = DragQueryFile((HDROP)hDrop, 0xFFFFFFFF, 0, 0);
      TCHAR szFilePath[MAX_PATH];
      for (DWORD i = 0; i < nFiles; i++)
      {
        DragQueryFile(hDrop, i, szFilePath, sizeof(szFilePath));
        Bitmap* bitmap = new Bitmap(szFilePath);
        if (bitmap && bitmap->GetLastStatus() == Ok)
        {
          SendMessageW(hEdit, EM_REPLACESEL, 0, (LPARAM)szFilePath);
          SendMessageW(hEdit, EM_REPLACESEL, 0, (LPARAM)TEXT("\r\n"));
          UINT    size = 0;
          UINT    count = 0;
          #define MAX_PROPTYPE_SIZE 30
          WCHAR strPropertyType[MAX_PROPTYPE_SIZE] = { 0 };
          WCHAR szText[1024] = { 0 };
          if (bitmap->GetPropertySize(&size, &count) == Ok)
          {
            wsprintfW(szText, L"There are %d pieces of metadata in the file.\r\n", count);
            SendMessageW(hEdit, EM_REPLACESEL, 0, (LPARAM)szText);
            wsprintfW(szText, L"The total size of the metadata is %d bytes.\r\n", size);
            SendMessageW(hEdit, EM_REPLACESEL, 0, (LPARAM)szText);
            PropertyItem* pPropBuffer = (PropertyItem*)GlobalAlloc(0, size);
            if (pPropBuffer)
            {
              if (bitmap->GetAllPropertyItems(size, count, pPropBuffer) == Ok)
              {
                for (UINT j = 0; j < count; ++j)
                {
                  // Convert the property type from a WORD to a string.
                  PropertyTypeFromWORD(
                    pPropBuffer[j].type, strPropertyType, MAX_PROPTYPE_SIZE);
                  wsprintfW(szText, L"Property Item %d\r\n", j);
                  SendMessageW(hEdit, EM_REPLACESEL, 0, (LPARAM)szText);
                  wsprintfW(szText, L"  id: 0x%x\r\n", pPropBuffer[j].id);
                  SendMessageW(hEdit, EM_REPLACESEL, 0, (LPARAM)szText);
                  wsprintfW(szText, L"  type: %s\r\n", strPropertyType);
                  SendMessageW(hEdit, EM_REPLACESEL, 0, (LPARAM)szText);
                  wsprintfW(szText, L"  length: %d bytes\r\n", pPropBuffer[j].length);
                  SendMessageW(hEdit, EM_REPLACESEL, 0, (LPARAM)szText);
                  if (pPropBuffer[j].type == 2)
                  {
                    CHAR szTextA[1024];
                    wsprintfA(szTextA, "  text: %s\r\n", (LPSTR)pPropBuffer[j].value);
                    SendMessageA(hEdit, EM_REPLACESEL, 0, (LPARAM)szTextA);
                  }
                }
              }
              GlobalFree(pPropBuffer);
            }
          }
          delete bitmap;
        }
      }
      DragFinish(hDrop);
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
  ULONG_PTR gdiToken;
  GdiplusStartupInput gdiSI;
  GdiplusStartup(&gdiToken, &gdiSI, NULL);
  TCHAR szClassName[] = TEXT("Window");
  MSG msg;
  WNDCLASS wndclass = {
    0,
    WndProc,
    0,
    0,
    hInstance,
    0,
    0,
    0,
    0,
    szClassName
  };
  RegisterClass(&wndclass);
  HWND hWnd = CreateWindow(
    szClassName,
    TEXT("ドラッグ＆ドロップされた JPEG の Exif 情報を表示する"),
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
  GdiplusShutdown(gdiToken);
  return (int)msg.wParam;
}
{% endhighlight %}

### 参考文献
- [Reading and Writing Metadata](https://docs.microsoft.com/en-us/windows/win32/gdiplus/-gdiplus-reading-and-writing-metadata-use)
- [Image::GetAllPropertyItems method (gdiplusheaders.h)](https://docs.microsoft.com/en-us/windows/win32/api/gdiplusheaders/nf-gdiplusheaders-image-getallpropertyitems)
