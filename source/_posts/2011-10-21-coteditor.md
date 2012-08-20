---
layout: post
title: CotEditorの検索パネル
tags: 
- Objective-C
---


CotEditorの検索パネルでReturnキーを押すとパネルが閉じてしまうので、その挙動を変えてみました。
実際はOgreKitのframeworkで検索パネルが実装されているので、CotEditor自体には手を加える必要はありませんでした。

SDKは10.6をターゲットにしてビルドしてます。PPCのバイナリは含まれていません。

[ここから][1] 手を加えたOgreKit.frameworkをダウンロードして、CotEditor.app/Contents/Frameworks/OgreKit.framework にコピーするだけ。Returnと一緒にShiftキーを押すと、以前と同じように検索後にパネルが閉じます。

追記: CotEditor本家に変更点が反映されたので、次のCotEditorのリリースではこの挙動がデフォルトになると思います。

[1]: https://dl.dropbox.com/u/86848/OgreKit_Find_Custom.zip
