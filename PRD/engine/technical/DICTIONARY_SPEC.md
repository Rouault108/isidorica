# 辞書技術仕様 (Dictionary Technical Specification)

Isidorica辞書の技術仕様。スキーマ定義、ファイル構成、処理フロー、バリデーションを含む。

**ステータス**: 📋 ドラフト（Phase 3で詳細を決定）

**関連ドキュメント**:
- [DICTIONARY_CONTENT_DESIGN.md](../../library/DICTIONARY_CONTENT_DESIGN.md): 辞書コンテンツ設計（哲学、対象範囲、項目設計）
- [CONTENT_FOUNDATIONS.md](../../shared/CONTENT_FOUNDATIONS.md): 辞書設計の理論的基盤
- [TOGGLETIPS_SPEC.md](../TOGGLETIPS_SPEC.md): Toggletips仕様

---

## 1. 層構造

辞書エントリーは以下の**5層構造**で構成する。

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Layer 1: FORM（形式）                                                   │
│  発音、綴り、形態素、語形変化                                                │
├─────────────────────────────────────────────────────────────────────────┤
│  Layer 2: DISPLAY（表示）                                                │
│  直感的説明、定義、例文 ← ユーザーが最初に見る層                               │
├─────────────────────────────────────────────────────────────────────────┤
│  Layer 3: QUALIA（意味構造）                                            │
│  クオリア構造、型強制 ← GLに基づく意味記述                               │
├─────────────────────────────────────────────────────────────────────────┤
│  Layer 4: CONTEXT（文脈）                                               │
│  語用論、分布、生態、歴史 ← 使用文脈と変遷                               │
├─────────────────────────────────────────────────────────────────────────┤
│  Layer 5: NETWORK（関係）                                               │
│  上位語、下位語、関連語、翻訳 ← 概念ネットワーク                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 型定義

本セクションでは、スキーマ定義で使用する型を定義する。

### なぜ型を定義するのか

従来の辞書スキーマ（単純なキー・値の列挙）では、**語の意味の生成的な側面**を表現できなかった。例えば：

- 「本を読み始めた」と「本を書き始めた」で「本」の意味が変わるのはなぜか？
- 「良いナイフ」と「良い人」で「良い」の意味が変わるのはなぜか？

これらの現象を説明するため、Isidoricaは**生成語彙論（GL）**と**型強制論（TCL）**に基づく型システムを採用する。

### GL/TCLの辞書スキーマへの適用

| 理論概念 | スキーマでの実現 | 説明 |
| :--- | :--- | :--- |
| **クオリア構造** | `qualia` | 語の意味を4つの側面（構成・形式・目的・生成）で構造化 |
| **ドットオブジェクト** | `DotObject` | 1つの語が複数の型を同時に持つ（例：本 = 物理オブジェクト • 情報） |
| **型強制** | `Coercion` | 文脈に応じて型が変換される現象（例：「本を読み始めた」で本 → 情報） |
| **イベント構造** | `event_structure` | 状態・過程・変化の時間的構造 |

**例：「本」のクオリア構造**

```yaml
qualia:
  constitutive:                 # 何から構成されるか
    components: [紙, インク, 製本]
  formal:                       # 何であるか（型）
    type:
      components: [physical-object, information]  # ドットオブジェクト
      dominant: information
  telic:                        # 何のためか（目的）
    purpose: 読む、情報を伝達する
  agentive:                     # どう生じたか（起源）
    origin: 著者が書き、出版社が製本した
```

この構造により、「本を読み始めた」で「本」が「情報」の意味になる理由を**型強制**として説明できる。

> **理論的詳細**: GL/TCLの理論的基盤については [CONTENT_FOUNDATIONS.md](../../shared/CONTENT_FOUNDATIONS.md) PART 1を参照。

---

### TypeScriptによるGL/TCL実装

GL/TCLの概念をTypeScript/Zodでどう実装するかを考察する。

#### ドット・オブジェクト → 交差型

GLのドット・オブジェクト（1つのエンティティが複数の型を同時に持つ）は、TypeScriptの**交差型（Intersection Type）**で表現できる。

```typescript
// GL: 本 = 物理オブジェクト • 情報
type PhysicalObject = { weight: number; material: string };
type Information = { content: string; author: string };

// TypeScript: 交差型
type Book = PhysicalObject & Information;

// Zod: .merge() または .and()
const PhysicalObjectSchema = z.object({ weight: z.number(), material: z.string() });
const InformationSchema = z.object({ content: z.string(), author: z.string() });
const BookSchema = PhysicalObjectSchema.merge(InformationSchema);
```

#### 型強制 → Zodのtransform/preprocess

TCLの型強制（文脈に応じて型が変換される）は、**ランタイムの意味解釈プロセス**であり、TypeScriptの静的型システムでは直接表現できない。Zodの`transform`/`preprocess`でランタイム変換として実装する。

