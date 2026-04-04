# Isidorica Diagram Library Specification

Isidorica向けのカスタムダイアグラムライブラリの設計仕様。
Mermaid/D2に代わり、Isidoricaのデザインシステムと完全に統合された静的ダイアグラムを提供する。

**ステータス**: 🔄 ドラフト

---

## 1. 概要

### 1.1 背景

- 既存ツール（Mermaid, D2）ではIsidoricaのデザインコンセプトとの統合が困難
- 完全なデザインコントロールと、アクセシビリティ（WCAG 2.2 AA）対応が必要
- Lit Web Componentsとして実装し、Isidorica-Engineに統合

### 1.2 設計方針

| 項目 | 方針 |
| :--- | :--- |
| **リポジトリ** | 別リポジトリとして開発（npm公開可能な汎用ライブラリ） |
| **技術スタック** | Lit Web Components + TypeScript |
| **出力形式** | SVG（スケーラブル、アクセシビリティ対応） |
| **デザイン** | Isidoricaデザインシステム準拠（デザイントークン使用） |
| **アクセシビリティ** | WCAG 2.2 AA必須 |

### 1.3 Laboratoryとの役割分担

| 特性 | Diagram | Laboratory |
| :--- | :--- | :--- |
| **目的** | 構造・関係の可視化 | 能動的推論の促進（予測誤差生成） |
| **インタラクション** | 閲覧のみ（ホバー、ズーム、パン） | 操作可能（パラメータ調整、実行） |
| **データ変更** | ❌ 不可（静的） | ✅ 可（動的） |
| **計算実行** | ❌ なし | ✅ あり（WASM/Pyodide） |
| **状態管理** | Stateless | Stateful |

> **設計根拠**: LEARNING_FOUNDATIONS.mdに基づき、静的な構造理解と能動的な予測誤差生成を明確に分離する。

---

## 2. サポートするダイアグラム種類

### 2.1 Phase 1（高優先度）

| 種類 | ID | 用途 |
| :--- | :--- | :--- |
| **フローチャート** | `flowchart` | 手順、アルゴリズム、意思決定フロー |
| **概念マップ** | `concept-map` | 概念間の関係、記事ネットワークの可視化 |

### 2.2 Phase 2（中優先度）

| 種類 | ID | 用途 |
| :--- | :--- | :--- |
| **シーケンス図** | `sequence` | プロセス間のやり取り、時系列 |
| **マインドマップ** | `mindmap` | 階層的な概念整理 |

### 2.3 Phase 3（低優先度）

| 種類 | ID | 用途 |
| :--- | :--- | :--- |
| **ER図** | `er` | データモデル、関係性 |
| **クラス図** | `class` | オブジェクト指向設計 |
| **状態遷移図** | `state` | 状態マシン |
| **ガントチャート** | `gantt` | スケジュール、タイムライン |

---

## 3. 記法仕様

### 3.1 基本構文

D2風の簡潔な記法を採用する。

```
::: diagram {type="flowchart" alt="処理フロー"}
node-id: ラベル
node-id2: ラベル2

node-id -> node-id2: 矢印ラベル
:::
```

### 3.2 フローチャート

```
::: diagram {type="flowchart" alt="ナッシュ均衡の計算フロー"}
start: 開始 {shape: circle}
input: 利得行列を入力 {shape: rect}
calculate: 最適応答を計算 {shape: rect}
check: 均衡点？ {shape: diamond}
output: 結果出力 {shape: rect}
end: 終了 {shape: circle}

start -> input -> calculate -> check
check -> output: yes
check -> calculate: no {style: dashed}
output -> end
:::
```

#### ノード属性

| 属性 | 値 | 説明 |
| :--- | :--- | :--- |
| `shape` | `rect`, `circle`, `diamond`, `parallelogram` | ノード形状 |
| `style` | `filled`, `outline` | スタイル |
| `color` | デザイントークン名 | 色指定 |

#### エッジ属性

| 記法 | 意味 |
| :--- | :--- |
| `->` | 通常の矢印 |
| `-->` | 破線矢印 |
| `<->` | 双方向矢印 |
| `--` | 線（矢印なし） |

### 3.3 概念マップ

```
::: diagram {type="concept-map" alt="ゲーム理論の概念関係"}
game-theory: ゲーム理論 {central: true}
nash: ナッシュ均衡
pareto: パレート最適
prisoner: 囚人のジレンマ

game-theory -> nash: 中心概念
game-theory -> pareto: 効率性基準
nash -> prisoner: 代表例
:::
```

### 3.4 シーケンス図

```
::: diagram {type="sequence" alt="リクエスト処理フロー"}
participant client: クライアント
participant server: サーバー
participant db: データベース

client -> server: リクエスト
server -> db: クエリ
db --> server: 結果
server --> client: レスポンス
:::
```

---

## 4. アクセシビリティ

### 4.1 必須要件

| 要件 | 実装 |
| :--- | :--- |
| **代替テキスト** | `alt` 属性必須。SVGに `<title>`, `<desc>` として埋め込み |
| **キーボード操作** | フォーカス可能、矢印キーでパン、+/-でズーム |
| **スクリーンリーダー** | `role="img"` + `aria-label`、ノード情報を構造化テキストで提供 |
| **色のコントラスト** | WCAG 2.2 AA準拠（4.5:1以上） |
| **モーション** | `prefers-reduced-motion` 対応 |

