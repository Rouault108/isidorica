# 記事間ネットワークアーキテクチャ (Network Architecture)

Isidoricaにおける概念・記事間のネットワーク構造の設計方針を定義する。

**ステータス**: Phase 2 確定
**改訂日**: 2026-01-12
**理論的基盤**: [DESIGN_PRINCIPLES.md](../../shared/DESIGN_PRINCIPLES.md), [CONTENT_FOUNDATIONS.md](../../shared/CONTENT_FOUNDATIONS.md), [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md)

---

## 1. 設計思想

### 1.1 目的

記事間ネットワークは、以下の2つのユースケースに対応する。

| ユースケース | ニーズ | 設計原則 |
| :--- | :--- | :--- |
| **体系的学習** | 「何から学べばいいか」「次に何を学ぶべきか」 | A-5（認知的徒弟制）、B-2（外的構造提供） |
| **ピンポイント参照** | 「Xについて知りたい」「関連する概念は？」 | E-1（ユーザーニーズ適応）、C-5（興味発達） |

> **参照**: 設計原則の詳細は [DESIGN_PRINCIPLES.md](../../shared/DESIGN_PRINCIPLES.md) を参照。

### 1.2 教科書機能と辞書機能の統合

Isidoricaは「教科書」と「辞書」の両方の機能を統合する（[DESIGN_PRINCIPLES.md §1.2](../../shared/DESIGN_PRINCIPLES.md)）。

| 機能 | 役割 | ネットワークが提供するもの |
| :--- | :--- | :--- |
| **教科書機能** | 体系的な学習コンテンツを提供 | 学習パス、前提関係 |
| **辞書機能** | 参照用コンテンツを検索・提供 | 概念間関係、意味検索 |

### 1.3 既存分類法の限界と解決策

従来の十進分類法（NDC/UDC/DDC）は**排他的ツリー構造**であり、多面性を持つ概念を表現できない。

| 問題点 | 説明 | 理論的背景 |
| :--- | :--- | :--- |
| **多面性の欠如** | 「ゲーム理論」は数学・経済学・生物学に属するが、1つの棚にしか置けない | Ranganathanファセット分類論 |
| **横断的視点の欠如** | 学際領域（脳科学+心理学+AI等）を扱えない | 知識の動的構築性（D-3） |

**解決策**: 概念を**ノード**、関係を**エッジ**とする**グラフ構造**を採用し、多対多の関係を表現可能にする。

> **参照**: ファセット分類論の詳細は [CONTENT_FOUNDATIONS.md §0.2](../../shared/CONTENT_FOUNDATIONS.md) を参照。

---

## 2. 3要素モデル

記事間ネットワークは以下の3つの要素で構成される。

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           3要素モデル                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────┐                                                │
│  │  概念辞書            │ ← 概念間の論理的・構造的関係（人間定義）        │
│  │  (Concept Dictionary)│                                               │
│  └──────────┬──────────┘                                                │
│             │                                                           │
│             ▼                                                           │
│  ┌─────────────────────┐                                                │
│  │  学習パス            │ ← キュレートされた学習順序（人間定義）          │
│  │  (Learning Paths)   │                                               │
│  └──────────┬──────────┘                                                │
│             │                                                           │
│             ▼                                                           │
│  ┌─────────────────────┐                                                │
│  │  意味検索            │ ← 意味的類似性に基づく発見（機械計算）          │
│  │  (Semantic Search)  │                                               │
│  └─────────────────────┘                                                │
│                                                                         │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                         │
│  ┌─────────────────────┐                                                │
│  │  記事               │ ← 概念を `topics` フィールドで参照              │
│  │  (Articles)         │                                               │
│  └─────────────────────┘                                                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.1 要素の概要

| 要素 | 役割 | 管理方法 | 担当 |
| :--- | :--- | :--- | :--- |
| **概念辞書** | 概念間の論理関係を定義 | `config/concepts.yml` | 人間（+ AI支援） |
| **学習パス** | キュレートされた学習順序を定義 | `config/learning-paths.yml` | 人間 |
| **意味検索** | 意味的類似性に基づく発見を支援 | ベクトル化（将来実装） | 機械 |

### 2.2 記事と概念の関係

記事は概念を**参照**し、概念間の関係を**継承**する。

