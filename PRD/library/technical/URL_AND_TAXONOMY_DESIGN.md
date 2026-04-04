# URL設計および分類戦略 (URL & Taxonomy Design)

本ドキュメントでは、Isidoricaにおける記事のURL構造、Slug命名規則、およびコンテンツ分類（Taxonomy）の戦略を定義する。

---

## 1. URL戦略 (URL Strategy)

### 1.1 基本方針: 言語プレフィックス + フラットSlug

Isidoricaは、**言語コード**をプレフィックスとし、その後に**フラットなSlug**を配置する構造を採用する。

*   **Pattern**: `https://{domain}/{lang}/{slug}`
*   **Example**: 
    *   日本語: `https://isidorica.org/ja/prisoner-dilemma`
    *   英語: `https://isidorica.org/en/prisoner-dilemma`

### 1.2 採用理由

*   **多言語対応**: 同じ概念（Slug）を異なる言語で提供可能。
*   **永続性 (Permanence)**: 「カテゴリ」は時代の解釈によって変化する（例：「数理哲学」は数学か哲学か？）。カテゴリをURLに含めると、カテゴリ移動のたびにURLが変わり、リンク切れ（Link Rot）が発生する。言語 + Slug構造であれば、内部的な管理ディレクトリを移動してもURLは不変であり続ける。
*   **シンプルさ (Simplicity)**: 冗長なパス（`/wiki/` や `/articles/`）を排除し、概念そのものへの最短アクセスを提供する。
*   **SEO**: 言語ごとに別URLを持つことで、hreflangタグによる言語切り替えが容易。

### 1.3 デフォルト言語とリダイレクト

| URL | 動作 |
| :--- | :--- |
| `/{slug}` （言語なし） | デフォルト言語（`ja`）にリダイレクト `/ja/{slug}` |
| `/{lang}/{slug}` | 指定言語の記事を表示 |
| `/{lang}/{slug}` （存在しない翻訳） | オリジナル言語にフォールバック、または「翻訳が存在しません」表示 |

### 1.4 言語コード

ISO 639-1 の2文字言語コードを使用する。

| コード | 言語 | 状態 |
| :--- | :--- | :--- |
| `ja` | 日本語 | オリジナル言語（デフォルト） |
| `en` | 英語 | 対応 |
| `zh` | 中国語 | 将来対応 |
| `ko` | 韓国語 | 将来対応 |

## 1.5 予約語 (Reserved Paths)

以下のパスは記事Slugまたは言語コードとして使用できない（ビルド時にバリデーションエラーとする）。

*   `about`
*   `search`
*   `paths`（学習パス用）
*   `topics`（トピック一覧用）
*   `archives`（パーマリンク用）
*   `api`
*   `admin`
*   `assets`

---

## 1.6 学習パス (Learning Paths) のURL構造

学習パスは記事の「集合」であり、記事とは異なるレベルのコンテンツのため、専用のディレクトリに配置する。

### URL構造

| ページ | URL | 説明 |
| :--- | :--- | :--- |
| **一覧ページ** | `/{lang}/paths` | 利用可能な学習パスの一覧 |
| **個別パス** | `/{lang}/paths/{path-id}` | 個別の学習パス（概要、記事リスト、進捗） |

例：
*   `/ja/paths` → 日本語の学習パス一覧
*   `/ja/paths/game-theory-basics` → 「ゲーム理論入門」パス

### 記事URLとの違い

| 観点 | 記事 | 学習パス |
| :--- | :--- | :--- |
| **URL** | `/{lang}/{slug}` （フラット） | `/{lang}/paths/{path-id}` （ディレクトリ内） |
| **性質** | 単一の概念を解説 | 複数の記事を順序付けた「メタコンテンツ」 |
| **衝突回避** | - | `/paths/` プレフィックスにより記事slugとの衝突なし |

> **詳細**: 学習パスのデータモデル、記事との関連については [NETWORK_ARCHITECTURE.md](./NETWORK_ARCHITECTURE.md) セクション3を参照。

---

## 1.7 Topics Index のURL構造

トピック一覧（Topics Index）および個別トピックページはPhase 4で導入する（詳細はセクション5.1参照）。

| ページ | URL | 説明 |
| :--- | :--- | :--- |
| **一覧ページ** | `/{lang}/topics` | 全トピックのリスト |
| **個別トピック** | `/{lang}/topics/{topic-id}` | ナビゲーションハブ（関連トピック、Featured Articles） |

