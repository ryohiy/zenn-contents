---
title: "npm パッケージを npm trusted publishing を利用して公開するようにしたので、その知見をまとめる"
emoji: "🛡️"
type: 'tech'
topics: ['frontend','npm','security','githubactions']
published_at: '2025-11-17 08:00'
published: true
---

## はじめに

こんにちは。フロントエンドエンジニアをしている ryo です。<br/>
先日、[Baseline Tooling Hackathon](https://baseline.devpost.com/) に参加して [baseline-search](https://github.com/ryohiy/baseline-search) というツールを作成しました。シンプルなツールですが個人的には気に入っていて、ハッカソン終了後も手隙の時にこのツールを少し改善したりしています。<br/>
そのリポジトリで最近 npm trusted publishing を利用した GHA でのパッケージ公開の設定をしたので、その知見をまとめようと思います。よろしくお願いします。

## npm trusted publishing とは

[npm trusted publishing](https://github.blog/changelog/2025-07-31-npm-trusted-publishing-with-oidc-is-generally-available/) とは、2025/7/31 に公開された、比較的まだ新しい機能です。これは OpenID Connect (OIDC) を npm の認証に用いてパッケージの公開が出来る機能です。
これにより以下のような利点があります：

- npm トークンを Github secrets で管理する必要がない
  - [Nx に対して行われた secrets で管理していたトークンを盗まれる](https://github.com/nrwl/nx/security/advisories/GHSA-cxm3-wv7p-598c)ようなリスクをなくせる
- 非常に有効期間の短いトークンにより npm パッケージの公開を行うため、トークン流出時に再利用されにくい
  - [OIDC の仕組み](https://docs.github.com/ja/actions/concepts/security/openid-connect#understanding-the-oidc-token)を利用している
- `--provenance`不要で、自動で provenance 情報が付与される
  - [provenanceの詳細](https://github.blog/jp/2023-04-26-introducing-npm-package-provenance/)
- 自身でトークンのローテション作業をする必要がない

従来の npm トークンを発行する手法よりもセキュアな運用のため、基本的にはこの手法を利用してパッケージ公開をするべきかと思っています。

## 設定方法

基本的に [npm の公式ドキュメント](https://docs.npmjs.com/trusted-publishers#configuring-trusted-publishing)に示されている通りのやり方で全く問題ないと思います。[npmjs.com](https://www.npmjs.com/) のコンソール画面で trusted publisher の追加をして、npm パッケージの公開を行っているワークフローに OIDC のための権限を付与する感じです。

ワークフローについてですが、私のリポジトリでは基本的に[こちらの公式ドキュメントにある設定](https://docs.npmjs.com/trusted-publishers#github-actions-configuration)をそのまま行っています。以下のリンクが私のリポジトリの設定ファイルですが、見ての通りとてもシンプルに利用する事ができます。
https://github.com/ryohiy/baseline-search/blob/main/.github/workflows/release.yml

公式ドキュメントには更に、推奨事項や、従来の手法からの移行に関する内容等の記載もありますので、実施する際はぜひそちらをご一読ください。

## まとめ

今回は GHA 上で npm パッケージを公開する際の npm trusted publishing というセキュアな手法についてまとめました。設定も簡単なので、従来の手法を利用されている方は是非使ってみていただければと思います。読んでいただきありがとうございました。

## 参考資料

- https://github.blog/changelog/2025-07-31-npm-trusted-publishing-with-oidc-is-generally-available/
- https://efcl.info/2025/09/07/npm-oidc/
- https://docs.npmjs.com/trusted-publishers#supported-cicd-providers
- https://blog.jxck.io/entries/2025-09-03/nx-incidents.html
- https://docs.github.com/ja/actions/concepts/security/openid-connect#benefits-of-using-oidc
