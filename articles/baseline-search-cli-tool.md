---
title: "Baseline情報をCLIで検索できるbaseline-searchを開発しました【Baseline Tooling Hackathon】"
emoji: "🔍"
type: "tech"
topics: ['cli', 'baseline', 'typescript', 'react','ink']
published: false
---

## はじめに
こんにちは、フロントエンドエンジニアのryoです。

Chromeチームが主催する[Baseline Tooling Hackathon](https://baseline.devpost.com/)に参加し、`baseline-search` というCLIツールを開発しました。本記事では、このツールについて紹介します。

## Baselineとは
[Baseline](https://web.dev/baseline)は、あるWeb Platform上の機能が主要なブラウザで動作するかについて明確な情報を提供する指標です。
この指標を活用する事で、自身が開発に携わっているアプリ・サイトで特定の新機能を用いて良いかどうかの基準を設けたりする事が出来ます。

完全な定義を確認したい場合は、web-featuresリポジトリで管理されている [Baseline.md](https://github.com/web-platform-dx/web-features/blob/main/docs/baseline.md)をご確認ください。


## baseline-searchとは
baseline-searchは、Web Platform機能のBaseline情報をCLI上から検索・閲覧できるツールです。
![Demo](https://github.com/ryohiy/baseline-search/blob/main/assets/baseline-search-demo.gif)


以下のコマンドですぐに利用する事が出来るため、ぜひお試しください！：

```bash
# 英語版（デフォルト）
npx baseline-search

# 日本語版
npx baseline-search --ja
```

### 

## 主な機能

### 1. フリーテキスト検索
Web Platform機能をキーワードで検索し、Baselineステータスを確認できます。

### 2. Baseline Target List
Baseline Widely AvailableやNewly Availableなど、ステータス別に機能を一覧表示できます。

### 3. Recent Updates
過去28日間にBaselineステータスが変更された機能を確認できます。Web標準の最新動向をキャッチアップするのに便利です。

### 4. 詳細情報の表示
各機能を選択すると、詳細なBaselineステータスやブラウザサポート情報を表示します。

## まとめ
baseline-searchは、Baseline情報をターミナルから素早く確認できるCLIツールです。開発中にブラウザを開かずにWeb Platform機能のサポート状況を確認したい方に便利だと思います。

ぜひ使ってみて、フィードバックをいただけると嬉しいです！