例：
*   `/ja/topics` → 日本語のトピック一覧
*   `/ja/topics/game-theory` → 「ゲーム理論」トピックページ

---

## 1.8 SEOタグ (hreflang & Canonical)

多言語サイトにおいて、検索エンジンに正確な情報を伝えるためのタグ設定。

### 1.8.1 hreflang タグ

「この記事には他の言語版がある」と検索エンジンに伝えるタグ。ユーザーの言語設定に応じた検索結果表示を実現する。

#### 基本構造

```html
<link rel="alternate" hreflang="ja" href="https://isidorica.org/ja/game-theory" />
<link rel="alternate" hreflang="en" href="https://isidorica.org/en/game-theory" />
<link rel="alternate" hreflang="x-default" href="https://isidorica.org/ja/game-theory" />
```

#### 各属性の意味

| 属性 | 意味 |
| :--- | :--- |
| `rel="alternate"` | 「代替版（別のバージョン）がある」 |
| `hreflang="ja"` | 言語コード（ISO 639-1） |
| `hreflang="x-default"` | どの言語にも該当しないユーザー向けのデフォルト |

#### Isidoricaでの運用

| ルール | 内容 |
| :--- | :--- |
| **全言語版をリスト** | 同じSlugを持つ全ての言語版を `alternate` として出力 |
| **x-default** | デフォルト言語（`ja`）を指定 |
| **翻訳版が存在しない場合** | その言語の hreflang は出力しない |
| **生成タイミング** | ビルド時に静的生成（SSG） |

#### 実装上の注意

hreflang 生成時、各記事ごとに全記事を検索すると O(n²) の計算量となりビルド時間が増大する。以下の効率的な実装を採用すること。

| 手法 | 内容 |
| :--- | :--- |
| **Slugインデックスの事前構築** | ビルド開始時に全記事をSlugでグループ化したインデックスを作成（O(n)） |
| **各記事ビルド時** | インデックスから同一Slugの言語版を取得（O(1)） |

### 1.8.2 Canonical タグ

「このURLが正規版である」と検索エンジンに伝えるタグ。重複コンテンツ問題を回避する。

#### 基本構造

```html
<link rel="canonical" href="https://isidorica.org/ja/game-theory" />
```

#### Isidoricaでの運用

| ページタイプ | Canonical の指定先 |
| :--- | :--- |
| **通常の記事ページ** | 自己参照（そのページ自身のURL） |
| **アーカイブページ** | **最新版**（Canonical URL）を指定。自分自身ではない。 |
| **クエリパラメータ付き** | パラメータなしのURLを指定 |

### 1.8.3 出力例

#### 通常の記事ページ (`/ja/game-theory`)

```html
<head>
  <link rel="canonical" href="https://isidorica.org/ja/game-theory" />
  <link rel="alternate" hreflang="ja" href="https://isidorica.org/ja/game-theory" />
  <link rel="alternate" hreflang="en" href="https://isidorica.org/en/game-theory" />
  <link rel="alternate" hreflang="x-default" href="https://isidorica.org/ja/game-theory" />
</head>
```

#### アーカイブページ (`/ja/archives/game-theory/abc12345`)

```html
<head>
  <link rel="canonical" href="https://isidorica.org/ja/game-theory" />
  <meta name="robots" content="noindex, follow" />
</head>
```

*   アーカイブは検索結果に表示させない（`noindex`）。
*   hreflang はアーカイブページには出力しない（最新版のみで管理）。

#### 翻訳版が存在しない記事 (`/ja/new-article`)

```html
<head>
  <link rel="canonical" href="https://isidorica.org/ja/new-article" />
  <link rel="alternate" hreflang="ja" href="https://isidorica.org/ja/new-article" />
  <link rel="alternate" hreflang="x-default" href="https://isidorica.org/ja/new-article" />
  <!-- 英語版が存在しないため、hreflang="en" は出力しない -->
</head>
```

---

## 2. パーマリンク戦略 (Permalink Strategy)

学術的な引用（Citation）に耐えうるため、記事の内容が変わっても参照先の内容が保証される**恒久的リンク（Permalink）**を提供する。

### 2.1 ハイブリッドURL方式

