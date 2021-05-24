---
layout: post
title: WebBrowserコントロールでGPUレンダリングを有効にするには
subtitle: WebGLを使うときはぜひ有効に
thumbnail-img: /assets/img/webbrowsergpurendering.png
tags: [C++,ブラウザ,GPU,IEコンポーネント]
comments: true
---

アプリケーションでWebBrowserコントロール（IEコンポーネント）を使用するとデフォルトではGPUレンダリングが無効となっているため Internet Explorer に比べレンダリングがカクついたりちらつきが発生することがあります。WebBrowser コントロールの GPU レンダリングを有効にするには下記のレジストリに値を書き込む必要があります。

`HKEY_CURRENT_USER\SOFTWARE\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_GPU_RENDERING`

- 値の名前: `exeのファイル名`
- 値のタイプ: `REG_DWORD`
- 値のデータ: `1`

![](/assets/img/webbrowsergpurendering.png){: .mx-auto.d-block :}

### 参考文献
- [Internet Feature Controls (D..H)](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/general-info/ee330731(v=vs.85)?redirectedfrom=MSDN#gpu_rendering)
- [C# WebBrowserコントロールのGPUレンダリングについて](https://social.msdn.microsoft.com/Forums/vstudio/ja-JP/1006a1ab-b5a1-4341-87fc-56d996a062cc)