# Toggletips 仕様 (Toggletips Specification)

`[[Term]]` 記法によるインライン参照機能の設計仕様を定義する。

**ステータス**: 📋 ドラフト
**作成日**: 2026-01-12
**理論的基盤**: [DESIGN_PRINCIPLES.md](../shared/DESIGN_PRINCIPLES.md) B-1（暗黙知明示化原則）

---

## 1. 概要

### 1.1 目的

Toggletipsは、記事本文中に登場する用語・概念に対して、**文脈を離れずに**簡潔な定義を提供する機能である。

| 機能 | 説明 |
| :--- | :--- |
| **限界コード表示** | 用語にホバー/タップで短い定義を表示 |
| **精密コード展開** | 「詳細」クリックでより詳しい説明を表示 |
| **記事リンク** | 関連記事がある場合はリンクを表示 |

### 1.2 理論的背景

Bernsteinのコード理論（[CORE_FOUNDATIONS.md](../shared/CORE_FOUNDATIONS.md) §2.3）に基づく。

| コード | 特性 | Toggletipsでの用途 |
| :--- | :--- | :--- |
| **限界コード (Restricted Code)** | 簡潔、文脈依存、共有知識を前提 | ホバー/タップ時の初期表示 |
| **精密コード (Elaborated Code)** | 詳細、文脈独立、明示的 | 「詳細」展開後の表示 |

> **設計原則**: B-1（暗黙知・学習ルール明示化原則）を実現する機能。

### 1.3 Topics との関係

Toggletipsと概念タグ（Topics）は**異なるシステム**である。

| システム | 目的 | データソース | 粒度 |
| :--- | :--- | :--- | :--- |
| **Topics** | 記事の分類・ナビゲーション | `config/topics.yml` | 分野レベル（game-theory, economics） |
| **Toggletips** | 用語の即時参照 | `config/topics.yml` + 記事 | 用語レベル（ナッシュ均衡, 量子もつれ） |

**統合ポイント**: Toggletipsは `topics.yml` で定義された概念を参照可能。ただし、すべてのTopicがToggletipとして表示されるわけではない。

---

## 2. 記法仕様

### 2.1 基本記法

```markdown
[[用語ID]]
```

**例**:
```markdown
ゲーム理論において、[[nash-equilibrium]]は各プレイヤーが
最適戦略を取っている状態を指す。
```

### 2.2 表示名のカスタマイズ

```markdown
[[ID|表示テキスト]]
```

**例**:
```markdown
[[nash-equilibrium|均衡点]]においては、どのプレイヤーも
単独で戦略を変更するインセンティブを持たない。
```

### 2.3 言語指定（多言語コンテンツ向け）

```markdown
[[ID:lang]]
```

**例**:
```markdown
[[game-theory:en]] is a mathematical framework...
```

---

## 3. コンテンツ定義

### 3.1 データソース

Toggletipsのコンテンツは以下の優先順位で取得される。

| 優先順位 | ソース | 条件 |
| :--- | :--- | :--- |
| 1 | `config/topics.yml` | `restricted`/`elaborated` が定義されている場合 |
| 2 | 記事 | 当該IDを `slug` または `topics[0]` に持つ記事が存在する場合 |
| 3 | フォールバック | 上記いずれも該当しない場合 |

### 3.2 topics.yml での定義

```yaml
# config/topics.yml

# 最小限（Toggletip定義なし → 記事から導出）
game-theory:
  ja: ゲーム理論
  en: Game Theory
  aliases: [げーむりろん]

# Toggletip定義あり
nash-equilibrium:
  ja:
    name: ナッシュ均衡
    restricted: 各プレイヤーが最適戦略を取っている均衡状態
    elaborated: |
      非協力ゲームにおいて、すべてのプレイヤーが互いの戦略を
      所与として自身の戦略を最適化している状態。
      いかなるプレイヤーも単独で戦略を変更することで
      利得を改善することができない。
  en:
    name: Nash Equilibrium
    restricted: A state where each player is playing their best response
  aliases: [ナッシュのきんこう]
```

### 3.3 記事からの導出

`topics.yml` に `restricted`/`elaborated` が定義されていない場合、記事から導出する。

| フィールド | 導出元 |
| :--- | :--- |
| **限界コード** | 記事の `description` フィールド（Frontmatter） |
| **精密コード** | 記事の冒頭セクション（最初の見出し前、または最初の段落。最大200字） |

**導出対象の記事**: 当該IDを以下のいずれかに持つ記事
- `slug` フィールド
- `topics` 配列の先頭

### 3.4 フォールバック

記事も定義も存在しない場合：

| フィールド | フォールバック値 |
| :--- | :--- |
| **表示名** | ID をタイトルケース化（`nash-equilibrium` → `Nash Equilibrium`） |
| **限界コード** | 表示しない（名前のみ表示） |
| **精密コード** | 表示しない |

---

## 4. UI仕様

### 4.1 表示トリガー

| デバイス | トリガー |
| :--- | :--- |
| **デスクトップ** | ホバー（マウスオーバー）またはクリック |
| **モバイル** | タップ |