| URLタイプ | 形式 | 役割 | 特徴 |
| :--- | :--- | :--- | :--- |
| **Canonical URL**<br>(人間用) | `/{lang}/{slug}` | 最新版の表示、SEO、シェア | コンテンツの更新とともに内容が変化する。<br>Slug変更時はリダイレクトされる。 |
| **Permanent URL**<br>(引用用) | `/{lang}/archives/{slug}/{hash}` | 参考文献としての引用、過去版参照 | **絶対不変**。特定時点の記事内容（Snapshot）を返す。<br>`robots: noindex` |

### 2.2 ハッシュの詳細仕様

| 項目 | 仕様 |
| :--- | :--- |
| **URL構造** | `/{lang}/archives/{slug}/{hash}` |
| **ハッシュ長** | **12文字**（SHA-256の先頭48bit） |
| **ハッシュ形式** | 小文字の16進数 |
| **衝突確率** | ~50% @ 1600万件（Isidoricaの規模では実質ゼロ） |

**例**:
```
/ja/archives/game-theory/a1b2c3d4e5f6
```

#### ハッシュ対象とR2保存データ

**原則**: **ハッシュ対象 = R2保存データ** でなければならない。

ハッシュが「コンテンツの同一性」を保証するため、ハッシュの計算対象とR2に保存されるデータは同一とする。

| 項目 | 仕様 |
| :--- | :--- |
| **ハッシュ対象** | 正規化後のMarkdownファイル全体（Frontmatter + 本文） |
| **R2保存データ** | 同上（正規化後のMarkdownファイル） |
| **除外フィールド** | `updated_at`（コンテンツの意味に影響しないフィールド） |

#### 正規化処理

ハッシュ計算およびR2保存の前に、以下の正規化処理を行う。

1.  `updated_at` フィールドを除外
2.  改行コードを `LF` に統一
3.  末尾の空白を除去
4.  ファイル末尾に改行を追加（POSIX準拠）
5.  SHA-256を計算し、先頭12文字をハッシュとして使用

#### 衝突対策と同一コンテンツ検出

`permalinks.json` で既存ハッシュを管理しているため、衝突検出と同一コンテンツの再利用が可能。

> **スキーマ詳細**: `config/permalinks.json` のスキーマは [CI_CD_PIPELINE.md](./CI_CD_PIPELINE.md) セクション3.1を参照。

| ケース | 判定 | 対処 |
| :--- | :--- | :--- |
| **新規コンテンツ** | ハッシュが `permalinks.json` に存在しない | 新規ハッシュとして登録、R2にアップロード |
| **同一コンテンツ** | ハッシュ一致、コンテンツ一致 | 既存ハッシュを使用（アップロード不要） |
| **異なるコンテンツ（衝突）** | ハッシュ一致、コンテンツ不一致 | ハッシュを16文字に延長して再計算・登録 |

**同一コンテンツとなるケース**:
*   **Revert（巻き戻し）**: 記事を v1 → v2 に更新後、v1 に戻した場合。v1のアーカイブは既にR2に存在するため、再アップロード不要。
*   **重複処理**: 同じ変更が誤って複数回適用された場合。
*   **`updated_at` のみの変更**: 本文が変わらずに `updated_at` だけ更新された場合（ハッシュ計算から除外しているため）。

**実装フロー**:
```
1. 正規化後のMarkdownからハッシュ（12文字）を計算
2. permalinks.json で同一ハッシュを検索
   ├─ 見つからない → 新規ハッシュとして登録、R2にアップロード
   ├─ 見つかった：
   │    ├─ コンテンツが同一 → 既存ハッシュを使用（アップロード不要）
   │    └─ コンテンツが異なる（衝突） → ハッシュを16文字に延長して再登録
```

### 2.3 アーキテクチャ

SSGのパフォーマンスとアーカイブの膨大なファイル数を両立するため、Cloudflare R2を活用する。

1.  **管理ファイル**: `config/permalinks.json` で各記事のハッシュ履歴とメタデータ（更新日、メッセージ）を管理。
2.  **アーカイブ保存 (CI)**: 記事更新時、CIが正規化後のMarkdownコンテンツを `isidorica-archives` バケット（R2）にアップロード。Gitリポジトリには保存しない。
3.  **アーカイブ閲覧 (Client)**: `/archives/...` へのアクセス時、Cloudflare Workers（Edge）で R2 から Markdown を取得しレンダリングする。

