# CookLangX 仕様 v0.1-draft

CookLangX は、料理レシピを実行可能な手順として表現するためのドメイン特化言語です。  
本仕様は、構造表現に YAML、操作行にコンパクトな DSL 文字列を用いる実用的なハイブリッドモデルを定義します。

## 1. 目的

CookLangX は次の目的を持ちます。

1. 人間が書きやすく、機械が読み取りやすいレシピ表現を実現する。
2. 依存関係と並列実行可能な作業を明示的に表現する。
3. 人数に応じた材料量スケーリングを信頼性高く行う。
4. タイマー、買い物リスト、栄養解析、将来の自動化との相互運用性を確保する。

## 2. 設計原則

1. レシピ記述は単なる文章ではなく、操作と状態遷移として扱う。
2. レシピはステップから成る有向非巡回グラフ（DAG）として扱う。
3. 自然言語ノートは構造化データと共存できる。
4. コア構文は小さく保ち、高度機能は能力レベルで段階化する。

## 3. 適合性に関する用語

本仕様における MUST、MUST NOT、SHOULD、SHOULD NOT、MAY は、RFC 2119 スタイルの慣用に従って解釈されます。

### 3.1 日本語話者向けの読み方

英語の規範語は、日常的な「べき」「推奨」とは重みが異なります。実装判断で迷わないため、次の意味で読み替えてください。

1. MUST: 必須。守らない実装は本仕様に適合しない。
2. MUST NOT: 禁止。行うと本仕様に適合しない。
3. SHOULD: 強い推奨。通常は守るべきで、外すなら合理的理由と影響説明が必要。
4. SHOULD NOT: 強い非推奨。通常は避けるべきで、採用するなら合理的理由が必要。
5. MAY: 任意。実装してもしなくても適合性は維持される。

補足:

1. 仕様文の「必須」「禁止」は互換性や安全性に直結するため、プロダクト都合で緩和しないこと。
2. 「推奨」「非推奨」は、性能・実装コスト・対象ユーザーに応じて例外を許容できる。
3. MAY 項目を未実装にする場合でも、ドキュメントで未対応機能を明示すると運用上の混乱を防げる。
4. 本仕様では、適合性判定の基準は英語規範語（MUST/SHOULD/MAY）を優先し、日本語訳は理解補助として扱う。

## 4. ファイル形式

1. 推奨拡張子: `.ckx`
2. 文字コード: UTF-8
3. 正規表現形式: YAML ドキュメント
4. MIME（提案）: `application/vnd.cooklangx+yaml`

## 5. ドキュメントモデル

CookLangX ドキュメントは次のトップレベルセクションで構成されます。

1. `meta`（必須）
2. `ingredients`（必須）
3. `tools`（任意）
4. `definitions`（任意）
5. `steps`（必須）

### 5.1 meta

`meta` は次を MUST で含むこと。

1. `name`（文字列）
2. `servings`（0 より大きい数値）

`meta` は次を MAY で含めてもよい。

1. `author`（文字列）
2. `lang`（BCP47 言語タグ。例: `ja`, `en-US`）
3. `tags`（文字列配列）
4. `time_total`（duration）

### 5.2 ingredients

`ingredients` は材料オブジェクトの配列です。

各材料は次を MUST で含むこと。

1. `id`（識別子）
2. `name`（文字列）

各材料は次を MAY で含めてもよい。

1. `amount`（数値）
2. `unit`（単位文字列）
3. `scalable`（真偽値、既定値 true）
4. `state`（文字列。例: `raw`, `diced`）
5. `substitute`（材料 id の配列）
6. `nutrition`（オブジェクト）

### 5.3 tools

`tools` は道具オブジェクトの配列です。

各道具は次を MUST で含むこと。

1. `id`（識別子）
2. `name`（文字列）

各道具は次を MAY で含めてもよい。

1. `capacity`（数値または文字列）
2. `temperature_range`（`min` と/または `max` を持つオブジェクト）

### 5.4 definitions

`definitions` は再利用可能なマクロ操作の配列です。

各定義は次を MUST で含むこと。

1. `id`（識別子）
2. `params`（文字列配列。空配列可）
3. `body`（操作行またはステップテンプレートの配列）

### 5.5 steps

`steps` は可読性のため順序付き配列ですが、実行順序は依存関係で決定されます。