```
記事 A の topics: [heliocentrism, astronomy]
記事 B の topics: [geocentrism, astronomy]

concepts.yml に:
  heliocentrism opposes geocentrism

→ 記事 A と 記事 B には「opposes」関係が自動的に継承される
→ 右サイドバーに「対立する見解: [記事B]」と表示可能
```

---

## 3. 概念辞書 (Concept Dictionary)

### 3.1 役割

概念辞書は、概念（Topic）間の**論理的・構造的関係**を定義する知識辞書である。

| 特性 | 説明 |
| :--- | :--- |
| **記事から独立** | 記事が存在しなくても概念間の関係は定義可能 |
| **論理的真実** | 教育的意図ではなく、概念間の客観的関係を記述 |
| **AI支援可能** | LLMを活用した半自動拡充を想定 |

### 3.2 理論的基盤

| 理論 | 適用 | 参照 |
| :--- | :--- | :--- |
| **辞書構造論 (Wiegand)** | マクロ・メディオ構造としての概念間参照 | [CONTENT_FOUNDATIONS.md §0.2](../../shared/CONTENT_FOUNDATIONS.md) |
| **生成語彙論 (GL)** | クオリア構造による関係タイプ分類 | [CONTENT_FOUNDATIONS.md §1.1](../../shared/CONTENT_FOUNDATIONS.md) |
| **MTT** | 語彙関数による形式化 | [CONTENT_FOUNDATIONS.md §1.4](../../shared/CONTENT_FOUNDATIONS.md) |
| **オントロジー工学** | 上位/下位概念、全体/部分 | WordNet, SUMO |

### 3.3 関係タイプ

関係タイプは以下のカテゴリに分類される。

#### 論理的関係

| Type | 方向性 | 説明 | 例 |
| :--- | :--- | :--- | :--- |
| `opposes` | 双方向 | 対立・矛盾・競合関係 | 天動説 ↔ 地動説 |
| `requires` | 一方向 | 論理的前提知識 | 一般相対性理論 → リーマン幾何学 |
| `causes` | 一方向 | 因果関係 | 温室効果ガス → 地球温暖化 |

> **将来検討**: `opposes` の細分化（`contradicts`, `critiques`, `competes_with`, `contrasts_with`）は [PROJECT_ROADMAP.md](../../shared/PROJECT_ROADMAP.md) の「将来の検討事項」を参照。


#### 構造的関係（GL: FORMALクオリアに対応）

| Type | 方向性 | 説明 | 例 |
| :--- | :--- | :--- | :--- |
| `instance_of` | 一方向 | 具体例 | 囚人のジレンマ → ゲーム理論 |
| `extends` | 一方向 | 拡張・一般化 | 特殊相対性理論 → 一般相対性理論 |
| `part_of` | 一方向 | 全体/部分関係 | エンジン → 自動車 |

#### 歴史的関係（GL: AGENTIVEクオリアに対応）

| Type | 方向性 | 説明 | 例 |
| :--- | :--- | :--- | :--- |
| `derived_from` | 一方向 | 歴史的継承・発展 | ニュートン力学 → 解析力学 |
| `supersedes` | 一方向 | 置換・後継 | 相対性理論 → ニュートン力学 |
| `influenced_by` | 一方向 | 思想的影響 | 実存主義 → キェルケゴール |

#### 機能的関係（GL: TELICクオリアに対応）

| Type | 方向性 | 説明 | 例 |
| :--- | :--- | :--- | :--- |
| `applies_to` | 一方向 | 適用関係 | ゲーム理論 → 経済学 |

#### 語彙的関係

| Type | 方向性 | 説明 | 例 |
| :--- | :--- | :--- | :--- |
| `synonym_of` | 双方向 | 同義・別名 | 認知的不協和 ↔ 認知的不整合 |

### 3.4 スキーマ

概念は**軽量なタグ**として管理する。詳細な説明は記事が担う。

#### `config/topics.yml`