### 2.4 アーカイブ生成タイミング

#### トリガー

| 項目 | 仕様 |
| :--- | :--- |
| **トリガー** | main ブランチへのPRマージ時（自動） |
| **方式** | 全履歴（変更があればアーカイブ作成） |
| **重複防止** | 同一コンテンツはスキップ（ハッシュで判定） |
| **permalinks.json更新** | CIが自動コミット |

#### CI/CDフロー

```
main ブランチへの PR マージ
  ↓
GitHub Actions: アーカイブ生成ジョブが起動
  ↓
変更された記事を検出（git diff）
  ↓
各記事に対して:
  1. 正規化処理
  2. ハッシュ計算（12文字）
  3. permalinks.json で重複チェック
     ├─ 重複なし → R2 にアップロード、permalinks.json を更新
     └─ 重複あり → スキップ
  ↓
permalinks.json をコミット（自動）
```

> **参照**: CI/CDパイプラインの詳細は [CI_CD_PIPELINE.md](./CI_CD_PIPELINE.md) を参照。

### 2.5 R2内のパス構造

**バケット**: `isidorica-archives`

| 項目 | 仕様 |
| :--- | :--- |
| **パス形式** | `{lang}/{slug}/{hash}.md` |
| **lang** | 言語コード（`ja`, `en` 等） |
| **slug** | 記事のSlug（言語間で共通） |
| **hash** | 正規化後コンテンツのSHA-256先頭12文字 |
| **拡張子** | `.md` |

**例**:
```
isidorica-archives/
├── ja/
│   └── game-theory/
│       ├── a1b2c3d4e5f6.md   # バージョン1
│       └── f7e8d9c0b1a2.md   # バージョン2
├── en/
│   └── game-theory/
│       └── c3d4e5f6a1b2.md
└── ...
```

> **注意**: リポジトリ内のディレクトリ構造（引用形式ベース、例: `humanities/philosophy/`）はR2パスに反映されない。R2パスは `{lang}/{slug}/{hash}.md` で統一し、URLと1:1で対応させる。

### 2.6 運用ルール

*   **引用推奨**: 記事フッターの「引用する」ボタン等は、Canonical URL ではなく Permanent URL を提示する。
*   **警告表示**: アーカイブページ閲覧時は「これは過去のバージョンです。最新版はこちら」というアラートを表示する。

#### 引用導線のUI要件

読者が学術的引用としてPermanent URLを確実に取得できるよう、以下の導線を設ける。

| 要素 | 仕様 |
| :--- | :--- |
| **「引用する」ボタン** | 記事フッターに配置。クリック時にPermanent URLをコピー可能な形式で表示 |
| **デフォルト表示URL** | Permanent URL（`/{lang}/archives/{slug}/{hash}`）を優先提示。Canonical URLは補足として表示 |
| **現在バージョンのハッシュ取得** | ビルド時に `permalinks.json` から最新ハッシュを埋め込み |
| **コピー機能** | ワンクリックでPermanent URLをクリップボードにコピー |

**リスク対策**: 読者がブラウザのアドレスバー（Canonical URL）をコピーして引用するリスクを軽減するため、「引用」ボタンを目立たせ、Permanent URLの重要性を説明するツールチップを表示する。

### 2.7 アーカイブページの挙動

#### レンダリング方式

**Cloudflare Workers（Edge Function）** を使用する。

| 観点 | 説明 |
| :--- | :--- |
| **理由** | アクセシビリティ（JS無効でも閲覧可能）と高速表示を優先 |
| **処理** | Workers が R2 から Markdown を取得し、HTMLを生成して返却 |

#### 処理フロー

```
ユーザーが /ja/archives/game-theory/{hash} にアクセス
  ↓
Cloudflare Workers が受け取る
  ↓
permalinks.json から game-theory の currentHash を取得
  ↓
hash == currentHash ?
  ├─ Yes → /ja/game-theory（Canonical URL）に 302 リダイレクト
  └─ No（過去版）:
       ↓
     R2 から ja/game-theory/{hash}.md を取得
       ↓
     Markdown → HTML 変換
       ↓
     警告バナー + メタデータを挿入してHTMLを返却
```

#### 最新版リダイレクト

