---
layout: post
title: 起動時ウィンドウをアクティブ化しないようにする(WPF)
subtitle: たった一つのプロパティを設定するのみ
thumbnail-img: /assets/img/showactivated.png
tags: [WPF,ウィンドウ]
comments: true
---

WPFで起動時ウィンドウをアクティブ化しないようにするには、XamlでWindowのプロパティで、
ShowActivated="False"と設定すると実現できます。

![](/assets/img/showactivated.png){: .mx-auto.d-block :}

[プロジェクトのダウンロード](https://github.com/kenjinote/ShowActivated/archive/master.zip)

[Xamlコード]
{% highlight xml linenos %}
<Window x:Class="ShowActivated.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:ShowActivated"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800" ShowActivated="False">
    <Grid>

    </Grid>
</Window>
{% endhighlight %}

### 参考文献
- [Window.ShowActivated プロパティ](https://docs.microsoft.com/ja-jp/dotnet/api/system.windows.window.showactivated)