```typescript
// 型強制のランタイム実装
const coerceType = (qualia: Qualia, context: string): string => {
  const formal = qualia.formal;
  
  // ドット・オブジェクトの場合、文脈に応じて型を選択
  if (typeof formal.type === 'object' && 'components' in formal.type) {
    const { components, dominant } = formal.type;
    
    // 文脈ルールに基づいて型を選択
    if (context === 'reading' && components.includes('information')) {
      return 'information';
    }
    if (context === 'carrying' && components.includes('physical-object')) {
      return 'physical-object';
    }
    return dominant || components[0];
  }
  
  return formal.type as string;
};

// Zodスキーマでの適用
const SemanticInterpretationSchema = z.object({
  entity: z.string(),
  context: z.string(),
  qualia: QualiaSchema,
}).transform((data) => ({
  ...data,
  resolvedType: coerceType(data.qualia, data.context),
}));
```

---

### TypeScript導入のメリットと課題

#### メリット

| メリット | 説明 |
| :--- | :--- |
| **静的型安全性** | コンパイル時にスキーマ違反を検出。YAMLとTypeScriptの型が一致しない場合にエラー |
| **IDE支援** | 自動補完、型ヒント、リファクタリング支援 |
| **ドキュメント効果** | 型定義がそのままドキュメントになる。スキーマの意図が明確 |
| **Zodとの統合** | ランタイムバリデーションと静的型の両方を1つのソースから生成 |

#### 課題

| 課題 | 詳細 |
| :--- | :--- |
| **静的型の限界** | 型強制は**ランタイム現象**。TypeScriptの静的型だけでは「文脈に応じた型変換」を表現できない |
| **理論と実装のギャップ** | GL/TCLは言語学理論であり、プログラミング言語の型システムとは目的が異なる |
| **複雑性の増大** | クオリア構造、型強制ルール、イベント構造をすべて型として表現すると、スキーマが複雑になる |
| **執筆者への負荷** | 型情報を正しく入力するには言語学の知識が必要。一般執筆者には難しい |

#### 解決策

| 課題 | 解決策 |
| :--- | :--- |
| **静的型の限界** | Zodの`transform`/`preprocess`でランタイム型強制を実装。静的型は「構造」を、ランタイムロジックは「意味解釈」を担当 |
| **理論と実装のギャップ** | 完全な理論的表現を目指さず、**実用上必要な部分のみ**を型として表現。理論的詳細はCONTENT_FOUNDATIONSに委ねる |
| **複雑性の増大** | 段階的開示と同様に、スキーマも**レベル分け**。基本フィールドは単純、GL/TCL構造は専門家向けEndmatterに配置 |
| **執筆者への負荷** | ガイド付きUI + AI支援（B.2で決定済み）。Zodスキーマはバリデーションとして機能し、誤った入力を検出 |

---

### TCL-TypeScript対応表

| TCL概念 | TypeScript/Zod実装 | 静的/ランタイム |
| :--- | :--- | :--- |
| **ドット・オブジェクト** | 交差型 `A & B` / `.merge()` | 静的 |
| **型強制** | `transform` / `preprocess` | ランタイム |
| **イベント構造** | Union型 `STATE \| PROCESS \| TRANSITION` | 静的 |
| **制約** | `.refine()` / `.superRefine()` | ランタイム |
| **パラメータ** | ジェネリクス `T<P>` | 静的 |

---

### 型のカテゴリ

以下の3カテゴリの型を定義する。

| カテゴリ | 内容 | 用途 |
| :--- | :--- | :--- |
| **基本型** | `string`, `ID`, `TypedID`, `date`, `I18n` | スキーマ全体で使用 |
| **Union型** | `EntryType`, `SemanticType`, `Register`等 | 値の選択肢を限定 |
| **複合型** | `DotObject`, `Constraint`, `Coercion`等 | GL/TCLの構造を表現 |

### 2.1 基本型

| 型 | 説明 | 例 |
| :--- | :--- | :--- |
| `string` | 任意のUTF-8文字列 | `"ナッシュ均衡"` |
| `ID` | 辞書エントリーID（`[a-z0-9-]+`） | `nash-equilibrium` |
| `TypedID` | タイプ付きID（`{type}/{id}`） | `person/john-nash` |
| `date` | ISO 8601日付 | `1879-03-14` |
| `I18n` | 多言語オブジェクト | `{ ja: "定義", en: "definition" }` |

---

#### ID型の詳細

| 型 | フォーマット | 用途 | バリデーション |
| :--- | :--- | :--- | :--- |
| `ID` | `[a-z0-9-]+` | 同一タイプ内の参照（例：`related`） | ファイル名と一致 |
| `TypedID` | `{type}/{id}` | 異なるタイプへの参照（例：`coined_by`） | ディレクトリ + ファイル名と一致 |

**ID vs TypedID の使い分け**:

```yaml
relations:
  broader: [nash-equilibrium]       # ID: 同じterm内の参照
  related: [dominant-strategy]      # ID: 同じterm内の参照

qualia:
  agentive: person/john-nash        # TypedID: personタイプへの参照
  
birth_place: place/princeton        # TypedID: placeタイプへの参照
```

