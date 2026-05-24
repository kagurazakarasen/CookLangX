# CookLangX 入門書

楽しみながら学び、最後には自分でレシピを書けるようになるためのハンズオン入門です。

この本のゴールは 3 つです。

1. CookLangX の考え方を直感でつかむ。
2. 仕様に沿って正しい `.ckx` を書けるようになる。
3. 依存関係やスケーリングを活用して、実用的なレシピを設計できるようになる。

---

## 0. この本の遊び方

設定:

- あなたは「キッチン自動化工房」の新人レシピエンジニア。
- 先輩はおいしいけれど口頭説明が長い。
- あなたの任務は、口頭レシピを CookLangX に変換し、誰でも再現可能にすること。

進め方:

1. 各章の「ミッション」を読む。
2. サンプルをそのまま写す。
3. 「やってみよう」を自力で書く。
4. 最後の総合課題で 1 本のレシピを完成させる。

前提:

- YAML の基本が少し分かると楽です。
- まったく初めてでも、サンプルを写経すれば進められます。

---

## 1. まずは全体像: CookLangX は何者か

### ミッション

「文章レシピ」を「実行可能な手順グラフ」に変える発想を理解する。

CookLangX は YAML と 1 行 DSL (`run`) のハイブリッドです。

- YAML 側: 材料、道具、手順の構造を記述
- DSL 側: 各手順で何をするかをコンパクトに記述

最小構成は次の 3 つです。

1. `meta`
2. `ingredients`
3. `steps`

最小サンプル:

```yaml
meta:
  name: Boiled Egg
  servings: 1

ingredients:
  - id: egg
    name: Egg
    amount: 1
    unit: piece

steps:
  - id: boil_egg
    run: boil $egg time=8min -> boiled_egg
```

ここで覚えるポイント:

1. `id` は機械が使う名前。
2. `run` は「動詞 + 引数 + 任意の出力」。
3. `-> boiled_egg` はステップの成果物名。

---

## 2. 材料と道具を定義する

### ミッション

材料 (`ingredients`) と道具 (`tools`) を正しく定義する。

### 材料の書き方

```yaml
ingredients:
  - id: onion
    name: Onion
    amount: 1
    unit: piece
    scalable: true

  - id: salt
    name: Salt
    amount: 2
    unit: g
    scalable: false
```

実務メモ:

1. `id` は `^[a-z][a-z0-9_]*$` に合わせる。
2. 人数スケーリングしたくないものは `scalable: false`。

### 道具の書き方

```yaml
tools:
  - id: pot
    name: Pot

  - id: frying_pan
    name: Frying Pan
```

### やってみよう

次を定義してみてください。

1. `garlic`（2 clove）
2. `olive_oil`（1 tbsp）
3. `skillet`（フライパン）

---

## 3. `run` DSL の読み書き

### ミッション

CookLangX の心臓部である 1 行 DSL を書けるようになる。

基本形:

```text
<action> <arg>* [-> <output_id>]
```

引数 (`arg`) は 3 種類です。

1. 参照: `$ingredient`, `%output`, `&tool`
2. キー値: `key=value`
3. 文字列: `"..."`

例:

```text
boil $spaghetti water=2L salt=2g -> boiled_pasta
fry $bacon heat=medium -> crisp_bacon
season %soup level="light" note="finish with olive oil" -> seasoned_soup
```

よくあるミス:

1. `heat==medium` のように `=` を 2 つ書く。
2. `%unknown_output` のように存在しない参照を書く。
3. `boil ->` のように出力名を省略する。

---

## 4. 依存関係を操る

### ミッション

手順を「順番のリスト」ではなく「DAG」として設計する。

CookLangX の実行順は次で決まります。

1. 明示依存: `depends_on`
2. 暗黙依存: `run` や `uses` の `%output` 参照

サンプル:

```yaml
steps:
  - id: boil_pasta
    run: boil $spaghetti water=2L -> boiled_pasta

  - id: cook_sauce
    run: simmer $tomato salt=1g -> tomato_sauce

  - id: combine
    run: combine %boiled_pasta %tomato_sauce -> pasta
    depends_on: [boil_pasta, cook_sauce]
```

設計のコツ:

1. 並列化できる工程は分ける。
2. 最後に `combine` で合流させる。
3. サイクルができないか常に確認する。

---

## 5. 時間・温度・スケーリング

### ミッション

レシピを人数変化に強い仕様にする。

### 時間

使える例:

1. `30s`
2. `8min`
3. `1h`

### 温度

使える例:

1. `180C`
2. `356F`
3. `low`, `medium`, `high`

### スケーリング式

人数変換は次式です。

$$
amount' = amount \times \frac{S_t}{S_o}
$$

- $S_o$: 元の人数
- $S_t$: 目標人数

例:

- 元 2 人前で 200g
- 目標 3 人前
- $200 \times \frac{3}{2} = 300$g

---

## 6. バリデーションの考え方

### ミッション

エラーコードを見て、どこを直せばよいか判断できるようになる。

代表例:

1. `CKX001`: 必須セクション欠落
2. `CKX002`: 識別子形式不正
3. `CKX003`: 識別子重複
4. `CKX004`: 未知参照
5. `CKX005`: 依存サイクル
6. `CKX008`: `servings` が正でない

デバッグ手順の型:

1. まず `meta/ingredients/steps` の存在を確認。
2. 次に `id` の重複と形式を確認。
3. 最後に `%output` と `depends_on` を追跡。

---

## 7. 写経課題: 10 分で 1 レシピ

次をそのまま写して、構造を体で覚えます。

```yaml
meta:
  name: Garlic Pasta
  servings: 2
  lang: ja
  spec_version: 0.1

ingredients:
  - id: spaghetti
    name: Spaghetti
    amount: 200
    unit: g
  - id: garlic
    name: Garlic
    amount: 2
    unit: clove
  - id: olive_oil
    name: Olive Oil
    amount: 2
    unit: tbsp
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

  - id: cook_garlic_oil
    run: saute $garlic $olive_oil heat=low -> garlic_oil
    tools: [frying_pan]
    time: 3min

  - id: finish
    run: combine %boiled_pasta %garlic_oil -> garlic_pasta
    depends_on: [boil_pasta, cook_garlic_oil]

  - id: plate
    run: serve %garlic_pasta
    depends_on: [finish]
```

---

## 8. 自力課題: あなたの一皿を設計する

### ミッション

自分の定番料理を 1 本、CookLangX で書く。

制約:

1. `ingredients` は 5 個以上。
2. `tools` を 2 個以上。
3. 並列可能なステップを 1 組以上作る。
4. `%output` 参照を 2 回以上使う。
5. `note` を 1 つ以上入れる。

提出前チェックリスト:

1. `meta.servings` は正の数か。
2. すべての `id` は小文字 + 数字 + `_` か。
3. 未定義の `$`, `%`, `&` を参照していないか。
4. `depends_on` に循環がないか。

---

## 9. つまずきポイント FAQ

Q1. 手順は配列順に実行されますか。  
A1. 可読順は配列ですが、実行順は依存関係で決まります。

Q2. `steps[].id` と `run` の `-> output` は同じである必要がありますか。  
A2. 必須ではありません。別名でも構いませんが、追跡しやすい命名が推奨です。

Q3. `note` は処理に影響しますか。  
A3. 依存セマンティクスでは無視されます。UI 表示用メモです。

Q4. YAML コメントと DSL コメントの違いは。  
A4. コメントは YAML の `#` のみです。DSL 独自コメントはありません。

---

## 10. 最終ミッション

次を満たす `.ckx` を 1 本作ってください。

1. 2 人前を 4 人前にスケールしても破綻しない。
2. 1 つの道具競合を避ける設計になっている。
3. 加熱工程に時間と温度を明記している。
4. 最後に `serve` で提供している。

ここまでできれば「楽しみながら学んで、自分で書ける」到達です。

---

## 付録 A: テンプレート

新規作成時にコピーして使える最小テンプレートです。

```yaml
meta:
  name: Your Dish Name
  servings: 2
  lang: ja
  spec_version: 0.1

ingredients:
  - id: sample_ingredient
    name: Sample Ingredient
    amount: 1
    unit: piece

tools:
  - id: sample_tool
    name: Sample Tool

steps:
  - id: step_one
    run: mix $sample_ingredient -> mixed_item

  - id: step_two
    run: serve %mixed_item
    depends_on: [step_one]
```

## 付録 B: おすすめ学習ルート

1. まず 3 本写経する。
2. 次に 1 本を自分の料理に置き換える。
3. その後、同じレシピを 2 人前と 5 人前で比較する。
4. 最後に `depends_on` を見直し、並列化できる箇所を探す。

このルートを 1 周すれば、CookLangX の基本は実戦レベルです。
