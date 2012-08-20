---
layout: post
title: Core Text APIを使用した際に起こる行間の問題について
tags: 
- Objective-C
---


Core Text APIを使用して日本語をレンダリングする際に、行間が英数文字と比べると異様に広くなる問題があります。詳しくは [E-WA’s Blog - Tweetbot：日本語テキストの行間について](http://ewa4618.vjck.com/2011/07/05/tweetbot%EF%BC%9A%E6%97%A5%E6%9C%AC%E8%AA%9E%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88%E3%81%AE%E8%A1%8C%E9%96%93%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6/) で説明されています。

CTLineをループして、Y座標をCTFramesetterに頼らず自力で指定し、CTLineDrawで一行ごと描画することでこの問題を回避できます。NSLayoutManagerを使用して英数字のみを描画した際に使用される行の高さを取得しています。

コードは [https://gist.github.com/1497649](https://gist.github.com/1497649) に置いてあります。