---

#### date型の詳細

ISO 8601形式。精度は3段階。

| 精度 | フォーマット | 例 | 用途 |
| :--- | :--- | :--- | :--- |
| **完全日付** | `YYYY-MM-DD` | `1879-03-14` | 正確な日付が判明している場合 |
| **年月** | `YYYY-MM` | `1879-03` | 月までは判明している場合 |
| **年のみ** | `YYYY` | `1879` | 年のみ判明している場合 |

**特殊ケース**:

| ケース | 記法 | 例 |
| :--- | :--- | :--- |
| **紀元前** | `-YYYY` | `-0384`（ソクラテス没年） |
| **期間** | `YYYY/YYYY` | `1939/1945`（第二次世界大戦） |
| **不確実** | `~YYYY` | `~1600`（おおよそ1600年頃） |

```typescript
// Zodでの実装例
const DateFormat = z.string().regex(
  /^~?-?\d{4}(-\d{2})?(-\d{2})?(\/~?-?\d{4}(-\d{2})?(-\d{2})?)?$/
);
```

---

#### I18n型の詳細

**構造**:

```yaml
I18n:
  ja: string   # 日本語（必須）
  en: string   # 英語（任意）
```

**拡張**: 将来的に他の言語を追加可能（`zh`, `ko`, `es`等）。

---

### 2.2 Union型

> **実装方針**: TypeScriptの`enum`は使用せず、リテラル型のUnionまたはZodの`z.enum()`を使用する。

```typescript
// 実装例
type EntryType = 'term' | 'person' | 'place' | 'work' | 'organization' | 'event' | 'species' | 'journal';

// Zodでの定義
const EntryType = z.enum(['term', 'person', 'place', 'work', 'organization', 'event', 'species', 'journal']);
```

#### EntryType（エントリータイプ）

```
term | person | place | work | organization | event | species | journal
```

#### POS（品詞）

```
noun | verb | adjective | adverb | preposition | conjunction | interjection | pronoun | determiner | particle
```

#### Register（使用域）

```
formal | informal | technical | colloquial | archaic | literary | slang
```

#### Frequency（頻度帯）

```
high | medium | low
```

#### EventStructureType（イベント構造タイプ）

```
STATE | PROCESS | TRANSITION
```

#### EquivalenceType（翻訳等価性）

```
full | partial | cultural
```

#### Status（ステータス）

```
draft | review | published
```

#### CertaintyLevel（確信度）

```
established | contested | emerging
```

#### LabType（Laboratoryタイプ）

```
LAB-VIZ | LAB-PARAM | LAB-CODE | LAB-EXPLORE | LAB-QUIZ
```

#### PlaceType（地名種別）

```
country | region | city | town | village | district | mountain | river | lake | sea | ocean | island | continent | landmark
```

#### WorkType（作品種別）

```
book | article | film | music | album | painting | sculpture | software | game | website | podcast | thesis | patent
```

#### OrgType（組織種別）

```
university | research-institute | company | corporation | startup | ngo | npo | government | ministry | court | military | hospital | museum | library | media | sports-team
```

#### EventType（イベント種別）

```
war | battle | revolution | election | treaty | conference | summit | disaster | pandemic | discovery | invention | founding
```

#### ConservationStatus（保全状況）

```
EX | EW | CR | EN | VU | NT | LC | DD | NE
```

> IUCN Red List categories: Extinct, Extinct in the Wild, Critically Endangered, Endangered, Vulnerable, Near Threatened, Least Concern, Data Deficient, Not Evaluated

#### Discipline（学問分野）

```
mathematics | physics | chemistry | biology | medicine | engineering | computer-science | economics | law | philosophy | psychology | sociology | anthropology | history | linguistics | literature | art | music | education | political-science | geography | astronomy | earth-science | environmental-science | agriculture | architecture | business | communication | religion | other
```

#### SemanticType（意味型）

GL/TCLにおける意味的な型。`qualia.formal.type`、`DotObject.components`、`Coercion.source`/`target`等で使用。

**基本型**:

```
entity | object | abstract | event | state | process | property | relation |
person | place | time | quantity | information | artifact | document
```

**カスタム型の許容**:

基本型にない型が必要な場合は `custom:` プレフィックスで記述可能。

```typescript
type SemanticType = 
  | 'entity' | 'object' | 'abstract' | 'event' | 'state' | 'process'
  | 'property' | 'relation' | 'person' | 'place' | 'time' | 'quantity'
  | 'information' | 'artifact' | 'document'
  | `custom:${string}`;  // カスタム型
```

**例**:

```yaml
qualia:
  formal:
    type: artifact                    # 基本型
    # または
    type: custom:musical-instrument   # カスタム型
```

**バリデーション**:
- 基本型は上記のUnion型に含まれるか検証
- `custom:` プレフィックスの場合はCI警告（レビュー対象としてマーク）

---

### 2.3 GL/TCL由来の複合型

#### DotObject（ドットオブジェクト）

