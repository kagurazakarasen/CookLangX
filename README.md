# CookLangX

CookLangX は、料理レシピを実行可能な手順として扱うための料理 DSL 仕様プロジェクトです。

このリポジトリでは、YAML と操作 DSL を組み合わせたハイブリッド設計により、次を実現することを目指します。

- 人間が読み書きしやすいレシピ記述
- 機械可読な依存関係と並列実行表現
- 人数変更に応じた分量スケーリング
- タイマー、買い物リスト、栄養解析、将来の自動化との連携

## 仕様書

現行の仕様ドラフトは次のファイルにあります。

- [CookLangXSpecification-draft.md](CookLangXSpecification-draft.md)

主な内容:

- ドキュメントモデル（meta, ingredients, tools, definitions, steps）
- Operation DSL の基本構文
- 時間・温度・単位・スケーリング
- 依存グラフ（DAG）による実行セマンティクス
- Core / Extended の能力レベル
- バリデーションエラーコード
- 正準例（カルボナーラ）

## 学習ガイドとサンプル

CookLangX を手を動かして学ぶための資料を用意しています。

- 入門ガイド: [HowTo.md](HowTo.md)
- 写経用サンプル: [samples/garlic_pasta.ckx](samples/garlic_pasta.ckx)
- レベル別サンプル（コメント付き）:
	- [samples/beginner_boiled_egg.ckx](samples/beginner_boiled_egg.ckx)
	- [samples/intermediate_oyako_don.ckx](samples/intermediate_oyako_don.ckx)
	- [samples/advanced_weeknight_curry.ckx](samples/advanced_weeknight_curry.ckx)

おすすめの読み進め方:

1. [HowTo.md](HowTo.md) を上から読む
2. 初級サンプルを写経する
3. 中級・応用サンプルで依存関係と設計意図を確認する

## このリポジトリの現状

- 仕様策定フェーズ（v0.1-draft）
- 実装（パーサ、バリデータ、実行エンジン）は今後追加予定

## 想定ユースケース

- レシピを構造化データとして管理したい
- 調理工程の依存関係と並列性を明確にしたい
- 調理支援アプリや音声ナビへの変換基盤を作りたい
- 将来的な IoT 家電連携の中間表現を整備したい

## 今後の候補

- DSL の形式 EBNF 定義
- 参照実装（Parser / Validator）の最小プロトタイプ
- サンプルレシピセットの追加
- 互換性テストケースの整備

## ライセンス

MIT License を採用しています。

- [LICENSE](LICENSE)
