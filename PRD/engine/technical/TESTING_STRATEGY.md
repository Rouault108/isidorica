# テスト戦略 (Testing Strategy)

Isidoricaプロジェクトにおけるテストの方針、フレームワーク選定、品質目標を定義する。

**ステータス**: ✅ 完了
**作成日**: 2026-01-12
**更新日**: 2026-01-13
**関連ADR**: [ADR-007 Quality Assurance](./ARCHITECTURE_DECISION_RECORD.md#27-quality-assurance-qa-adr-007)

---

## 1. 目的

本ドキュメントでは、Isidoricaプロジェクトにおけるテストの**戦略・方針・基準**を定義する。

1. **品質基準の定義**: テストで達成すべき品質レベルを明確化
2. **一貫性の確保**: チーム全体で統一されたテストアプローチ
3. **意思決定の記録**: テスト戦略に関する決定と根拠

---

## 2. テスト哲学

### 2.1 基本原則

| 原則 | 説明 |
| :--- | :--- |
| **品質は後付けではない** | テストは開発プロセスに統合される。事後の品質保証ではなく、開発中に品質を作り込む |
| **テストピラミッド** | 高速なUnitテストを多く、遅いE2Eテストを少なく。コスト効率の良いテスト配分 |
| **アクセシビリティは必須** | A11yテストはオプションではなく必須要件（WCAG 2.2 AA準拠） |
| **継続的検証** | すべてのPRでテストを実行し、品質劣化を防ぐ |
| **失敗は早く** | 問題は開発サイクルの早い段階で検出する（Shift Left） |

### 2.2 テストピラミッド

```
           ┌─────────────────────┐
           │         E2E        │  少量：重要なユーザーフロー
           ├─────────────────────┤
           │     Integration    │  少〜中量：モジュール間結合
           ├─────────────────────┤
           │      Component     │  中量：UIコンポーネント
           ├─────────────────────┤
           │        Unit        │  多量：ロジック、ユーティリティ
           └─────────────────────┘
```

| 層 | 目的 | 実行速度 | 量 | 信頼性 |
| :--- | :--- | :--- | :--- | :--- |
| **Unit** | 個別関数・モジュールの正確性 | 高速（ミリ秒） | 多 | 高 |
| **Component** | UIコンポーネントの振る舞い・外観 | 中速（秒） | 中 | 中〜高 |
| **Integration** | モジュール間の連携 | 中速（秒） | 少〜中 | 中 |
| **E2E** | ユーザーシナリオの完遂 | 低速（数秒〜） | 少 | 低〜中 |

### 2.3 TDD/BDDの適用範囲

| 領域 | アプローチ | 理由 |
| :--- | :--- | :--- |
| **ユーティリティ関数** | TDD推奨 | 純粋関数、明確な入出力 |
| **Transformer** | TDD推奨 | AST変換ロジック、エッジケースが多い |
| **UIコンポーネント** | Storybook駆動 | ビジュアル確認しながら開発 |
| **E2E** | BDD | ユーザーシナリオベース |

### 2.4 テスト優先順位（リソース制約時）

時間やリソースが限られる場合の優先順位。

| 優先度 | テスト種別 | 理由 |
| :--- | :--- | :--- |
| **1（最優先）** | Unit Test | 高速、ROI最大、問題の早期発見 |
| **2** | Accessibility Test | 法的リスク、ユーザー影響大 |
| **3** | Integration Test | 結合障害の検出 |
| **4** | Security Test | 脆弱性検出 |
| **5** | E2E Test | 主要フローのみに絞る |
| **6** | Visual Regression | 差分検出（手動確認で代替可） |

> **注意**: この優先順位は緊急時の判断指針。通常はすべてのテストを実行する。

---

## 3. テスト分類

### 3.1 機能テスト

#### 3.1.1 Unit Test

| 項目 | 内容 |
| :--- | :--- |
| **ツール** | **Vitest** |
| **対象** | ユーティリティ関数、カスタムhooks、Transformer |
| **実行タイミング** | PRごと、pre-commit |
| **カバレッジ目標** | 90%以上 |

**選定理由**:
- Vite互換の高速テストランナー
- TypeScript対応、ESMネイティブ
- Jest互換API

**テスト対象の例**:
```typescript
// lib/utils/slug.ts のテスト
describe('generateSlug', () => {
  it('should convert Japanese to kebab-case', () => {
    expect(generateSlug('ゲーム理論')).toBe('game-theory');
  });
  
  it('should handle special characters', () => {
    expect(generateSlug('C++')).toBe('cpp');
  });
});
```

#### 3.1.2 Component Test

| 項目 | 内容 |
| :--- | :--- |
| **ツール** | **Storybook Interaction Testing** |
| **対象** | UIコンポーネント（UI、イベント） |
| **実行タイミング** | PRごと |

**選定理由**:
- 実ブラウザでWeb Componentsをレンダリング
- Testing Library代替として採用
- Shadow DOM内部も検証可能

> **参照**: [ADR-007](./ARCHITECTURE_DECISION_RECORD.md) - Testing Library不採用の理由

**テスト例**:
```typescript
// toggletip-element.stories.ts
export const Opens: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const toggletip = canvas.getByRole('button');
    
    await userEvent.click(toggletip);
    
    await expect(canvas.getByRole('tooltip')).toBeVisible();
  },
};
```

#### 3.1.3 Integration Test

| 項目 | 内容 |
| :--- | :--- |
| **ツール** | **Vitest** |
| **対象** | モジュール間の連携、ビルドパイプライン |
| **実行タイミング** | PRごと |

**テスト対象**:

| 対象 | 説明 |
| :--- | :--- |
| **Transformer連携** | Markdown → AST → HTML の変換パイプライン |
| **ビルドプロセス** | Velite → Astro の連携 |
| **データ整合性** | topics.yml と記事の `topics` フィールドの整合 |

**テスト例**:
```typescript
describe('Build Pipeline', () => {
  it('should transform markdown to correct HTML', async () => {
    const markdown = '# Title\n\n[[term]]';
    const result = await buildPipeline.transform(markdown);
    
    expect(result).toContain('<h1>Title</h1>');
    expect(result).toContain('<toggletip-element');
  });
});
```

#### 3.1.4 E2E Test

| 項目 | 内容 |
| :--- | :--- |
| **ツール** | **Playwright** |
| **対象** | 重要なユーザーフロー、ルーティング、検索 |
| **実行タイミング** | リリース前、main マージ後 |

**選定理由**:
- クロスブラウザ対応（Chromium, Firefox, WebKit）
- 高い安定性
- 並列実行

**テスト対象**:

| フロー | 説明 | 優先度 |
| :--- | :--- | :--- |
| **記事閲覧** | トップページ → 記事一覧 → 記事詳細 | 高 |
| **検索** | 検索入力 → 結果表示 → 記事遷移 | 高 |
| **Toggletip** | 用語クリック → ツールチップ表示 | 中 |
| **Laboratory** | コード実行 → 結果表示 | 中 |
| **学習パス** | パス選択 → 記事ナビゲーション | 中 |

**E2Eシナリオ例（Gherkin形式）**:

```gherkin
# tests/e2e/article-navigation.spec.ts

Feature: 記事閲覧
  読者が記事を閲覧できる

  Scenario: トップページから記事詳細への遷移
    Given トップページを開いている
    When 「ゲーム理論」カテゴリをクリック
    And 「ナッシュ均衡」記事をクリック
    Then 記事タイトルが「ナッシュ均衡」と表示される
    And 目次が表示される
    And Toggletipが機能する

  Scenario: 検索から記事への遷移
    Given トップページを開いている
    When 検索ボックスに「量子」と入力
    And 検索結果「量子もつれ」をクリック
    Then 記事タイトルが「量子もつれ」と表示される
```

> **注意**: Gherkin形式はドキュメント用。実際のテストはPlaywright APIで記述。

#### 3.1.5 Property-Based / Fuzz Testing

| 項目 | 内容 |
| :--- | :--- |
| **ツール** | **fast-check** |
| **対象** | IFMパーサー、Transformer、Markdown変換 |
| **実行タイミング** | PRごと |

**目的**:
- 開発者が想定しないエッジケースの検出
- パーサーのクラッシュ耐性検証
- 複雑なDirective構文への堅牢性確保

**テスト例**:
```typescript
import fc from 'fast-check';

describe('IFM Parser', () => {
  it('should not crash on any input', () => {
    fc.assert(
      fc.property(fc.string(), (input) => {
        // どんな入力でもクラッシュしない
        expect(() => parseIFM(input)).not.toThrow();
      })
    );
  });

  it('should preserve content in round-trip', () => {
    fc.assert(
      fc.property(fc.string(), (input) => {
        const parsed = parseIFM(input);
        const rendered = renderIFM(parsed);
        // パース可能な入力は往復で保持される
        if (isValidIFM(input)) {
          expect(rendered).toContain(extractContent(input));
        }
      })
    );
  });
});
```

**適用対象**:

| 対象 | 検証する性質 |
| :--- | :--- |
| **Toggletip構文** `[[...]]` | ネスト、特殊文字、超長文字列への耐性 |
| **Directive構文** `:::` | 不正な構文でクラッシュしない |
| **Frontmatter** | 不正なYAMLでも安全にエラー処理 |

> **参照**: [fast-check ドキュメント](https://fast-check.dev/)

### 3.2 非機能テスト

#### 3.2.1 Accessibility Test

| 項目 | 内容 |
| :--- | :--- |
| **ツール** | **axe-core** (Storybook addon)、**Lighthouse**、**eslint-plugin-lit-a11y** |
| **対象** | 全コンポーネント、全ページ |
| **実行タイミング** | PRごと |
| **基準** | WCAG 2.2 AA準拠 |

**検証項目**:

| 項目 | 検証方法 | 自動化 |
| :--- | :--- | :--- |
| **キーボード操作** | Storybook Interaction Testing | ✅ |
| **フォーカス管理** | Storybook Interaction Testing | ✅ |
| **ARIAラベル** | axe-core | ✅ |
| **コントラスト比** | axe-core | ✅ |
| **見出し構造** | axe-core | ✅ |
| **スクリーンリーダー互換性** | 手動検証 | ❌ |

> **スクリーンリーダーテストの自動化について**:
> スクリーンリーダーのUX検証は**完全な自動化が不可能**。理由：
> - スクリーンリーダー間（NVDA/VoiceOver/JAWS）で挙動が異なる
> - 音声出力の適切さはプログラムで判断できない
> - ARIA属性の存在はaxe-coreで自動検証済み（部分的カバー）

> **スクリーンリーダー手動検証のガイド**:
> - 主要ページ（トップ、記事詳細、検索）をNVDA/VoiceOverで操作
> - 見出しナビゲーション、Toggletip展開、Laboratory実行結果の読み上げを確認
> - リリース前に最低1回実施（将来的にチェックリスト化を検討）

> **参照**: [ADR-007](./ARCHITECTURE_DECISION_RECORD.md) - A11y テスト戦略

**Lintによる静的検証**:
```bash
# eslint-plugin-lit-a11y によるLitテンプレートのA11yチェック
pnpm lint:a11y
```

#### 3.2.2 Performance Test

| 項目 | 内容 |
| :--- | :--- |
| **ツール** | **Lighthouse CI** |
| **対象** | 代表的なページ |
| **実行タイミング** | PRごと |

**目標値**:

| 指標 | 目標 | PRブロック |
| :--- | :--- | :--- |
| **Performance** | 90以上 | ❌ 警告のみ |
| **Accessibility** | 100 | ✅ |
| **Best Practices** | 90以上 | ❌ 警告のみ |
| **SEO** | 90以上 | ❌ 警告のみ |

#### 3.2.3 Security Test

| 項目 | 内容 |
| :--- | :--- |
| **ツール** | **pnpm audit**、**gitleaks**、**CodeQL**、**CSP検証** |
| **対象** | 依存関係、コミット履歴、ソースコード、ビルド成果物 |
| **実行タイミング** | PRごと、pre-commit、週次 |

**検証項目**:

| 項目 | ツール | PRブロック |
| :--- | :--- | :--- |
| **依存関係脆弱性** | `pnpm audit` | high以上でブロック |
| **Secrets検出** | `gitleaks` | ✅ |
| **SAST（静的解析）** | `CodeQL` | high/critical でブロック |
| **CSPヘッダー** | カスタムスクリプト | ✅ |
| **外部リソース参照** | ビルド後検証 | ❌ 警告のみ |

**gitleaks 設定**:
- **実行タイミング**: pre-commit（husky）、PRごと（CI）
- **検出対象**: APIキー、トークン、パスワード、秘密鍵等
- **除外設定**: `.gitleaksignore` でテスト用のダミー値を除外

**CodeQL 設定**:
- **実行タイミング**: PRごと、週次スケジュール（main ブランチ）
- **対象言語**: JavaScript/TypeScript
- **検出対象**: XSS、インジェクション、安全でないデータフロー等
- **設定ファイル**: `.github/workflows/codeql.yml`
- **誤検知対応**: `.github/codeql/codeql-config.yml` で除外設定

**DAST（動的解析）を導入しない理由**:
- Isidoricaは**静的サイト（Astro SSG）** であり、動的なサーバーサイド処理がない
- 主要なセキュリティリスク（XSS等）はCSP検証とCodeQLでカバー
- **将来の再検討条件**: アカウント機能や動的APIを実装する場合（認証・認可やインジェクション攻撃のリスクが増加するため導入）

> **参照**: [NON_FUNCTIONAL_REQUIREMENTS.md](../NON_FUNCTIONAL_REQUIREMENTS.md) §5 セキュリティ要件

#### 3.2.4 Visual Regression Test

| 項目 | 内容 |
| :--- | :--- |
| **ツール** | **Storybook + Chromatic** |
| **対象** | UIコンポーネントの外観 |
| **実行タイミング** | PRごと |

**選定理由**:
- Storybookとの統合
- ブラウザ環境でのスナップショット
- 差分検出とレビューワークフロー

### 3.3 コンテンツ品質テスト

記事コンテンツ自体の品質検証。

#### 3.3.1 構造検証

| チェック | ツール | 対象 | PRブロック |
| :--- | :--- | :--- | :--- |
| **Frontmatter** | Zod | 必須フィールド、型 | ✅ |
| **見出し構造** | 内製バリデーションライブラリ | h1は1つ、順序 | ✅ |
| **リンク** | 内製バリデーションライブラリ | 内部リンク整合性 | ✅ |
| **Directive** | IFMパーサー | 構文エラー | ✅ |

> **参照**: [ADR-007](./ARCHITECTURE_DECISION_RECORD.md) - Content Quality

#### 3.3.2 文章品質

| チェック | ツール | 対象 | PRブロック |
| :--- | :--- | :--- | :--- |
| **表記ゆれ** | textlint | 用語統一 | ✅（自動修正） |
| **文体・構造** | textlint | 技術文書規約 | ❌ 警告のみ |
| **Isidorica固有ルール** | textlint (カスタム) | ひらく語、読者呼びかけ禁止等 | ✅ |

> **参照**: [ADR-007](./ARCHITECTURE_DECISION_RECORD.md) - Writing (Japanese) 

---

## 4. テスト命名規約

### 4.1 ファイル命名

| テスト種別 | パターン | 例 |
| :--- | :--- | :--- |
| **Unit Test** | `*.test.ts` | `slug.test.ts` |
| **Integration Test** | `*.integration.test.ts` | `build-pipeline.integration.test.ts` |
| **E2E Test** | `*.spec.ts` | `article-navigation.spec.ts` |
| **Story** | `*.stories.ts` | `toggletip-element.stories.ts` |

### 4.2 describe/it 命名

**パターン**: `describe('[対象]', () => { it('should [期待動作] when [条件]', ...) })`

```typescript
// ✅ 良い例
describe('generateSlug', () => {
  it('should return kebab-case when given Japanese text', () => { ... });
  it('should return empty string when given empty input', () => { ... });
  it('should throw error when given null', () => { ... });
});

// ❌ 悪い例
describe('slug tests', () => {
  it('works', () => { ... });
  it('test 2', () => { ... });
});
```

### 4.3 テストの構造（AAA パターン）

```typescript
it('should return formatted date when given valid timestamp', () => {
  // Arrange（準備）
  const timestamp = 1704067200000;
  
  // Act（実行）
  const result = formatDate(timestamp);
  
  // Assert（検証）
  expect(result).toBe('2024-01-01');
});
```

---

## 5. テストの組織化

### 5.1 コロケーション vs 分離

| 方式 | 適用先 | 理由 |
| :--- | :--- | :--- |
| **コロケーション**（同じディレクトリ） | Unit Test, Stories | テスト対象との関連が明確。ファイル移動時に追従しやすい |
| **分離**（`tests/` ディレクトリ） | E2E, Integration, Fixtures | 複数モジュールにまたがるため、特定のソースに紐づかない |

> **具体的なディレクトリ構成**: リポジトリルートの `README.md` を参照。

---

## 6. テスト品質要件

### 6.1 基本方針

**カバレッジは目標ではなく診断ツールである。**

高いカバレッジ数値を追求すると、意味のないテスト（アサーションのない実行、形式的なテスト）が増え、技術的負債となる。代わりに、意味のあるテストを定性的に定義する。

### 6.2 テストの必須条件

| 対象 | 要件 |
| :--- | :--- |
| **公開関数** | すべての公開関数（export）に最低1つのテスト |
| **エッジケース** | 実装時に識別したエッジケースを網羅 |
| **Transformer** | サポートする各構文に対するテスト（入出力例をドキュメント化） |
| **バグ修正** | 修正したバグに対するリグレッションテスト |
| **UIコンポーネント** | Storybookに登録（インタラクションテスト推奨） |
| **E2E** | 主要ユーザーフローのスモークテスト |

### 6.3 カバレッジの扱い

| 方針 | 内容 |
| :--- | :--- |
| **可視化** | PRにカバレッジレポートを表示（診断情報として） |
| **警告** | 大幅な低下（5%以上）はPRコメントで警告 |
| **ブロッキング** | 特定の数値をブロッキング基準としない |
| **レビュー** | レビュアーがカバレッジ低下の妥当性を判断 |

> **なぜ数値目標を設定しないか**:
> - Goodhartの法則: 「指標が目標になると、良い指標ではなくなる」
> - カバレッジ≠品質: 100%カバレッジでもアサーションが不十分ならバグは防げない
> - 意味のないテストの増加を防ぐ

### 6.4 カバレッジレポート

| 項目 | 内容 |
| :--- | :--- |
| **生成タイミング** | PRごと（CIで自動生成） |
| **フォーマット** | HTML（Vitest UI） |
| **保存先** | GitHub Actions Artifacts（30日保持） |
| **閲覧方法** | PRコメントにリンク自動投稿 |

### 6.5 カバレッジ対象外

以下はカバレッジ計算から除外（`vitest.config.ts` で設定）：

| 除外対象 | 理由 |
| :--- | :--- |
| **設定ファイル** (`*.config.ts`) | テスト不要 |
| **型定義ファイル** (`*.d.ts`) | テスト不要 |
| **Storyファイル** (`*.stories.ts`) | テストそのもの |
| **モックファイル** (`__mocks__/*`) | テストそのもの |
| **E2Eテスト** (`tests/e2e/*`) | カバレッジ対象外 |

---

## 7. テストデータ管理

### 7.1 フィクスチャ

| 種類 | 配置 | 用途 |
| :--- | :--- | :--- |
| **記事データ** | `tests/fixtures/articles/` | Markdown記事のサンプル |
| **概念データ** | `tests/fixtures/topics/` | topics.yml のサンプル |
| **学習パス** | `tests/fixtures/learning-paths/` | learning-paths.yml のサンプル |

### 7.2 テストダブル戦略

| 種類 | 用途 | いつ使うか |
| :--- | :--- | :--- |
| **Stub** | 固定値を返す | 外部依存の置き換え |
| **Mock** | 呼び出しを検証 | 副作用の検証が必要なとき |
| **Spy** | 元の実装を保持しつつ監視 | 呼び出し回数の検証 |
| **Fake** | 簡略化した実装 | ファイルシステム（memfs）等 |

**判断基準**:
- **モックを使う**: 外部API、ファイルI/O、時間依存
- **モックを避ける**: 純粋関数、小さなユーティリティ

### 7.3 モック対象と方法

| 対象 | モック方法 |
| :--- | :--- |
| **ファイルシステム** | memfs または Vitest の `vi.mock` |
| **外部API** | MSW (Mock Service Worker) |
| **ブラウザAPI** | Vitest browser mode |
| **日時** | Vitest の `vi.useFakeTimers` |

### 7.4 テスト環境の差異管理

ローカル開発環境とCI環境の差異を認識し、適切に対処する。

| 項目 | ローカル (開発者) | CI (GitHub Actions) |
| :--- | :--- | :--- |
| **OS** | Windows 10/11 | Ubuntu 22.04 (`ubuntu-latest`) |
| **Node.js** | 開発者依存 | `package.json` の `engines` で固定 |
| **ブラウザ** | 開発者環境のブラウザ | Headless (Playwright管理) |
| **タイムゾーン** | JST (UTC+9) | UTC |
| **ファイルパス** | `\` (バックスラッシュ) | `/` (スラッシュ) |

**差異による問題の防止策**:

| 問題 | 対策 |
| :--- | :--- |
| **パス区切り文字** | `path.join()` / `path.resolve()` を使用。ハードコードしない |
| **タイムゾーン依存** | `vi.useFakeTimers` で固定、またはUTC基準で記述 |
| **OS固有機能** | 使用を避ける。必要な場合は条件分岐 |
| **改行コード** | Git設定（`.gitattributes`）で `text=auto` |
| **環境変数** | `.env.test` で統一、CI secrets で管理 |

> **方針**: CIで通過したテストはローカルでも通過すべき。差異を最小化する。

---

## 8. テストのメンテナンス

### 8.1 テスト更新の責任

| 変更種別 | テスト更新責任 |
| :--- | :--- |
| **機能追加** | 機能実装者がテストも追加 |
| **バグ修正** | 修正者がリグレッションテストを追加 |
| **リファクタリング** | リファクタリング者がテストを更新 |
| **依存関係更新** | 更新者がテスト通過を確認 |

### 8.2 テスト削除の基準

テストを削除しても良いケース：

| ケース | 条件 |
| :--- | :--- |
| **対象コードの削除** | 対象機能が削除された場合 |
| **重複テスト** | 同じ振る舞いを検証する別のテストがある場合 |
| **Flaky Test** | 1週間以内に修正できない場合（Issue必須） |
| **過度に脆いテスト** | 実装詳細に依存し、頻繁に壊れる場合 |

> **禁止**: カバレッジを下げるためだけの削除

### 8.3 テストコードレビューのポイント

PRレビュー時にテストコードを確認する観点：

| 観点 | チェック項目 |
| :--- | :--- |
| **網羅性** | エッジケース、境界値をテストしているか |
| **独立性** | 他のテストに依存していないか |
| **可読性** | テスト名から何をテストしているかわかるか |
| **保守性** | 実装詳細ではなく振る舞いをテストしているか |
| **速度** | 不必要に遅いテストになっていないか |

---

## 9. 失敗時の対応

### 9.1 テスト失敗フロー

```
テスト失敗
    ↓
┌─────────────────────────────────────────┐
│ 1. エラーメッセージを確認               │
│ 2. 失敗したテストのみを再実行           │
│    pnpm test -- --filter="テスト名"    │
│ 3. デバッグモードで実行                 │
│    pnpm test:debug                     │
│ 4. 原因を特定して修正                   │
│ 5. 全テストを再実行して確認             │
└─────────────────────────────────────────┘
```

### 9.2 Flaky Test 対策

**Flaky Test**: 同じコードで成功したり失敗したりする不安定なテスト

| 対策 | 説明 |
| :--- | :--- |
| **検出** | CIで3回リトライ、2回以上失敗したらFlaky判定 |
| **報告** | Flaky Testは即座にIssue作成 |
| **対応期限** | 1週間以内に修正または削除 |
| **一時的スキップ** | `it.skip` + TODO コメント + Issue番号 |

**Flaky Testの主な原因と対策**:

| 原因 | 対策 |
| :--- | :--- |
| **タイミング依存** | 明示的な待機（`waitFor`）、Fake Timers |
| **テスト間の依存** | テストの独立性確保、セットアップ/クリーンアップ |
| **ランダム性** | シード値の固定 |
| **外部依存** | モック化 |

### 9.3 デバッグ方法

```bash
# 特定のテストのみ実行
pnpm test -- --filter="generateSlug"

# デバッグモード（VSCode連携）
pnpm test:debug

# 詳細ログ出力
pnpm test -- --reporter=verbose

# Playwrightデバッグ
pnpm test:e2e -- --debug
```

---

## 10. ローカル開発

### 10.1 コマンド

```bash
# Unit Test
pnpm test              # 全テスト実行
pnpm test:watch        # ウォッチモード
pnpm test:coverage     # カバレッジ付き
pnpm test:debug        # デバッグモード

# Component Test
pnpm storybook         # Storybook起動
pnpm test:storybook    # Storybook Interaction Test

# E2E Test
pnpm test:e2e          # Playwright実行
pnpm test:e2e:debug    # Playwrightデバッグモード

# Lint
pnpm lint              # Biome lint
pnpm lint:a11y         # A11y lint (ESLint)

# All Checks
pnpm check             # 全チェック実行（CI相当）
```

### 10.2 Pre-commit Hook

`husky` + `lint-staged` で設定。

| フック | 内容 | スキップ方法 |
| :--- | :--- | :--- |
| **pre-commit** | Format, Lint, Type Check | `git commit --no-verify` |
| **pre-push** | Unit Test | `git push --no-verify` |

> **注意**: `--no-verify` は緊急時のみ使用。CIで必ず検出される。

---

## 11. ツールスタック

| カテゴリ | ツール | 用途 |
| :--- | :--- | :--- |
| **Unit Test** | Vitest | ロジック、Transformer |
| **Property-Based Test** | fast-check | パーサー堅牢性、エッジケース検出 |
| **Component Test** | Storybook Interaction Testing | UI、イベント |
| **Integration Test** | Vitest | モジュール連携 |
| **Visual Regression** | Chromatic | スナップショット比較 |
| **E2E Test** | Playwright | ユーザーフロー |
| **Accessibility** | axe-core, eslint-plugin-lit-a11y | A11y検証 |
| **Performance** | Lighthouse CI | パフォーマンス計測 |
| **Security** | pnpm audit, gitleaks, CodeQL | SCA、Secrets検出、SAST |
| **Content Quality** | textlint | 日本語品質 |
| **Content Structure** | 内製バリデーションライブラリ | 構造検証 |

### 11.1 ツール選定の原則

ツールの永続性と代替可能性を考慮し、以下の原則に従う。

| 原則 | 説明 |
| :--- | :--- |
| **標準API優先** | Jest互換API（Vitest）、WebDriver/CDP互換（Playwright）など、業界標準に準拠 |
| **ロックイン回避** | テストロジックとツールAPIを分離。シナリオ記述は可能な限り汎用形式で |
| **代替可能性** | 単一ベンダー依存を避ける（例: CodeQLはSemgrepで代替可能） |
| **保守状況の監視** | 年次でツールの保守状況を確認。非推奨化の兆候があれば移行計画を策定 |

**ツール別のリスク評価**:

| リスク | ツール | 軽減策 |
| :--- | :--- | :--- |
| **低** | Vitest, axe-core, Playwright | Jest/Cypress等への移行が容易 |
| **中** | Storybook, CodeQL | エコシステム依存。代替にコストあり |
| **低** | fast-check, textlint | 概念自体は汎用。テストロジックは再利用可能 |

---

## 12. 今後の課題

| 項目 | 説明 | 優先度 |
| :--- | :--- | :--- |
| **Mutation Testing** | テストの品質検証（Stryker等） | 低 |
| **Contract Testing** | API契約テスト（将来的にAPIを公開する場合） | 低 |
| **Load Testing** | Laboratory の同時実行負荷 | 低 |
| **A11y自動手動ハイブリッド** | スクリーンリーダー検証のパターン化 | 中 |
| **数値カバレッジ目標の導入** | チーム拡大時、レビュー判断の標準化のため最低ライン（例: 80%）を検討 | 低 |

---

## 13. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [ARCHITECTURE_DECISION_RECORD.md](./ARCHITECTURE_DECISION_RECORD.md) | ADR-007 QA |
| [CI_CD_PIPELINE.md](./CI_CD_PIPELINE.md) | CI/CD統合（実行詳細） |
| [NON_FUNCTIONAL_REQUIREMENTS.md](../NON_FUNCTIONAL_REQUIREMENTS.md) | 品質基準 |
| [CONTRIBUTING.md](../CONTRIBUTING.md) | 開発者ガイド |