| 項目 | 仕様 |
| :--- | :--- |
| **条件** | アクセスされたハッシュが `permalinks.json` の `currentHash` と一致 |
| **動作** | Canonical URL（`/{lang}/{slug}`）に **302 (Temporary Redirect)** |
| **理由（SEO）** | 最新版は常にCanonical URLで表示し、重複コンテンツを回避 |
| **理由（UX）** | 最新版を見たいユーザーを自然にCanonical URLへ導く |

> **重要**: リダイレクトは **302 (Temporary Redirect)** を使用。301 (Permanent) だとブラウザがキャッシュし、将来ハッシュが過去版になった時に正しく動作しなくなる。

#### 表示要素（過去版のみ）

| 要素 | 表示内容 | 位置 |
| :--- | :--- | :--- |
| **警告バナー** | 「これは過去のバージョンです。最新版はこちら」 | ページ上部（目立つ位置） |
| **アーカイブ日時** | このバージョンが作成された日時 | ページ上部またはフッター |
| **最新版へのリンク** | Canonical URL へのリンク | 警告バナー内 |
| **バージョン履歴** | **提供しない**（GOVERNANCE.md準拠） | - |

> **注意**: バージョン履歴UIは提供しない。理由は [GOVERNANCE.md](../GOVERNANCE.md) セクション1「記事履歴の非公開」を参照。

#### SEO対応（過去版のみ）

| 項目 | 設定 |
| :--- | :--- |
| **robots** | `noindex, nofollow`（検索エンジンにインデックスさせない） |
| **canonical** | 最新版（Canonical URL）を指定 |

---

## 3. スラッグ命名規則 (Slug Rules)

記事のIDとなるSlug（=ファイル名）は、以下のルールに従う。

1.  **英語ベース**: URLの可読性と互換性のため、日本語ではなく英単語を使用する。
    *   OK: `general-relativity`
    *   NG: `ippan-soutairon` (ローマ字), `一般相対性理論` (日本語URL)
2.  **ケバブケース**: 全て小文字、単語間はハイフンで繋ぐ。
    *   OK: `game-theory`
    *   NG: `GameTheory`, `game_theory`
3.  **予測可能性**: 執筆者が推測しやすい一般的な名称を用いる。
    *   **略語制限**: `math`, `bio` など広く認知された略語以外は原則使用禁止。
    *   OK: `quantum-physics`
    *   NG: `q-phys`
4.  **概念IDとしての一意性 (Concept Uniqueness)**:
    *   Slugは単なるファイル名ではなく、**全言語共通の概念ID**である。
    *   **ルール**: 翻訳版を作成する際、必ずオリジナル版と同じSlugを使用しなければならない。
    *   Example:
        *   オリジナル: `ja/philosophy/game-theory.md`
        *   英語翻訳: `en/economics/game-theory.md` (ディレクトリが異なってもSlugは一致必須)
    *   **CI強制**: PR作成時、翻訳版のSlugがオリジナル版のSlugと一致しているか自動チェックし、不一致の場合はマージをブロックする。

---

## 4. ディレクトリ構造 (管理用)

物理的なファイル配置（リポジトリ内）は、**多言語対応**と**翻訳管理**を考慮した構造を採用する。
URLには影響しないが、検索性と管理コスト低減のため、以下のルールを設ける。

### 4.1 基本構造: 言語ベース + 主題サブディレクトリ

第1階層は**言語コード**、第2階層は**主題（Subject）** とする。

```
content/articles/
├── ja/                       # 日本語（オリジナル言語）
│   ├── philosophy/           # 哲学
│   │   ├── kant-ethics.md
│   │   └── plato-republic.md
│   ├── technology/           # 技術
│   │   └── react-hooks.md
│   └── misc/                 # 分類困難・未定
│       └── ...
│
├── en/                       # 英語（翻訳版）
│   ├── philosophy/
│   │   ├── kant-ethics.md    ← ja/philosophy/kant-ethics.md の翻訳
│   │   └── plato-republic.md
│   └── technology/
│       └── react-hooks.md
│
└── zh/                       # 中国語（将来）
    └── ...
```

### 4.2 設計理由

| 理由 | 説明 |
| :--- | :--- |
| **翻訳版の特定が容易** | 同じslugのファイルを言語ディレクトリで照合可能 |
| **自動化との親和性** | オリジナル更新時に翻訳版を自動特定できる |
| **URLとの整合性** | `/{lang}/{slug}` 形式のURLと一致 |
| **既存の主題分類を維持** | 言語内では従来の主題ディレクトリを使用 |

