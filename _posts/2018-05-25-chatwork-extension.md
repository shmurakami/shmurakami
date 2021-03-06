---
layout: post
posted: 2018-05-25
title: ChatWorkのChrome Extensionを作った
description: 
---

# tl;dr

GitHubにあげてます。[https://github.com/shmurakami/chatwork-completion](https://github.com/shmurakami/chatwork-completion)
今のところ公開するつもりはありません。
このExtensionでなにかあったとしても責任は取れませんので、自己責任でご利用ください。

---

そういえば先週からChatWork株式会社に入社してます。よろしくお願いします。Happyさんどうもありがとう。お礼を言いそびれているのでここでｗ またいずれ飯でも。

なお、この記事に限らず私が発信する内容は私個人の考えであり所属する組織には一切関係ありません。

---

さて、そういうわけで仕事でチャットワークを使い始めているのだけど。前職では完全にSlackでChatOpsだぜーとかやってたくらいだからSlackの手癖がまだ残っている状態なわけです。

基本的にキーボードでいろんな作業を完結したいタイプで、チャット上でもそれは当然同じ。
Slackはキーボードショートカットや/コマンドが充実していて大体のことにマウスを使う必要がなくとても便利でした。

先日の[Specイベント](https://slackhq.com/introducing-spec-slacks-first-developer-conference-26736fc6f92c)でもslash commandの拡充が発表されてました。
イメージですがSlackのユーザにはエンジニアが多いので相性が良いのでしょう。

チャットワークにもキーボードショートカットがいくつかあってよく使わせてもらってますが、個人的に1番ほしい、そして手癖が抜けないメンションを送りたいときのショートカットが無いのがつらい。
Slackだと`@`を打つとchannel内のユーザリストが表示されて、誰にメンションを送るか選択できます。インクリメンタルにフィルタもかけてくれます。

こんなやつ

![image](https://user-images.githubusercontent.com/1549858/40548404-9477d21c-606f-11e8-908c-0270b8b4773c.png)

こんなリストを表示するショートカットが欲しい、ということでChrome Extensionで実装することにしました。

ということでこの記事は、こんなリストを作るためにどんなことをしようとしたのか、という記録です。
しようとしたのか、ということで、実際にはしていません。

何故かと言うと、チャットワークはすでに似たようなToリストを持っていて

![image](https://user-images.githubusercontent.com/1549858/40548476-d3e35e58-606f-11e8-8c35-2cc7198710a0.png)

このボタンのクリックイベントだけ発火させてやれば十分だということに気づいてしまったのでそれをするだけになりました。

そもそもこのボタンを押すためにキーボードから手を動かしたくない、ということだったんだから、代わりにクリックイベントを発火させられればそれで十分だったんだよね。


# 実装するつもりだったもの

上記のような事情で、以下は実際にやるつもりだったこと、最終的にやったことです。


## 要件

- メンバーのリスト
- `@` キーを押したら反応する
- 単語の区切りなら動くが文章の途中では動かない. 要するに正規表現だと `\s@` もしくは `^@` の状態になったら動くようにする
- インクリメンタルにフィルタリングされる
- キーボードで上下にフォーカスを移動できる. エンターで選択
- マウスのホバーでもフォーカスを移動できる. クリックで選択

こんな感じの要件が自分の期待するUIです。

`@`で発火するところ以外はチャットワークにすでに実装されてたので実際には何も考える必要がなかったです。なんか申し訳なくなった。


## 技術要素

この要件をブラウザ上で満たそうと思ったときに、「メッセージ入力欄(textarea)にブラウザ上のフォーカスは残しつつ、つまり文字入力は続けつづ、上下でリストを移動できるようにする」というのがあまりピンと来てません。
selectタグでやろうとするとtextareaからフォーカス外れちゃうし、上下キーのイベントも取得して制御してるの？もっと良い方法あるのでは？といった感じ。

よく分からなかったのでブラウザ上でSlackを開き、メンションリストを表示させてDev ToolでHTMLを見ることに。大体以下のような感じでした。`li`タグ以下は関係ないのでバッサリ切ってます。

```html
<ul class="type_members" role="listbox">
    <li role="option" 
        tabindex="-1"
        class="tab_complete_ui_item active" 
        id="tab_complete_ui_item_0" 
        data-type="member" 
        data-index="0" 
        aria-selected="true"> 
        <span>...</span>
    </li>
</ul>
```

普通のリストとして記述されてますね。
気になったのは `role="listbox"`。role属性はデザイン系のフレームワークだったりツールを使うとたまに見かけるけど、これで良い感じのリストとして何か表現できるのでは？と当たりをつけてググることにした。

Slackの実装方法はこれ以上深く追ってないので特にそっちには期待しないでください。


### WAI-ARIA

listboxでググったら[WAI-ARIAのページ](https://www.w3.org/TR/wai-aria-1.1/#listbox)が見つかりました。WAI-ARIAって見たことあるけどよく知らないっていうものだったけど、W3CによるWebアクセシビリティに関するプロジェクトのことらしい。

その辺を見てたら欲しいものそのまんまのlistboxのサンプルを載せてるページを発見。 [https://www.w3.org/TR/wai-aria-practices/examples/listbox/listbox-scrollable.html](https://www.w3.org/TR/wai-aria-practices/examples/listbox/listbox-scrollable.html)

![image](https://user-images.githubusercontent.com/1549858/40553377-684883ea-607d-11e8-9b52-77db1cc519a7.png)

このページで読み込まれてるjsの中にlistbox.jsというのがあって、W3C的にどうしてるのかが分かって非常に良かった。興味があれば読んでみると良いのではないでしょうか。500行くらいの大して難しくない実装でした。

listbox.jsの中では、カーソルキーのキーボードイベントを取得してフォーカスを移動してることが分かりました。予想した通りじゃないか。それだけだとフォーカスが描画範囲から外れてもスクロールの追従とかやってくれないからカーソルイベントに合わせてスクロール位置の調整とかもしないといけない。まぁそうだよなぁ、と理解できるけど、泥くさくやってるのだなぁ。


### 描画要素へのフォーカス

W3Cがそうしてるならしょうがない、ということで自分でlistboxを実装しようとしたけど、同じようにやってもも`ul`要素にキーボードのフォーカスがあたらない。
ググったところ、 `ul`とかのような本来フォーカスが当たらない要素にフォーカスを当てるには(?)、 `tabindex` 属性を追加しないといけないらしい。というかtabindex属性を追加するとそういうことができるらしい。
見返してみたらSlackにもW3Cのサンプルにもあったわ。

HTMLのドキュメントにはこんなふうに記載されてる模様

![image](https://user-images.githubusercontent.com/1549858/40550680-c73b7220-6075-11e8-8f08-da7d4260a477.png)


### Extension側の実装

Chrome Extensionは数年前にも実装したことがあったけど、その頃からあんまり変わってないな？という印象。
manifest書いてbackgroundで動くスクリプトやら、content_scriptとして動かすjsを書いてっていう感じ。

querySelectorも賢いし特にjQueryとか使う必要も無いな、といった状態なのは余計なもの入れなくて済むのでありがたい。

今のところ1ファイルに全部押し込んでるけど、Chromeがまだimport/exportをサポートしてるわけでもないし、requireもできないから普段のノリでjsを書こうとするとビルドツールを入れなきゃいけなさそうだなぁとなった。

しかしたった数個のスクリプトのためにwebpackやらbrowserifyを入れるのはとてもToo much感が... とはいえ、jsを分割して読み込み順に気をつけながら、とかもうやってらんない。babelでこの辺どうにかできたかな？


### 非同期コンテンツへの対応

チャットワークを使ったことがある人だと分かるかもしれないけど、サイトにアクセスすると非同期でコンテンツを取ってきてレンダリングする方式なんですよね。そういうサイトだとChrome拡張のcontent_scriptの実行タイミングではどうしてもDOMが出来上がってないので要素が見つからなくてエラーになります。

そういうときに `MutationObserver` っていうのが使える模様。

> MutationObserver provides developers with a way to react to changes in a DOM. It is designed as a replacement for Mutation Events defined in the DOM3 Events specification.

DOMの変更に対してリアクションする機能を提供してくれる。もともとMutation EventsっていうのがあったけどこれはもうWeb Standardから除外されてて、今はこのMutationObserverを使うらしい。

このObserverをキレイに実装してくれてるgistを見つけたのでほぼそのまま使わせてもらってる [https://gist.github.com/jwilson8767/db379026efcbd932f64382db4b02853e](https://gist.github.com/jwilson8767/db379026efcbd932f64382db4b02853e)

MutationObserverで目的の要素がレンダリングされたらextensionの処理を動かすようにして無事に実行できた。

Observerを使って目的のDOMが生成されたのを検知したらDOMに対してもろもろイベントリスナーをセットする、というわけです。


# このあと

Toリストはこんな事情で割と簡単に実装できたけど、あとはできれば`>`で始まる行を引用タグで囲うようにしたい。

こんな拡張作りましたよっていう話をフロントエンドチームの方に話したらプルリクくれって言われたので、時間を作ってやっていきたい。けど既存のUXから見ると異質な動きな気がするからちょっと気後れしますね。

最近プライベートでフロントまわりしか触ってないな