```yaml
# 概念タグの定義（最小限）
# Tooltip定義（restricted/elaborated）は任意。未定義時は記事から導出。

game-theory:
  ja: ゲーム理論
  en: Game Theory
  aliases: [げーむりろん, theory-of-games]

heliocentrism:
  ja: 地動説
  en: Heliocentrism
  aliases: []

# Tooltip定義が必要な場合（記事がない、または上書きが必要な場合）
quantum-entanglement:
  ja:
    name: 量子もつれ
    restricted: 離れた粒子間に生じる量子力学的な相関
    elaborated: |
      量子力学において、複数の粒子が強い相関を持ち、
      一方の測定結果が他方に即座に影響を与える現象。
  en:
    name: Quantum Entanglement
    restricted: Quantum correlation between distant particles
  aliases: [entanglement]
```

> **Tooltip機能の詳細**: `[[Term]]` 記法によるインライン参照機能は [TOGGLETIPS_SPEC.md](../../engine/TOGGLETIPS_SPEC.md) を参照。

#### `config/topic-relations.yml`

```yaml
# 概念間の関係定義

- from: heliocentrism
  to: geocentrism
  type: opposes

- from: general-relativity
  to: special-relativity
  type: extends

- from: general-relativity
  to: riemannian-geometry
  type: requires
```

### 3.5 設計原則との対応

| 設計原則 | 概念辞書が提供するもの |
| :--- | :--- |
| **A-4（転移促進）** | 異なるドメイン間の構造的類似性を `extends`, `applies_to` で明示 |
| **D-1（知識形成過程可視化）** | `opposes`, `supersedes` で論争・改訂の関係を可視化 |
| **D-2（認識的多様性）** | `opposes` で対立する見解を並置 |
| **B-1（暗黙知明示化）** | Toggletips による即時参照（→ [TOGGLETIPS_SPEC.md](../../engine/TOGGLETIPS_SPEC.md)） |

---

## 4. 学習パス (Learning Paths)

### 4.1 役割

学習パスは、学習者に提示する**キュレートされた学習順序**を定義する。

| 特性 | 説明 |
| :--- | :--- |
| **教育的意図** | 概念辞書の `requires` とは異なり、「おすすめ順序」を表現 |
| **線形構造** | 学習者が迷わないよう、明確な順序を提供 |
| **難易度別** | beginner / intermediate / advanced で分類 |

### 4.2 理論的基盤

| 理論 | 適用 | 参照 |
| :--- | :--- | :--- |
| **足場かけ (Vygotsky)** | 学習者のZPDに応じた段階的支援 | [LEARNING_FOUNDATIONS.md §6.2](../../shared/LEARNING_FOUNDATIONS.md) |
| **認知負荷理論 (Sweller)** | 内在的負荷を調整し、外在的負荷を最小化する順序設計 | [LEARNING_FOUNDATIONS.md §1.4](../../shared/LEARNING_FOUNDATIONS.md) |
| **処理水準理論 (Craik)** | 深い処理を促す順序設計 | [LEARNING_FOUNDATIONS.md §2.5](../../shared/LEARNING_FOUNDATIONS.md) |
| **遠隔転移の困難さ (Barnett)** | 近接転移を優先するパス設計 | [LEARNING_FOUNDATIONS.md §2.2](../../shared/LEARNING_FOUNDATIONS.md) |

### 4.3 概念辞書の `requires` との違い

| 項目 | 概念辞書の `requires` | 学習パス |
| :--- | :--- | :--- |
| **性質** | 論理的必然（AなしにBは理解不可能） | 教育的推薦（Aから学ぶとBが分かりやすい） |
| **定義者** | 概念間の客観的関係 | 教育者のキュレーション |
| **構造** | グラフ（多対多） | 線形リスト |
| **例** | 「一般相対性理論」は「リーマン幾何学」を requires | 「ゲーム理論入門」パス: 囚人のジレンマ → ナッシュ均衡 → パレート効率 |

**設計上の帰結**: 
- 概念辞書の `requires` は**自動的に**「この記事を読むための前提知識」として表示
- 学習パスは**キュレートされた**「おすすめの学習順序」として表示

### 4.4 スキーマ

```yaml
# config/learning-paths.yml

- id: game-theory-basics
  title: ゲーム理論入門
  description: ゲーム理論の基礎概念を学ぶパス
  level: beginner
  prerequisites: []
  next_paths:
    - game-theory-advanced
  steps:
    - prisoner-dilemma
    - nash-equilibrium
    - pareto-efficiency
    - mixed-strategy

- id: modern-philosophy
  title: 現代哲学入門
  level: beginner
  steps:
    - phenomenology
    - existentialism
    - hermeneutics
```

