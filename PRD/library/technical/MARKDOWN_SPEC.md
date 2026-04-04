# Isidorica Markdown Specification (IFM)

本ドキュメントは、Isidoricaで使用されるMarkdownの拡張仕様（Isidorica Flavored Markdown: IFM）を定義する。
本仕様は、執筆者向けのガイドラインであると同時に、パーサーおよびバリデーター実装の要件定義書でもある。

**ステータス**: ✅ 完了

---

## 1. 基本方針 (Philosophy)

1.  **Semantic over Presentation**: 見た目ではなく「意味」を記述する。
2.  **Explicit Structure**: 記事間の関係性（依存関係・対立関係）や文書構造を明示的に定義する。
3.  **Future-Proof**: 将来的な機械可読性（ベクトル検索、AI解析）を見据え、厳密なスキーマを採用する。

---

## 2. サポートするMarkdown記法

IFMは、[Markdown Guide](https://www.markdownguide.org/) に準拠しつつ、Isidorica固有の拡張を追加する。

### 2.1 Basic Syntax（全て採用）

| 記法 | 例 | 備考 |
| :--- | :--- | :--- |
| 見出し | `# H1`, `## H2`, `### H3` | H1は記事タイトルのみ使用 |
| 段落 | 空行で区切る | - |
| 改行 | 行末に2スペース or `<br>` | - |
| 太字 | `**bold**` | 用語の初出、キーワード |
| イタリック | `*italic*` | 書名、外来語、強調 |
| 引用 | `> quote` | - |
| リスト（順序なし） | `- item` | - |
| リスト（順序あり） | `1. item` | - |
| コード（インライン） | `` `code` `` | - |
| 水平線 | `---` | - |
| リンク | `[text](url)` | - |
| 画像 | `![alt](src)` | 詳細は10節参照 |

### 2.2 Extended Syntax

| 記法 | 採用 | 例 | 備考 |
| :--- | :--- | :--- | :--- |
| **Tables** | ✅ | `\| A \| B \|` | 拡張記法あり（2.4節参照） |
| **Fenced Code Blocks** | ✅ | ` ```python ` | 拡張機能あり（2.8節参照） |
| **Footnotes** | ✅ | `[^1]` | 出典ではない補足説明に使用 |
| **Heading IDs** | ✅ | `## Heading {#custom-id}` | 目次、アンカーリンク |
| **Definition Lists** | ✅ | `Term\n: Definition` | Phase 2（定義）での用語整理 |
| **Strikethrough** | ✅ | `~~strikethrough~~` | 訂正表示 |
| **Subscript** | ✅ | `H~2~O` | 化学式等 |
| **Superscript** | ✅ | `x^2^` | 指数、日付（1^st^）等 |
| **Emoji Shortcodes** | ✅ | `:tent:` → ⛺ | 執筆者の利便性向上 |
| **Automatic URL Linking** | ✅ | `https://...` → リンク化 | - |
| **Abbreviations** | ✅ | `*[API]: ...` | abbreviations.yml連携（2.6節参照） |
| **Ruby Annotation** | ✅ | `{漢字\|よみ}` | 自動ルビ + 動的制御（2.7節参照） |
| **Task Lists** | ❌ | `- [ ] task` | Quiz Directiveで代替 |
| **Highlight** | ❌ | `==highlight==` | 認知負荷増大のため不採用 |

> **設計根拠（Highlight不採用）**: USER_BEHAVIOR_MODEL.md 3.4節「注意の制約」に基づき、強調の多用は認知負荷を増大させる。`**bold**` と `*italic*` で十分であり、追加の強調レイヤーは「構造で導く」設計思想に反する。

### 2.3 Footnotes vs 出典記法

| 用途 | 記法 | 例 |
| :--- | :--- | :--- |
| **出典（書籍・論文）** | `[@id, p.XX]` | `[@neumann-1944, p. 45]` |
| **補足説明（出典以外）** | `[^1]` | `[^1]: 補足説明文` |

### 2.4 拡張テーブル記法

GFMテーブルを拡張し、複雑なデータ表現を可能にする。

#### 2.4.1 基本構文（GFM互換）

```markdown
| 列A | 列B | 列C |
| :--- | :---: | ---: |
| 左寄せ | 中央 | 右寄せ |
```

#### 2.4.2 セル内改行

1つのセル内で複数行のテキストを表示する。行末にバックスラッシュ `\` を置き、次の行でインデントして継続する。

```markdown
| 概念 | 説明 |
| :--- | :--- |
| ナッシュ均衡 | どのプレイヤーも一方的に \
|              | 戦略を変更するインセンティブがない状態。 |
| パレート最適 | 誰かを改善するには \
|              | 他の誰かを悪化させる必要がある状態。 |
```

**HTML出力**:
```html
<td>どのプレイヤーも一方的に<br>戦略を変更するインセンティブがない状態。</td>
```

#### 2.4.3 セル結合

| 記号 | 機能 | 意味 |
| :--- | :--- | :--- |
| `->` | colspan（右方向結合） | このセルを左のセルと結合 |
| `^` | rowspan（上方向結合） | このセルを上のセルと結合 |

**例: colspan（右方向結合）**

```markdown
| 戦略 | 相手: 協力 | 相手: 裏切り |
| :--- | :--- | :--- |
| ヘッダー | 結合されたセル | -> |
| 自分: 協力 | (-1, -1) | (-3, 0) |
```

**例: rowspan（上方向結合）**

```markdown
| カテゴリ | サブカテゴリ | 値 |
| :--- | :--- | :--- |
| グループA | 項目1 | 100 |
| ^ | 項目2 | 200 |
| グループB | 項目3 | 300 |
```

**HTML出力**:
```html
<tr>
  <td rowspan="2">グループA</td>
  <td>項目1</td>
  <td>100</td>
</tr>
<tr>
  <td>項目2</td>
  <td>200</td>
</tr>
```

#### 2.4.4 テーブルキャプション

キャプションや出典が必要な表には `:::table` directiveを使用する。
**表番号は自動採番されるため、執筆者はキャプションのみ記述する。**

```markdown
::: table {id="table:payoff-matrix" caption="囚人のジレンマの利得行列"}
| 戦略 | 協力 | 裏切り |
| :--- | :--- | :--- |
| 協力 | (-1, -1) | (-3, 0) |
| 裏切り | (0, -3) | (-2, -2) |

出典: [@tanaka-game-theory-2020, p. 45]
:::
```

**HTML出力**（番号は自動付与）:
```html
<figure class="table-figure" id="table:payoff-matrix">
  <figcaption><span class="table-number">表 1:</span> 囚人のジレンマの利得行列</figcaption>
  <table>...</table>
</figure>
```

| 属性 | 必須 | 説明 |
| :--- | :--- | :--- |
| `caption` | No | テーブルキャプション（番号なしで記述、番号は自動付与） |
| `id` | No | テーブルID（相互参照用、`table:` プレフィックスで指定） |

**相互参照の例**:
```markdown
::: table {id="table:payoff-matrix" caption="囚人のジレンマの利得行列"}
...
:::

:ref:[table:payoff-matrix] を参照してください。  → 「表 1」と表示
```

> **参照**: 相互参照記法の詳細は5.2節を参照。

#### 2.4.5 エスケープ

拡張記法の特殊文字をリテラルとして表示したい場合は、バックスラッシュでエスケープする。

| 文字 | エスケープ | 表示 |
| :--- | :--- | :--- |
| `\` (行末) | `\\` | `\` |
| `->` | `\->` | `->` |
| `^` | `\^` | `^` |

```markdown
| 記号 | 意味 |
| :--- | :--- |
| \-> | 右矢印（リテラル） |
| \^ | キャレット（リテラル） |
```

---

### 2.5 定義リスト記法 (Definition Lists)

用語とその定義を記述するための記法。Phase 2（概念定義）での用語解説に適している。

#### 2.5.1 基本構文

```markdown
用語
: 用語の定義文。

別の用語
: 定義文にはMarkdownを含めることも可能。
: 複数の定義がある場合は行を分けて記述。
```

**HTML出力**:
```html
<dl>
  <dt>用語</dt>
  <dd>用語の定義文。</dd>
  <dt>別の用語</dt>
  <dd>定義文にはMarkdownを含めることも可能。</dd>
  <dd>複数の定義がある場合は行を分けて記述。</dd>
</dl>
```

#### 2.5.2 複数行の定義

定義が複数行にわたる場合は、インデント（4スペースまたは1タブ）で継続する。

```markdown
ナッシュ均衡
: ゲーム理論における解概念の一つ。
  どのプレイヤーも一方的に戦略を変更するインセンティブがない状態を指す。
  
  1950年にジョン・ナッシュによって提唱された。
```

#### 2.5.3 ネストされた定義リスト

定義内にさらに定義リストをネストすることも可能。

```markdown
ゲーム理論
: 複数の意思決定者間の相互作用を分析する数学的手法。

  非協力ゲーム
  : プレイヤー間の拘束力のある合意がないゲーム。

  協力ゲーム
  : プレイヤー間で拘束力のある合意が可能なゲーム。
```

#### 2.5.4 アクセシビリティ

`<dl>`, `<dt>`, `<dd>` はスクリーンリーダーで適切に読み上げられる。
長い定義リストには見出し（`##`）を併用し、構造を明確にすること。

---

### 2.6 略語記法 (Abbreviations)

略語を `<abbr>` タグに変換し、初学者に略語の意味を伝える。

#### 2.6.1 基本動作（自動展開）

記事の `topics` に基づき、`abbreviations.yml` から自動的に略語を展開する。
執筆者は通常何も書く必要がない。

```markdown
---
topics: [programming, web]
---

APIを使ってDOMを操作します。
```

**HTML出力**（自動変換）:
```html
<abbr title="Application Programming Interface">API</abbr>を使って
<abbr title="Document Object Model">DOM</abbr>を操作します。
```

#### 2.6.2 ローカル定義（上書き）

特殊な意味で略語を使用する場合、記事内でローカル定義を行う。
ローカル定義は `abbreviations.yml` より優先される。

```markdown
---
topics: [hardware]
---

*[API]: Advanced Physical Interface

この記事では特殊なAPIについて説明します。
```

#### 2.6.3 abbreviations.yml スキーマ

`abbreviations.yml` は略語のグローバル定義を管理する。

```yaml
# 単一定義
HTML:
  full: HyperText Markup Language
  scope: [web, programming]

# 複数定義（分野によって異なる）
API:
  definitions:
    - full: Application Programming Interface
      scope: [programming, software, web]
    - full: Active Pharmaceutical Ingredient
      scope: [medicine, pharmacology]
```

#### 2.6.4 処理ルール

| 状況 | 処理 |
| :--- | :--- |
| abbreviations.yml に存在し、スコープ一致 | 自動展開 |
| abbreviations.yml に存在しない | 変換しない + CI で Info 通知 |
| スコープ一致なし | 変換しない + CI で Warning |
| 複数スコープ一致（曖昧性） | ビルドエラー（ローカル定義で解決） |
| ローカル定義あり | ローカル定義を優先 |

> **参照**: パーサーでの処理詳細は [IFM_PARSER_SPEC.md](../../engine/technical/IFM_PARSER_SPEC.md) 4.6節を参照。

---

### 2.7 インライン・グロッサリー (Inline Glossary)

用語の定義をツールチップとして表示し、ページ遷移なしで文脈理解を支援する（認知的負荷の軽減）。
Isidoricaの理念に基づき、「出典に基づく厳密性」と「直感的な理解」を両立させる**二層構造**を採用する。

#### 2.7.1 記法

カスタムリンク記法 `[[Term]]` を使用する。

| 記法 | 挙動 | 用途 |
| :--- | :--- | :--- |
| `[[Term]]` | `terms.yml` から定義を取得 | 一般的な用語参照 |
| `[[Term|Text]]` | 本文中の表示文字を変更 | 文法活用（例: `[[Apple|りんご]]`） |
| `[[Term||Def]]` | **定義**（Definition）を上書き | 文脈依存の定義 |
| `[[Term|||Int]]` | **直感**（Intuition）を上書き | 文脈依存の例え話 |
| `[[Term||Def|Int]]` | 両方を上書き | 完全なローカル定義 |

#### 2.7.2 terms.yml スキーマ

[DICTIONARY_CONTENT_DESIGN.md](../DICTIONARY_CONTENT_DESIGN.md) で定義された**生成語彙論 (Generative Lexicon)** に基づく意味論的辞書データ。

**配置場所**: Libraryリポジトリ `data/terms.yml`

**設計思想**: 従来の意味列挙型辞書（SEL）ではなく、Pustejovsky (1995) の生成語彙論を採用。意味は固定的ではなく、クオリア構造と型強制によって文脈から動的に生成される。

```yaml
# data/terms.yml
nash-equilibrium:
  # 表示層（一般読者向け）
  display:
    intuition:
      ja: コンビニが同じ場所に集まっているのを見たことはありますか？ライバル同士なのに近くに出店するのは不思議ですが、実は「誰も動きたくなくなる」状態なのです。
    definition:
      ja: 各プレイヤーが互いに最適な戦略をとっており、誰も単独で戦略を変更する動機がない状態。
      en: A state in which each player is playing their best response to the strategies of the others.

  # 意味構造層（クオリア構造）
  qualia:
    constitutive:
      components: [players, strategies, payoff-function]
      description:
        ja: プレイヤー集合、戦略集合、利得関数から構成される
    formal:
      type: StrategyProfile
      supertypes: [equilibrium-concept, solution-concept]  # TTR: 多重継承
      constraints:  # TCL/TTR: 型制約
        - predicate: "∀i ∈ Players: σ_i = BR(σ_{-i})"
          natural_language:
            ja: 各プレイヤーが最適応答している
      parameters:  # TTR: 依存型パラメータ
        - name: n
          type: PositiveInteger
          description:
            ja: プレイヤー数
      description:
        ja: 戦略の組。均衡概念の一種
    telic:
      purpose: achieve-stable-outcome
      description:
        ja: 安定的な戦略配置を達成するため
    agentive:
      origin: game-theoretic-analysis
      description:
        ja: ゲーム理論的分析によって特定される
    event_structure:
      type: TRANSITION
      initial_state:
        ja: 未均衡状態
      final_state:
        ja: 均衡状態

  # 型強制（意味シフト）
  coercion:
    - context: "NE を破る"
      source: State
      target: Norm
      trigger:  # TCL: 型強制の発生条件
        predicate: "λx. violate(x)"
        natural_language:
          ja: 規範的解釈を要求する動詞の目的語
      interpretation:
        ja: 規範として解釈される
    - context: "NE に達する"
      source: State
      target: Process
      trigger:
        predicate: "λx. achieve(x)"
        natural_language:
          ja: 到達を表す動詞の目的語
      interpretation:
        ja: 到達プロセスとして解釈される

  # 補足層
  pragmatics:
    contexts: [経済学, ゲーム理論, 戦略的意思決定]
    register: 学術的・専門的
    collocation: [〜に達する, 〜から逸脱する]
    presupposition: プレイヤーが合理的であること
    speech_act: 安定性・最適性を暗示する

  distributional:
    neighbors: [pareto-efficiency, dominant-strategy, mixed-strategy]
    prototype: 非協力ゲームにおける戦略的均衡状態
    periphery: [日常語としての膠着状態]

  history:
    origin: "Nash, J. (1950). Equilibrium points in n-person games."
    evolution: 当初は非協力ゲームの解概念。後に進化ゲーム理論へ拡張。

  ecology:
    embodiment: 「動かない」という身体経験に基づく安定性のメタファー
    scenario: 複数のプレイヤーが互いの選択を予測しながら戦略を決定する場面
    frame_elements:
      PLAYERS: 意思決定を行う主体
      STRATEGIES: 各プレイヤーの選択肢
      EQUILIBRIUM: 誰も逸脱しない戦略の組

  relations:
    broader: equilibrium-concept
    narrower: [subgame-perfect-equilibrium, bayesian-nash-equilibrium]
    related: [pareto-efficiency]
```

**フィールド定義**:

| 層 | フィールド | 必須 | 説明 |
| :--- | :--- | :--- | :--- |
| **Display** | `display.intuition` | ❌ | 日常的説明（限定コード）。`locale -> text` |
| **Display** | `display.definition` | ✅ | 論理的定義（精密コード）。`locale -> text` |
| **Qualia** | `qualia.constitutive` | ❌ | 構成要素 |
| **Qualia** | `qualia.formal` | ❌ | 型・カテゴリ |
| **Qualia** | `qualia.formal.supertypes[]` | ❌ | 多重継承（TTR） |
| **Qualia** | `qualia.formal.constraints[]` | ❌ | 型制約（TCL/TTR） |
| **Qualia** | `qualia.formal.parameters[]` | ❌ | 依存型パラメータ（TTR） |
| **Qualia** | `qualia.telic` | ❌ | 目的 |
| **Qualia** | `qualia.agentive` | ❌ | 生成方法 |
| **Qualia** | `qualia.event_structure` | ❌ | イベント構造（STATE/PROCESS/TRANSITION） |
| **Coercion** | `coercion[]` | ❌ | 型強制の文脈リスト |
| **Coercion** | `coercion[].trigger` | ❌ | 型強制の発生条件（TCL） |
| **Supplementary** | `pragmatics` | ❌ | 語用論的記述 |
| **Supplementary** | `distributional` | ❌ | 分散意味表現 |
| **Supplementary** | `history` | ❌ | 歴史的変遷 |
| **Supplementary** | `ecology` | ❌ | エコロジカル観点 |
| **Supplementary** | `relations` | ❌ | 知識グラフ |

#### 2.7.3 表示要件（Toggletip）

アクセシビリティ（WCAG 2.2 AA）と品質担保のため、以下の仕様でレンダリングする。文字数の制限ではなくあくまで表示する際のロジックの話である。

1.  **二層構造のUI**（データソース: `terms.yml`）:
    *   **上段**: `definition` フィールド。出典に基づく厳密な定義（必須）。
    *   **下段**: `intuition` フィールド。限定コードによる直感的な補助（任意）。視覚的に厳密な定義と区別する。
2.  **肥大化防止**:
    *   **表示量**: 最大 **2-3行**（約100-150文字を目安、ただし強制ではない）
    *   **表示ロジック**: `definition` の最初の1文 + `intuition` の最初の1文を表示
    *   **切り詰め**: 長い場合は文末で切り詰め、「続きを読む」リンクで辞書ページへ誘導
    *   **`terms.yml` への影響なし**: フィールド自体に文字数制限は設けない。辞書ページでは全文表示
3.  **インタラクション**:
    *   クリック/タップで開閉（`button` + `aria-expanded`）。
    *   Escキーで閉じる。
    *   キーボードフォーカス可能。


### 2.8 ルビ記法 (Ruby Annotation)

漢字にルビ（読み仮名）を付与し、読者の日本語レベルに応じて表示を動的に制御する。

#### 2.8.1 基本動作（自動ルビ）

ビルド時に形態素解析（Sudachi）を使用し、漢字に自動でルビを付与する。
執筆者は通常何も書く必要がない。

```markdown
歴史について学びます。所謂ナッシュ均衡について説明します。
```

**自動変換後**:
```html
<ruby data-level="basic">歴史<rp>（</rp><rt>れきし</rt><rp>）</rp></ruby>について学びます。
<ruby data-level="joyo">所謂<rp>（</rp><rt>いわゆる</rt><rp>）</rp></ruby>
<ruby data-level="joyo">ナッシュ均衡<rp>（</rp><rt>なっしゅきんこう</rt><rp>）</rp></ruby>について説明します。
```

#### 2.8.2 明示的指定（上書き）

自動ルビが不正確な場合、執筆者が明示的に指定できる。明示的指定は自動ルビより優先される。

```markdown
{東海林|しょうじ}さんは{今日|きょう}も元気です。
```

#### 2.8.3 漢字レベル分類

| レベル | 対象 | 漢字数 |
| :--- | :--- | :--- |
| `basic` | 教育漢字（小学校で習う） | 1,026字 |
| `joyo` | 常用漢字（中学以降で習う） | 追加1,110字 |
| `non-joyo` | 常用漢字外 | その他 |

#### 2.8.4 ユーザー設定（表示制御）

ルビの表示はユーザー設定で動的に切り替えられる。

| 設定 | 対象ユーザー | 表示されるルビ |
| :--- | :--- | :--- |
| **全表示** | 小学生、漢字初学者など | すべて |
| **教育漢字以外** | 中高生 | joyo + non-joyo |
| **常用漢字外のみ** | 一般成人（デフォルト） | non-joyo のみ |
| **非表示** | ルビ不要な人 | なし |

> **参照**: パーサーでの処理詳細は [IFM_PARSER_SPEC.md](../../engine/technical/IFM_PARSER_SPEC.md) 4.6節を参照。

### 2.9 コードブロック拡張 (Fenced Code Block Extensions)

通常のフェンスコードブロックを拡張し、教育的な機能を追加する。

#### 2.9.1 基本構文

言語識別子の後に属性を指定する。

````markdown
```python showLineNumbers title="sort.py" {3,5-7}
def quick_sort(arr):
    if len(arr) <= 1:
        return arr  # ハイライト
    pivot = arr[0]
    left = [x for x in arr[1:] if x < pivot]   # ハイライト
    right = [x for x in arr[1:] if x >= pivot]  # ハイライト
    return quick_sort(left) + [pivot] + quick_sort(right)  # ハイライト
```
````

#### 2.9.2 属性一覧

| 属性 | 説明 | 例 |
| :--- | :--- | :--- |
| `showLineNumbers` | 行番号を表示 | `showLineNumbers` |
| `{行番号}` | ハイライト行 | `{3}`, `{1-5}`, `{1,3,5}` |
| `+{行番号}` | 追加行（緑、Diff表示） | `+{3}`, `+{1-3}` |
| `-{行番号}` | 削除行（赤、Diff表示） | `-{2}`, `-{1-2}` |
| `title="..."` | ファイル名/タイトル | `title="sort.py"` |
| `noCollapse` | 自動折りたたみを無効化 | `noCollapse` |

#### 2.9.3 行番号指定の構文

| 構文 | 説明 | 例 |
| :--- | :--- | :--- |
| `{3}` | 1行のみ | 3行目 |
| `{1,3,5}` | 複数行 | 1, 3, 5行目 |
| `{1-5}` | 範囲 | 1〜5行目 |
| `{1,3-5,8}` | 組み合わせ | 1, 3〜5, 8行目 |

#### 2.9.4 Diff表示

コードの変更を示す。行番号で指定するため、コード本体はそのままコピー可能。

````markdown
```python title="greet.py" +{3} -{2}
def greet(name):
    print("Hello, " + name)  # 削除（赤）
    print(f"Hello, {name}")   # 追加（緑）
```
````

#### 2.9.5 自動折りたたみ

**25行以上**のコードブロックは自動的に折りたたみ表示される。

- 最初の15行を表示
- 「残り XX 行を表示」で展開可能
- `noCollapse` 属性で無効化可能

#### 2.9.6 領域マーカー（Region Marker）

import文や設定コードなど、「コピーには必要だが説明の本質ではない」部分を折りたたむ。

````markdown
```python
# @region imports
import numpy as np
import pandas as pd
from typing import List
# @endregion

def quick_sort(arr: List[int]) -> List[int]:
    if len(arr) <= 1:
        return arr
    pivot = arr[0]
    left = [x for x in arr[1:] if x < pivot]
    right = [x for x in arr[1:] if x >= pivot]
    return quick_sort(left) + [pivot] + quick_sort(right)
```
````

**表示**:
```
┌─────────────────────────────────────┐
│ ▶ imports (3行)                     │  ← クリックで展開
├─────────────────────────────────────┤
│ def quick_sort(arr: List[int]):     │
│     ...                             │
└─────────────────────────────────────┘
```

**ルール**:

| ルール | 説明 |
| :--- | :--- |
| **必須: 非領域行が1行以上** | コードブロック内に `@region` 外の行が必須 |
| **ネスト禁止** | 領域のネストは不可 |
| **閉じ忘れ** | CIが `@endregion` を自動追記 + 報告 |
| **全行が領域内** | 領域マーカーを無効化して全て表示 + CI警告 |

#### 2.9.7 パッケージマネージャータブ

`npm` を言語識別子として使用すると、npm/yarn/pnpm/bun のタブUIが自動生成される。

````markdown
```npm
npm install react
```
````

**自動変換**:

| 入力 | npm | yarn | pnpm | bun |
| :--- | :--- | :--- | :--- | :--- |
| `npm install react` | そのまま | `yarn add react` | `pnpm add react` | `bun add react` |
| `npm run dev` | そのまま | `yarn dev` | `pnpm dev` | `bun dev` |

#### 2.9.8 コピーボタン

すべてのコードブロックにコピーボタンが自動付与される。

- 折りたたまれた部分も含めて全体がコピーされる
- 領域マーカー（`# @region` 等）は**コピーに含まれる**（実際のコードとして必要なため）

> **参照**: パーサーでの処理詳細は [IFM_PARSER_SPEC.md](../../engine/technical/IFM_PARSER_SPEC.md) 4.7節を参照。

### 2.10 サポートしない記法 (Unsupported Syntax)

Isidoricaの理念（アクセシビリティ、認知負荷の軽減、セキュリティ）に基づき、以下の記法・機能はサポートしない。

#### 2.10.1 メディア

| 記法 | 理由 | 代替手段 |
| :--- | :--- | :--- |
| **動画 (`<video>`)** | アクセシビリティ、パフォーマンス | ダイアグラム、Laboratory |
| **音声 (`<audio>`)** | アクセシビリティ | テキストによる説明 |
| **自動再生メディア** | `prefers-reduced-motion` 違反 | ユーザー操作必須 |

> **例外**: 歴史的映像等の一次資料は**別ページ**で提供可能。詳細は [GOVERNANCE.md](../GOVERNANCE.md) を参照。

#### 2.10.2 埋め込み

| 記法 | 理由 | 代替手段 |
| :--- | :--- | :--- |
| **iframe** | セキュリティ（CSP）、外部依存 | Laboratory |
| **外部スクリプト** | セキュリティ、プライバシー | Laboratory（サンドボックス） |
| **コメント埋め込み（Disqus等）** | プライバシー、外部依存 | Agora（内製） |
| **広告タグ** | 公共財としての設計思想 | 不可 |

#### 2.10.3 スタイリング

| 記法 | 理由 | 代替手段 |
| :--- | :--- | :--- |
| **生のHTML** | 構造の一貫性破壊、セキュリティ | Directive記法 |
| **インラインCSS** | アクセシビリティ、デザインの一貫性 | デザイントークン |
| **カスタムスタイル** | コントラスト比の保証不可 | 標準の強調記法 |

#### 2.10.4 UI

| 記法 | 理由 | 代替手段 |
| :--- | :--- | :--- |
| **汎用アコーディオン** | 情報の隠蔽による認知負荷 | 見出し + 構造化 |
| **汎用タブ（目的不明確）** | 認知負荷 | `language-tabs`（目的明確） |

> **注意**: コードブロックの領域マーカーや `:::language-tabs` は目的が明確なため許可。汎用的な「情報を隠す」UIは、認知的倹約家が開かない可能性があるため不可。

#### 2.10.5 既にExtended Syntaxで不採用としたもの

| 記法 | 理由 | 代替手段 |
| :--- | :--- | :--- |
| **Task Lists (`- [ ]`)** | 既に不採用 | `:::quiz` |
| **Highlight (`==text==`)** | 認知負荷増大 | `**bold**`, `*italic*` |

---

## 3. Frontmatter Schema

各記事の先頭にはYAML形式のFrontmatterを必須とする。

### 3.1 必須フィールド

| Field | Type | Description |
| :--- | :--- | :--- |
| **title** | `string` | 記事のタイトル。 |
| **description** | `string` | 記事の要約（Abstract）。SEO、OGP、サイト内検索、AI文脈理解に使用。**100-140文字推奨**。 |
| **topics** | `string[]` | 記事を分類するタグ（Topics Masterで管理された語彙）。 |

### 3.2 任意フィールド

デフォルト値や自動推論が効くため、通常は記述不要。

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **cover** | `string` | 自動生成 | OGP画像パス。未指定時はタイトルとトピックから動的生成。 |
| **discipline** | `string` | 親ディレクトリから推論 | 引用スタイル決定用の分野。例外的なスタイル適用が必要な場合のみ指定。 |
| **variables** | `object` | `{}` | 本文内で `{{key}}` として展開する変数。詳細は3.7節参照。 |

### 3.3 自動生成フィールド（Frontmatterに記述不要）

以下のフィールドはビルド時にGitから自動生成される。

| Field | Description |
| :--- | :--- |
| **slug** | URLパス。**ファイル名から自動生成**（例: `game-theory.md` → `/game-theory`）。 |
| **created_at** | 記事の作成日（Gitログから取得） |
| **updated_at** | 記事の最終更新日（Gitログから取得） |
| **authors** | 記事の著者リスト（Gitコミット履歴 + `authors.yml` から生成） |

### 3.4 記述例

**ミニマル（推奨）**:
```yaml
---
title: ゲーム理論
description: 戦略的な意思決定を数理的に分析する理論。ナッシュ均衡や囚人のジレンマを通じて、競争と協力のメカニズムを解明します。
topics: ['math', 'economics']
---
```

**フル指定（例外時）**:
```yaml
---
title: ゲーム理論
description: 戦略的な意思決定を数理的に分析する理論。ナッシュ均衡や囚人のジレンマを通じて、競争と協力のメカニズムを解明します。
topics: ['math', 'economics']
discipline: economics
cover: ./images/game-theory-cover.png
variables:
  version: "2.0"
---
```

### 3.5 引用形式の決定ロジック

記事の引用形式は以下の優先順位で決定される。

#### 優先順位

| 優先度 | 条件 | 例 |
| :--- | :--- | :--- |
| **1** | Frontmatterに `discipline` が明示指定 | `discipline: psychology` → APA |
| **2** | 親ディレクトリが分野に該当 | `economics/game-theory.md` → Chicago (Author-Date) |
| **3** | いずれも該当しない | **Harvard**（デフォルト） |

#### 分野と引用形式のマッピング

21分野のマッピングが定義されている。詳細は [GOVERNANCE.md](../GOVERNANCE.md) セクション1.1を参照。

| 引用形式 | 対象分野（例） |
| :--- | :--- |
| **APA 7th Edition** | psychology, education, sociology, linguistics |
| **Vancouver** | medicine, biology |
| **OSCOLA** | law |
| **Chicago (Notes-Bibliography)** | philosophy, history, art, religion |
| **Chicago (Author-Date)** | economics, politics |
| **Nature Style** | natural-science, physics, chemistry |
| **AMS** | mathematics |
| **MLA 9th Edition** | literature |
| **IEEE** | engineering, computer-science |
| **Harvard**（デフォルト） | その他 / 該当なし |

#### 具体例

| ファイルパス | `discipline` | 適用される引用形式 |
| :--- | :--- | :--- |
| `economics/game-theory.md` | 未指定 | Chicago (Author-Date)（親ディレクトリ推論） |
| `economics/game-theory.md` | `psychology` | APA 7th Edition（明示指定が優先） |
| `computer-science/react-hooks.md` | 未指定 | IEEE（親ディレクトリ推論） |
| `misc/general-article.md` | 未指定 | Harvard（デフォルト） |

> **参照**: 引用形式マッピングの詳細とCSLスタイル名は [GOVERNANCE.md](../GOVERNANCE.md) セクション1.1を参照。

### 3.6 ネットワーク定義について

> **Note**: 記事間の関係性（依存関係、対立関係など）や学習パスは、Markdownファイル内ではなく、リポジトリルートの外部定義ファイル（`concept-relations.yml`, `learning-paths.yml`）で一元管理する。詳細は [NETWORK_ARCHITECTURE.md](./NETWORK_ARCHITECTURE.md) を参照。

### 3.7 変数 (Variables)

Frontmatterで定義した値を本文内で展開する機能。バージョン番号や共通のリポジトリURLなどの管理に適している。

```markdown
---
slug: react-tutorial
title: React チュートリアル
variables:
  version: "18.2.0"
  repo: "facebook/react"
---

現在のバージョンは {{version}} です。
リポジトリは [GitHub](https://github.com/{{repo}}) にあります。
```

#### 3.7.1 構文

| 構文 | 説明 |
| :--- | :--- |
| `{{変数名}}` | 変数の値を展開（文字列のみ） |

#### 3.7.2 定義済み変数

以下の変数はFrontmatterで明示的に定義しなくても使用可能。

| 変数 | 説明 |
| :--- | :--- |
| `{{slug}}` | 記事のslug |
| `{{title}}` | 記事のタイトル |
| `{{updated_at}}` | 最終更新日 |

#### 3.7.3 ルール

| ルール | 説明 |
| :--- | :--- |
| **未定義変数** | ビルドエラー |
| **型** | 文字列のみ展開可能 |
| **スコープ** | 記事単位（グローバル変数は現在サポート外） |

---

## 4. Block Directives (`:::`)

独自コンポーネントやセマンティックなブロックには `:::` (Generic Directives) を使用する。

### 4.1 コールアウト (Callouts)

情報の役割に応じて、以下の3カテゴリのコールアウトを使い分ける。これらは視覚的なアンカーとなり、スキミングや復習を助ける。

#### 4.1.1 汎用 (Admonitions)

メタ情報や読書体験のサポート。

*   `:::note`: 補足情報
*   `:::tip`: ヒント、コツ
*   `:::warning`: 注意点、落とし穴
*   `:::danger`: 重大な警告

```markdown
:::note {title="補足"}
これは一般的な補足情報です。
:::
```

#### 4.1.2 学術的構造 (Structural / Tier 1)

知識の骨格となる厳密な情報。「定義」や「定理」を明確に区切ることで、コンテキストの境界を明示する。

*   `:::definition {title="用語"}`: 用語の厳密な定義
*   `:::theorem {title="定理名"}`: 定理、公理、法則
*   `:::proof`: 証明（**デフォルトで折りたたみ**、`{open}` 属性で展開状態で表示）

```markdown
:::definition {title="ナッシュ均衡"}
各プレイヤーが互いに最適な戦略をとっている状態。
:::

:::theorem {title="ナッシュの定理"}
任意の有限ゲームには、混合戦略の範囲で少なくとも一つのナッシュ均衡が存在する。
:::
```

#### 4.1.3 認知的足場 (Scaffolding / Tier 2)

理解を助け、認知負荷を下げるための補助情報。Tier 1の厳密さを補完する。

*   `:::intuition`: 直感的な説明、アナロジー、「ざっくり言うと」
*   `:::column`: 歴史的背景、余談、発展的な話題

```markdown
:::intuition
「誰も自分だけでは動きようがない膠着状態」と考えると分かりやすいです。
:::
```

### 4.2 Laboratory (`leaf`)
インタラクティブな実験・シミュレーションコンポーネント。中身は空（または代替テキスト）とし、属性で制御する。

```markdown
::: laboratory {id="prisoner-dilemma-sim" height="400px"}
:::
```

### 4.3 Layouts (`container`)
複雑なレイアウトを表現するためのコンテナ。

```markdown
::: grid {cols=2}
左側のコンテンツ
:::
右側のコンテンツ
:::
```

### 4.4 Reader Adaptation Directives (`parsed`)

読者の習熟度（Explorer / Learner / Scholar）に応じてコンテンツを切り替えるためのDirective。
詳細は [READER_ADAPTATION_DESIGN.md](../../engine/READER_ADAPTATION_DESIGN.md) を参照。

```markdown
## Phase 1: 予測誤差の喚起

::: explorer
ゲーム理論は、複数の意思決定者が互いに影響し合う状況を分析する数学の一分野です。
:::

::: learner
ゲーム理論は、複数の主体（プレイヤー）が存在し、各プレイヤーの結果（利得）が
自分の行動だけでなく他のプレイヤーの行動にも依存する状況を分析する数学的枠組みです。

::: laboratory {id="game-theory-intro"}
:::
:::

::: scholar
> **学術的補足**: 本記事ではNash均衡を中心に扱う。協力ゲーム（Shapley値、コア）、
> メカニズムデザイン（Myerson, Maskin）については別記事を参照。
:::
```

| Directive | 対象モード | 表示ルール |
| :--- | :--- | :--- |
| `:::explorer` | Explorer | Explorerモード時のみ表示 |
| `:::learner` | Learner | Learnerモード時のみ表示（デフォルト） |
| `:::scholar` | Scholar | Scholarモード時のみ表示 |

> **フォールバック**: モード別コンテンツが未定義の場合、Learnerコンテンツを表示する。

### 4.5 Quiz Directive (`parsed`)

確認問題・理解度チェックのためのDirective。Phase 4（確認問題）およびExplorer末尾のQuizに使用。

```markdown
::: quiz {id="nash-equilibrium-check"}
### Q1: ナッシュ均衡とは？

以下の説明のうち、ナッシュ均衡を正しく説明しているものはどれですか？

- [ ] すべてのプレイヤーが最大の利得を得ている状態
- [x] どのプレイヤーも一方的に戦略を変更するインセンティブがない状態
- [ ] 協力によって全員が得をする状態

::: feedback {correct}
正解です！ナッシュ均衡は「誰も一方的に戦略を変えたくない」状態を指します。
:::

::: feedback {incorrect}
惜しい！Learnerモードで詳しく学ぶと、この概念がより明確になります。
:::
:::
```

| 属性 | 説明 |
| :--- | :--- |
| `id` | Quiz識別子（必須） |
| `optional` | `true` の場合、スキップ可能であることを明示 |

> **理論的根拠**: Quiz設計は流暢性の錯覚（USER_BEHAVIOR_MODEL.md 3.6節）への対策として機能する。

### 4.6 Agora Prompt Directive (`leaf`)

Agoraへの議論誘導（Scholarモード向け）。

```markdown
::: agora {topic="nash-equilibrium" prompt="ナッシュ均衡の計算複雑性について議論しましょう"}
:::
```

### 4.7 Diagram Directive (`leaf`)

静的なダイアグラム（フローチャート、概念マップ等）を埋め込む。
ダイアグラムライブラリは別リポジトリで開発し、Isidorica-Engineに統合する。

```markdown
::: diagram {type="flowchart" alt="ナッシュ均衡の計算フロー"}
start: 開始
input: 利得行列を入力
calculate: 各戦略の最適応答を計算
check: 均衡点存在？
  yes -> output
  no -> iterate
output: ナッシュ均衡を出力

start -> input -> calculate -> check
iterate -> calculate
:::
```

| 属性 | 必須 | 説明 |
| :--- | :--- | :--- |
| `type` | Yes | ダイアグラム種類（`flowchart`, `sequence`, `er`, `mindmap`等） |
| `alt` | Yes | 代替テキスト（アクセシビリティ必須） |
| `laboratory` | No | 関連するLaboratory ID。指定すると「操作する」ボタンを表示 |
| `id` | No | ダイアグラムID（相互参照用） |

#### Laboratoryとの使い分け

| 特性 | Diagram | Laboratory |
| :--- | :--- | :--- |
| **目的** | 構造・関係の可視化 | 能動的推論の促進 |
| **インタラクション** | 閲覧のみ（ホバー、ズーム） | 操作可能（パラメータ調整、実行） |
| **データ変更** | ❌ 不可 | ✅ 可 |
| **計算実行** | ❌ なし | ✅ あり（WASM） |

> **設計根拠**: LEARNING_FOUNDATIONS.mdに基づき、静的な構造理解（ダイアグラム）と能動的な予測誤差生成（Laboratory）を明確に分離する。インタラクティブな操作が必要な場合は `laboratory` 属性でLaboratoryに誘導する。

> **参照**: ダイアグラムライブラリの詳細仕様は [DIAGRAM_LIBRARY_SPEC.md](../../engine/technical/DIAGRAM_LIBRARY_SPEC.md) を参照。

### 4.8 Tabs Directive (`container`)

並列的な情報をタブで切り替えて表示する（漸進的開示）。
コードの言語切り替えだけでなく、解説のレベル別提示（直感/厳密）や、図解と数式の切り替えなど、多様な用途に使用できる。

#### 構造

| 要素 | 役割 |
| :--- | :--- |
| `:::tab-group` | コンテナDirective。タブグループ全体をラップ。 |
| `:::tab` | 各タブのコンテンツを格納。`title` 属性が必須。 |

```markdown
:::tab-group
:::tab {title="Python"}
Pythonの実装例...
:::
:::tab {title="JavaScript"}
JSの実装例...
:::
:::
```

#### 使用例: 説明のアプローチ切り替え

```markdown
:::tab-group

:::tab {title="直感的理解"}
比喩を使って概念を掴みます。
...
:::

:::tab {title="厳密な定義"}
数式を使って定義します。
$$ f(x) = \dots $$
:::

:::
```

### 4.9 Steps Directive (`container`)

手順や論理プロセスを視覚的なタイムラインとして表示する（チャンキング支援）。
複雑な論理展開をステップごとに分解し、学習者の認知負荷を軽減する。

#### 構造

`:::steps` ブロック内の **順序付きリスト (`1.`)** を自動的にステップコンポーネントとしてレンダリングする。

*   **タイトル**: リストアイテム先頭の **太字 (`**Text**`)** をステップのタイトルとして扱う。
*   **内容**: その後のテキストやブロック要素（コード、数式等）をステップの内容として扱う。

#### 使用例

```markdown
:::steps

1. **仮定の設定**
   まずは問題を単純化するために、摩擦がない系を仮定します。
   
   $$ F = ma $$

2. **運動方程式の立式**
   この系における運動方程式は以下のようになります。
   物体にかかる力は重力のみであるため...

3. **解の導出**
   微分方程式を解くと...

:::
```

---

## 5. Inline Directives (`:`)

インラインで使用する軽量なDirective。

### 5.1 キーボードキー (`:kbd:`)

キーボード入力を表示する。

```markdown
:kbd:[Ctrl] + :kbd:[C] でコピーします。
:kbd:[Cmd] + :kbd:[Shift] + :kbd:[P] でコマンドパレットを開きます。
```

**HTML出力**:
```html
<kbd>Ctrl</kbd> + <kbd>C</kbd> でコピーします。
```

### 5.2 参照 (`:ref:`)

番号付き要素（数式、図、表）への参照リンクを生成する。
**プレフィックスからラベルも自動生成されるため、「式」「図」「表」を手動で書く必要はない。**

```markdown
:ref:[eq:gauss-sum] より、1からnまでの和は...
:ref:[fig:nash-equilibrium] に示すように...
:ref:[table:payoff-matrix] を参照してください。
```

**HTML出力**:
```html
<a href="#eq:gauss-sum">式 (1)</a> より、1からnまでの和は...
<a href="#fig:nash-equilibrium">図 1</a> に示すように...
<a href="#table:payoff-matrix">表 1</a> を参照してください。
```

#### サポートするプレフィックス

| プレフィックス | 対象 | 自動生成されるラベル |
| :--- | :--- | :--- |
| `eq:` | 数式 | 「式 (番号)」 |
| `fig:` | 図 | 「図 番号」 |
| `table:` | 表 | 「表 番号」 |

#### ID定義方法

各要素でIDを定義する方法：

**数式**（`$$ ... $$ {#id}`）:
```markdown
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$ {#eq:gauss-sum}

:ref:[eq:gauss-sum] より...  → 「式 (1)」と表示
```

**図**（`:::figure {id="..."}`）:
```markdown
::: figure {id="fig:nash-equilibrium" src="./images/nash-eq.svg" alt="ナッシュ均衡の図解"}
ナッシュ均衡における戦略の相互作用
:::

:ref:[fig:nash-equilibrium] に示すように...  → 「図 1」と表示
```

**表**（`:::table {id="..."}`）:
```markdown
::: table {id="table:payoff-matrix" caption="囚人のジレンマの利得行列"}
| | 協力 | 裏切り |
|---|:---:|:---:|
| **協力** | (-1, -1) | (-3, 0) |
| **裏切り** | (0, -3) | (-2, -2) |
:::

:ref:[table:payoff-matrix] を参照してください。  → 「表 1」と表示
```

> **注意**: IDは `プレフィックス:名前` の形式で付与する（例: `eq:gauss-sum`, `fig:nash-equilibrium`）。番号とラベルは自動生成されるため記述不要。

### 5.3 アイコン (`:icon:`)

UIアイコンを表示する。

```markdown
設定は :icon:[settings] から変更できます。
:icon:[warning] この操作は取り消せません。
```

**HTML出力**:
```html
設定は <span class="icon icon-settings" aria-hidden="true"></span> から変更できます。
```

### 5.4 一覧

| Directive | 説明 | 例 |
| :--- | :--- | :--- |
| `:kbd:[キー]` | キーボードキー | `:kbd:[Enter]` |
| `:ref:[id]` | 番号付き要素への参照 | `:ref:[eq:gauss-sum]` |
| `:icon:[名前]` | アイコン | `:icon:[settings]` |

---

## 6. 数式レンダリング (Math Rendering)

### 6.1 基本方針

| 項目 | 方針 |
| :--- | :--- |
| **レンダラー** | **KaTeX互換の独自実装**。IFMパーサーの一部として実装する |
| **静的数式** | ビルド時にHTML生成（サーバーサイドレンダリング） |
| **動的数式** | Laboratory連動時はクライアントサイドレンダリング |
| **構文** | LaTeX記法 |
| **アクセシビリティ** | MathML出力を有効化 |

> **Note**: rehype/remarkは使用せず、IFMパーサーとして独自実装する。数式レンダリングエンジンはKaTeXと同等の機能を持つ独自実装とする。

### 6.2 記法

#### インライン数式

```markdown
アインシュタインの有名な式 $E = mc^2$ は...
```

#### ブロック数式

```markdown
$$
\int_{a}^{b} f(x) \, dx = F(b) - F(a)
$$
```

#### 番号付き数式

```markdown
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$ {#eq:gauss-sum}

:ref:[eq:gauss-sum] より...
```

| 属性 | 説明 |
| :--- | :--- |
| `{#eq:id}` | 数式ID（相互参照用、`eq:` プレフィックスで指定） |

> **参照**: 相互参照記法（`:ref:`）の詳細は5.2節を参照。

### 6.3 レンダリング方式

| 方式 | 使用場面 | メリット |
| :--- | :--- | :--- |
| **静的（ビルド時）** | 記事本文の数式 | LCP向上、初期表示高速 |
| **動的（クライアント）** | Laboratoryスライダー連動 | パラメータに応じて数式更新 |

#### 静的レンダリングの例

```markdown
二次関数 $f(x) = ax^2 + bx + c$ の頂点は...
```

ビルド時にHTMLへ変換：

```html
<span class="math-inline">
  <span class="katex">...</span>
  <math>...</math> <!-- MathML for a11y -->
</span>
```

#### 動的レンダリングの例（Laboratory連動）

```markdown
::: laboratory {id="quadratic-function" type="param"}
:::
```

スライダー操作で $a$, $b$, $c$ を変更すると、数式 $f(x) = ax^2 + bx + c$ がリアルタイムで更新される。

### 6.4 対応するLaTeX機能

| 機能 | 対応 | 例 |
| :--- | :--- | :--- |
| 基本演算 | ✅ | `+`, `-`, `*`, `/`, `=` |
| 上付き・下付き | ✅ | `x^2`, `x_i` |
| 分数 | ✅ | `\frac{a}{b}` |
| 根号 | ✅ | `\sqrt{x}`, `\sqrt[n]{x}` |
| 総和・積分 | ✅ | `\sum`, `\int`, `\prod` |
| ギリシャ文字 | ✅ | `\alpha`, `\beta`, `\gamma` |
| 行列 | ✅ | `\begin{matrix}...\end{matrix}` |
| 括弧サイズ | ✅ | `\left(`, `\right)` |
| テキスト | ✅ | `\text{...}` |
| 太字・イタリック | ✅ | `\mathbf{}`, `\mathit{}` |

### 6.5 アクセシビリティ

| 対応 | 説明 |
| :--- | :--- |
| **MathML出力** | スクリーンリーダー対応のためMathMLを併記 |
| **altテキスト** | シンプルな数式は `aria-label` で読み上げ文を提供 |
| **複雑な数式** | 前後の文脈で意味を説明することを執筆者に推奨 |

---

## 7. 6Phase記事構造ガイドライン

Isidoricaの記事は、LEARNING_FOUNDATIONS.md 4章で定義された「6Phase螺旋」に従って構成される。
この構造は**推奨**であり、Lintでの強制ではないが、Learnerモード向け記事では強く推奨する。

### 7.1 Phase構造

| Phase | 見出し例 | 目的 | 必須Directive |
| :--- | :--- | :--- | :--- |
| **Phase 1** | `## 予測誤差の喚起` | 読者の素朴な予測を引き出し、覆す | `:::laboratory` 推奨 |
| **Phase 2** | `## 定義` | 概念の正式な定義を提示 | - |
| **Phase 3** | `## 機序` | 概念の動作原理を段階的に説明 | `:::laboratory` 推奨 |
| **Phase 4** | `## 確認問題` | 理解度のチェック | `:::quiz` 必須 |
| **Phase 5** | `## 文脈・歴史` | 概念の歴史的・社会的意義 | - |
| **Phase 6** | `## 応用・転移` | 異なる文脈への転移例 | `:::laboratory` 推奨 |

### 7.2 サンプル構造

```markdown
---
slug: game-theory
title: ゲーム理論
description: 複数の意思決定者が互いに影響し合う状況を分析する数学的枠組み。
topics: [math, economics]
primary_discipline: economics
---

## Phase 1: 予測誤差の喚起

::: explorer
ゲーム理論の概要...
:::

::: learner
あなたは今、囚人のジレンマに直面しています。どちらを選びますか？

::: laboratory {id="prisoners-dilemma-intro"}
:::
:::

## Phase 2: 定義

::: learner
ゲーム理論とは...
:::

## Phase 3: 機序

::: learner
ナッシュ均衡の計算方法...

::: laboratory {id="nash-equilibrium-calculator"}
:::
:::

## Phase 4: 確認問題

::: quiz {id="game-theory-check" optional}
...
:::

## Phase 5: 文脈・歴史

::: learner
フォン・ノイマンとモルゲンシュテルンによる...
:::

## Phase 6: 応用・転移

::: learner
ゲーム理論は経済学だけでなく、政治学、生物学にも応用されています。

::: laboratory {id="game-theory-applications"}
:::
:::

::: scholar
> **未解決問題**: Nash均衡の計算はPPAD完全であり...

::: agora {topic="game-theory" prompt="ゲーム理論の限界について議論しましょう"}
:::
:::
```

> **参照**: 6Phaseモデルの詳細は [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) 4章を参照。

---

## 8. Document Structure Rules (Validation)

将来的なチャンク化（AI解析）のため、以下の構造ルールをLintで強制する。

1.  **見出し階層の厳守**: `H1` (Title) -> `H2` (Section) -> `H3` (Subsection) の順序を守る。スキップ（`H2`の次に`H4`など）は禁止。
2.  **セクション構造**: 原則として、見出しの直下には導入文（Introductory text）が必要。いきなり次の見出しが来る構造は避ける（チャンクの意味が途切れるため）。

---

## 9. 出典記法 (Reference Syntax)

### 9.1 インライン引用

本文中で出典を参照する際は、以下の記法を使用する。

```markdown
ゲーム理論はvon NeumannとMorgensternによって確立された [@neumann-1944, p. 45]。
複数出典を参照する場合は [@neumann-1944; @nash-1950] のように記述する。
```

| 記法 | 説明 | 例 |
| :--- | :--- | :--- |
| `[@id]` | 出典IDのみ | `[@neumann-1944]` |
| `[@id, p. XX]` | ページ番号付き | `[@neumann-1944, p. 45]` |
| `[@id, pp. XX-YY]` | ページ範囲 | `[@neumann-1944, pp. 45-67]` |
| `[@id; @id2]` | 複数出典 | `[@neumann-1944; @nash-1950]` |
| `[@id, chap. X]` | 章番号 | `[@neumann-1944, chap. 3]` |

### 9.2 出典ID

出典IDは `bibliography.json` で管理される。IDの命名規則：

| 形式 | 説明 | 例 |
| :--- | :--- | :--- |
| **著者-年** | 単著 | `@neumann-1944` |
| **著者-et-al-年** | 3人以上 | `@smith-et-al-2020` |
| **著者1-著者2-年** | 2人 | `@neumann-morgenstern-1944` |

### 9.3 参考文献リスト

記事末尾に自動生成される参考文献リストは、`primary_discipline` に基づいて適切なスタイル（Harvard, APA, Chicago等）でフォーマットされる。

```markdown
## 参考文献

<!-- 自動生成：bibliography.json から引用された出典のみ表示 -->
```

> **参照**: 出典管理の詳細は [GOVERNANCE.md](../GOVERNANCE.md) セクション1を参照。

### 9.4 コードスニペットの引用

コードスニペットの出典は、出典元に応じて管理方法が異なる。

#### 自作コード

記事のために執筆者が作成したコードは**出典不要**。Isidoricaのライセンスで公開される。

#### 引用コード（出典元別の管理）

| 出典元 | 管理方法 | 記法 |
| :--- | :--- | :--- |
| **書籍・論文** | `bibliography.json` | `[@source-id, p. 45]` |
| **公式ドキュメント** | `bibliography.json`（Webページとして） | `[@mdn-fetch]` |
| **OSS・GitHub** | コード内コメント（インライン） | `# Source: URL` + `# License:` |
| **Stack Overflow** | コード内コメント（インライン） | `# Source: URL` + `# CC BY-SA` |

#### OSS・GitHubからの引用

```python
# Source: https://github.com/example/repo/blob/abc123/src/main.py#L10-L20
# License: MIT
def hello():
    print("Hello, World!")
```

| 要素 | 説明 |
| :--- | :--- |
| `Source:` | コミットハッシュを含むパーマリンク |
| `License:` | 元コードのライセンス（MIT, Apache-2.0 等） |

#### Stack Overflowからの引用

```javascript
// Source: https://stackoverflow.com/a/12345678
// Author: username
// License: CC BY-SA 4.0
function example() { }
```

| 要素 | 説明 |
| :--- | :--- |
| `Source:` | 回答のパーマリンク |
| `Author:` | 回答者のユーザー名 |
| `License:` | CC BY-SA 4.0（Stack Overflow標準） |

> **参照**: コード引用の詳細は [GOVERNANCE.md](../GOVERNANCE.md) セクション10.5を参照。

---

## 10. 画像・図表記法

### 10.1 基本方針

画像・図表の記法は、**用途に応じて2種類を使い分ける**。

| 用途 | 記法 | キャプション | 出典 |
| :--- | :--- | :--- | :--- |
| **図表（アカデミック用途）** | `:::figure` directive | 必須 | 外部画像は必須 |
| **インライン画像（アイコン等）** | `![alt](src)` | 不要 | 不要 |

> **設計根拠**: Isidoricaの設計思想「Semantic over Presentation」に基づき、キャプションや出典が必要な図表には明示的な `:::figure` directiveを使用する。CommonMarkの画像記法（`![alt](src)`）にイタリック慣習でキャプションを付ける方法は、セマンティックでなく、アクセシビリティ（WCAG 2.2 AA）の観点からも不十分なため採用しない。

### 10.2 figure directive

キャプションや出典が必要な図表に使用する。
**図番号は自動採番されるため、執筆者はキャプションのみ記述する。**

```markdown
::: figure {id="fig:prisoners-dilemma" src="./images/prisoners-dilemma.svg" alt="囚人のジレンマの利得行列"}
囚人のジレンマの利得行列

複数行のキャプションも記述可能。

出典: [@tanaka-game-theory-2020, p. 45]
:::
```

| 属性 | 必須 | 説明 |
| :--- | :--- | :--- |
| `src` | Yes | 画像パス |
| `alt` | Yes | 代替テキスト（アクセシビリティ） |
| `id` | No | 図表ID（相互参照用、`fig:` プレフィックスで指定） |

**相互参照の例**:
```markdown
::: figure {id="fig:nash-equilibrium" src="./images/nash-eq.svg" alt="ナッシュ均衡の図解"}
ナッシュ均衡における戦略の相互作用
:::

:ref:[fig:nash-equilibrium] に示すように...  → 「図 1」と表示
```

> **参照**: 相互参照記法の詳細は5.2節を参照。

#### HTML出力

```html
<figure id="fig:prisoners-dilemma">
  <img src="./images/prisoners-dilemma.svg" alt="囚人のジレンマの利得行列">
  <figcaption>
    <span class="figure-number">図 1:</span> 囚人のジレンマの利得行列
    <cite>出典: Tanaka (2020), p. 45</cite>
  </figcaption>
</figure>
```

### 10.3 通常のMarkdown画像

キャプション・出典が不要なインライン画像（アイコン、装飾等）に使用する。

```markdown
設定画面を開くには ![設定アイコン](./icons/settings.svg) をクリックします。
```

> **注意**: `./images/external/` 配下の画像に対してこの記法を使用すると、Lintで警告が表示される。

### 10.4 ディレクトリ規約

画像の種類に応じてディレクトリを分離し、CIでの出典チェックを可能にする。

| ディレクトリ | 用途 | 出典 |
| :--- | :--- | :--- |
| `./images/external/` | 外部から取得した画像 | 必須 |
| `./images/original/` | 自作の図表・イラスト | 不要 |
| `./icons/` | UIアイコン等 | 不要 |

### 10.5 出典ルール

| 画像タイプ | 出典 | 許諾 |
| :--- | :--- | :--- |
| **外部画像** | 必須（`:::figure` 内に記載） | 必須（PRに証拠添付） |
| **自作図表** | 不要 | - |
| **元データあり** | 元データの出典を `:::figure` 内に明記 | - |

> **参照**: 画像管理ワークフローの詳細は [IMAGE_MANAGEMENT.md](../../engine/IMAGE_MANAGEMENT.md) を参照。

---

## 11. 関連ドキュメント

| ドキュメント | 関連箇所 |
| :--- | :--- |
| [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) | 6Phaseモデル（4章）、教育設計の理論的基盤 |
| [USER_BEHAVIOR_MODEL.md](../../shared/USER_BEHAVIOR_MODEL.md) | 流暢性の錯覚（3.6節）、Quiz設計の根拠 |
| [READER_ADAPTATION_DESIGN.md](../../engine/READER_ADAPTATION_DESIGN.md) | Explorer/Learner/Scholar Directiveの設計根拠 |
| [GOVERNANCE.md](../GOVERNANCE.md) | 出典管理、引用形式マッピング |
| [NETWORK_ARCHITECTURE.md](./NETWORK_ARCHITECTURE.md) | 外部ファイルでの関係管理 |
| [IMAGE_MANAGEMENT.md](../../engine/IMAGE_MANAGEMENT.md) | 画像管理ワークフロー |