### 4.3 主題サブディレクトリ（引用形式ベース）

サブディレクトリは**引用形式（Citation Style）の自動推論**に使用される。
[GOVERNANCE.md](../GOVERNANCE.md) セクション1.1のマッピング定義と連動する。

#### ディレクトリ一覧

| ディレクトリ | 分野 | 引用形式 |
| :--- | :--- | :--- |
| `psychology/` | 心理学 | APA |
| `education/` | 教育学 | APA |
| `sociology/` | 社会学 | APA |
| `linguistics/` | 言語学 | APA |
| `medicine/` | 医学 | Vancouver |
| `biology/` | 生物学 | Vancouver |
| `law/` | 法学 | OSCOLA |
| `philosophy/` | 哲学 | Chicago (Notes) |
| `history/` | 歴史 | Chicago (Notes) |
| `art/` | 芸術 | Chicago (Notes) |
| `religion/` | 宗教学 | Chicago (Notes) |
| `economics/` | 経済学 | Chicago (Author-Date) |
| `politics/` | 政治学 | Chicago (Author-Date) |
| `natural-science/` | 自然科学（汎用） | Nature |
| `physics/` | 物理学 | Nature |
| `chemistry/` | 化学 | Nature |
| `mathematics/` | 数学 | AMS |
| `literature/` | 文学 | MLA |
| `engineering/` | 工学 | IEEE |
| `computer-science/` | コンピュータサイエンス | IEEE |
| `misc/` | 分類困難・未定 | Harvard（デフォルト） |

#### ディレクトリ構造例

```
content/articles/
├── ja/
│   ├── psychology/           # APA
│   │   └── cognitive-bias.md
│   ├── philosophy/           # Chicago (Notes)
│   │   ├── kant-ethics.md
│   │   └── plato-republic.md
│   ├── mathematics/          # AMS
│   │   └── set-theory.md
│   ├── computer-science/     # IEEE
│   │   └── react-hooks.md
│   ├── economics/            # Chicago (Author-Date)
│   │   └── game-theory.md
│   └── misc/                 # Harvard（デフォルト）
│       └── ...
│
├── en/                       # 英語（翻訳版）
│   └── ...（同構造）
│
└── zh/                       # 中国語（将来）
    └── ...
```

#### 設計原則

| 原則 | 説明 |
| :--- | :--- |
| **引用形式の自動推論** | ディレクトリから引用形式を自動決定。Frontmatterでの指定は不要 |
| **専門家の期待に応える** | 各分野の専門家が慣れ親しんだ引用形式を使用 |
| **執筆者の直感的な配置** | 分野名で配置場所を判断可能 |
| **URLに影響しない** | 後から分野を移動しても `/slug` は不変 |

#### 境界領域の扱い

*   執筆者が「主たる視点」と考える分野のディレクトリに配置する。
*   迷ったら `misc/` に置くことを許容し、定期的に管理者が適切な場所へ移動する（URLは変わらないため安全）。
*   Frontmatterで `discipline` を明示指定すれば、ディレクトリの推論を上書き可能。

### 4.4 オリジナル言語の管理

| 項目 | 内容 |
| :--- | :--- |
| **オリジナル言語** | Frontmatter の `original_language` で指定 |
| **デフォルト** | `ja`（日本語） |
| **翻訳元の特定** | 同じ slug を持つオリジナル言語版のファイル |

```yaml
# en/technology/react-hooks.md
slug: react-hooks
title: React Hooks Basics
original_language: ja  # このslugのオリジナルは ja/ にある
```

### 4.5 翻訳版のディレクトリ追従

翻訳版は、オリジナルと**同じ主題ディレクトリ**に配置しなければならない。引用形式や分類の整合性を保つためである。

#### ルール

| ルール | 説明 |
| :--- | :--- |
| **ディレクトリ一致強制** | 翻訳版はオリジナルと同じ主題ディレクトリに配置 |
| **CIでの警告** | 不一致の場合、PRで警告を表示（ブロックはしない） |
| **移動PR自動作成** | オリジナルのディレクトリが変更された場合、翻訳版の移動PRを自動作成 |

