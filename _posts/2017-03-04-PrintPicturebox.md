---
layout: post
title: Pictureboxに貼り付けた画像を印刷する(VB)
thumbnail-img: /assets/img/printpicturebox.png
tags: [Visual Basic,Picturebox,印刷]
comments: true
---

[Visual Basicコード]
{% highlight vb linenos %}
Public Class Form1

    Private Sub Button1_Click(sender As Object, e As EventArgs) Handles Button1.Click

        ' Pictureボックスに線分を描画した画像を設定
        Dim img As New Bitmap(PictureBox1.Width, PictureBox1.Height)
        Dim g As Graphics = Graphics.FromImage(img)
        g.DrawLine(Pens.Black, 0, 0, 100, 100)
        g.Dispose()
        PictureBox1.Image = img

        ' 印刷
        Dim printDocument As New System.Drawing.Printing.PrintDocument
        AddHandler printDocument.PrintPage, AddressOf PrintPage
        Dim printDialog As New PrintDialog
        printDocument.OriginAtMargins = True
        printDocument.DocumentName = "印刷ドキュメント"
        printDialog.Document = printDocument
        If printDialog.ShowDialog() = DialogResult.OK Then
            printDocument.Print()
        End If

    End Sub

    Private Sub PrintPage(ByVal sender As Object, ByVal e As System.Drawing.Printing.PrintPageEventArgs)
        Using img As Image = Me.PictureBox1.Image
            e.Graphics.DrawImage(img, e.MarginBounds)
            e.HasMorePages = False
        End Using
    End Sub

End Class
{% endhighlight %}
