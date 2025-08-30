---
title: "なぜCSS @layerが生まれたのか Cascade Layersの背景と仕組みを理解する"
emoji: "🎨"
type: "tech" 
topics: ['frontend', 'css','web']
published_at: '2025-08-26 07:00'
published: true
---

## はじめに
こんにちは、フロントエンドエンジニアのryoです。
多くの方はご存知かもしれませんが、Tailwind CSSでは @layer を活用してスタイリングの構造化が行われています。

個人的にはとても整理された手法で良いなと思いつつも、それに関連してちょっと詰まったりもしました。また、詳細度では十分じゃなかったんだっけ？なぜlayerが必要だったのか？という疑問も湧いてきたため、@layer に焦点を当てて深掘りして、自分の理解を整理するためにブログにまとめてみようと思いました。

内容については、可能な限り調べて慎重に書いたつもりですが、もし誤った内容があればご指摘いただければ幸いです。それでは始めていきます。

## @layer とは
[MDNのドキュメント](https://developer.mozilla.org/ja/docs/Web/CSS/@layer)には以下のように説明されています。
> @layer は CSS のアットルールで、Cascade Layers（カスケードレイヤー）を宣言するために使用し、また複数のCascade Layersがある場合に、優先順位を定義するためにも使用することができます。

この説明にあるCascade Layersという用語は何を指すのでしょうか。また、CSSには既に[Specificity（詳細度）](https://developer.mozilla.org/ja/docs/Web/CSS/CSS_cascade/Specificity)による優先順位付けのアルゴリズムが存在するのに、なぜ新たに優先順位を定義するためのアットルールが必要だったのでしょうか。

これらの疑問を解消するために、まずはCascadeの仕組みから掘り下げていきます。

## Cascade
CSSによってスタイルが適用される際は、[Cascade（カスケード）](https://developer.mozilla.org/ja/docs/Web/CSS/CSS_cascade/Cascade)のアルゴリズムによってスタイルの優先順位の計算が行われて、一番優先順位が高いスタイルが適用されています。

まずは、CSS における Cascadeを実行する際の処理を整理していきます。ここでは一旦Cascade Layersは考えずに、基本的な仕組みから見ていきましょう。

### Cascade は何をするのか
Cascade は その要素のプロパティ宣言から提供される[Declared Values](https://www.w3.org/TR/css-cascade-5/#declared-value)をソートされていない状態で受け取り、最大で一つの[Cascaded Value](https://www.w3.org/TR/css-cascade-5/#cascaded-value)を決めるステップです。
優先順位付けの基準は、現在以下の様に決まっており、上から順に実行してDeclared Valuesをソートしていき、完全にsortされたリストを出力する事で一つの"winning value"を決定します。(ただし、Cascadeによって出力されたlistが空の事もあり、この時はcascaded valueは存在しません)

1. Origin and Importance
2. Context
3. Element-Attached Styles
4. Layers
5. Specificity
6. Order of Appearance

Layersは一旦説明せずに飛ばしつつ、それぞれの基準について整理していきます。

### Origin and Importance
CSS はまずユーザー、Author、そしてユーザーエージェント（ブラウザ）という3つの「core origins」間の力関係のバランスから始まります。
- User-agent stylesheets（ユーザーエージェントスタイルシート）
    - ざっくりブラウザが提供するスタイルシートの事
- Author stylesheets（作成者スタイルシート）
    - 最も一般的なスタイルシートで、開発者によって作成されたシートを指す
- User stylesheets（ユーザースタイルシート）
    - ブラウザが提供する機能によるwebサイトのユーザーが独自に設定するスタイルシート

そして、上記のOrigins（オリジン）の他にCSSの拡張で以下の二つのadditional Originsが定義されています。
- Animation Origin
- Transition Origin

[本ブログ執筆の現在では上記の通り、5つがOriginsとして定義されています](https://www.w3.org/TR/css-cascade-5/#cascading-origins)。

#### !important キーワード
!important フラグはCascade内の宣言を選択するルールを変更します。 
https://developer.mozilla.org/ja/docs/Web/CSS/important
normal と比べるとOriginsの優先順位が逆転します。core origins と 重要度 による優先順位付けは降順で以下の通りです:
1. User-agentのimportant付きスタイル
2. Userのimportant付きスタイル
3. Authorのimportant付きスタイル
4. Authorの通常スタイル
5. Userの通常スタイル
6. User-agentの通常スタイル

#### Origin and Importanceでの優先順位
additional origins も追加すると、優先順位付けは降順で以下の通り、8段階になります:
1. トランジション中のスタイル
2. User-agentのimportant付きスタイル
3. Userのimportant付きスタイル
4. Authorのimportant付きスタイル
5. アニメーション中のスタイル
6. Authorの通常スタイル
7. Userの通常スタイル
8. User-agentの通常スタイル

この評価が[Origin and Importance](https://www.w3.org/TR/css-cascade-5/#cascade-origin)です。

### Context
異なるカプセル化コンテキスト（Shadow DOMなどのネストされたツリーコンテキスト）から提供されるスタイル宣言の優先順位に関する基準です。
詳細: https://www.w3.org/TR/css-cascade-5/#cascade-context

### Element-Attached Styles
要素に直接付与されたスタイル（style属性など）と、セレクタ経由で適用されるスタイルの優先順位に関する基準です。
詳細: https://www.w3.org/TR/css-cascade-5/#cascade-element-attached

### Specificity
Specificity（詳細度）による優先順位付けです。ID、クラス、要素タイプの3つの重みで計算されます。

### Order of Appearance
Order of Appearance（登場順序）による優先順位付けです。同じ優先順位のルールが複数ある場合、後に記述されたものが優先されます。

### 開発者視点でのCascade
開発者視点では、Author Originsの中でスタイルを実装していく訳なので、基本的にSpecificityでの優先順位を意識してスタイルを実装しているかと思います。
(!importantの有無は一番最初のsortロジックに効いてくるので、そういった意味でもスタイルの複雑さを上げてしまうな。とは思いました)。

ここのロジックについて完全に正確なドキュメントを参照したい場合は、https://www.w3.org/TR/css-cascade-5/#cascade-sort をご参照ください。

## @layer はなぜ生まれたか
ここまでCascadeの仕組みを見てきましたが、実は開発者にとってはSpecificityとOrder of Appearanceだけでスタイルを管理することには限界がありました。その課題と、それを解決するための取り組みを見ていきましょう。

### CSS設計手法の登場
スタイルの複雑さに対応するため、様々なCSS設計手法が生まれました。その代表例として、[ITCSS（Inverted Triangle CSS）](https://www.xfive.co/blog/itcss-scalable-maintainable-css-architecture)を見ていきます。

ITCSSはHarry Roberts氏によって提唱されたCSS設計手法です。私が探した限りでは、2016年に公開された[こちらの記事](https://www.creativebloq.com/web-design/manage-large-css-projects-itcss-101517528)が一番古いようには見えますので、この辺りから一般に浸透し始めたようです。この手法は、開発時のCSSによるスタイリングの難しさに立ち向かうために生まれました。

#### なぜCSS設計手法が必要だったのか
CSSを用いたWebのフロントエンド開発は年々複雑になっていました。スタイリングもリッチな実装が求められ、関わる開発者も増えていったからです。

ここで重要なのは、開発で影響するOriginsは全てAuthor stylesheetsであるという点です。つまり、基本的にはSpecificityだけで制御していく必要があります。SpecificityはID列、CLASS列、TYPE列の3つの値で決まりますが、CSSが複雑になるほど詳細度の競合が発生し、SpecificityでCascaded Valueが決まらないケースが増えます。そうなるとOrder of Appearance（登場順序）という非常に曖昧な基準に頼ることになります。

その結果、保守性やスケーラビリティに課題が出てきました。

#### ITCSSのアプローチ
ITCSSは、CSSを複数のレイヤー（Settings、Tools、Generic、Elements、Objects、Components、Utilities）に分割し、逆三角形の構造で整理する手法です。上から下に向かって、より具体的で高いSpecificityを持つスタイルを配置することで、Specificityの競合を防ぎ、予測可能なCSSを実現しようとしていました。

ITCSSに限らず、様々なCSS設計手法が近しいアプローチで、スケーラブルでメンテナンス性の高いCSSを実現しようとしていました。

### 設計手法の限界と新たな提案
しかし、これらの設計手法にも課題がありました：
- 記法のルールでしか縛れないため、運用が繊細
- サードパーティライブラリによって簡単に壊れる可能性
- チーム全員が設計手法を理解し、遵守する必要がある

そこで、2019年10月29日、Miriam氏によって[Cascadeの基準に関する提案](https://github.com/w3c/csswg-drafts/issues/4470)がなされました。

提案の内容を簡易的にまとめると：
- デザインシステムの定義では、トークン、デフォルト、パターン、コンポーネントといった抽象化レイヤーを構築することが一般的（OOCSS、Atomic Design、ITCSSなど）
- これらを実現するには、レイヤーに応じたSpecificityの慎重な管理が必要で、サードパーティツールによって簡単にバランスが崩れる
- Cascading OriginsとimportantがUA/user/author間で同じ問題を解決しているように、Author Origin内でもカスタムオリジン（レイヤー）を制御できるようにすれば、多くの「Specificity問題」を解決できる

つまり、開発者が概念的に作っていたレイヤーを、CSS仕様として正式にサポートしようという提案でした。

[Miriam氏によるCascade Layersの仕様に関するブログ](https://www.miriamsuzanne.com/specs/cascade-5/)も、この背景をとても分かりやすく説明しています。

なお、MDNでは簡単な@layerに関する課題も用意されているので、実際に触ってみるのも良いかと思います。
https://developer.mozilla.org/ja/docs/Learn_web_development/Core/Styling_basics/Test_your_skills/Cascade#%E8%AA%B2%E9%A1%8C_2

## Cascade Layers
2019年のMiriam氏の提案を受けて、Cascade Layersが標準化されました。

[Cascade Layers](https://www.w3.org/TR/css-cascade-5/#cascade-layers)の仕様では以下のように説明されています。
> In the same way that cascade origins provide a balance of power between user and author styles, cascade layers provide a structured way to organize and balance concerns within a single origin. Rules within a single cascade layer cascade together, without interleaving with style rules outside the layer.

つまり、Originsがユーザーとauthorのスタイル間でバランスを提供するのと同じように、Cascade Layersは単一のOrigin内で関心事を整理し、バランスを取る構造的な方法を提供します。各レイヤー内のルールは、レイヤー外のルールと混在することなく、独立してCascadeされます。

これはまさに、様々なCSS設計手法がSpecificityのルール化によって作っていた抽象化レイヤーを、CSSの仕様として正式にサポートできるようになったということです。

先ほどCascadeセクションで触れた通り、LayersはSpecificityの評価の前に位置します。これにより、Specificityに頼らずにスタイルの優先順位を制御できるようになったのです。

Cascadeにおけるlayerの基準の詳細はこちらで確認できます：
https://www.w3.org/TR/css-cascade-5/#cascade-layering

### 設計思想から標準仕様へ
ここまでの流れを整理すると：
- CSSの複雑化に伴い、Specificityだけでの管理に限界が生じた
- ITCSSなどの設計思想が抽象化レイヤーという解決策を提示した
- その概念がCascade Layersとして標準化された

次は、@layerの具体的な使い方を見ていきましょう。

## @layer の詳細
ここでは、@layerの具体的な仕様と動作を見ていきます。

### レイヤーの優先順位
@layerの仕様で最も重要なのは、レイヤーの優先順位の仕組みです。[MDNのドキュメント](https://developer.mozilla.org/ja/docs/Web/CSS/@layer)に掲載されている以下の図は、1, 2, ..., N の順に宣言されたレイヤーの優先順位を示しています。

![レイヤーの優先順位の図（MDNより引用）](/images/css-cascade-layers/cascade-layers.png)
*出典: [MDN - @layer](https://developer.mozilla.org/ja/docs/Web/CSS/@layer)*

この図を見ると、Origin and Importanceの基準によってimportant付きのAuthor宣言と通常のAuthor宣言で分けられ、その中でLayersによる評価が行われていることが分かります。また、Origin and Importanceの基準と同様、important付きバケットでのレイヤーの優先順位は通常の宣言とは逆転していることも分かりますね。

特に注意が必要なのは、通常のAuthor宣言内では**レイヤー外の宣言が最も優先される**という点です。既存のWebアプリに@layerを導入する際は、この仕様により意図しないスタイルの適用が起きる可能性があります（実際、私もこの点で躓き、それが本記事を書くきっかけの一つになりました）。

### 実装への影響
この仕様により、既存のCSSに@layerを追加する際は、既存のスタイルをどのレイヤーに配置するか、あるいはレイヤー外に残すかを慎重に検討する必要があります。

Tailwind CSSではすでにCascade Layersを用いた実装が行われているので、次はそちらを見てみましょう。

## Tailwind CSS における @layer の活用
Tailwind CSSは早い段階から@layerの概念を取り入れ、v4で本格的にCascade Layersを採用しました。その変遷を見てみます。

### v3での独自実装
Tailwind CSS v3では、[ドキュメント](https://v3.tailwindcss.com/docs/adding-custom-styles#using-css-and-layer)でlayerで分ける理由が説明されています。ただし、この時点ではまだCascade Layersの仕様が確定していなかったため、ITCSSのルールに則ってTailwind CSSが独自に実装した@layerディレクティブでした。

### v4でのCascade Layers採用
[Tailwind CSS v4のリリース](https://tailwindcss.com/blog/tailwindcss-v4#designed-for-the-modern-web)で、正式にCascade Layersを採用したことが発表されました。

v4でのlayerを用いたカスタマイズ手法はこちらで確認できます：
https://tailwindcss.com/docs/adding-custom-styles#using-custom-css

### Tailwind CSSのレイヤー構造
Tailwind CSS v4でのCascade Layersの構造について説明する公式ドキュメントは見つかりませんでしたが、[GitHubのソースコード](https://github.com/tailwindlabs/tailwindcss/blob/e4c0255e3aabafb83e7324f2ac816c13370b8c15/packages/tailwindcss/index.css#L1)を確認すると、以下のように定義されています：

`@layer theme, base, components, utilities;`

[v3のドキュメント](https://v3.tailwindcss.com/docs/adding-custom-styles#using-css-and-layer)では、各レイヤーの役割が以下のように説明されています：
- base layer: リセットルールやプレーンなHTML要素へのデフォルトスタイル
- components layer: ユーティリティで上書き可能なクラスベースのスタイル
- utilities layer: 常に他のスタイルより優先される、単一目的の小さなクラス

実際に、`.text-neutral-900`のようなクラスはutilitiesレイヤーに、Tailwind CSSがh1要素に適用するスタイルはbaseレイヤーに配置されていることが、開発者ツールで確認できます。

### 既存プロジェクトへの影響
Tailwind CSSがCascade Layersを使用しているため、Tailwind CSS導入以前にAuthor宣言でCSSを記述していた場合、適切なレイヤーに宣言を移動する必要があります。そうしないと、レイヤー外の宣言が優先されるという仕様により、意図しないスタイルが適用される可能性があります。

この対処法については、[Tailwind CSSのカスタムスタイルのドキュメント](https://tailwindcss.com/docs/adding-custom-styles#adding-base-styles)で詳しく説明されています。

## まとめ
Cascade Layers がどの様にして生まれたか、背景知識を整理しつつまとめることができて、個人的に良い学習機会になりました。
CascadeアルゴリズムとOriginsの仕組みを改めて理解できたことも大きな収穫でした。
この記事が、同じような疑問を持つ方の理解の助けになれば幸いです。読んでいただきありがとうございました！

## 参考資料
- https://www.w3.org/TR/css-cascade-5
- https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascade/Cascade
- https://developer.mozilla.org/ja/docs/Web/CSS/@layer
- https://www.miriamsuzanne.com/specs/cascade-5/
- https://github.com/w3c/csswg-drafts/issues/4470
- https://www.xfive.co/blog/itcss-scalable-maintainable-css-architecture
- https://www.creativebloq.com/web-design/manage-large-css-projects-itcss-101517528
- https://tailwindcss.com/blog/tailwindcss-v4#designed-for-the-modern-web
- https://tech-blog.rakus.co.jp/entry/20211210/layer
- https://blog.jxck.io/entries/2023-01-07/new-css-capabilities-for-component.html#@layer