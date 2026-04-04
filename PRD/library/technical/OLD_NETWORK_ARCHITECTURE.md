# 記事間ネットワークアーキテクチャ (Network Architecture)

本ドキュメントでは、Isidoricaにおける記事間のネットワーク構造（知識グラフ）の設計方針を定義する。

**ステータス**: Phase 2 確定

---

## 1. 設計思想

### 1.1 「辞書」ではなく「教科書」

Isidoricaは単なる「点の集合（辞書）」ではなく、**「線の集合（教科書）」** を目指す。

| アプローチ | 説明 | 問題点 |
| :--- | :--- | :--- |
| **辞書型**（Wikipedia等） | 記事はバラバラに存在。関連項目は「See Also」で列挙 | 初心者は「どれから読めばいいの？」と迷子になる |
| **教科書型**（Isidorica） | 記事間に「学習順序」「論理関係」を定義 | 体系的な学習が可能 |

### 1.2 既存分類法の限界

従来の十進分類法（NDC/UDC/DDC）は、**「1冊の本をどの棚に置くか」を決める排他的なツリー構造**である。

| 問題点 | 説明 |
| :--- | :--- |
| **多面性（Polyhierarchy）の欠如** | 「ゲーム理論」は数学でもあり、経済学でもあり、生物学でもある。しかし1つの棚にしか置けない |
| **横断的（Transdisciplinary）視点の欠如** | 脳科学（医学）と心理学（人文学）とAI（情報学）が交差する領域を扱えない |
| **時代への固定** | 19〜20世紀の学問体系に基づいており、新しい学際領域を表現できない |

### 1.3 解決策: ハイブリッド分類

Isidoricaでは、**ファセット分類法**と**グラフ構造**を融合したハイブリッド分類方式を採用する。

```
従来: 棚に置く（排他的ツリー）
     ↓
Isidorica: タグを付けて繋ぐ（多対多グラフ）
```

### 1.4 記事と概念（Topics）の関係

Isidoricaのネットワーク設計において、**記事と概念（Topics）の関係**は以下のように定義される。

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     概念ネットワーク (Topics Network)                    │
│                                                                         │
│    game-theory ─────── economics ─────── mathematics                    │
│         │                   │                  │                        │
│    nash-equilibrium    behavioral-econ    probability                   │
│         │                                      │                        │
│    pareto-efficiency                      bayes-theorem                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                              ↑
                    ┌─────────┴─────────┐
                    │                   │
            ┌───────┴───────┐   ┌───────┴───────┐
            │   記事A       │   │   記事B       │
            │ topics:       │   │ topics:       │
            │ - game-theory │   │ - game-theory │
            │ - nash-eq     │   │ - economics   │
            │ - economics   │   │ - behavioral  │
            └───────────────┘   └───────────────┘