GLにおける複合型。1つのエンティティが複数の型を**同時に**持つ。

**構造**:

```yaml
DotObject:
  components: [string]   # 必須：2つ以上の型名
  dominant: string       # 任意：文脈で支配的な型
```

**記述例**:

```yaml
# 「本」のドットオブジェクト
qualia:
  formal:
    type:
      components: [physical-object, information]
      dominant: information
```

**バリデーション**:
- `components`は2つ以上の型名を含む
- `dominant`は`components`に含まれる型名でなければならない

---

#### Constraint（制約）

型に対する制約条件。

**構造**:

```yaml
Constraint:
  property: string     # 必須：制約対象のプロパティ
  operator: Operator   # 必須：比較演算子
  value: any           # 必須：制約値
```

**Operator enum**:

```
eq | ne | gt | gte | lt | lte | in | contains | exists
```

**記述例**:

```yaml
qualia:
  formal:
    type: nash-equilibrium
    constraints:
      - property: players
        operator: gte
        value: 2
```

---

#### Parameter（パラメータ）

パラメータ化された型の引数。

**構造**:

```yaml
Parameter:
  name: string         # 必須：パラメータ名
  type: string         # 必須：型名
  description: I18n    # 任意：説明
  default: any         # 任意：デフォルト値
```

**記述例**:

```yaml
qualia:
  formal:
    type: nash-equilibrium
    parameters:
      - name: n
        type: number
        description:
          ja: プレイヤー数
```

---

#### Coercion（型強制）

文脈によって型が強制的に変換される現象。

**構造**:

```yaml
Coercion:
  context: string      # 必須：強制が発生する文脈
  source: string       # 必須：元の型
  target: string       # 必須：変換後の型
  trigger:             # 必須：強制のトリガー
    predicate: string
    natural_language: I18n
  interpretation: I18n # 任意：解釈の説明
```

**記述例**:

```yaml
coercion:
  - context: reading
    source: physical-object
    target: information
    trigger:
      predicate: begin
      natural_language:
        ja: 「読み始める」は情報へのアクセスを開始することを意味する
    interpretation:
      ja: 本の内容（情報）を読むこと
```

---

## 3. スキーマ定義

### 3.1 辞書エントリー詳細スキーマ

```yaml
# Isidorica Dictionary Entry Schema v2.0

{term_id}:
  # ═══════════════════════════════════════════════════════════
  # Layer 1: FORM（形式）
  # ═══════════════════════════════════════════════════════════
  form:
    headword: string                    # 見出し語
    pronunciation:
      ipa: string                       # IPA表記
      audio: string                     # 音声ファイルパス（任意）
    orthography:
      variants: [string]                # 異表記（例: colour/color）
      script: string                    # 文字体系（例: Latin, Kanji）
    morphology:
      pos: string                       # 品詞
      inflections: object               # 語形変化（任意）
      derivations: [string]             # 派生語（任意）

  # ═══════════════════════════════════════════════════════════
  # Layer 2: DISPLAY（表示）← ユーザーが最初に見る層
  # ═══════════════════════════════════════════════════════════
  display:
    intuition:                          # 限定コード（日常的説明）
      ja: string
      en: string
    definition:                         # 精密コード（形式的定義）
      ja: string
      en: string
    examples:                           # 例文
      - sentence:
          ja: string
          en: string
        context: string                 # 文脈タグ（任意）
        source: string                  # 出典（任意）

  # ═══════════════════════════════════════════════════════════
  # Layer 3: QUALIA（意味構造）← GLに基づく
  # ═══════════════════════════════════════════════════════════
  qualia:
    constitutive:                       # 構成（何から成るか）
      components: [string]
      description:
        ja: string
    formal:                             # 形式（何であるか）
      type: string | DotObject
      supertypes: [string]
      constraints: [Constraint]
      parameters: [Parameter]
      description:
        ja: string
    telic:                              # 目的（何のためか）
      purpose: string
      description:
        ja: string
    agentive:                           # 生成（どう生じるか）
      origin: string
      description:
        ja: string
    event_structure:                    # イベント構造
      type: STATE | PROCESS | TRANSITION
      initial_state:
        ja: string
      final_state:
        ja: string
      process:
        ja: string

  coercion:                             # 型強制
    - context: string
      source: string
      target: string
      trigger:
        predicate: string
        natural_language:
          ja: string
      interpretation:
        ja: string

  # ═══════════════════════════════════════════════════════════
  # Layer 4: CONTEXT（文脈）
  # ═══════════════════════════════════════════════════════════
  pragmatics:                           # 語用論
    register: string                    # 使用域（学術的、日常的など）
    contexts: [string]                  # 使用文脈
    collocation: [string]               # 共起表現
    discourse_function: string          # 談話機能（談話標識など）
    presupposition: string              # 前提

  distribution:                         # 分布
    frequency: string                   # 頻度帯（高/中/低）
    neighbors: [string]                 # 意味的近傍語
    prototype: string                   # プロトタイプ的用法
    periphery: [string]                 # 周辺的用法

  ecology:                              # 生態
    frame: string                       # フレーム（FrameNet参照）
    frame_elements: object              # フレーム要素
    embodiment: string                  # 身体化メタファー
    image_schema: string                # イメージスキーマ
    cultural_context: string            # 文化的文脈
    scenario: string                    # 典型的シナリオ

  history:                              # 歴史
    etymology:
      origin: string                    # 語源言語
      original_form: string             # 原形
      pathway: [string]                 # 借用経路
    semantic_change:
      - period: string
        change_type: string             # 意味変化タイプ
        description:
          ja: string
    first_attestation: string           # 初出

  # ═══════════════════════════════════════════════════════════
  # Layer 5: NETWORK（関係）
  # ═══════════════════════════════════════════════════════════
  relations:
    broader: [string]                   # 上位語
    narrower: [string]                  # 下位語
    related: [string]                   # 関連語
    see_also: [string]                  # 参照
    antonyms: [string]                  # 対義語
    synonyms: [string]                  # 同義語

  translations:                         # 翻訳等価語
    - lang: string
      term: string
      equivalence_type: string          # full | partial | cultural
      notes: string

  # ═══════════════════════════════════════════════════════════
  # 体験的グラウンディング
  # ═══════════════════════════════════════════════════════════
  laboratory:
    - id: string
      type: string                      # LAB-VIZ, LAB-PARAM, LAB-CODE等
      description:
        ja: string

  # ═══════════════════════════════════════════════════════════
  # メタ情報
  # ═══════════════════════════════════════════════════════════
  meta:
    status: string                      # draft | review | published
    certainty_level: string             # established | contested | emerging
    sources: [string]                   # 主要出典
    last_updated: date
    contributors: [string]
```

