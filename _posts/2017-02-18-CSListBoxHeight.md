---
layout: post
title: [C#][Windows Forms] リストボックスの項目の高さを変更する
subtitle: これはテスト投稿です。
tags: ["C#",ListBox]
comments: true
---

ListBoxの項目の高さを変更します。

{% highlight csharp linenos %}
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      InitializeComponent();

      listBox1.DrawMode = DrawMode.OwnerDrawFixed;
      listBox1.ItemHeight = 200;

      listBox1.Items.Add("aaa");
      listBox1.Items.Add("bbb");
      listBox1.Items.Add("ccc");
    }

    private void listBox1_DrawItem(object sender, DrawItemEventArgs e)
    {
        if (e.Index == -1)
            return;

        e.DrawBackground();
        string itemString = (string)((ListBox)sender).Items[e.Index];
        e.Graphics.DrawString(itemString, e.Font, new SolidBrush(e.ForeColor),
             new RectangleF(e.Bounds.X, e.Bounds.Y, e.Bounds.Width, e.Bounds.Height));
        e.DrawFocusRectangle();
    }
  }
}
{% endhighlight %}