```

| 構成要素 | 役割 |
| :--- | :--- |
| **概念（Topics）** | ネットワークの**ノード**。学問的概念を表す。概念間の関係は `concept-relations.yml` で定義 |
| **記事（Articles）** | 概念の**集合体**。複数の Topics を束ねることで、概念間を橋渡しする存在 |

**意味合い**:
*   記事は「この記事は topic A と topic B を扱っている」という宣言
*   Topics のネットワーク上に記事が「浮かぶ」のではなく、記事が Topics を「束ねる」
*   同じ Topic を持つ記事は、その Topic を通じて関連付けられる

**設計上の帰結**:
*   Topics の粒度は統一する必要がない（広い概念と狭い概念が共存可能）
*   記事には、扱う概念を網羅的に付与する（検索可能性向上）
*   階層構造は `concept-relations.yml`（`instance_of` 等）で表現

---

## 2. 3+1層ネットワークモデル

記事間の繋がりを、その**性質**に応じて4つのレイヤーで定義する。

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           記事 (Node)                                    │
├─────────────────────────────────────────────────────────────────────────┤
│  Layer 1: Dependency Layer     │  「AにはBが必要」有向グラフ（DAG）       │
│  (学習順序)                     │  → 学習パス（カリキュラム）を形成         │
├─────────────────────────────────────────────────────────────────────────┤
│  Layer 2: Semantic Layer       │  「対立」「派生」「包含」など論理関係      │
│  (論理関係)                     │  → 知識の構造化を担う                    │
├─────────────────────────────────────────────────────────────────────────┤
│  Layer 3: Facet Layer          │  「分野」「トピック」などのタグ           │
│  (分類タグ)                     │  → 横断的な検索（Discovery）を担う       │
├─────────────────────────────────────────────────────────────────────────┤
│  Layer 4: Implicit Vector Layer│  文脈的な意味的近接性                    │
│  (意味的近接性)                 │  → 将来的にベクトル化で自動計算          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.1 Layer 1: Dependency Layer（学習順序）

**役割**: 「Aを理解するにはBが必要である」という有向非巡回グラフ（DAG）を定義する。

**理論的基盤**:

| 理論 | 適用 | 参照 |
| :--- | :--- | :--- |
| **足場かけ (Vygotsky)** | 学習者のZPD（発達の最近接領域）に応じた段階的支援 | [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) §6.2 |
| **認知負荷理論 (Sweller)** | 内在的負荷を調整し、外在的負荷を最小化する順序設計 | [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) §1.4 |
| **処理水準理論 (Craik & Lockhart)** | 深い処理を促す順序設計 | [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) §2.5 |
| **遠隔転移の困難さ (Barnett & Ceci)** | 近接転移を優先するパス設計 | [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) §2.2 |

**管理方法**: 外部ファイル `config/learning-paths.yml` で線形パスとして一元管理。

#### Semantic Layer の `requires`（後述）との違い

| 項目 | Dependency Layer（learning-paths.yml） | Semantic Layer の `requires` |
| :--- | :--- | :--- |
| **目的** | カリキュラム（学習コース）の定義 | 論理的な前提関係の記録 |
| **性質** | 教育的意図を持った「おすすめ順序」 | 概念間の「必然的な依存」 |
| **例** | 「ゲーム理論入門」コースの順序 | 「一般相対性理論」は「リーマン幾何学」を requires |
| **使い分け** | 読者に提示するナビゲーション | 記事単体で参照される前提知識 |

#### スキーマ

```yaml
# config/learning-paths.yml
- id: game-theory-basics
  title: ゲーム理論入門
  description: ゲーム理論の基礎概念を学ぶパス
  level: beginner              # 難易度: beginner / intermediate / advanced
  discipline: economics        # 主分野（引用形式連動）
  prerequisites: []            # このパスを始める前に完了すべきパス
  next_paths:                  # このパス完了後に推奨するパス
    - game-theory-advanced
  steps:
    - prisoner-dilemma
    - nash-equilibrium
    - pareto-efficiency
    - mixed-strategy

- id: modern-philosophy
  title: 現代哲学入門
  level: beginner
  discipline: philosophy
  prerequisites: []
  next_paths:
    - existentialism-deep-dive
  steps:
    - phenomenology
    - existentialism
    - hermeneutics
```

**UXへの効果**:
- 記事冒頭に「この記事を読むのに必要な知識」を表示
- 「カリキュラム（学習ルート）」ページを自動生成
- 難易度別フィルタリング
- 進捗トラッキング（将来的なアカウント機能）

#### 記述順序のルール

`learning-paths.yml` 内のパス定義順序は、記事が複数のパスに所属する場合の**デフォルト表示優先順位**に影響する。

| ルール | 説明 |
| :--- | :--- |
| **基礎パスを上位に記述** | 入門・基礎的なパスをファイルの先頭に配置 |
| **応用パスは下位に記述** | 発展・応用的なパスは後ろに配置 |

**理由**: 記事に直接アクセスした場合、最初に定義されているパスがデフォルト表示される。Learnerにとって「最も基礎的・体系的な文脈」で出会うことが望ましいため、基礎パスを優先する。

> **参照**: 複数パス所属時のUI仕様は [ARTICLE_PAGE_DESIGN.md](../ARTICLE_PAGE_DESIGN.md) セクション3.2を参照。

> **UI詳細**: 具体的な表示形式・レイアウトはPhase 3で `INFORMATION_ARCHITECTURE.md` に定義する。

### 2.2 Layer 2: Semantic Layer（論理関係）

**役割**: **概念（Concept）間**の論理的関係を辞書として定義する。記事ではなく、概念レベルでの意味ネットワークを構築する。

**情報学的背景**: WordNetのような**意味ネットワーク（Semantic Network）** に基づく。概念間の関係を「ラベル付きエッジ」で表現するグラフ構造であり、特定の記事に依存しない普遍的な知識辞書として機能する。

**管理方法**: 外部ファイル `config/concept-relations.yml` で概念間の関係を一元管理。

#### 概念辞書型アプローチ

| 項目 | 説明 |
| :--- | :--- |
| **relations.yml の役割** | 概念（Concept/Topic）間の関係辞書。記事の存在とは独立 |
| **記事との関係** | 記事は `topics` フィールドで概念をタグ付けし、概念間の関係を**継承**する |
| **記事が存在しない場合** | 問題なし。辞書は記事とは独立して存在できる |
| **メリット** | 新しい記事が追加されたとき、その記事が持つ概念に基づいて自動的に関係が適用される |

#### スキーマ

```yaml
# config/concept-relations.yml
# 概念间の関係を定義。記事とは独立した知識辞書。