---

## 4. 配置基準

辞書ファイルは **Frontmatter + 本文 + Endmatter** の3部構成。各配置先の基準を定義する。

### 4.1 配置先の役割

| 配置先 | 役割 | 基準 |
| :--- | :--- | :--- |
| **Frontmatter** | エントリーの識別・メタデータ | ルーティング・UI表示に必要な情報。タイプ固有の必須項目を含む |
| **本文** | 叙述的コンテンツ | Markdown記法、`[[Term]]`、数式、コードブロック、Laboratoryが使用可能 |
| **Endmatter** | 構造化データ | 段階的開示の「詳細」「完全」レベルで表示される情報。機械可読なYAML |

### 4.2 専門知識が必要な項目の扱い

Frontmatterには専門知識が必要な項目も含まれる（例：`species`の`scientific_name`）。

**原則**: 執筆者は知っている範囲で記述すればよい。不足はレビュープロセスで補完する。

| 原則 | 説明 |
| :--- | :--- |
| **参入障壁の最小化** | 執筆者が専門知識不足で項目を埋められなくてもPR提出可能 |
| **補完者の優先順位** | 1. 他のコントリビューター → 2. レビュワー |
| **出典必須** | 補完時も必ず出典を明記。公式データベース（GBIF, ITIS, NCBI Taxonomy等）も出典として有効 |
| **レビュワー補完の許容** | 確認方法を知っていれば少労力で可能な項目（学名等）はレビュワーが補完してもよい |

> **理論的根拠**: Isidoricaの出典至上主義により、補完時も検証可能性が担保される。専門知識の有無で執筆者を排除しない。

---

## 5. 必須 vs 任意フィールド

辞書ファイル（`content/dictionary/{type}/{id}.md`）における必須・任意フィールド。

| 配置先 | フィールド | 必須 | 説明 |
| :--- | :--- | :--- | :--- |
| **Frontmatter** | `name` | ✅ | 表示名 |
| **本文** | `## 直感` | 推奨 | 直感的説明（リッチコンテンツ） |
| **本文** | `## 定義` | 推奨 | 形式的定義 |
| **本文** | `## 例` | 推奨 | 例文 |
| **本文** | `## Laboratory` | - | 体験的グラウンディング |
| **Endmatter（一般）** | `discipline` | - | 学問分野 |
| **Endmatter（一般）** | `register` | - | 使用域 |
| **Endmatter（一般）** | `collocation` | - | コロケーション |
| **Endmatter（一般）** | `related` | - | 関連概念 |
| **Endmatter（一般）** | `origin_year` | - | 語源年/提唱年 |
| **Endmatter（一般）** | `coined_by` | - | 提唱者 |
| **Endmatter（一般）** | `purpose` | - | 目的 |
| **Endmatter（一般）** | `prototype` | - | 典型例 |
| **Endmatter（一般）** | `inflection` | - | 屈折パラダイム |
| **Endmatter（一般）** | `neighbors` | - | 分布的関連語 |
| **Endmatter（専門）** | `qualia` | - | クオリア構造 |
| **Endmatter（専門）** | `relations` | - | 語彙関係 |
| **Endmatter（専門）** | `event_structure` | - | イベント構造 |
| **Endmatter（専門）** | `coercion` | - | 型強制パターン |
| **Endmatter（専門）** | `ecology` | - | 認知的文脈 |
| **Endmatter（専門）** | `blend` | - | 概念ブレンド |
| **Endmatter（専門）** | `pragmatics` | - | 語用論 |
| **Endmatter（専門）** | `history` | - | 意味変化の年表 |