#### オリジナルのディレクトリ変更時の挙動

```
オリジナル記事の主題ディレクトリが変更された
    ↓
CIがオリジナルと翻訳版のディレクトリを比較
    ↓
不整合を検出
    ↓
翻訳版の移動PRを自動作成
    ├─ PRタイトル: 「chore: 翻訳版のディレクトリ追従 ({slug})」
    └─ 変更内容: `{lang}/{old-dir}/{slug}.md` → `{lang}/{new-dir}/{slug}.md`
    ↓
レビュワーがマージ
```

#### 不整合中の影響

| 項目 | 影響 |
| :--- | :--- |
| **URL** | 影響なし（URLに主題ディレクトリは含まれない） |
| **引用形式** | 翻訳版だけ古い引用形式で表示される可能性あり（短期間なら許容） |

> **参照**: 移動PR自動作成のワークフローは [CI_CD_PIPELINE.md](./CI_CD_PIPELINE.md) セクション5を参照。

---

## 5. 分類とタギング (Taxonomy)

記事の発見可能性（Discovery）を高めるため、物理ディレクトリとは別に「意味上のネットワーク」を構築する。

### 5.1 Topics (Tags)

多対多のタグ付けシステム。ファセット分類を実現する。

#### 基本構造

*   **Frontmatter**:
    ```yaml
    title: 囚人のジレンマ
    slug: prisoner-dilemma
    topics:
      - game-theory
      - ethics
      - psychology
    ```

#### 用途

| 用途 | 説明 |
| :--- | :--- |
| **検索フィルター** | 検索時にtopicsで絞り込みが可能 |
| **Related Articlesの計算** | 共通topicsを持つ記事を関連記事として表示 |
| **記事のメタデータ** | 記事ページにtopicsをタグとして表示 |

#### トピックページ戦略

トピックページは、「概念記事」との役割重複を避けつつ、**ナビゲーションハブ**として機能させる。

| フェーズ | 方針 |
| :--- | :--- |
| **Phase 4〜** | Topics Index (`/{lang}/topics`) + 個別トピックページ (`/{lang}/topics/{id}`) を作成。個別ページは「ナビゲーションハブ」として構成（下記参照）。 |
| **Phase 7以降** | ベクトル化による意味的近接性を活用し、関連トピック・関連記事の自動推薦を追加。 |

#### 個別トピックページの構成

概念記事が「解説」を担うのに対し、トピックページは「ナビゲーション（地図）」を担う。

| 要素 | 説明 | 概念記事との差別化 |
| :--- | :--- | :--- |
| **関連トピックへのリンク** | `concept-relations.yml` で定義された関係（`requires`, `extends`, `opposes` 等）を表示 | 概念記事は「この概念とは何か」、トピックページは「周辺に何があるか」 |
| **Featured Articles** | 編集者が選んだ入門・深掘り記事を2〜3件ハイライト | 概念記事は「自身が入門」、トピックページは「入口を複数提示」 |
| **全記事リスト（折りたたみ）** | 該当トピックを持つ全記事。デフォルトは折りたたみ | メインコンテンツではなく補助情報 |
| **ローカルグラフ（オプション）** | 関連トピック・記事をノード図で可視化 | 視覚的なナビゲーション |

> **設計原則**: トピックページに「解説文」を書かない。解説が必要な場合は、通常の概念記事（`/{lang}/{slug}`）を作成し、トピックページからリンクする。これにより二重管理を回避する。

#### マスター管理

Topics は `config/topics.yml` で一元管理される。自由なタグ付けではなく、管理された語彙（Controlled Vocabulary）を使用する。

| 項目 | 内容 |
| :--- | :--- |
| **管理ファイル** | `config/topics.yml` |
| **命名規則** | 英語ケバブケース、単数形優先、略語禁止 |
| **多言語対応** | `id`（言語中立）+ `locales.{lang}.name`（表示名） |
| **別名（aliases）** | 表記ゆれを吸収する別名を定義可能 |
| **description** | トピックページのヘッダーに表示する短い概要（1〜2文） |

#### topics.yml の description フィールド

トピックページが「ただのリンク集」にならないよう、各トピックに短い概要（description）を持たせる。