# 概念の対立関係
- concept_a: heliocentrism        # 地動説
  concept_b: geocentrism          # 天動説
  type: opposes                   # 対立

# 概念の拡張関係
- concept_a: general-relativity   # 一般相対性理論
  concept_b: special-relativity   # 特殊相対性理論
  type: extends                   # 拡張

# 概念の前提関係
- concept_a: general-relativity   # 一般相対性理論
  concept_b: riemannian-geometry  # リーマン幾何学  
  type: requires                  # 前提知識

# 概念の具体例関係
- concept_a: prisoner-dilemma     # 囚人のジレンマ
  concept_b: game-theory          # ゲーム理論
  type: instance_of               # 具体例
```

#### 概念と記事の関係（継承ロジック）

```
記事 A の topics: [heliocentrism, astronomy]
記事 B の topics: [geocentrism, astronomy]

concept-relations.yml に:
  heliocentrism opposes geocentrism

→ 記事 A と 記事 B には「opposes」関係が自動的に適用される
→ 右サイドバーに「対立する見解: [記事B]」と表示
```

**メリット**:
- 概念辞書を一度定義すれば、該当する概念を持つ全記事に自動適用
- 記事の追加・削除に関係なく、概念間の関係は維持される
- AIによる辞書の拡充が容易（「この概念辞書を分析して、不足を追加して」）

#### 記事固有の関係（オプション）

概念レベルでは捉えられない、記事固有の関係がある場合のみ `config/article-relations.yml` を使用する。

```yaml
# config/article-relations.yml（オプション）
# 概念レベルでは表現できない、記事固有の関係のみ定義

- source: copernicus-biography    # 人物記事
  target: copernican-heliocentrism
  type: introduces                # この人物がこの概念を提唱した
