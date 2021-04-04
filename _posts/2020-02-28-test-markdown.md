---
layout: post
title: サンプル投稿
subtitle: 各投稿にはサブタイトルをつけられます
gh-repo: kenjinote/cmdchat
gh-badge: [star, fork, follow]
tags: [test]
comments: true
---

この記事は確認用のデモの投稿となります。マークダウンで記述しています。
マークダウン記法を学ぶには [こちら](https://markdowntutorial.com/) のページが参考になります。

**太字のテキストはアスタ2つで囲います**

## 2つ目の章立てはシャープ２つを行先頭に付加します

テーブルのサンプルは下記となります。

| 番号 | 次の数字 | 前の数字 |
| :------ |:--- | :--- |
| 五 | 六 | 四 |
| 十 | 十一 | 九 |
| 七 | 八 | 六 |
| 二 | 三 | 一 |


画像を貼り付けるには下記のようにします。

![Crepe](https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg)

センターぞろえにするには下記のようにします。

![Crepe](https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg){: .mx-auto.d-block :}

コードを貼り付ける場合は、チルダ3つで囲います。

~~~
var foo = function(x) {
  return(x + 5);
}
foo(3)
~~~

シンタックスハイライトを使うには、バッククォート3つで囲み、初めのバッククォート3つの行末に言語を指定します。

```javascript
var foo = function(x) {
  return(x + 5);
}
foo(3)
```

各行に行番号をつけるには下記のように記述します。

{% highlight javascript linenos %}
var foo = function(x) {
  return(x + 5);
}
foo(3)
{% endhighlight %}

## ボックス
お知らせや注意、エラーなどは下記のように枠で表示することが可能です。:

### お知らせ

{: .box-note}
**Note:** お知らせですよ。

### 警告

{: .box-warning}
**Warning:** これは警告です。

### エラー

{: .box-error}
**Error:** これはエラーです。