各ステップは次を MUST で含むこと。

1. `id`（識別子）
2. `run`（DSL 操作行）

各ステップは次を MAY で含めてもよい。

1. `uses`（材料参照および/または出力参照の配列）
2. `tools`（tool id の配列）
3. `time`（duration）
4. `temperature`（temperature）
5. `depends_on`（step id の配列）
6. `state_in`（オブジェクト）
7. `state_out`（オブジェクト）
8. `produces`（出力識別子）
9. `yield`（amount と unit を持つオブジェクト）
10. `when`（式文字列。Extended 能力）
11. `repeat`（繰り返し記述子。Extended 能力）
12. `note`（自然言語ノート）

## 6. 識別子および参照ルール

### 6.1 識別子

識別子の正規表現:

```text
^[a-z][a-z0-9_]*$
```

識別子は大文字小文字を区別します。

### 6.2 参照

1. 材料参照: `$<ingredient_id>`
2. ステップ出力参照: `%<step_or_output_id>`
3. DSL 引数内の道具参照: `&<tool_id>`（省略形、任意）

例:

```text
$spaghetti
%boiled_pasta
&frying_pan
```

## 7. Operation DSL

`run` は 1 行の操作を保持します。

### 7.1 コア形式

```text
<action> <arg>* [-> <output_id>]
```

定義:

1. `<action>` は識別子。
2. `<arg>` は次のいずれか。
   - 参照（`$id`, `%id`, `&id`）
   - キー値トークン（`key=value`）
   - クォート文字列（`"..."`）

例:

```text
boil $spaghetti water=2L salt=2g -> boiled_pasta
fry $bacon heat=medium -> crisp_bacon
mix %boiled_pasta %crisp_bacon $egg -> pasta_mix
```

### 7.2 推奨プリミティブアクション

コア実装のアクション名は、少なくとも次を SHOULD で含むことが推奨されます。

1. `wash`
2. `cut`
3. `mix`
4. `boil`
5. `steam`
6. `fry`
7. `saute`
8. `bake`
9. `simmer`
10. `season`
11. `combine`
12. `rest`
13. `serve`

実装は追加のアクション名を MAY でサポートして構いません。

## 8. 時間と温度

### 8.1 Duration

duration の字句形式:

1. `30s`
2. `8min`
3. `1h`
4. ISO 8601 duration を MAY でサポートしてよい（`PT8M`）。

正規化後の標準単位は秒です。

### 8.2 Temperature

サポートされる形式:

1. `180C`
2. `356F`
3. 定性的火加減（`low`, `medium`, `high`）

数値温度と定性的火加減が同時にある場合、数値温度を優先します。

## 9. 単位とスケーリング

### 9.1 単位セット

推奨組み込み単位:

1. 質量: `mg`, `g`, `kg`
2. 体積: `ml`, `l`, `tsp`, `tbsp`, `cup`
3. 個数: `piece`, `clove`, `leaf`

### 9.2 スケーリング規則

次を与える:

1. 元の人数 $S_o$
2. 目標人数 $S_t$

`scalable=true` の各材料について、スケール後の量は次式とする。

$$
amount' = amount \times \frac{S_t}{S_o}
$$

`scalable=false` の場合、amount は変更してはならない（MUST）。

丸め方針は設定可能であることが SHOULD。既定は個数単位以外を小数第 2 位の四捨五入（half-up）とします。

## 10. 依存関係と実行セマンティクス

1. ステップは明示 `depends_on` に依存する。
2. ステップは `run` または `uses` の `%output` 参照から暗黙依存も持つ。
3. 最終依存グラフは非循環（acyclic）でなければならない（MUST）。
4. 経路制約のない 2 ステップは並列実行してよい（MAY）。
5. スケジューラは道具競合を尊重しつつ、クリティカルパス短縮を優先することが SHOULD。

### 10.1 道具競合ルール

同一の排他的道具インスタンスを必要とする準備完了ステップ同士は、同時実行してはならない（MUST NOT）。

## 11. 状態遷移モデル

各ステップは `state_in` と `state_out` を MAY で定義できます。

例:

```yaml
state_in:
  onion: raw
state_out:
  onion: translucent
```

状態制約がある場合:

1. `state_in` は先行する生成ステップに照らして検証されることが SHOULD。
2. 解決不能な必要状態は、厳格度モードに応じて警告またはエラーを出すことが SHOULD。

## 12. 自然言語との共存

`note` により人間向けヒントを保持できます。

実装要件:

1. 依存関係セマンティクスでは `note` を無視しなければならない（MUST）。
2. UI、音声ガイド、印刷出力で `note` を表示してよい（MAY）。

## 13. 能力レベル

### 13.1 Core

Core 実装は次を MUST でサポートすること。

1. 第 5 章のセクション
2. 7.1 の DSL
3. 8.1 の時間表現
4. 第 9 章のスケーリング
5. 第 10 章の DAG 検証

### 13.2 Extended

Extended は次を追加します。

1. `when` による条件付きステップ有効化
2. `repeat` ループ
3. `definitions` マクロ展開
4. 栄養・買い物リスト連携メタデータ

## 14. バリデーション規則

適合バリデータは機械可読なエラーコードを報告することが SHOULD。

推奨エラーセット:

1. `CKX001`: 必須セクション欠落
2. `CKX002`: 識別子形式不正
3. `CKX003`: 識別子重複
4. `CKX004`: 不明な材料/道具/出力参照
5. `CKX005`: 依存サイクル検出
6. `CKX006`: duration 形式不正
7. `CKX007`: temperature 形式不正
8. `CKX008`: servings が正でない
9. `CKX009`: 未サポート単位
10. `CKX010`: マクロ展開失敗

## 15. 正準例（カルボナーラ）

```yaml
meta:
  name: Carbonara
  servings: 2
  lang: ja
  spec_version: 0.1

ingredients:
  - id: spaghetti
    name: Spaghetti
    amount: 200
    unit: g
  - id: egg
    name: Egg
    amount: 2
    unit: piece
  - id: bacon
    name: Bacon
    amount: 80
    unit: g
  - id: salt
    name: Salt
    amount: 2
    unit: g

tools:
  - id: pot
    name: Pot
  - id: frying_pan
    name: Frying Pan

steps:
  - id: boil_pasta
    run: boil $spaghetti water=2L salt=2g -> boiled_pasta
    tools: [pot]
    time: 8min
    temperature: 100C

  - id: cook_bacon
    run: fry $bacon heat=medium -> crisp_bacon
    tools: [frying_pan]
    time: 4min

  - id: make_egg_mix
    run: mix $egg -> egg_mix
    time: 1min

  - id: combine
    run: combine %boiled_pasta %crisp_bacon %egg_mix -> carbonara
    depends_on: [boil_pasta, cook_bacon, make_egg_mix]
    note: Keep heat low to avoid scrambling egg.

  - id: plate
    run: serve %carbonara
    depends_on: [combine]
```

## 16. 参考: 内部パイプライン

典型的な処理パイプライン:

1. YAML をパースする。
2. DSL 行を AST ノードへパースする。
3. シンボル表（ingredients, tools, outputs, step ids）を解決する。
4. 依存 DAG を構築する。
5. 制約を検証する。
6. 実行計画、タイマー、派生成果物を出力する。

## 17. JSON 相互交換マッピング（参考）

実装は同一セマンティクスを持つ正規化 JSON を公開してよい（MAY）。

推奨最小フィールド:

1. `meta`
2. `ingredients`
3. `tools`
4. `steps`
5. `graph`（`nodes`, `edges`）

## 18. バージョニングと前方互換

1. ドキュメントは `meta.spec_version`（例: `0.1`）を含むことが SHOULD。
2. 未知フィールドは strict mode 有効時を除き無視しなければならない（MUST）。
3. 破壊的な文法変更はメジャーバージョンを上げなければならない（MUST）。

## 19. セキュリティと安全性の考慮

1. 本フォーマットは食品安全を保証しない。
2. 実装は加熱不足リスクパターンに対する警告を出せることが SHOULD。
3. 自動機器制御の前に、道具指示と温度指示を検証することが SHOULD。

## 20. v0.2 に向けた未解決項目

1. DSL の形式 EBNF
2. 標準アクションオントロジーと多言語エイリアス
3. アレルゲンおよび栄養プロファイルスキーマ
4. 決定的スケジューラ方針プロファイル
5. スマート家電向け IoT コマンドバインディングプロファイル