```

#### Relation Types（関係タイプ）

**ステータス**: 暑定（運用開始後に追加・変更の可能性あり）

**理論的基盤**: 関係タイプは以下の言語学理論に基づいて分類される。

| 理論 | 提供する概念 | 参照 |
| :--- | :--- | :--- |
| **オントロジー工学** | 上位/下位概念、全体/部分 | WordNet, SUMO |
| **生成語彙論 (GL)** | クオリア構造（FORMAL, TELIC, AGENTIVE） | [CONTENT_FOUNDATIONS.md](../../shared/CONTENT_FOUNDATIONS.md) §1.1 |
| **MTT** | 語彙関数（LF） | [CONTENT_FOUNDATIONS.md](../../shared/CONTENT_FOUNDATIONS.md) §1.4 |
| **辞書構造論 (Wiegand)** | マクロ・メディオ構造 | [CONTENT_FOUNDATIONS.md](../../shared/CONTENT_FOUNDATIONS.md) §0.2 |

##### 確定タイプ（カテゴリ別）

**論理的関係**:

| Type | 方向性 | 説明 | 例 |
| :--- | :--- | :--- | :--- |
| `opposes` | 双方向 | 対立・批判関係 | 天動説 ↔ 地動説 |
| `requires` | 一方向 | 論理的前提知識 | 一般相対性理論 → リーマン幾何学 |

**構造的関係**（GL: FORMALクオリアに対応）:

| Type | 方向性 | 説明 | 例 |
| :--- | :--- | :--- | :--- |
| `instance_of` | 一方向 | 具体例 | 囚人のジレンマ → ゲーム理論 |
| `extends` | 一方向 | 拡張・一般化 | 特殊相対性理論 → 一般相対性理論 |
| `part_of` | 一方向 | 全体/部分関係 | エンジン → 自動車 |

**歴史的関係**（GL: AGENTIVEクオリアに対応）:

| Type | 方向性 | 説明 | 例 |
| :--- | :--- | :--- | :--- |
| `derived_from` | 一方向 | 歴史的継承・発展 | ニュートン力学 → 解析力学 |
| `supersedes` | 一方向 | 置換・後継 | 相対性理論 → ニュートン力学 |
| `influenced_by` | 一方向 | 思想的影響 | 実存主義 → キェルケゴール |

**機能的関係**（GL: TELICクオリアに対応）:

| Type | 方向性 | 説明 | 例 |
| :--- | :--- | :--- | :--- |
| `applies_to` | 一方向 | 適用関係 | ゲーム理論 → 経済学 |

**語彙的関係**:

| Type | 方向性 | 説明 | 例 |
| :--- | :--- | :--- | :--- |
| `synonym_of` | 双方向 | 同義・別名 | 認知的不協和 ↔ 認知的不整合 |

#### 双方向関係の扱い

| 方向性 | 定義方法 | 処理 |
| :--- | :--- | :--- |
| **一方向** | `concept_a` → `concept_b` のみ定義 | そのまま使用 |
| **双方向** | `concept_a` → `concept_b` のみ定義 | ビルド時に逆方向も自動生成 |

双方向タイプ（`opposes`, `synonym_of`）は、片方を定義すれば逆方向のリンクも自動的に生成される。

**AI支援**: 将来的に、LLMを活用して `concept-relations.yml` の拡充を半自動化することを想定。

### 2.3 Layer 3: Facet Layer（分類タグ）

**役割**: 従来のタグ付けシステム。緩やかなグルーピングを行い、予期せぬ分野間の「飛び地」を繋ぐ。

**理論的基盤**: Ranganathan(1933) の**ファセット分類論 (Faceted Classification)** に基づく。

| RanganathanのPMEST | Isidoricaでの適用 |
| :--- | :--- |
| **Personality (P)** | 概念の核心（topic自体） |
| **Matter (M)** | 扱う素材・対象（情報論, アルゴリズム等） |
| **Energy (E)** | 活動・プロセス（解析, 証明等） |
| **Space (S)** | 適用される空間・文脈（生物学, 経済学等） |
| **Time (T)** | 時代・時期（古代, 近代等） |

この理論は「単一の階層に属さない」概念の多面的な分類を可能にする。

> **参照**: 辞書構造論におけるアクセス構造の詳細は [CONTENT_FOUNDATIONS.md](../../shared/CONTENT_FOUNDATIONS.md) §0.2 を参照。

**管理方法**: 記事内Frontmatter `topics` フィールドで分散定義。

```yaml
# 記事例: prisoner-dilemma.md
slug: prisoner-dilemma
title: 囚人のジレンマ
topics:
  - game-theory
  - ethics
  - psychology
  - economics