### 4.5 設計原則との対応

| 設計原則 | 学習パスが提供するもの |
| :--- | :--- |
| **A-5（認知的徒弟制）** | 段階的な学習順序（Modeling → Scaffolding → Fading） |
| **B-2（外的構造提供）** | 学習者に明確な道筋を提供 |

---

## 5. 意味検索 (Semantic Search)

### 5.1 役割

意味検索は、**機械が計算する**意味的類似性に基づく発見機能である。

| 特性 | 説明 |
| :--- | :--- |
| **機械計算** | 人間が定義するのではなく、ベクトル化により自動計算 |
| **発見支援** | 人間が気づかない関連性を発見（Serendipity） |
| **将来実装** | Phase 7以降で実装予定 |

### 5.2 理論的基盤

| 理論 | 適用 | 参照 |
| :--- | :--- | :--- |
| **分散意味論** | 意味は文脈（周囲の言葉）によって規定される | 分布仮説 |

### 5.3 現フェーズでの対応

**直接実装はしない**（将来フェーズで対応）。

**準備として**:
- `description`（Abstract）を必須化 → ベクトル化の種データ
- 見出し構造を厳格化 → チャンク化に備える

### 5.4 設計原則との対応

| 設計原則 | 意味検索が提供するもの |
| :--- | :--- |
| **C-5（興味発達）** | 新奇性のある関連記事を発見 |
| **E-1（ユーザーニーズ適応）** | 参照モードでの「もっと読む」推薦 |

---

## 6. 記事と概念の統合

### 6.1 記事の `topics` フィールド

記事は `topics` フィールドで概念辞書の概念を参照する。

```yaml
# articles/prisoner-dilemma.md
---
slug: prisoner-dilemma
title: 囚人のジレンマ
topics:
  - game-theory
  - ethics
  - psychology
  - economics
---
```

### 6.2 関係の継承

記事が持つ `topics` を通じて、概念間の関係が記事に継承される。

```
記事 A の topics: [heliocentrism]
記事 B の topics: [geocentrism]

concepts.yml の relations:
  heliocentrism opposes geocentrism

→ 記事 A と 記事 B は自動的に「opposes」関係を持つ
→ UIに「対立する見解」として表示可能
```

### 6.3 概念語彙の管理

| 項目 | 設計 |
| :--- | :--- |
| **識別子（id）** | 英語ケバブケースで言語中立。URLやコード内で使用 |
| **表示名** | `locales.{lang}.name` で言語別に定義 |
| **description** | `locales.{lang}.description` で概念ページのヘッダー用 |
| **必須言語** | `ja` のみ必須。他言語は任意 |
| **フォールバック** | 言語がない場合は `ja` → `en` → `id` の順で表示 |

---

## 7. 設計原則

### 7.1 概念と記事の分離 (Separation of Concepts and Articles)

**原則**: 概念辞書は記事の存在に依存しない。

| 利点 | 説明 |
| :--- | :--- |
| **先行定義** | 記事がなくても概念間の関係を定義可能 |
| **再利用** | 同じ概念辞書を複数の記事集合で利用可能 |
| **AI支援** | 概念辞書の拡充はAI支援と相性が良い |

### 7.2 人間定義と機械計算の分離

| 担当 | 要素 | 理由 |
| :--- | :--- | :--- |
| **人間** | 概念辞書、学習パス | 論理関係・教育的意図は人間が判断すべき |
| **機械** | 意味検索 | 大量の記事間の潜在的近接性は人間には把握不能 |

### 7.3 関係性の外部ファイル化

**原則**: 記事（ノード）の属性と、概念間の関係（エッジ）は別々に管理する。

| 管理対象 | 管理場所 | 理由 |
| :--- | :--- | :--- |
| **記事の属性** | Markdownファイル内Frontmatter | 記事自身の性質 |
| **概念・関係** | `config/concepts.yml` | 関係性の更新時に記事を修正する必要がなくなる |
| **学習パス** | `config/learning-paths.yml` | パスの順序変更が1行で済む |

---

## 8. 設定ファイル配置

