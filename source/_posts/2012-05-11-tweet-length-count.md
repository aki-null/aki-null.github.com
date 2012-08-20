---
layout: post
title: ツイートの文字数
tags: 
- Twitter
- Objective-C
---


Twitter には1つの投稿につき140文字までしか送信できないルールがある。普通に考えると、それぞれのプログラミング言語の API を使って普通に文字数を数えれば思うだろう。しかし、この実装が簡単に見える機能にはいろいろな罠が潜んでいる。

<!--more-->

## URL

Twitter に投稿するツイートに含まれるリンクはすべて t.co という Twitter 公式の URL 短縮サービスによって短縮されてしまうのだが、「140文字まで」という制限はURL を短縮したあとの文字数に適用される。つまり、Twitter 内部の URL 認識する使用を完全に再現しないと、正確に最終的な文字数を数えることは出来ない。更に、t.co で短縮化された URL の文字数は静的ではないので、[test/configuration][1] API から定期的に取得して記録しておく必要がある。

次に、https で始まる URL が t.co によって短縮された場合、t.co 化された URL も https で始まるようになる。なので、先程の API から両方の URL の長さの値が取得することができる。この記事を書いている時点では通常の URL の場合20文字、https で始まる URL の場合は21文字となっている。

上のルールを適用すると、

`My website! http://aki-null.net`

という文字列の文字数は、この記事を書いている現時点では32文字ということになる。

## Unicode

まず、Twitter のルールに従って文字数を数えるためには、Unicode 正規化を行わなければならない。Twitter は文字数を数えるときは正規化形式C (Normalization Form Canonical Composition) を使用することを指定している。使用している言語・API によって変換する方法は違うが、Objective-C の場合、`NSString`クラスに`- (NSString *)precomposedStringWithCanonicalMapping`という API が用意されている。

更に、Twitter は文字を数えるとき、バイト数ではなく、最終的な文字数を数える。つまり、使用している言語、API によっては、正確な文字数をそのまま取得することはできない場合がある。例えば Objective-C を Mac や iPhone で使っている場合、通常文字列を扱うには`NSString`クラスを使用する。`NSString`は内部的に文字を格納するために UTF-16 を採用している。そして、`NSString`の`- (NSUInteger)length` という API はサロゲートペアを考慮せずに文字数を計算するので、3バイトから4バイトの文字は2文字として数えられてしまう。Objective-C を使っている場合、`CFStringIsSurrogateHighCharacter`と`CFStringIsSurrogateLowCharacter`を使うことで、指定した文字がサロゲートペアの一部であるかを確認することができる。

## まとめ

正確に文字数を数えるのは非常に難しい。そこで Twitter はいくつか URL を認識したり、文字数を数えるライブラリを複数の言語向けに公開している。

- Objective-C: [twitter-text-objc][2]
- Java: [twitter-text-java][3]
- Javascript: [twitter-text-js][4]
- Ruby: [twitter-text-rb][5]

自分で正確な URL 認識を実装したい場合は、[twitter-text-comformance][6] リポジトリにあるテストデータをユニットテストに使用すると良いだろう。

[1]: https://dev.twitter.com/docs/api/1/get/help/configuration
[2]: https://github.com/twitter/twitter-text-objc
[3]: https://github.com/twitter/twitter-text-java
[4]: https://github.com/twitter/twitter-text-js
[5]: https://github.com/twitter/twitter-text-rb
[6]: https://github.com/twitter/twitter-text-conformance