```

#### topics のマスター管理

概念辞書（`concept-relations.yml`）との語彙統一を担保するため、topics はマスターファイルで一元管理する。

> **スキーマ詳細**: `config/topics.yml` の形式定義、具体例、多言語対応については [セクション5.3](#53-configtopicsyml-スキーマ) を参照。

##### Topic 命名規則

| ルール | 説明 | 例 |
| :--- | :--- | :--- |
| **英語ケバブケース** | 小文字、ハイフン区切り | `game-theory`, `general-relativity` |
| **単数形優先** | 複数形は避ける | `algorithm` ✅ / `algorithms` ❌ |
| **略語を避ける** | フルスペル推奨 | `quantum-mechanics` ✅ / `qm` ❌ |
| **一般的な英語表記** | 学術的に確立された用語 | `heliocentrism` ✅ / `sun-centered-theory` ❌ |
| **概念レベル** | 記事タイトルではなく概念名 | `game-theory` ✅ / `game-theory-introduction` ❌ |

##### Topics 付与のガイドライン

> **参照**: Topics と記事の関係については [1.4 記事と概念（Topics）の関係](#14-記事と概念topicsの関係) を参照。

| ルール | 説明 | 例 |
| :--- | :--- | :--- |
| **網羅的に付ける** | 記事が扱う概念は、粒度に関わらず**すべて**付ける | `game-theory`, `nash-equilibrium` の両方を付けてよい |
| **冗長でも構わない** | 上位概念と下位概念の両方を付けても問題ない | 検索可能性が向上する |
| **主題に限定** | 記事が「主に扱う」概念のみ。言及しただけの概念は付けない | 「経済学入門」に `nash-equilibrium` は不要 |
| **概念レベルのみ** | 固有名詞（人名、作品名）ではなく、概念を付ける | `copernicus` ❌ / `heliocentrism` ✅ |

##### バリデーションフロー

```
┌─────────────────────────────────────────────────────────────┐
│                      執筆者がPR作成                          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│              GitHub Actions: topics-check.yml                │
│                                                             │
│  1. 記事の topics を抽出                                     │
│  2. topics.yml と照合                                        │
│     ├─ マッチ → ✅ 通過                                      │
│     ├─ alias マッチ → 🔧 自動修正（正規IDに置換してコミット）   │
│     └─ 未登録 → 🏷️ ラベル付与 `new-topic`                    │
│                                                             │
│  3. 未登録 Topic がある場合のみレビュワーに通知                │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│            レビュワーが未登録Topicを判断                      │
│                                                             │
│  A) 既存 Topic の表記ゆれ → 執筆者に修正依頼                   │
│  B) 正当な新規 Topic → topics.yml に追加するPRを作成           │
│  C) 粒度が不適切 → より一般的/具体的なTopicを提案              │
└─────────────────────────────────────────────────────────────┘
```

##### バリデーションパターン

| ケース | 処理 | 人間の介入 |
| :--- | :--- | :--- |
| **正規ID マッチ** | ✅ 通過 | なし |
| **alias マッチ** | 🔧 自動修正してコミット | なし |
| **未登録** | 🏷️ `new-topic` ラベル付与、レビュワーに通知 | 必要 |

##### ローカルツール（オプション）

| ツール | 機能 | ステータス |
| :--- | :--- | :--- |
| **CLI** | `pnpm topics:check` で記事のTopicをバリデーション | Phase 4 で実装 |
| **VS Code 拡張** | Frontmatter 編集時に Topic を自動補完、未登録警告 | 将来検討 |

##### Web 執筆支援ツール

```
[Topic 入力フィールド]
┌─────────────────────────────────────────┐
│  🔍 game                                │
│  ───────────────────────────────────────│
│  ✅ game-theory (ゲーム理論)             │
│  ✅ game-design (ゲームデザイン)         │
│  ➕ 新規登録: "game-xxx"                │
└─────────────────────────────────────────┘