---

## 6. タイプ別項目定義

### 6.1 共通項目（全タイプ）

以下の項目は全エントリータイプで使用可能。

#### Frontmatter

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **name** | ✅ | string | 表示名 |
| **pronunciation** | - | string | 発音（カタカナ）。外来語・人名由来語のみ |
| **image** | - | object | メイン画像（下記参照） |

**image項目の構造**:

```yaml
image:
  src: ./images/example.jpg  # 相対パス（CIがR2 URLに自動変換）
  alt: 画像の説明            # 必須（アクセシビリティ）
  caption: キャプション       # オプション
  license: CC BY-SA 4.0      # オプション（外部画像は必須）
```

#### 本文セクション

| セクション | 必須 | 説明 | 理論的根拠 |
| :--- | :--- | :--- | :--- |
| `## 直感` | 推奨 | 限定コード（日常的説明）。`[[Term]]`可 | 言語コード論 |
| `## 定義` | 推奨 | 精密コード（形式的定義）。数式・`[[Term]]`可 | 定義の理論 |
| `## 例` | 推奨 | 代表的な例文、コードブロック | ユーザー行動研究 |
| `## Laboratory` | - | 体験的グラウンディング。`:::laboratory` | 循環性問題の外部接地 |
| `## 出典` | - | 参考文献（`[@cite]`記法） | 検証可能性 |

#### Endmatter共通項目

**一般執筆者向け**:

| 項目 | 型 | 説明 | 理論的根拠 |
| :--- | :--- | :--- | :--- |
| **discipline** | Discipline | 学問分野 | 辞書類型学 |
| **register** | Register | 使用域（formal/informal/technical） | コーパス辞書学 |
| **collocation** | string[] | コロケーション | コーパス辞書学 |
| **related** | ID[] | 関連概念 | 語彙関係論 |
| **origin_year** | number | 語源年/提唱年 | 歴史意味論 |
| **coined_by** | TypedID | 提唱者（personへの参照） | 歴史意味論 |
| **purpose** | string | 目的（TELIC簡易版） | GL |
| **prototype** | TypedID | 典型例 | プロトタイプ理論 |
| **neighbors** | ID[] | 分布的関連語（自動生成可） | 分布意味論 |

**専門家向け**:

| 項目 | 型 | 説明 | 理論的根拠 |
| :--- | :--- | :--- | :--- |
| **qualia** | object | formal, constitutive, telic, agentive | GL |
| **relations** | object | broader, narrower, synonyms, antonyms, meronyms | 語彙関係論 |
| **event_structure** | object | STATE/PROCESS/TRANSITION | GL |
| **coercion** | object[] | 型強制パターン | TCL/TTR |
| **ecology** | object | scenario, frame_elements, embodiment | 認知意味論 |
| **blend** | object | 概念ブレンド | 概念ブレンド理論 |
| **pragmatics** | object | 語用論 | 語用論 |
| **history** | object[] | 意味変化の年表 | 歴史意味論 |

---

### 6.2 term（専門用語）

共通項目に加えて以下を使用可能。

#### Frontmatter追加項目

なし（共通項目のみ）

#### Endmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **etymology** | - | string | 語源 |

---

### 6.3 person（人物）

#### Frontmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **birth_date** | 推奨 | date | 生年月日 |
| **death_date** | - | date | 没年月日（存命なら空） |
| **occupation** | 推奨 | string | 職業・肩書き |

#### Endmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **birth_place** | - | TypedID | 出生地（placeへの参照） |
| **nationality** | - | string | 国籍 |
| **known_for** | 推奨 | TypedID[] | 主な業績（termへの参照） |
| **works** | - | TypedID[] | 主な作品（workへの参照） |
| **affiliations** | - | TypedID[] | 所属組織（organizationへの参照） |

---

### 6.4 place（地名）

#### Frontmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **place_type** | ✅ | PlaceType | 種別（country, city, mountain, river等） |

#### Endmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **coordinates** | 推奨 | object | 座標（緯度/経度） |
| **country** | - | TypedID | 所属国（placeへの参照） |
| **area** | - | number | 面積（km²） |
| **population** | - | number | 人口 |
| **elevation** | - | number | 標高（m） |
| **established** | - | number | 設立年 |

---

### 6.5 work（作品）

#### Frontmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **work_type** | ✅ | WorkType | 種別（book, film, music, software, painting等） |
| **publication_date** | 推奨 | date | 発表年 |
| **creator** | 推奨 | TypedID | 著者・作者（personへの参照） |

#### Endmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **publisher** | - | TypedID | 出版社（organizationへの参照） |
| **genre** | - | string | ジャンル |
| **language** | - | string | 言語（原語） |
| **license** | - | string | ライセンス（ソフトウェアの場合） |

