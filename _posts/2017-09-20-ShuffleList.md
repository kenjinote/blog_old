---
layout: post
title: 配列をランダムに並び替える
subtitle: ゲームの基本は並び替え
tags: [C++,アルゴリズム,配列]
comments: true
---

配列をランダムに並び替える方法はいろいろとあると思いますがまず簡単に思いつく方法は、配列のすべての要素をそれそれ配列のランダムな番目の要素と交換する方法です。

<script src="https://gist.github.com/kenjinote/c31873f7ea608cee40771912a20c108d.js"></script>

また、動的配列として std::vector を使う場合は std::shuffle を使ってランダムに並び替えることもできます。

<script src="https://gist.github.com/kenjinote/f86659c29cb43ea821e7d03815de879d.js"></script>