→ 新規登録選択時は理由入力を求め、レビュワー承認後に追加
```

**UXへの効果**:
- `/topics/game-theory` などのインデックスページを自動生成
- 関連記事表示（同じトピックを持つ記事）
- 検索フィルタリング
- トピック階層によるブラウジング

> **UI詳細**: 具体的な表示形式・レイアウトはPhase 3で `INFORMATION_ARCHITECTURE.md` に定義する。

### 2.4 Layer 4: Implicit Vector Layer（意味的近接性）

**役割**: 人間が定義しきれない文脈的な近接性（Proximity）を自動計算する。

**計算言語学的背景**: **分散意味論（Distributional Semantics）** に基づく。「意味は文脈（周囲の言葉）によって規定される」という分布仮説から、記事のベクトル表現を生成し、類似度を計算する。

**現フェーズでの対応**:
- **直接実装はしない**（将来フェーズで対応）
- **準備として**:
  - `description`（Abstract）を必須化 → ベクトル化の種データ
  - 見出し構造を厳格化 → チャンク化に備える
  - `keywords` フィールド → ベクトル生成時の重み付けヒント

**将来的な実装イメージ**:
- 全記事の `description` をベクトル化（OpenAI Embedding等）
- 高次元空間に記事をマッピング
- 「この記事と意味的に近い記事」を自動推薦

### 2.5 Facet Layer と Vector Layer の補完関係

Facet Layer（L3）と Vector Layer（L4）は異なる役割を持ち、補完的に機能する。

#### 本質的な違い

| 観点 | Facet Layer (L3) | Vector Layer (L4) |
| :--- | :--- | :--- |
| **定義方法** | 人間が明示的に定義 | 機械が自動計算 |
| **構造** | 離散的（カテゴリ） | 連続的（ベクトル空間） |
| **関係の性質** | 包含関係（この記事は topic A に属する） | 距離関係（この記事は記事 B に近い） |
| **説明可能性** | 高い（なぜ関連しているか説明可能） | 低い（類似度は数値だが理由は不明瞭） |
| **Serendipity** | 低い（予測可能な関連） | 高い（予期しない関連を発見） |
| **Cold Start** | なし（記事作成時に topics を付与） | あり（記事が少ないと計算精度が低い） |

#### 役割の分担

| ユースケース | 適したLayer | 理由 |
| :--- | :--- | :--- |
| **「同じ分野の記事」表示** | Facet | topics が一致 = 同分野 |
| **「この記事を読んだ人は」推薦** | Vector | 意味的類似性 = 興味が近い |
| **検索フィルタリング** | Facet | 明示的なカテゴリで絞り込み |
| **「予期しない関連」の発見** | Vector | 人間が気づかない意味的近接性 |
| **カテゴリナビゲーション** | Facet | 階層的なブラウジング |
| **「もっと読む」推薦** | Vector | 文脈的な類似記事 |

#### 関連記事表示の戦略

| フェーズ | 戦略 | 実装 |
| :--- | :--- | :--- |
| **Phase 5-6（短期）** | Facet ベース | 同じ topics を持つ記事を表示 |
| **Phase 7以降（中長期）** | Vector ベースへ移行 | 意味的類似度による推薦。Facet は検索・ナビゲーションに特化 |

**設計上の帰結**:
*   **Facet Layer**: ナビゲーション、検索、カテゴリ表示に特化（「このトピックの記事」）
*   **Vector Layer**: 関連記事推薦、Serendipity に特化（「関連記事」「もっと読む」）
*   **両者の共存**: Fac et は説明可能な関連、Vector は意外な関連を提供

---

## 3. 設計原則

### 3.1 関係性の外部ファイル化（Separation of Concerns）

**原則**: 記事（ノード）の属性と、記事間の関係（エッジ）は別々に管理する。

| 管理対象 | 管理場所 | 理由 |
| :--- | :--- | :--- |
| **記事の属性**（タイトル、要約、タグ） | Markdownファイル内 Frontmatter | 記事自身の性質 |
| **記事間の関係**（依存、対立、パス） | 外部YAML（`concept-relations.yml`, `learning-paths.yml`） | 関係性の更新時に記事を修正する必要がなくなる |

**利点**:
- **保守性向上**: 学習パスの順序入れ替えが1行で済む
- **可視化容易**: グラフファイルだけを読めば全体構造を把握可能
- **AI支援**: 外部ファイルへの追記はAIによる半自動生成と相性が良い
- **役割分担**: 記事執筆（文章）と構造編集（グラフ）を分離可能

### 3.2 人間が定義すべき領域 vs 機械が計算すべき領域

| 領域 | 担当 | 理由 |
| :--- | :--- | :--- |
| **Dependency Layer** | 人間 | 「学ぶ順序」は教育的意図を反映する。機械には推論困難 |
| **Semantic Layer** | 人間（+ AI支援） | 論理関係は人間が判断すべき。AIは候補提案のみ |
| **Facet Layer** | 人間 | タグ付けは人間の判断 |
| **Vector Layer** | 機械 | 大量の記事間の潜在的近接性は人間には把握不能 |

---

## 4. 設定ファイル配置

```
isidorica/
├── config/
│   ├── disciplines.yml      # 分野リスト + 引用形式マッピング
│   ├── concept-relations.yml # 意味的関係（Semantic Layer）- 概念辞書
│   ├── learning-paths.yml   # 学習パス（Dependency Layer）
│   └── topics.yml           # トピックマスター（Facet Layer）
├── content/
│   └── articles/
│       └── *.md
└── src/
    └── schemas/             # Zodスキーマ定義
```

---

## 5. スキーマ定義

### 5.1 `config/learning-paths.yml` スキーマ

```yaml
# 配列形式
- id: string             # 必須: パスID（URL用、ケバブケース）
  title: string          # 必須: パス名
  description: string    # 任意: パスの説明
  level: string          # 任意: beginner / intermediate / advanced（デフォルト: beginner）
  discipline: string     # 任意: 主分野（引用形式連動）
  prerequisites:         # 任意: 前提パスIDの配列
    - string
  next_paths:            # 任意: 次に推奨するパスIDの配列
    - string
  steps:                 # 必須: 記事slugの配列
    - string
    - string
```

### 5.2 `config/concept-relations.yml` スキーマ

```yaml
# 配列形式 - 概念間の関係辞書（記事とは独立）
- concept_a: string      # 必須: 概念A（topicsと同じ語彙を使用）
  concept_b: string      # 必須: 概念B
  type: string           # 必須: 関係タイプ
  note: string           # 任意: 補足説明
