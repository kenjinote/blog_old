---
layout: post
title: コンピュータに数独を解かせる
subtitle: 悪用厳禁
thumbnail-img: /assets/img/solvesudoku.png
tags: [C++,アルゴリズム,パズル,ゲーム]
comments: true
---

数独を解くプログラムです。初期で定まっている数字を指定し、`解く`ボタンをクリックすると数独の解いてきます。

![](/assets/img/solvesudoku.png){: .mx-auto.d-block :}

[プロジェクトのダウンロード](https://github.com/kenjinote/SolveSudoku/archive/master.zip)

[C++コード]
{% highlight cpp linenos %}
#pragma comment(linker,"\"/manifestdependency:type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")

#include <windows.h>
#include <windowsx.h>

TCHAR szClassName[] = TEXT("Window");
#define NumberSize 9
#define MatrixSize (NumberSize * NumberSize)
#define nSpan 32
#define nTop 50

class Number
{
	int number;
	BOOL fixed;
public:
	Number() : number(0), fixed(FALSE) {}
	BOOL IsHint() const { return fixed; }
	operator int() const { return number; }
	int operator=(int n) { number = n; return number; }
	void SetHint(int n) { number = n; fixed = n ? TRUE : FALSE; }
};

class NumberArray
{
	Number items[MatrixSize];
	void getTryNumber(int tryNum[], int n);
	BOOL tryPlace(int n);
public:
	NumberArray() {};
	BOOL GetAnswer();
	Number* operator[](int n) { return &items[n]; }
};

BOOL NumberArray::GetAnswer()
{
	for (int i = 0; i < MatrixSize; i++)
	{
		if (!items[i].IsHint()) items[i] = 0;
	}
	return tryPlace(0);
}

BOOL NumberArray::tryPlace(int n)
{
	int tryNumber[NumberSize + 1], *pTryNumber;
	if (n == MatrixSize) return TRUE;
	if (items[n].IsHint()) return tryPlace(n + 1);

	getTryNumber(tryNumber, n);
	for (pTryNumber = tryNumber; *pTryNumber; pTryNumber++)
	{
		items[n] = *pTryNumber;
		if (tryPlace(n + 1)) return TRUE;
		items[n] = 0;
	}
	return FALSE;
}

void NumberArray::getTryNumber(int tryNum[], int n)
{
	int tmpnum[NumberSize];
	const int offset[] = {
		1, 1, NumberSize - 2,
		1, 1, NumberSize - 2,
		1, 1, NumberSize - 2
	};
	int i, j;
	int index;

	for (i = 0; i < NumberSize; i++) tmpnum[i] = i + 1;

	//横のチェック
	index = (n / NumberSize)*NumberSize;
	for (i = 0; i < NumberSize; i++)
	{
		int number = items[index];
		if (number) tmpnum[number - 1] = 0;
		index++;
	}

	//縦のチェック
	index = n % NumberSize;
	for (i = 0; i < NumberSize; i++)
	{
		int number = items[index];
		if (number) tmpnum[number - 1] = 0;
		index += NumberSize;
	}

	//ブロックのチェック
	index = (n / 3) / 9 * 27 + ((n / 3) % 3) * 3;
	for (i = 0; i < NumberSize; i++)
	{
		int number = items[index];
		if (number) tmpnum[number - 1] = 0;
		index += offset[i];
	}

	for (i = 0, j = 0; i < NumberSize; i++)
	{
		if (tmpnum[i]) tryNum[j++] = tmpnum[i];
	}
	tryNum[j] = 0;
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
	static HWND hButton;
	static NumberArray numbers;
	static int nCurrentIndex;
	switch (msg)
	{
	case WM_CREATE:
		hButton = CreateWindow(TEXT("BUTTON"), TEXT("解く"),
			WS_VISIBLE | WS_CHILD, 0, 0, 0, 0, hWnd, (HMENU)IDOK,
			((LPCREATESTRUCT)lParam)->hInstance, 0);
		{
			const int table[] = {
				7,0,0,0,5,0,3,0,0,
				0,5,0,0,0,2,0,0,0,
				0,0,8,0,7,0,6,0,2,
				0,0,0,1,0,0,0,9,0,
				5,0,9,0,6,0,7,0,4,
				0,6,0,0,0,3,0,0,0,
				9,0,1,0,8,0,5,0,0,
				0,0,0,9,0,0,0,6,0,
				0,0,7,0,3,0,0,0,1,
			};
			for (int i = 0; i < MatrixSize; i++)
			{
				if (table[i])
					(numbers[i])->SetHint(table[i]);
			}
		}
		break;
	case WM_LBUTTONDOWN:
	case WM_RBUTTONDOWN:
		{
			const int nX = GET_X_LPARAM(lParam);
			const int nY = GET_Y_LPARAM(lParam);
			nCurrentIndex = nX / nSpan + ((nY - nTop) / nSpan) * NumberSize;
			HMENU hMyMenu = CreatePopupMenu();
			AppendMenu(hMyMenu, MF_ENABLED | MF_STRING, 100, TEXT("空白"));
			for (int i = 1; i < 10; ++i)
			{
				TCHAR szNumber[2] = { 0 };
				wsprintf(szNumber, TEXT("%d"), i);
				AppendMenu(hMyMenu, MF_ENABLED | MF_STRING, 100 + i, szNumber);
			}
			AppendMenu(hMyMenu, MF_SEPARATOR, 0, 0);
			AppendMenu(hMyMenu, MF_ENABLED | MF_STRING, 110, TEXT("全部クリア"));

			POINT point = { nX,nY };
			ClientToScreen(hWnd, &point);

			TrackPopupMenu(hMyMenu, TPM_LEFTALIGN, point.x, point.y, 0, hWnd, NULL);

			//DeleteMenu()
			DestroyMenu(hMyMenu);
		}
		break;
	case WM_PAINT:
		{
			PAINTSTRUCT ps;
			const HDC hdc = BeginPaint(hWnd, &ps);

			for (int i = 0; i <= NumberSize; ++i)
			{
				MoveToEx(hdc, 0, nSpan * i + nTop, 0);
				LineTo(hdc, nSpan * NumberSize, nSpan * i + 50);
				MoveToEx(hdc, nSpan * i, 0 + nTop, 0);
				LineTo(hdc, nSpan * i, nSpan * NumberSize + 50);
			}

			TCHAR szText[1024];
			for (int x = 0; x < NumberSize; x++)
			{
				for (int y = 0; y < NumberSize; y++)
				{
					const int nNumber = (int)(*numbers[y*NumberSize + x%NumberSize]);
					if (!nNumber) continue;
					wsprintf(szText, TEXT("%d"), nNumber);
					TextOut(hdc, x * nSpan + 12, y * nSpan + nTop + 8, szText, lstrlen(szText));
				}
			}

			EndPaint(hWnd, &ps);
		}
		break;
	case WM_SIZE:
		MoveWindow(hButton, 10, 10, nSpan * NumberSize - 20, 32, TRUE);
		break;
	case WM_COMMAND:
		if (LOWORD(wParam) == IDOK)
		{
			const BOOL result = numbers.GetAnswer();
			InvalidateRect(hWnd, 0, 1);
			if (result)
				MessageBox(hWnd, TEXT("解が得られました"), TEXT("確認"), 0);
			else
				MessageBox(hWnd, TEXT("解が得られませんでした"), TEXT("確認"), 0);
		}
		else if (LOWORD(wParam) >= 100 && LOWORD(wParam) <= 109)
		{
			(numbers[nCurrentIndex])->SetHint(LOWORD(wParam) - 100);
			InvalidateRect(hWnd, 0, 1);
		}
		else if (LOWORD(wParam) == 110)
		{
			for (int i = 0; i < MatrixSize; i++)
			{
				(numbers[i])->SetHint(0);
			}
			InvalidateRect(hWnd, 0, 1);
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
	const WNDCLASS wndclass = {
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
	RECT rect = { 0, 0,
		nSpan * NumberSize + 1,
		nSpan * NumberSize + nTop + 1 };
	const DWORD dwStyle = WS_CAPTION | WS_SYSMENU | WS_MINIMIZEBOX;
	AdjustWindowRect(&rect, dwStyle, FALSE);
	const HWND hWnd = CreateWindow(
		szClassName,
		TEXT("数独を解く"),
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
###   参考
- 工学社『C++活用研究』

※「数独」は株式会社ニコリの商標登録です。