### 4.2 SVG出力例

```html
<figure class="isidorica-diagram" role="img" aria-label="ナッシュ均衡の計算フロー">
  <svg viewBox="0 0 800 600">
    <title>ナッシュ均衡の計算フロー</title>
    <desc>開始から利得行列入力、最適応答計算、均衡点判定、結果出力までのフローチャート</desc>
    <!-- SVGコンテンツ -->
  </svg>
  <details class="diagram-text-alt">
    <summary>テキスト版を表示</summary>
    <ol>
      <li>開始</li>
      <li>利得行列を入力</li>
      <li>最適応答を計算</li>
      <li>均衡点が存在するか判定</li>
      <li>存在する場合: 結果出力</li>
      <li>存在しない場合: 計算に戻る</li>
      <li>終了</li>
    </ol>
  </details>
</figure>
```

---

## 5. デザイン仕様

### 5.1 デザイン目標

- **洗練されたミニマリズム**: Isidoricaのデザイン思想に準拠
- **学術的な品格**: 過剰な装飾を避け、情報を明確に伝える
- **ダークモード完全対応**: CSS変数でテーマ切り替え

### 5.2 デザイントークン

```css
:root {
  /* ダイアグラム専用トークン */
  --diagram-bg: var(--color-surface);
  --diagram-node-bg: var(--color-surface-elevated);
  --diagram-node-border: var(--color-border);
  --diagram-edge: var(--color-text-secondary);
  --diagram-edge-arrow: var(--color-text-primary);
  --diagram-label: var(--color-text-primary);
  --diagram-highlight: var(--color-accent);
}

[data-theme="dark"] {
  --diagram-bg: var(--color-surface-dark);
  /* ... */
}
```

### 5.3 視覚スタイル

| 要素 | スタイル |
| :--- | :--- |
| **ノード** | 大きめの角丸（8px）、微細なシャドウ |
| **エッジ** | 2px線、スムーズなベジェ曲線 |
| **ラベル** | システムフォント、適切なサイズ（14px-16px） |
| **背景** | オプションでグリッド背景 |

---

## 6. 技術アーキテクチャ

### 6.1 パッケージ構成

```
@isidorica/diagram-lib
├── packages/
│   ├── core/           # パーサー・AST・レイアウトエンジン
│   ├── renderer-svg/   # SVGレンダラー
│   ├── lit-component/  # Lit Web Component
│   └── themes/         # テーマ定義
├── docs/               # ドキュメントサイト
└── examples/           # サンプル集
```

### 6.2 処理パイプライン

```
Markdown Source
    ↓
IFM Parser (diagram directive検出)
    ↓
Diagram Parser (テキスト → AST)
    ↓
Layout Engine (ノード配置計算)
    ↓
SVG Renderer (AST → SVG)
    ↓
Lit Web Component (DOM埋め込み)
```

### 6.3 レイアウトエンジン

| ダイアグラム種類 | レイアウトアルゴリズム |
| :--- | :--- |
| フローチャート | Dagre（階層レイアウト） |
| 概念マップ | Force-directed |
| シーケンス図 | カスタム（時系列） |

---

## 7. Laboratory連携

### 7.1 遷移ボタン

`laboratory` 属性が指定された場合、ダイアグラム下部に「操作する」ボタンを表示する。

```markdown
::: diagram {type="flowchart" alt="..." laboratory="nash-equilibrium-sim"}
...
:::
```

**レンダリング結果**:
```html
<figure class="isidorica-diagram">
  <svg>...</svg>
  <a href="#laboratory-nash-equilibrium-sim" class="diagram-lab-link">
    <span class="icon">🔬</span>
    実際に操作する
  </a>
</figure>
```

---

## 8. 開発ロードマップ

| Phase | 内容 | 時期 |
| :--- | :--- | :--- |
| **Phase 3** | 仕様策定完了、リポジトリ作成 | PRD Phase 3 |
| **Phase 4.1** | パーサー・AST基盤 | PRD Phase 4 |
| **Phase 4.2** | フローチャート実装 | PRD Phase 4 |
| **Phase 4.3** | 概念マップ実装 | PRD Phase 4 |
| **Phase 4.4** | Isidorica-Engine統合 | PRD Phase 4 |
| **Phase 5** | 追加ダイアグラム種類 | 公開後 |

---

## 9. 関連ドキュメント

| ドキュメント | 関連箇所 |
| :--- | :--- |
| [MARKDOWN_SPEC.md](../../library/technical/MARKDOWN_SPEC.md) | 4.7節 Diagram Directive |
| [LABORATORY_DESIGN.md](../LABORATORY_DESIGN.md) | Laboratory機能との役割分担 |
| [LEARNING_FOUNDATIONS.md](../../shared/LEARNING_FOUNDATIONS.md) | 静的理解 vs 能動的推論の設計根拠 |
| [GEMINI.md](../../../../.gemini/GEMINI.md) | デザイン原則、アクセシビリティ要件 |