```

**有効な `type` 値**:
- `opposes`（双方向）
- `extends`（一方向）
- `instance_of`（一方向）
- `derived_from`（一方向）
- `requires`（一方向）
- `synonym_of`（双方向）※将来追加予定

### 5.3 `config/topics.yml` スキーマ

#### 形式定義

```yaml
# 配列形式
- id: string             # 必須: 正規ID（英語ケバブケース、言語中立）
  domain: string         # 任意: 主分野（参考情報）
  parent: string         # 任意: 親トピック（階層構造）
  locales:               # 必須: 言語別の名称とエイリアス
    ja:                  # 必須: 日本語
      name: string       # 必須: 表示名
      description: string # 任意: トピックページのヘッダー・meta description用（1〜2文）
      aliases:           # 任意: 許容される別名
        - string
    en:                  # 任意: 英語
      name: string
      description: string
      aliases:
        - string
    # 他言語も同様に追加可能
```

#### 具体例

```yaml
# config/topics.yml
# すべてのTopicを一元管理。concept-relations.yml と同じ語彙を使用。

- id: game-theory
  domain: economics
  parent: mathematics
  locales:
    ja:
      name: ゲーム理論
      description: 複数の意思決定者が相互に影響を与え合う状況を数学的に分析する理論。
      aliases:
        - げーむりろん
    en:
      name: Game Theory
      description: A mathematical framework for analyzing strategic interactions.
      aliases:
        - theory-of-games

- id: heliocentrism
  domain: astronomy
  locales:
    ja:
      name: 地動説
      description: 太陽を中心に地球やその他の惑星が公転しているとする天文学のモデル。
      aliases: []
    en:
      name: Heliocentrism
      description: The astronomical model where the Earth and planets revolve around the Sun.
      aliases: []

- id: prisoner-dilemma
  domain: game-theory
  parent: game-theory
  locales:
    ja:
      name: 囚人のジレンマ
      description: 個人の合理的選択が集団的に非合理な結果を招くゲーム理論の代表的な例。
      aliases:
        - しゅうじんのじれんま
    en:
      name: Prisoner's Dilemma
      description: A classic game theory example where individual rational choices lead to collective irrationality.
      aliases:
        - prisoners-dilemma
        - prisonerdilemma
```

#### 多言語設計

| 項目 | 設計 |
| :--- | :--- |
| **識別子（id）** | 英語ケバブケースで言語中立。URLやコード内で使用 |
| **表示名** | `locales.{lang}.name` で言語別に定義 |
| **description** | `locales.{lang}.description` でトピックページのヘッダー・meta description用（1〜2文、50〜150字程度） |
| **別名（aliases）** | `locales.{lang}.aliases` で言語別に定義 |
| **必須言語** | `ja` のみ必須。他言語は任意 |
| **フォールバック** | 言語がない場合は `ja` → `en` → `id` の順で表示 |
| **拡張性** | 新しい言語は `locales.de`, `locales.fr` 等で追加可能 |

### 5.4 `config/disciplines.yml` スキーマ

```yaml
default:
  citation_style: string   # デフォルトCSLスタイル
  citation_name: string    # 表示名

disciplines:
  [discipline_id]:         # 分野ID（primary_discipline の値）
    name: string           # 日本語名
    name_en: string        # 英語名
    citation_style: string # CSLスタイル
    citation_name: string  # 引用形式の表示名
