---
layout: post
title: バブルソートとクイックソートの比較（その２）
subtitle: ソードの実行をGifアニメに
thumbnail-img: /assets/img/sortanimationgif.gif
tags: [C++,ソート,アルゴリズム,GIFアニメーション]
comments: true
---

バブルソートとクイックソートのソートの過程をアニメーションGIFに出力するプログラムです。
{% highlight cpp linenos %}
void CreateSortAnimationGif(SORT_TYPE type, LPCTSTR lpszFilePath, int nListSize, int nWidth, int nHeight, float fFrameRate, Gdiplus::Color color)
{% endhighlight %}

上記の関数で出力を行っています。
SORT_TYPE type　←　バブルソートかクイックソートか選択  
LPCTSTR lpszFilePath　←　出力するアニメーションGIFのファイルパスを指定  
int nListSize　←　ソートするリストのサイズを指定。あまりでかいと終わるのに時間がかかったりメモリが確保できなかったりします。  
int nWidth　←　出力するGIFの横幅  
int nHeight　←　出力するGIFの縦幅  
float fFrameRate　←　フレームレート、アニメーションGIFの再生スピードをコントロールできる  
Gdiplus::Color color　←　グラフの色を指定  

[プロジェクトのダウンロード](https://github.com/kenjinote/SortAnimationGIF/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#pragma comment(lib, "gdiplus")

#include <windows.h>
#include <gdiplus.h>
#include <vector>
#include <string>
#include <array>
#include <random>

#include "GifEncoder.h"

TCHAR szClassName[] = TEXT("Window");

void DrawGraph(Gdiplus::Bitmap *bmp, int nWidth, int nHeight, int *data, int size, Gdiplus::Color color)
{
    Gdiplus::Graphics g(bmp);
    g.Clear(Gdiplus::Color::Black);
    float dmax = 0.0f;
    for (int i = 0; i < size; ++i) {
        dmax = max(data[i], dmax);
    }
    const float penwidth = (float)nWidth / (float)size;
    for (int i = 0; i < size; ++i) {
        const float x = i * penwidth + penwidth / 2.0f;
        g.DrawLine(&Gdiplus::Pen(color, (float)penwidth), Gdiplus::PointF(x, (float)nHeight), Gdiplus::PointF(x, nHeight - data[i] * nHeight / dmax));
    }
}

void BubbleSort(int* data, int size, CGifEncoder *pGifEncoder, Gdiplus::Bitmap *bmp, int nWidth, int nHeight, Gdiplus::Color color)
{
    BOOL bFlag;
    int k = 0;
    do {
        bFlag = FALSE;
        for (int i = 0; i < size - 1 - k; ++i) {
            if (data[i] > data[i + 1]) {
                bFlag = TRUE;
                const int nTmp = data[i];
                data[i] = data[i + 1];
                data[i + 1] = nTmp;
                DrawGraph(bmp, nWidth, nHeight, data, size, color);
                pGifEncoder->AddFrame(bmp);
            }
        }
        ++k;
    } while (bFlag);
}

void QuickSort(int start, int end, int *data, int size, CGifEncoder *pGifEncoder, Gdiplus::Bitmap *bmp, int nWidth, int nHeight, Gdiplus::Color color)
{
    int lower, upper, div, temp;
    if (start >= end) {
        return;
    }
    div = data[start];
    for (lower = start, upper = end; lower < upper;) {
        while (lower <= upper && data[lower] <= div) {
            ++lower;
        }
        while (lower <= upper&&data[upper] > div) {
            --upper;
        }
        if (lower < upper) {
            temp = data[lower];
            data[lower] = data[upper];
            data[upper] = temp;
            DrawGraph(bmp, nWidth, nHeight, data, size, color);
            pGifEncoder->AddFrame(bmp);
        }
    }
    temp = data[start];
    data[start] = data[upper];
    data[upper] = temp;
    DrawGraph(bmp, nWidth, nHeight, data, size, color);
    pGifEncoder->AddFrame(bmp);
    QuickSort(start, upper - 1, data, size, pGifEncoder, bmp, nWidth, nHeight, color);
    QuickSort(upper + 1, end, data, size, pGifEncoder, bmp, nWidth, nHeight, color);
}

void InitList(int* pList, int nSize)
{
    for (int i = 0; i < nSize; ++i) {
        pList[i] = i;
    }
}

void Shuffle(int* pList, int nSize)
{
    std::array<std::seed_seq::result_type, std::mt19937::state_size> seed_data;
    std::random_device rd;
    std::generate_n(seed_data.data(), seed_data.size(), std::ref(rd));
    std::seed_seq seq(std::begin(seed_data), std::end(seed_data));
    std::mt19937 engine(seq);
    for (int i = 0; i < nSize; ++i) {
        const int j = engine() % nSize;
        if (i != j) {
            int temp = pList[i];
            pList[i] = pList[j];
            pList[j] = temp;
        }
    }
}

enum SORT_TYPE {BUBBLE_SORT, QUICK_SORT};
void CreateSortAnimationGif(SORT_TYPE type, LPCTSTR lpszFilePath, int nListSize, int nWidth, int nHeight, float fFrameRate, Gdiplus::Color color)
{
    int* data = new int[nListSize];
    if (data) {
        Gdiplus::Bitmap *bmp = new Gdiplus::Bitmap(nWidth, nHeight);
        if (bmp) {
            CGifEncoder gifEncoder;
            gifEncoder.SetFrameSize(nWidth, nHeight);
            gifEncoder.SetFrameRate(fFrameRate);
            gifEncoder.StartEncoder(std::wstring(lpszFilePath));
            InitList(data, nListSize);
            Shuffle(data, nListSize);
            DrawGraph(bmp, nWidth, nHeight, data, nListSize, color);
            for (int i = 0; i < 50; ++i)
                gifEncoder.AddFrame(bmp);
            if (type == BUBBLE_SORT)
                BubbleSort(data, nListSize, &gifEncoder, bmp, nWidth, nHeight, color);
            else
                QuickSort(0, nListSize - 1, data, nListSize, &gifEncoder, bmp, nWidth, nHeight, color);
            for (int i = 0; i < 49; ++i)
                gifEncoder.AddFrame(bmp);
            gifEncoder.FinishEncoder();
            delete bmp;
        }
        delete[]data;
    }
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
    static HWND hButton;
    switch (msg)
    {
    case WM_CREATE:
        hButton = CreateWindow(TEXT("BUTTON"), TEXT("GIFの作成"), WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)IDOK, ((LPCREATESTRUCT)lParam)->hInstance, 0);
        break;
    case WM_SIZE:
        MoveWindow(hButton, 10, 10, 256, 32, TRUE);
        break;
    case WM_COMMAND:
        if (LOWORD(wParam) == IDOK)
        {
            CreateSortAnimationGif(BUBBLE_SORT, TEXT("bubble.gif"), 64, 256, 256, 50.0f, Gdiplus::Color(248, 183, 84));
            CreateSortAnimationGif(QUICK_SORT, TEXT("quick.gif"), 64, 256, 256, 50.0f, Gdiplus::Color(155, 216, 236));
            MessageBox(hWnd, TEXT("出力が完了しました。"), TEXT("確認"), 0);
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
    Gdiplus::GdiplusStartupInput gdiSI;
    Gdiplus::GdiplusStartup(&gdiToken, &gdiSI, NULL);
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
        TEXT("バブルソートとクイックソートの様子をアニメーションGIFとして出力"),
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
    Gdiplus::GdiplusShutdown(gdiToken);
    return (int)msg.wParam;
}
{% endhighlight %}