---

### 6.6 organization（組織）

#### Frontmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **org_type** | ✅ | OrgType | 種別（university, company, ngo, government等） |
| **founded** | 推奨 | number | 設立年 |

#### Endmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **dissolved** | - | number | 解散年（現存なら空） |
| **headquarters** | - | TypedID | 本部所在地（placeへの参照） |
| **founder** | - | TypedID | 創設者（personへの参照） |
| **parent_org** | - | TypedID | 上位組織（organizationへの参照） |
| **website** | - | string | 公式サイトURL |

---

### 6.7 event（出来事）

#### Frontmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **event_type** | ✅ | EventType | 種別（war, revolution, election, conference等） |
| **start_date** | ✅ | date | 開始日 |
| **end_date** | - | date | 終了日（継続中なら空） |

#### Endmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **location** | 推奨 | TypedID | 場所（placeへの参照） |
| **participants** | - | TypedID[] | 参加者・当事者 |
| **outcome** | - | string | 結果・帰結 |
| **casualties** | - | number | 死傷者数 |

---

### 6.8 species（生物種）

#### Frontmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **scientific_name** | ✅ | string | 学名（ラテン名） |

#### Endmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **taxonomy** | ✅ | object | 分類（界・門・綱・目・科・属・種） |
| **common_names** | - | object | 一般名（各言語） |
| **conservation_status** | - | ConservationStatus | 保全状況（IUCN分類） |
| **habitat** | - | string | 生息地 |
| **distribution** | - | string | 分布域 |

---

### 6.9 journal（学術雑誌）

#### Frontmatter追加項目

なし（共通項目のみ）

#### Endmatter追加項目

| 項目 | 必須 | 型 | 説明 |
| :--- | :--- | :--- | :--- |
| **issn** | 推奨 | string | ISSN |
| **publisher** | 推奨 | TypedID | 出版社（organizationへの参照） |
| **discipline** | 推奨 | Discipline | 分野 |
| **first_issue** | - | number | 創刊年 |
| **impact_factor** | - | number | インパクトファクター |
| **open_access** | - | boolean | オープンアクセスか |
| **website** | - | string | 公式サイトURL |

---

### 6.10 Phase 7以降で追加予定（暫定）

| タイプ | 主な固有項目 |
| :--- | :--- |
| **word** | pos（品詞）, conjugation, collocation |
| **chemical** | formula, molar_mass, density, melting_point |
| **language** | iso_code, speakers, language_family |
| **celestial_body** | body_type, mass, radius, orbital_period |
| **law** | jurisdiction, enacted_date, status |

---

## 7. ファイル構成

### 7.1 ディレクトリ構造

```
isidorica-library/
├── data/
│   └── dictionary-index.yml     ← Toggletips用（AI生成可）
│
└── content/
    └── dictionary/
        ├── term/
        │   ├── nash-equilibrium.md
        │   └── entropy.md
        ├── person/
        │   └── john-nash.md
        ├── place/
        │   └── paris.md
        ├── work/
        │   └── origin-of-species.md
        ├── organization/
        │   └── unesco.md
        └── event/
            └── world-war-ii.md
```

---

### 7.2 辞書エントリーファイル

辞書エントリーは `content/dictionary/{type}/{id}.md` に配置され、**Frontmatter + 本文 + Endmatter** の3部構成で記述する。

**id/typeの自動推論**:
- `id`: ファイル名から推論（`nash-equilibrium.md` → `nash-equilibrium`）
- `type`: ディレクトリから推論（`dictionary/term/` → `term`）

---

### 7.3 Endmatter記法

辞書ファイルの末尾に構造化メタデータを配置。

```markdown
<!-- ENDMATTER
qualia:
  formal: 状態
  constitutive: [player, strategy, payoff]
  telic: equilibrium-prediction
  agentive: john-nash

relations:
  broader: game-theory
  narrower: [pure-strategy-nash, mixed-strategy-nash]
  related: [dominant-strategy, pareto-optimal]
ENDMATTER -->
```

---

## 8. 処理フロー

辞書ファイル（`content/dictionary/*.md`）の処理フロー。

```
┌─────────────────────────────────────┐
│  content/dictionary/{type}/{id}.md  │
│  - Frontmatter + 本文 + Endmatter   │
│  - [[Term]]許容                     │
│  - リッチコンテンツ（数式、コード） │
└─────────────────────────────────────┘
    ↓
IFM_PARSER（カスタムパーサー）
    ↓
┌─────────────────────────────────────┐
│  パース結果                          │
│  - frontmatter: object              │
│  - content: AST                     │
│  - endmatter: object                │
└─────────────────────────────────────┘
    ↓
辞書ページテンプレート
    ↓
┌─────────────────────────────────────┐
│  辞書ページ (/dictionary/{id})       │
│  - セクション別レンダリング          │
│  - 段階的開示UI                     │
└─────────────────────────────────────┘
```

---

## 9. バリデーション

辞書ファイルのCIバリデーション。