```

> **参照**: 詳細は [GOVERNANCE.md](../GOVERNANCE.md) のセクション1.1を参照。

### 5.5 バリデーション

ビルド時に以下をチェックする:

| チェック項目 | エラー時の動作 |
| :--- | :--- |
| `learning-paths.yml`: `steps` 内の slug が実在するか | ビルドエラー |
| `concept-relations.yml`: `concept_a`, `concept_b` が `topics.yml` に存在するか | 警告 |
| `concept-relations.yml`: `type` が定義済みの値か | 警告（未知のタイプは許容） |
| `learning-paths.yml`: 循環参照がないか | ビルドエラー |
| `learning-paths.yml`: `prerequisites`, `next_paths` のパスIDが実在するか | ビルドエラー |
| `disciplines.yml`: `discipline` が定義済みか | 警告 |
| **記事の `topics`**: `topics.yml` に存在するか | 未登録は `new-topic` ラベル付与 |
| **記事の `topics`**: alias マッチ | 🔧 自動修正（正規IDに置換） |

---

## 6. 理論的基盤

本ドキュメントの設計は、以下の理論的基盤文書に依拠する。

| 基盤文書 | 主要な適用箇所 | 対応レイヤー |
| :--- | :--- | :--- |
| [CONTENT_FOUNDATIONS.md](../../shared/CONTENT_FOUNDATIONS.md) | 辞書構造論、GL、MTT | L2: Semantic |
| [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) | 認知負荷理論、足場かけ | L1: Dependency |
| [CORE_FOUNDATIONS.md](../../shared/CORE_FOUNDATIONS.md) | 教育的公正、ケイパビリティ | 全体設計 |

### 6.1 言語学的基盤（CONTENT_FOUNDATIONS）

| 理論 | Isidoricaでの適用 | 対応レイヤー |
| :--- | :--- | :--- |
| **辞書構造論 (Wiegand)** | マクロ・メディオ構造としての概念間参照 | L2: Semantic |
| **辞書機能論 (Tarp)** | 認知機能（学習パス）、伝達機能（検索） | L1, L3 |
| **生成語彙論 (GL)** | クオリア構造による関係タイプ分類 | L2: Semantic |
| **MTT** | 語彙関数による形式化 | L2: Semantic |
| **ファセット分類 (Ranganathan)** | 多面的分類を可能にするPMESTファセット | L3: Facet |

### 6.2 学習理論的基盤（LEARNING_FOUNDATIONS）

| 理論 | Isidoricaでの適用 | 対応レイヤー |
| :--- | :--- | :--- |
| **足場かけ (Vygotsky)** | 学習者のZPDに応じた段階的支援 | L1: Dependency |
| **認知負荷理論 (Sweller)** | 内在的負荷調整、外在的負荷最小化 | L1: Dependency |
| **処理水準理論 (Craik)** | 深い処理を促す順序設計 | L1: Dependency |
| **スキーマ理論** | 概念間の関係性を明示し、スキーマ構築を支援 | L2: Semantic |

### 6.3 オントロジー工学

| 概念 | Isidoricaでの適用 |
| :--- | :--- |
| **上位/下位概念（Hypernym/Hyponym）** | `instance_of` 関係で表現 |
| **全体/部分（Holonym/Meronym）** | `part_of` タイプで表現 |
| **継承（Inheritance）** | `extends` 関係で表現 |

### 6.4 計算言語学

| 概念 | Isidoricaでの適用 |
| :--- | :--- |
| **分散意味論（Distributional Semantics）** | Implicit Vector Layer（将来実装） |
| **意味ネットワーク（Semantic Network）** | Semantic Layer |

---

## 7. 逆引きとUIへの反映

外部ファイルから記事への逆引きは、ビルド時に計算してページに反映する。

| 逆引き | 表示場所 | 表示内容 |
| :--- | :--- | :--- |
| **この記事を含む学習パス** | 記事ヘッダー or サイドバー | パス名、現在位置、前後の記事 |
| **この記事を前提とする記事** | 記事フッター | 「この記事の後に読む」リスト |
| **関連する記事（Semantic）** | 記事フッター | 対立・拡張等の関係を持つ記事 |
| **同じトピックの記事** | 記事サイドバー | topics が一致する記事 |

> **UI詳細**: 具体的な表示形式・レイアウト・表示件数はPhase 3で `INFORMATION_ARCHITECTURE.md` に定義する。

---

## 8. 今後の課題

本セクションに関連する今後の課題は [PROJECT_ROADMAP.md](../../shared/PROJECT_ROADMAP.md) を参照。

| 項目 | 該当セクション |
| :--- | :--- |
| 設定ファイル作成 | Phase 4.3 |
| Relation Types の追加検討 | Phase 2（ネットワークアーキテクチャ） |
| topics マスター管理 | Phase 4.3 |
| 設定ファイルの定期棚卸し | [GOVERNANCE.md](../GOVERNANCE.md) セクション8.4 |

---

## 9. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [MARKDOWN_SPEC.md](./MARKDOWN_SPEC.md) | Frontmatterスキーマ（`topics`, `primary_discipline` フィールド等） |
| [URL_AND_TAXONOMY_DESIGN.md](./URL_AND_TAXONOMY_DESIGN.md) | URL構造、ディレクトリ構造、タギング戦略 |
| [GOVERNANCE.md](../GOVERNANCE.md) | 引用形式マッピング、分野リスト管理 |
| [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) | 学習設計の認知モデル |
| [SERVICE_CONCEPT.md](../../shared/SERVICE_CONCEPT.md) | 「教科書」としてのサービス設計 |
| [PROJECT_ROADMAP.md](../../shared/PROJECT_ROADMAP.md) | 実装スケジュール（Phase 4.3で設定ファイル作成） |