```
isidorica/
├── config/
│   ├── concepts.yml          # 概念辞書（概念定義 + 関係定義）
│   └── learning-paths.yml    # 学習パス
├── content/
│   └── articles/
│       └── *.md              # 記事（topics で概念を参照）
└── src/
    └── schemas/              # Zodスキーマ定義
```

---

## 9. バリデーション

ビルド時に以下をチェックする。

| チェック項目 | エラー時の動作 |
| :--- | :--- |
| `learning-paths.yml`: `steps` 内のslugが実在するか | ビルドエラー |
| `concepts.yml`: `relations` の `from`, `to` が概念として定義されているか | 警告 |
| `concepts.yml`: `type` が定義済みの値か | 警告 |
| `learning-paths.yml`: 循環参照がないか | ビルドエラー |
| 記事の `topics`: `concepts.yml` に存在するか | 未登録は `new-topic` ラベル付与 |

---

## 10. 逆引きとUIへの反映

外部ファイルから記事への逆引きは、ビルド時に計算してページに反映する。

| 逆引き | 表示場所 | 表示内容 |
| :--- | :--- | :--- |
| **この記事を含む学習パス** | 記事ヘッダー or サイドバー | パス名、現在位置、前後の記事 |
| **前提知識（requires）** | 記事ヘッダー | 「この記事を読むのに必要な知識」 |
| **関連する記事（Semantic）** | 記事サイドバー | 対立・拡張等の関係を持つ記事 |
| **同じ概念の記事** | 記事サイドバー | topics が一致する記事 |

---

## 11. 理論的基盤

本ドキュメントの設計は、以下の理論的基盤文書に依拠する。

| 基盤文書 | 主要な適用箇所 |
| :--- | :--- |
| [DESIGN_PRINCIPLES.md](../../shared/DESIGN_PRINCIPLES.md) | 設計原則の演繹 |
| [CONTENT_FOUNDATIONS.md](../../shared/CONTENT_FOUNDATIONS.md) | 辞書構造論、GL、MTT |
| [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) | 足場かけ、認知負荷理論 |
| [CORE_FOUNDATIONS.md](../../shared/CORE_FOUNDATIONS.md) | 教育的公正、ケイパビリティ |

### 11.1 言語学的基盤

| 理論 | 適用 |
| :--- | :--- |
| **辞書構造論 (Wiegand)** | マクロ・メディオ構造としての概念間参照 |
| **生成語彙論 (GL)** | クオリア構造による関係タイプ分類 |
| **ファセット分類論 (Ranganathan)** | 多面的分類を可能にする構造 |

### 11.2 学習理論的基盤

| 理論 | 適用 |
| :--- | :--- |
| **足場かけ (Vygotsky)** | 学習パスによる段階的支援 |
| **認知負荷理論 (Sweller)** | 内在的負荷調整、外在的負荷最小化 |

### 11.3 オントロジー工学

| 概念 | 適用 |
| :--- | :--- |
| **上位/下位概念（Hypernym/Hyponym）** | `instance_of` 関係 |
| **全体/部分（Holonym/Meronym）** | `part_of` 関係 |
| **継承（Inheritance）** | `extends` 関係 |

---

## 12. 今後の課題

| 項目 | 該当フェーズ |
| :--- | :--- |
| 概念辞書の初期構築 | Phase 4 |
| 学習パスの初期構築 | Phase 4 |
| 意味検索の実装 | Phase 7以降 |
| AI支援による概念辞書拡充 | 将来検討 |

---

## 13. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [MARKDOWN_SPEC.md](./MARKDOWN_SPEC.md) | Frontmatterスキーマ（`topics` フィールド等） |
| [DESIGN_PRINCIPLES.md](../../shared/DESIGN_PRINCIPLES.md) | 理論的設計原則 |
| [INTEGRATED_DESIGN_GUIDELINES.md](../../shared/INTEGRATED_DESIGN_GUIDELINES.md) | 統合設計指針 |
| [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) | 学習理論基盤 |
| [CONTENT_FOUNDATIONS.md](../../shared/CONTENT_FOUNDATIONS.md) | コンテンツ提示の理論基盤 |
| [PROJECT_ROADMAP.md](../../shared/PROJECT_ROADMAP.md) | 実装スケジュール |