### 4.2 表示コンテンツ（初期状態）

```
┌─────────────────────────────────────┐
│ ナッシュ均衡                         │  ← タイトル（name）
│                                     │
│ 各プレイヤーが最適戦略を取っている    │  ← 限界コード（restricted）
│ 均衡状態                            │
│                                     │
│ [詳細を見る] [記事を読む]            │  ← アクションリンク
└─────────────────────────────────────┘
```

### 4.3 詳細展開後

```
┌─────────────────────────────────────┐
│ ナッシュ均衡                         │
│                                     │
│ 非協力ゲームにおいて、すべての       │  ← 精密コード（elaborated）
│ プレイヤーが互いの戦略を所与として    │
│ 自身の戦略を最適化している状態。     │
│ いかなるプレイヤーも単独で戦略を     │
│ 変更することで利得を改善すること     │
│ ができない。                        │
│                                     │
│ [記事を読む]                        │
└─────────────────────────────────────┘
```

### 4.4 リンク表示条件

| リンク | 表示条件 |
| :--- | :--- |
| **詳細を見る** | `elaborated` が定義されている場合 |
| **記事を読む** | 当該IDに対応する記事が存在する場合 |

### 4.5 アクセシビリティ

| 要件 | 実装 |
| :--- | :--- |
| **キーボードナビゲーション** | Tab でフォーカス、Enter/Space で展開 |
| **スクリーンリーダー** | `role="tooltip"`, `aria-describedby` |
| **フォーカス管理** | 閉じた後に元の要素にフォーカスを戻す |
| **Escape キー** | Tooltip を閉じる |

> **参照**: WCAG 2.2 AA準拠。詳細は [NON_FUNCTIONAL_REQUIREMENTS.md](./NON_FUNCTIONAL_REQUIREMENTS.md) を参照。

---

## 5. パース処理

### 5.1 処理フロー

```
Markdownソース
    ↓
[[Term]] パターン検出（remark プラグイン）
    ↓
topics.yml / 記事から定義取得
    ↓
<toggletip-element> Web Component に変換
    ↓
HTML出力
```

### 5.2 未定義の用語

`[[Term]]` が以下のいずれにも該当しない場合：

1. `topics.yml` に存在しない
2. 該当する記事が存在しない

**処理**:
- ビルド時に警告を出力
- 出力HTMLでは通常テキストとして表示（リンクなし）
- `data-toggletip-missing="true"` 属性を付与（開発時のデバッグ用）

---

## 6. 実装コンポーネント

### 6.1 Web Component

```html
<toggletip-element
  term-id="nash-equilibrium"
  lang="ja"
  aria-describedby="toggletip-nash-equilibrium"
>
  ナッシュ均衡
</toggletip-element>
```

### 6.2 Lit コンポーネント（概要）

```typescript
@customElement('toggletip-element')
export class ToggletipElement extends LitElement {
  @property() termId: string = '';
  @property() lang: string = 'ja';
  @state() isOpen: boolean = false;
  @state() isExpanded: boolean = false;
  
  // ... 実装詳細は Phase 6 で策定
}
```

---

## 7. パフォーマンス考慮

| 考慮点 | 対策 |
| :--- | :--- |
| **初期ロード** | Tooltip コンテンツは遅延ロード（ホバー/タップ時に取得） |
| **キャッシュ** | 一度取得した定義はセッション内でキャッシュ |
| **バンドルサイズ** | Toggletip コンポーネントはコード分割 |

---

## 8. バリデーション

### 8.1 ビルド時チェック

| チェック項目 | エラー時の動作 |
| :--- | :--- |
| `[[Term]]` が `topics.yml` または記事に存在するか | 警告（ビルドは継続） |
| `restricted` が定義されているが `name` がない | エラー |
| `elaborated` に `[[` が含まれていないか | 警告（無限再帰防止） |

### 8.2 CI/CD 統合

```yaml
# .github/workflows/validate.yml
- name: Validate Toggletips
  run: pnpm run validate:toggletips
```

---

## 9. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [NETWORK_ARCHITECTURE.md](../library/technical/NETWORK_ARCHITECTURE.md) | topics.yml スキーマ定義 |
| [MARKDOWN_SPEC.md](../library/technical/MARKDOWN_SPEC.md) | Markdown記法仕様 |
| [IFM_PARSER_SPEC.md](./technical/IFM_PARSER_SPEC.md) | パーサー処理仕様 |
| [DESIGN_PRINCIPLES.md](../shared/DESIGN_PRINCIPLES.md) | B-1（暗黙知明示化原則） |
| [CORE_FOUNDATIONS.md](../shared/CORE_FOUNDATIONS.md) | Bernsteinコード理論 |

---

## 10. 今後の課題

| 項目 | 説明 | 優先度 |
| :--- | :--- | :--- |
| **相互参照検出** | 記事内で未定義の用語を自動検出し、Toggletip候補として提案 | 低 |
| **AI支援** | LLMによる限界/精密コードの自動生成支援 | 低 |
| **多言語同期** | 言語間での定義の同期・翻訳支援 | 中 |