| ルール | 検証方法 | エラー時の動作 |
| :--- | :--- | :--- |
| **Frontmatter必須項目** | `name` が存在するか | CIエラー |
| **タイプ固有必須項目** | タイプ別の必須項目（例：person の `birth_date`） | CI警告 |
| **Endmatter構文** | `<!-- ENDMATTER ... ENDMATTER -->` の正規表現マッチ | パースエラー |
| **Endmatter型チェック** | Zodスキーマによるバリデーション | CIエラー |
| **ID参照の存在** | `coined_by`, `birth_place` 等のID参照が存在するか | CI警告 |
| **画像ファイルの存在** | `image.src` で指定されたファイルが存在するか | CIエラー |

---

## 10. 設計原則

辞書ファイル（本仕様）とToggletipsインデックス（`TOGGLETIPS_SPEC.md`）は、**同一の概念を異なる目的で表現する**二層構造をとる。

| 層 | 目的 | 表現形式 |
| :--- | :--- | :--- |
| **Toggletipsインデックス** | 即時参照（ツールチップ表示） | 軽量・プレーンテキスト |
| **辞書ファイル** | 詳細閲覧（辞書ページ） | リッチ・構造化データ |

この二層構造を理解するため、以下に両者の違いを整理する。

### 10.1 リッチコンテンツの許容

Toggletipsインデックスは無限遡及・描画負荷を防止するためプレーンテキストのみ許容するが、辞書ファイルはリッチコンテンツを許容する。

| 記法 | Toggletipsインデックス | 辞書ファイル |
| :--- | :--- | :--- |
| **`[[Term]]`** | ❌ 禁止（無限遡及防止） | ✅ 許容 |
| **Markdown記法** | ❌ 禁止（プレーンテキスト） | ✅ 許容 |
| **数式（LaTeX）** | ❌ 禁止 | ✅ 許容 |
| **コードブロック** | ❌ 禁止 | ✅ 許容 |
| **画像** | ❌ 禁止 | ✅ 許容 |
| **Laboratory** | ❌ 禁止 | ✅ 許容 |

### 10.2 Toggletipsインデックスとの関係

両者は独立して管理され、段階的に作成できる。

| 観点 | 説明 |
| :--- | :--- |
| **独立管理** | 辞書ファイルとTogletipsインデックスは別々に管理される |
| **段階的作成** | Toggletipsインデックスのみで`[[Term]]`機能が動作。辞書ファイルは後から追加可能 |
| **同期チェック** | 辞書ファイルが存在するがTogletipsインデックスに未登録の場合はCI警告 |

> **参照**: Toggletipsインデックスの詳細は [TOGGLETIPS_SPEC.md](../TOGGLETIPS_SPEC.md) を参照。

---

## Appendix: 未決定事項

以下の事項は今後の設計で決定予定。

### B.1 型名の語彙 ✅ 決定済み

**決定**: Union型 + カスタム型の許容（`custom:` プレフィックス）

> **参照**: Appendix A.2「SemanticType（意味型）」を参照。

### B.2 執筆者UX ✅ 決定済み

**決定**: 誰でも編集可能 + ガイド付きUI + AI支援 + レビューで品質担保

| 観点 | 設計 |
| :--- | :--- |
| **編集権限** | 誰でも編集可能（専門家・非専門家を問わず） |
| **ガイド付きUI** | `qualia`構造を選択式で入力できるフォームを提供 |
| **AI支援** | 本文から`qualia`を推論し、入力候補として提案 |
| **出典必須** | GL/TCL構造を追加する場合は出典を明記 |
| **レビュー** | PRレビューで検証。誤りがあれば修正リクエスト |
| **段階的開示** | 初心者は「簡易入力」（目的、関連語のみ）から始められる |

**理論的根拠**: Isidoricaの理念「すべての人が知識にアクセスできる」に基づき、編集権限も開放。品質はレビュープロセスで担保。

### B.3 活用方法 ✅ 決定済み

**決定**: 段階的開示 + 概念ネットワーク + ID参照リンク

| 活用方法 | 説明 | 優先度 |
| :--- | :--- | :--- |
| **段階的開示の完全表示** | 「意味構造を見る」で`qualia`, `coercion`を展開表示 | 高 |
| **概念ネットワーク** | `relations` + `qualia.formal.supertypes`でグラフ表示 | 中 |
| **ID参照リンク** | `qualia.agentive`等のID参照を辞書・記事へリンク | 高 |

**将来的な拡張**:

| 活用方法 | 説明 |
| :--- | :--- |
| **検索拡張** | 上位型・同義語を用いた検索結果拡張 |
| **意味推論** | 型強制パターンを用いた多義語の意味推論 |
| **Toggletipでの活用** | Toggletipに`qualia`の要約を表示 |
| **Laboratory連携** | `qualia.telic`に基づくLaboratoryの提案 |

> **参照**: 段階的開示の設計は [DICTIONARY_CONTENT_DESIGN.md](../../library/DICTIONARY_CONTENT_DESIGN.md) セクション4を参照。