| 項目 | 仕様 |
| :--- | :--- |
| **フィールド** | `locales.{lang}.description` |
| **長さ** | 1〜2文（50〜150文字程度） |
| **用途** | トピックページのヘッダーに表示、`<meta description>` として使用 |
| **役割** | 「このトピックは何に関する集まりか」という最低限のコンテキスト提供 |

**例**:
```yaml
# config/topics.yml
game-theory:
  locales:
    ja:
      name: ゲーム理論
      description: 複数の意思決定者が相互に影響を与え合う状況を数学的に分析する理論。経済学、政治学、生物学など幅広い分野で応用される。
    en:
      name: Game Theory
      description: A mathematical framework for analyzing situations where multiple decision-makers influence each other.
  aliases:
    - 戦略的相互作用
```

> **terms.yml との棲み分け**: `terms.yml` は用語のインライン定義（Toggletip用、厳密な定義 + 直感的補助）、`topics.yml` の description はトピックページ用の短い概要。役割が異なるため、重複は許容する。

#### バリデーション

| ケース | 処理 |
| :--- | :--- |
| **正規ID マッチ** | ✅ 通過 |
| **alias マッチ** | 🔧 自動修正（正規IDに置換してコミット） |
| **未登録** | 🏷️ `new-topic` ラベル付与、レビュワーに通知 |

> **詳細**: Topics のスキーマ、命名規則、バリデーションフローの詳細は [NETWORK_ARCHITECTURE.md](./NETWORK_ARCHITECTURE.md) セクション2.3および5.3を参照。

### 5.2 Related Articles (Graph)

記事間の関連性は、以下の方法で定義・生成される。

#### 明示的関係（Semantic Layer）

概念間の論理関係は `config/concept-relations.yml` で一元管理される。

| 関係タイプ | 説明 | 例 |
| :--- | :--- | :--- |
| `opposes` | 対立・批判関係 | 天動説 ↔ 地動説 |
| `extends` | 拡張・一般化 | 特殊相対性理論 → 一般相対性理論 |
| `instance_of` | 具体例 | 囚人のジレンマ → ゲーム理論 |
| `requires` | 論理的前提 | 一般相対性理論 → リーマン幾何学 |

#### 暗黙的関係（自動計算）

| 方法 | 説明 |
| :--- | :--- |
| **共通Topics** | 同じ `topics` を持つ記事を関連記事として表示 |
| **リンク構造** | 被リンク数や相互リンクから関連度を計算 |
| **意味的近接性** | 将来的にベクトル化で自動計算（Phase 7以降） |

> **詳細**: 関連記事の定義方法、概念辞書型アプローチ、Relation Types の詳細は [NETWORK_ARCHITECTURE.md](./NETWORK_ARCHITECTURE.md) セクション2.2を参照。

---

## 6. 執筆者支援 (Authoring Support)

ディレクトリレスな検索性を補完し、特定のエディタ機能に依存しない執筆体験を提供する。

### 6.1 カタログインデックス (Catalog Index)
CI/CDプロセスにより、リポジトリルートに `CATALOG.md` を自動生成・更新する。

*   **Format**: 全記事のタイトル、Slug、物理パス、タグを網羅したMarkdownテーブル。
*   **Usage**: 執筆者は `CATALOG.md` を開いてキーワード検索することで、既存記事の有無や現在の配置場所を即座に把握できる。
*   **Benefit**: GitHubのWeb UIや、任意のテキストエディタで閲覧可能であり、専用ツールを必要としない。

### 6.2 記事作成CLI (Scaffolding Tool)

適切なディレクトリ配置とFrontmatter入力を支援するCLIツール（`pnpm new-article`）を提供する。

執筆者は「分野」を選択するだけで、引用形式に基づく適切なディレクトリにファイルが自動作成される。これにより、ディレクトリ構造を意識する必要がなくなる。

> **詳細**: CLIツールの仕様、対話フロー、設計原則については [CLI_TOOLS_SPEC.md](../../engine/technical/CLI_TOOLS_SPEC.md) を参照。

---
## 7. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [NETWORK_ARCHITECTURE.md](./NETWORK_ARCHITECTURE.md) | 記事間ネットワーク構造（3+1層モデル、学習パス、意味的関係） |
| [MARKDOWN_SPEC.md](./MARKDOWN_SPEC.md) | Frontmatterスキーマ、Directive記法 |
| [GOVERNANCE.md](../GOVERNANCE.md) | 出典管理、レビュープロセス |
