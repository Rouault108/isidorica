# 翻訳設計の理論的基盤 (Translation Foundations)

## 序論

### 本文書の位置づけ

本文書は、Isidoricaの翻訳設計が依拠する**翻訳学・対照言語学の理論的基盤**を解説する。

翻訳等価論、翻訳規範論、対照言語学、ポストコロニアル翻訳論等の諸理論は、多言語プラットフォームとしてのIsidoricaの翻訳戦略の根拠を提供する。本文書は理論の**カタログ**として機能し、各理論のOrigin（起源）、Core Mechanism（核心メカニズム）、Criticism（批判・限界）を一貫したフォーマットで提示する。

**関連文書**:
- 共通理論基盤: [CORE_FOUNDATIONS.md](./CORE_FOUNDATIONS.md)
- 設計適用: [TRANSLATION_DESIGN.md](../library/TRANSLATION_DESIGN.md)
- 辞書設計基盤: [CONTENT_FOUNDATIONS.md](./CONTENT_FOUNDATIONS.md)
- 学習理論: [LEARNING_FOUNDATIONS.md](./LEARNING_FOUNDATIONS.md)

### 本文書の構成

本文書は、翻訳設計に関連する理論を**8つの分野**に組織化し、付録として他文書との接続・参考資料を提供する。

1. **翻訳等価論（§1）**: 翻訳の「目標」を定義する — 形式的等価、動的等価、スコポス理論
2. **翻訳不可能性と対処（§2）**: 言語固有性への対応 — 借用、説明的翻訳、適応
3. **翻訳規範論（§3）**: 翻訳の社会的規範 — Touryの規範理論、妥当性と受容可能性
4. **対照言語学（§4）**: 言語間の構造的差異 — 語彙類型論、文法カテゴリの相違
5. **自然意味論メタ言語（§5）**: 言語横断的な意味定義 — Wierzbickaの意味プリミティブ
6. **機械翻訳と人間翻訳（§6）**: 技術活用 — NMT、LLM、ポストエディット、翻訳記憶
7. **翻訳批評・翻訳倫理（§7）**: 品質評価と透明性 — 翻訳の不可視性、MQM、書き換え論
8. **ポストコロニアル翻訳論（§8）**: 権力と翻訳 — 英語中心主義批判

**付録**:
- 他の理論基盤との接続（§9）
- スキーマへの反映（§10）
- 年表（§11）
- 推奨文献（§12）

### 統合的視座

本文書の諸理論は、**スコポス理論**を統合的な軸として組織化される。

> **スコポス原則**: 翻訳は目的によって決まる。

Isidoricaのスコポスは「**読者が概念を理解し、能力を獲得すること**」である。このスコポスが、等価の種類の選択（§1）、翻訳不可能性への対処戦略（§2）、翻訳規範の設計（§3）を規定する。

### 分野別インデックス

| 分野 | セクション |
|:---|:---|
| **1 翻訳等価論** | 1.1 形式的等価と動的等価, 1.2 Kollerの等価類型, 1.3 スコポス理論, 1.4 等価概念への批判 |
| **2 翻訳不可能性と対処** | 2.1 翻訳不可能性の問題, 2.2 Benjamin純粋言語論, 2.3 対処戦略, 2.4 Dictionary of Untranslatables |
| **3 翻訳規範論** | 3.1 Toury翻訳規範理論, 3.2 妥当性と受容可能性 |
| **4 対照言語学** | 4.1 語彙類型論, 4.2 Thinking for Speaking, 4.3 文法カテゴリの差異 |
| **5 自然意味論メタ言語** | 5.1 NSMとは, 5.2 NSMによる概念定義 |
| **6 機械翻訳と人間翻訳** | 6.1 機械翻訳の発展, 6.2 LLM時代の翻訳, 6.3 ポストエディット, 6.4 ローカリゼーション, 6.5 翻訳記憶 |
| **7 翻訳批評・翻訳倫理** | 7.1 翻訳の不可視性, 7.2 翻訳者の倫理, 7.3 翻訳品質評価, 7.4 翻訳と書き換え |
| **8 ポストコロニアル翻訳論** | 8.1 翻訳と権力, 8.2 英語中心主義批判 |

---

## 1. 翻訳等価論 (Translation Equivalence)

翻訳とは、ある言語で表現された意味を別の言語で「同じ」意味として再現することである。しかし、「同じ意味」とは何を指すのか？この問いへの答えが**等価（equivalence）**の理論である。

> 「翻訳は裏切りである」（Traduttore, traditore — イタリアの諺）

### 1.1 形式的等価と動的等価 (Nida 1964)

##### Origin

**Eugene Nida (1964)** *Toward a Science of Translating* において、聖書翻訳の実践から以下の区別を導入した。Nidaはアメリカ聖書協会の翻訳コンサルタントとして、非西洋文化圏への聖書翻訳に従事した経験から、形式への忠実さと意味への忠実さの間のジレンマを理論化した。

##### Core Mechanism

| 等価の種類 | 説明 | 優先するもの |
| :--- | :--- | :--- |
| **形式的等価（Formal Equivalence）** | 原文の形式（語順、文構造、修辞）を可能な限り保持 | 原文への忠実さ |
| **動的等価（Dynamic Equivalence）** | 原文が読者に与えた効果を、訳文の読者にも与える | 読者の反応・理解 |

**例：聖書の翻訳**

> 原文（ギリシャ語）：「私の杯から飲め」
> 
> - **形式的等価**: 「私の杯から飲め」（原文の比喩を保持）
> - **動的等価**: 「私の苦しみを分かち合え」（比喩を解釈して伝える）

##### Criticism & Limitation

- **二項対立の単純化**: 形式と動的の二分法は連続体であり、明確に分離できない
- **読者反応の測定困難**: 「同じ効果」を客観的に測定する方法が不明確
- **文化的仮定**: 「自然な」訳文という概念自体が文化依存的

---

### 1.2 Werner Koller の等価類型 (1979)

##### Origin

**Werner Koller (1979)** *Einführung in die Übersetzungswissenschaft* において、Nidaの二分法をより精緻化し、5つの等価類型を提唱した。ドイツ語圏の翻訳学（Übersetzungswissenschaft）の伝統に立脚している。

##### Core Mechanism

| 等価類型 | 説明 | 例 |
| :--- | :--- | :--- |
| **内容的等価（Denotative）** | 指示対象が同じ | dog → 犬 |
| **含意的等価（Connotative）** | 連想・ニュアンスが同じ | home → 家庭/ホーム? |
| **テクスト規範的等価（Text-normative）** | テクストタイプの規範に従う | 契約書は法律文体で |
| **語用論的等価（Pragmatic）** | 読者への効果が同じ | ジョークの翻訳 |
| **形式的等価（Formal-aesthetic）** | 韻律・修辞・形式が同じ | 詩の翻訳 |

##### Criticism & Limitation

- **類型間の優先順位が不明確**: 複数の等価が競合した場合の判断基準が欠如
- **静的な分類**: 翻訳プロセスの動的側面を捉えていない
- **言語ペア依存性**: 特定の言語ペアでのみ有効な分類の可能性

---

### 1.3 スコポス理論 (Skopos Theory)

##### Origin

**Hans Vermeer (1978)** および **Katharina Reiss & Vermeer (1984)** *Grundlegung einer allgemeinen Translationstheorie* において、翻訳行為論（Translatorisches Handeln）の文脈で提唱された。「スコポス」はギリシャ語で「目的」を意味する。

##### Core Mechanism

> **スコポス原則**: 翻訳は目的によって決まる。同じ原文でも、目的が異なれば異なる翻訳が正当化される。

**例：料理レシピの翻訳**

| 目的（スコポス） | 翻訳方法 |
| :--- | :--- |
| 家庭での調理 | 現地で入手可能な食材に置き換え、計量単位を変換 |
| 文化研究 | 原文の食材名を保持し、注釈を付ける |

**Reissのテクストタイプ論 (Reiss 1971)**

**Katharina Reiss (1971)** *Möglichkeiten und Grenzen der Übersetzungskritik* において、ビューラーの言語機能論に基づくテクストタイプ分類を提唱した。Vermeerとの共著でスコポス理論を発展させる以前の業績。

| タイプ | 機能 | 対応するテクスト |
| :--- | :--- | :--- |
| **情報的** | 内容伝達 | 新聞記事、教科書 |
| **表現的** | 芸術的表現 | 文学作品 |
| **訴え的** | 行動喚起 | 広告、演説 |

**Nordのテクストタイプと翻訳戦略 (Nord 1997)**

**Christiane Nord (1997)** *Translating as a Purposeful Activity* は、Reissのテクストタイプ論とスコポス理論を統合し、実務的な翻訳戦略を体系化した。

| テクストタイプ | 機能 | 翻訳戦略 |
| :--- | :--- | :--- |
| **情報的テクスト** | 情報伝達 | 内容的等価を優先 |
| **表現的テクスト** | 芸術表現 | 形式的・美的側面を重視 |
| **アピール的テクスト** | 読者への働きかけ | 効果の等価を優先 |
| **ファティックテクスト** | 関係維持 | 社会的機能を優先 |

##### Criticism & Limitation

- **原文軽視の危険**: 目的が何でも正当化されうるという批判
- **翻訳者の権限問題**: 依頼者の目的と翻訳者の判断の衝突
- **目的の特定困難**: 複合的な目的を持つテクストへの対応

---

### 1.4 等価概念への批判

##### Origin

**Mary Snell-Hornby (1988)** *Translation Studies: An Integrated Approach* および **Anthony Pym (2010)** *Exploring Translation Theories* が等価概念の根本的批判を展開した。

##### Core Mechanism

Snell-Hornbyの主張:
> 等価は存在しない。翻訳は常に「適応（adaptation）」であり、原文との関係は連続的なスペクトラムである。

Pymの分析:
- 等価概念の歴史的変遷を分析し、等価は固定的な基準ではなく、翻訳実践の中で交渉されるものだと論じた

##### Criticism & Limitation

- **代替案の欠如**: 等価を否定した後の代替的枠組みが不十分
- **実務との乖離**: 翻訳実務者は依然として「等価」を参照枠として使用

---

## 2. 翻訳不可能性と対処 (Untranslatability)

ある言語にはあるが、別の言語には対応する語がない——この現象を**翻訳不可能性（untranslatability）**と呼ぶ。

### 2.1 翻訳不可能性の問題 (Catford 1965)

##### Origin

**J.C. Catford (1965)** *A Linguistic Theory of Translation* において、翻訳不可能性を言語学的に分析した。構造主義言語学の影響下で、言語間の対応関係を体系的に記述しようとした。

##### Core Mechanism

| 種類 | 説明 | 例 |
| :--- | :--- | :--- |
| **言語的不可能性** | 文法・音韻レベルの対応がない | 日本語の敬語体系 → 英語 |
| **文化的不可能性** | 文化的概念の対応がない | 「義理」「もったいない」→ 英語 |

##### Criticism & Limitation

- **言語構造主義の限界**: 言語は閉じたシステムではなく、常に変化・借用が起こる
- **不可能性の相対性**: 時間とともに借用語として定着する可能性（例: 「Tsunami」）

---

### 2.2 Walter Benjamin の純粋言語論 (1923)

##### Origin

**Walter Benjamin (1923)** *Die Aufgabe des Übersetzers*（翻訳者の使命）において、翻訳を哲学的に考察した。フランクフルト学派の批評理論、ユダヤ神秘主義の影響を受けている。

##### Core Mechanism

> 翻訳は原文に「第二の生命」を与える。翻訳を通じて、原文は自らを超え、「純粋言語（reine Sprache）」に近づく。

翻訳を「原文の劣化コピー」ではなく、**言語と意味の本質への探求**として再定義する。各言語は「純粋言語」の断片であり、翻訳はそれらを補完し合う行為である。

##### Criticism & Limitation

- **神秘主義的**: 実務的な翻訳指針を提供しない
- **検証不可能**: 「純粋言語」の存在は経験的に検証できない
- **解釈の困難**: テクスト自体が難解で多様な解釈を生む

---

### 2.3 対処戦略

##### Origin

翻訳学の実務的伝統から発展した戦略群。**Vinay & Darbelnet (1958)** の直接翻訳と斜訳の区分が基礎となっている。

##### Core Mechanism

| 戦略 | 説明 | 例 |
| :--- | :--- | :--- |
| **借用（Borrowing）** | 原語をそのまま使用 | 「Zeitgeist」「ikigai」 |
| **翻訳借用（Calque）** | 語の構造を母語で再構成 | 「摩天楼」← skyscraper |
| **説明的翻訳（Explicitation）** | 意味を説明する長い表現に置換 | 「Schadenfreude」→「他人の不幸を喜ぶ感情」 |
| **適応（Adaptation）** | 機能的に同等な概念に置換 | 「ティータイム」→「おやつの時間」 |
| **省略（Omission）** | 翻訳不能な要素を削除 | 駄洒落の翻訳で意味だけ残す |
| **補償（Compensation）** | 別の箇所で同等の効果を作る | 失われた韻を別の箇所で再現 |

##### Criticism & Limitation

- **戦略選択の基準**: どの戦略を選ぶかは翻訳者の判断に委ねられる
- **情報損失の不可避性**: いずれの戦略も完全な情報保持を保証しない

---

### 2.4 Dictionary of Untranslatables (Cassin 2004)

##### Origin

**Barbara Cassin et al. (2004/2014)** *Vocabulaire Européen des Philosophies / Dictionary of Untranslatables* は、哲学における翻訳不可能な概念の百科事典である。フランス国立科学研究センター（CNRS）のプロジェクトとして開始された。

##### Core Mechanism

「翻訳不可能」を「完全には翻訳できず、常に問題を提起し続ける」と再定義し、翻訳の歴史そのものを記録した。各項目は概念の翻訳履歴と問題点を詳述する。

##### Criticism & Limitation

- **ヨーロッパ中心主義**: 主にヨーロッパ言語間の哲学用語に焦点
- **静的な記録**: オンライン版がなく、継続的更新が困難

---

## 3. 翻訳規範論 (Translation Norms)

翻訳は社会的・文化的規範に規定された行為である。翻訳者は常に、ターゲット文化における「適切さ」と「受容可能性」の規範に制約される。

### 3.1 Touryの翻訳規範理論 (1995)

##### Origin

**Gideon Toury (1995)** *Descriptive Translation Studies and Beyond* において、記述的翻訳学（Descriptive Translation Studies）の枠組みで翻訳規範を体系化した。イスラエルのテルアビブ学派の中心人物。

**多元システム論（Polysystem Theory）との関係**

Touryの理論は、**Itamar Even-Zohar (1978)** の多元システム論を基盤とする。多元システム論は、文学を「中心」と「周縁」の動的な系として捗える。翻訳文学は受容文化において中心的位置を占めることもあれば、周縁的位置に留まることもある。この位置が翻訳規範を規定する。

##### Core Mechanism

> 翻訳は真空の中で起こるのではない。翻訳者は常に、ターゲット文化における「適切さ」と「受容可能性」の規範に制約される。

**規範の三層構造**

| 規範 | 説明 |
| :--- | :--- |
| **初期規範（Initial Norm）** | 原文忠実か訳文自然かの基本姿勢 |
| **予備規範（Preliminary Norms）** | 何を翻訳するか/しないかの政策 |
| **操作規範（Operational Norms）** | 具体的な翻訳戦略 |

##### Criticism & Limitation

- **規範の特定困難**: 規範は暗黙的であり、事後的にしか特定できない
- **規範の可変性**: 時代・地域によって変化し、普遍的な規範は存在しない
- **規範と個人**: 翻訳者個人の選択と社会的規範の関係が不明確

---

### 3.2 妥当性と受容可能性 (Adequacy vs. Acceptability)

##### Origin

Touryが提唱した翻訳分析の軸。翻訳のポジションを連続体として捉える。

##### Core Mechanism

| 軸 | 説明 | 特徴 |
| :--- | :--- | :--- |
| **妥当性（Adequacy）** | 原文の規範に従う | 異質性を保持、読みにくい可能性 |
| **受容可能性（Acceptability）** | ターゲット文化の規範に従う | 流暢だが、原文の特徴が失われる |

翻訳者は常にこの二極の間でバランスを取る。

##### Criticism & Limitation

- **二項対立の単純化**: 実際の翻訳は複雑であり、単純な軸では捉えきれない
- **評価的含意**: 「妥当性」という語が価値判断を含意してしまう

---

## 4. 対照言語学 (Contrastive Linguistics)

言語は世界の分節化（どこで語の境界を引くか）において異なる。翻訳者はこれらの差異を理解し、適切に対処する必要がある。

### 4.1 語彙類型論 (Lexical Typology)

##### Origin

**Nicholas Evans (2010)** *Semantic Typology and Spatial Cognition* において、語彙化パターンの言語間差異を体系的に研究した。言語類型論の枠組みで意味論的多様性を分析する。

##### Core Mechanism

**例：色の語彙化**

| 言語 | 青と緑の区別 |
| :--- | :--- |
| **英語** | blue / green（別々の語） |
| **日本語** | 青/緑（区別するが、「青信号」のように青が緑を含む用法あり） |
| **グルジ語（パプアニューギニア）** | 一つの語（grue）が青と緑を包括 |

##### Criticism & Limitation

- **データの偏り**: 研究対象言語がヨーロッパ言語に偏っている
- **変化の無視**: 言語接触による語彙変化を十分に捉えていない

---

### 4.2 Slobin の Thinking for Speaking (1996)

##### Origin

**Dan Slobin (1996)** が「言語が思考を制約する」というサピア・ウォーフ仮説を、発話時の思考に限定して再定式化した。

##### Core Mechanism

> **Thinking for Speaking**: 私たちは話すために思考する際、母語の文法カテゴリに沿って経験を組織化する。

**例：移動表現**

| 言語タイプ | 移動の表現 | 例 |
| :--- | :--- | :--- |
| **動詞枠組み（Verb-framed）** | 経路が動詞 | スペイン語 "subir corriendo"（走って上がる） |
| **衛星枠組み（Satellite-framed）** | 経路が副詞・前置詞 | 英語 "run up" |
| **均衡型** | 動詞と副詞が均等に分担 | 日本語「走って上がる」 |

##### Criticism & Limitation

- **限定的な影響**: 言語が思考に与える影響は発話時に限定される
- **個人差の無視**: 二言語話者の存在が理論を複雑化

---

### 4.3 文法カテゴリの差異

##### Origin

対照言語学の基本的知見。文法カテゴリの言語間差異は翻訳の根本的問題である。

##### Core Mechanism

| カテゴリ | 日本語 | 英語 |
| :--- | :--- | :--- |
| **数** | 通常は明示しない | 単数/複数が必須 |
| **冠詞** | なし | 定冠詞/不定冠詞が必須 |
| **主語** | 省略可能 | 通常は必須 |
| **敬語** | 文法的に体系化 | 語彙的に表現 |
| **証拠性** | 語彙的に表現 | （一部言語では文法的） |

翻訳時には、ソース言語で暗黙的な情報をターゲット言語で明示化（explicitation）する必要がしばしばある。

##### Criticism & Limitation

- **二項対立の限界**: 言語間の差異は連続体であり、二項対立では捉えきれない
- **変異の無視**: 方言、レジスターによる差異を見落とす

---

## 5. 自然意味論メタ言語 (Natural Semantic Metalanguage)

言語に依存しない意味定義を目指す理論的枠組み。

### 5.1 NSMとは (Wierzbicka 1996)

##### Origin

**Anna Wierzbicka (1996)** *Semantics: Primes and Universals* において提唱された。ポーランド生まれのオーストラリアの言語学者で、言語普遍性の研究に貢献した。

##### Core Mechanism

> **NSM仮説**: すべての言語にはおよそ65の基本概念（プリミティブ）が存在し、これらを使えばあらゆる複雑な概念を定義できる。

**意味プリミティブの例**

| カテゴリ | プリミティブ |
| :--- | :--- |
| 実体詞 | I, YOU, SOMEONE, SOMETHING, PEOPLE, BODY |
| 精神述語 | THINK, KNOW, WANT, FEEL, SEE, HEAR |
| 評価詞 | GOOD, BAD |
| 論理 | NOT, MAYBE, CAN, BECAUSE, IF |

##### Criticism & Limitation

- **プリミティブの恣意性**: なぜこれらがプリミティブかの根拠が不明確
- **翻訳実務との乖離**: 実際の翻訳でNSMを使う場面は限定的
- **文化依存性**: 「普遍的」とされるプリミティブも文化的バイアスを含む可能性

---

### 5.2 NSMによる概念定義

##### Origin

Wierzbickaとその共同研究者たちによって発展した概念定義の方法論。

##### Core Mechanism

文化固有の概念も普遍的な語彙で「解凍」できる。

**例：「Schadenfreude」のNSM定義**

```
X feels something good
because X knows something bad happened to someone else
X thinks: this is good for me because of this
```

この定義は、ドイツ語を知らなくても理解できる。

##### Criticism & Limitation

- **冗長性**: シンプルな概念も長い説明になりがち
- **精度の限界**: 複雑な概念のニュアンスを完全には捉えられない

---

## 6. 機械翻訳と人間翻訳 (Machine and Human Translation)

技術進歩が翻訳ワークフローを変革している。機械翻訳の能力と限界を理解することは翻訳設計に不可欠である。

### 6.1 機械翻訳の発展

##### Origin

機械翻訳は1950年代のルールベースシステムから始まり、統計モデル、ニューラルネットワークを経て、現在の大規模言語モデル（LLM）に至る。

##### Core Mechanism

| 世代 | 年代 | アプローチ | 特徴 |
| :--- | :--- | :--- | :--- |
| **RBMT** | 1950s-1980s | ルールベース | 文法規則と辞書 |
| **SMT** | 1990s-2000s | 統計ベース | 大規模コーパスからの学習 |
| **NMT** | 2010s- | ニューラルネットワーク | 文脈理解、流暢さの向上 |
| **LLM** | 2020s- | 大規模言語モデル | Few-shot学習、指示理解 |

##### Criticism & Limitation

- **稀少言語の精度**: 訓練データが少ない言語では品質が低下
- **専門分野の精度**: 専門用語や文脈理解に限界
- **文化的ニュアンス**: 暗黙的な文化的意味を捉えにくい

---

### 6.2 LLM時代の翻訳 (LLM-based Translation)

##### Origin

GPT-4、Claude等の大規模言語モデルは、従来のNMTとは異なる特性を持つ。2020年代に商用化が進んだ。

##### Core Mechanism

| 特性 | NMT | LLM |
| :--- | :--- | :--- |
| **学習方法** | 対訳コーパスで訓練 | 汎用的な言語理解 |
| **指示理解** | 不可 | プロンプトで翻訳スタイルを指定可能 |
| **用語統一** | 外部用語集が必要 | 用語集をプロンプト内で指定可能 |
| **文化適応** | 限定的 | 文化的文脈を説明すれば適応可能 |

##### Criticism & Limitation

- **一貫性の欠如**: 同じ入力でも出力が変動しうる
- **幻覚（Hallucination）**: 原文にない内容を生成する可能性
- **検証可能性**: なぜそう翻訳したかの説明が困難

---

### 6.3 ポストエディット (Post-Editing)

##### Origin

機械翻訳の出力を人間が修正するプロセス。**Sharon O'Brien (2012)** がポストエディットの認知負荷と効率のトレードオフを研究した。

##### Core Mechanism

| レベル | 説明 | 用途 |
| :--- | :--- | :--- |
| **Light PE** | 最低限の修正（意味が通じる程度） | 社内文書、参考資料 |
| **Full PE** | 完全な修正（出版品質） | 公開コンテンツ |

##### Criticism & Limitation

- **認知負荷**: 低品質の機械翻訳を修正するよりゼロから翻訳する方が効率的な場合もある
- **翻訳者のスキル変化**: ポストエディットは翻訳とは異なるスキルセットを要求

---

### 6.4 ローカリゼーション理論 (Localization Theory)

##### Origin

**LISA（Localization Industry Standards Association）**および**GALA（Globalization and Localization Association）**が標準化したフレームワーク。ソフトウェア産業から発展した。

##### Core Mechanism

| 概念 | 定義 | 例 |
| :--- | :--- | :--- |
| **国際化（i18n）** | ソフトウェアを多言語対応可能な設計にすること | Unicode対応、日付形式の抽象化 |
| **地域化（L10n）** | 特定の地域・言語向けにコンテンツを適応すること | 日本語翻訳、日付形式を「年/月/日」に |
| **グローバリゼーション（g11n）** | i18n + L10n を含む全体戦略 | 多言語展開計画 |

**技術的ローカリゼーション要素**

| 要素 | 説明 |
| :--- | :--- |
| **日付・時刻形式** | 地域ごとに異なる形式 |
| **数値形式** | 桁区切り、小数点の差異 |
| **テキスト方向（RTL）** | アラビア語・ヘブライ語等の右→左表記 |
| **文字エンコーディング** | 多言語文字の表現 |

##### Criticism & Limitation

- **技術偏重**: 文化的適応より技術的側面に焦点
- **標準化の限界**: すべてのコンテンツタイプに適用できるわけではない

---

### 6.5 翻訳記憶 (Translation Memory)

##### Origin

1980年代にCAT（Computer-Assisted Translation）ツールの中核技術として発展した。**Trados（1984）**、**SDL**等の企業が商用化を推進。

##### Core Mechanism

翻訳記憶は、過去に翻訳された文（セグメント）とその訳文をデータベースに保存し、類似の文が出現した際に再利用する技術。

| 概念 | 説明 |
| :--- | :--- |
| **セグメント** | 翻訳の単位となる文または句 |
| **完全一致（Exact Match）** | 100%一致するセグメント |
| **ファジーマッチ** | 部分的に一致するセグメント（例：85%一致） |
| **用語集（Termbase）** | 用語とその訳語の対応表 |

##### Criticism & Limitation

- **文脈の欠如**: セグメント単位での一致は文脈を無視しうる
- **品質の継承**: 過去の誤訳が再利用される危険
- **創造性の制約**: 一貫性重視が表現の多様性を制限

---

### 6.6 翻訳と認知 (Cognitive Translatology)

##### Origin

**Shreve, Gregory M. & Angelone, Erik (Eds.) (2010).** *Translation and Cognition.* John Benjamins. — 翻訳認知科学の包括的概説。

**Muñoz Martín, Ricardo (2010).** On paradigms and cognitive translatology. In G.M. Shreve & E. Angelone (Eds.), *Translation and Cognition*. — 翻訳認知学の方法論的議論。

**Carl, Michael, Bangalore, Srinivas & Schaeffer, Moritz (Eds.) (2016).** *New Directions in Empirical Translation Process Research.* Springer. — 翻訳プロセス研究の最新動向。

##### Core Mechanism

翻訳認知学（Cognitive Translatology）は、翻訳を**認知プロセス**として研究する分野である：

| 研究手法 | 内容 |
|:---|:---|
| **Think-Aloud Protocol（TAP）** | 翻訳者が翻訳中に思考を口頭で報告。内省データの収集 |
| **Keystroke Logging** | キーストロークと時間間隔を記録。翻訳プロセスの可視化 |
| **Eye-Tracking** | 視線の動きを追跡。注意配分、原文と訳文の参照パターン |
| **脳イメージング（fMRI, EEG）** | 翻訳中の神経活動を測定 |

**翻訳の認知モデル**:

| モデル | 内容 |
|:---|:---|
| **問題解決モデル** | 翻訳は一連の問題解決プロセス。困難な箇所で処理時間が増大 |
| **二重プロセス理論** | 自動的処理（Type 1）と意識的処理（Type 2）の相互作用 |
| **翻訳の専門性（Expertise）** | 熟達翻訳者と初心者の認知プロセスの差異。チャンキング、自動化 |

**認知負荷と翻訳**:

翻訳は高い認知負荷を課す作業である。ワーキングメモリは原文理解、訳出案生成、訳文評価を同時に処理する。§LEARNING_FOUNDATIONS 1.4のスキーマ理論、§CORE_FOUNDATIONS 2.3の認知負荷理論と接続。

##### Criticism & Limitation

| 批判 | 内容 |
|:---|:---|
| **TAP の限界** | 言語化できない認知プロセスを捉えられない。言語化自体がプロセスを変化させる |
| **生態学的妥当性** | 実験室条件と実際の翻訳作業環境の乖離 |
| **普遍性への疑問** | 言語ペア、テキストタイプ、個人差による変動が大きい |

---

### 6.7 専門分野翻訳 (Specialized Translation)

##### Origin

**Šarčević, Susan (1997).** *New Approach to Legal Translation.* Kluwer Law International. — 法律翻訳の理論的基盤。

**Montalt, Vicent & González Davies, Maria (2007).** *Medical Translation Step by Step.* St. Jerome. — 医療翻訳の実践的ガイド。

##### Core Mechanism

**法律翻訳（Legal Translation）**:

| 特徴 | 内容 |
|:---|:---|
| **法体系の差異** | 法的概念は法体系（Common Law, Civil Law等）に依存。「同等」の概念が存在しないことが多い |
| **規範的テクスト** | 法律テクストは行為を規制する。翻訳の誤りは法的帰結を持つ |
| **専門用語** | 法律用語は厳密に定義され、一般言語と意味が異なることがある |
| **機能的等価** | 概念的等価が不可能な場合、機能的に同等な効果を持つ訳を目指す |

**医療翻訳（Medical Translation）**:

| 特徴 | 内容 |
|:---|:---|
| **患者安全** | 翻訳の誤りは患者の安全に直結。高い正確性が要求される |
| **テクストタイプの多様性** | 臨床試験文書、患者向け説明文書、論文等で異なる戦略が必要 |
| **用語の標準化** | ICD（国際疾病分類）、MedDRA等の国際標準用語 |
| **患者リテラシー** | 患者向け文書では専門用語の平易な説明が必要 |

**共通の課題**:

| 課題 | 内容 |
|:---|:---|
| **翻訳者の専門知識** | 法律・医療の専門知識が不可欠。言語能力だけでは不十分 |
| **専門家との協働** | 法律家・医療専門家との連携が品質を左右 |
| **規制要件** | 認証翻訳（Certified Translation）、規制文書の特殊要件 |

##### Criticism & Limitation

| 批判 | 内容 |
|:---|:---|
| **アクセシビリティ** | 専門知識の要件が翻訳者の参入障壁となる |
| **標準化の限界** | 国際標準用語があっても、法的・文化的文脈は標準化できない |
| **AIの影響** | 専門分野翻訳へのAI適用可能性と限界は未解明 |

---

## 7. 翻訳批評と翻訳倫理 (Translation Criticism and Ethics)

翻訳の質をどう評価し、翻訳者はどのような責任を負うかを論じる。

### 7.1 翻訳の不可視性 (Venuti 1995)

##### Origin

**Lawrence Venuti (1995)** *The Translator's Invisibility* において、翻訳者が「透明」であるべきという規範が、翻訳者の労働と創造性を不可視化していると批判した。

##### Core Mechanism

| 戦略 | 説明 | 効果 |
| :--- | :--- | :--- |
| **同化（Domestication）** | ターゲット文化に合わせて翻訳 | 読みやすいが、異文化性が消える |
| **異化（Foreignization）** | ソース文化の痕跡を残す | 異文化性を保つが、読みにくい |

##### Criticism & Limitation

- **二項対立の単純化**: 実際の翻訳は両方の戦略を混合している
- **異化の限界**: 極端な異化は読者を遠ざける
- **権力批判の限界**: すべての同化が権力的支配とは限らない

---

### 7.2 翻訳者の倫理 (Pym 2012)

##### Origin

**Anthony Pym (2012)** *On Translator Ethics* において、翻訳者の倫理を体系的に論じた。

##### Core Mechanism

| 原則 | 説明 |
| :--- | :--- |
| **忠実性（Fidelity）** | 原文の意味を正確に伝える義務 |
| **誠実性（Honesty）** | 翻訳の限界を認め、必要に応じて注記する |
| **責任（Accountability）** | 翻訳の帰結に対する責任 |
| **協働（Cooperation）** | 著者・読者・依頼者との協調 |

##### Criticism & Limitation

- **原則間の衝突**: 忠実性と読者への責任が衝突する場合の判断基準が不明確
- **文脈依存性**: 倫理原則は翻訳の目的や文脈によって異なる

---

### 7.3 翻訳品質評価 (House 1997, MQM 2015)

##### Origin

**Juliane House (1997, 2015)** *Translation Quality Assessment* が体系的な品質評価モデルを提唱。**DFKI/TAUS (2015)** がMQM（Multidimensional Quality Metrics）を開発した。

##### Core Mechanism

**House のモデル**

| 評価基準 | 説明 |
| :--- | :--- |
| **Overt Translation** | 翻訳であることが明示される（文学翻訳） |
| **Covert Translation** | 翻訳であることが隠される（実用翻訳） |

**MQM エラータイプ**

| エラータイプ | 説明 | 重大度 |
| :--- | :--- | :--- |
| **正確性（Accuracy）** | 意味の誤り、省略、追加 | 重大 |
| **流暢性（Fluency）** | 文法、スタイル、可読性 | 中 |
| **用語（Terminology）** | 用語集との一貫性 | 中 |
| **スタイル（Style）** | 文体の適切性 | 軽微 |

##### Criticism & Limitation

- **主観性**: 品質評価は評価者によって異なる
- **コスト**: 詳細な品質評価は時間とコストがかかる
- **文脈依存性**: 品質基準は翻訳の目的によって異なる

---

### 7.4 翻訳と書き換え (Lefevere 1992)

##### Origin

**André Lefevere (1992)** *Translation, Rewriting, and the Manipulation of Literary Fame* において、翻訳を「書き換え（rewriting）」の一形態として捉える理論を提唱した。操作学派（Manipulation School）の中心人物。

##### Core Mechanism

翻訳は、批評・解説・選集編纂・映画化等と並ぶ「書き換え」の一種であり、受容文化の規範に沿ってテクストを「操作」する行為である。

| 概念 | 説明 |
| :--- | :--- |
| **パトロネージュ** | 文学生産を支配する権力システム（出版社、批評家、教育機関） |
| **イデオロギー** | 書き換えを規定する信念体系 |
| **詩学** | ジャンルやスタイルに関する規範 |

##### Criticism & Limitation

- **文学翻訳への偏重**: 非文学テクストへの適用が限定的
- **能動性の過大評価**: 翻訳者の操作が意図的であるかのような含意
- **記述と規範の混同**: 「操作」という語の否定的含意

---

## 8. ポストコロニアル翻訳論 (Postcolonial Translation Theory)

翻訳と権力の関係を批判的に分析する。

### 8.1 翻訳と権力 (Spivak 1993, Niranjana 1992)

##### Origin

**Gayatri Spivak (1993)** *The Politics of Translation* および **Tejaswini Niranjana (1992)** *Siting Translation* が翻訳と植民地主義の関係を分析した。

##### Core Mechanism

> 翻訳は常に権力関係の中で行われる。「周辺」から「中心」への翻訳と、その逆では、意味が異なる。

翻訳が支配-被支配関係を再生産しうることを示した。

##### Criticism & Limitation

- **理論偏重**: 実務的な翻訳指針を提供しない
- **本質主義への危険**: 「中心」と「周辺」の固定化
- **変化への対応**: グローバル化による権力関係の変化を捉えきれない

---

### 8.2 英語中心主義への批判

##### Origin

翻訳学とポストコロニアル研究の交差点で発展した批判的視点。

##### Core Mechanism

| パターン | 説明 | 問題 |
| :--- | :--- | :--- |
| **英語→周辺言語** | 英語のコンテンツを各国語に翻訳 | 英語が「原典」、他は「派生」という階層 |
| **周辺言語→英語→周辺言語** | 非英語圏間の翻訳も英語を経由 | 二重の意味損失 |

##### Criticism & Limitation

- **英語の実用性**: 英語がリンガフランカとして機能している現実
- **代替案の不在**: 英語中心を批判しても具体的な代替案が不足

---

## 9. 他の理論基盤との接続 (Cross-Reference to Other Foundations)

### 9.1 証言の認識論との接続

[CORE_FOUNDATIONS.md §2.1](./CORE_FOUNDATIONS.md) の証言の認識論は、翻訳設計に以下の含意を持つ：

| 証言の認識論の概念 | 翻訳への適用 |
| :--- | :--- |
| **証言者の信頼性** | 翻訳者も「証言者」として機能。翻訳の正確性は翻訳者の能力と誠実性に依存 |
| **証言的不正義** | 翻訳者の判断が軽視される「翻訳の不可視性」問題との接続 |
| **署名責任原則** | レビュワーが翻訳コンテンツの全責任を負う（STRATEGY.md §2.2参照） |

---

### 9.2 教育的公正との接続

[CORE_FOUNDATIONS.md §1](./CORE_FOUNDATIONS.md) の教育的公正は、翻訳の意義を以下のように基礎づける：

| 教育的公正の概念 | 翻訳への適用 |
| :--- | :--- |
| **分配的正義** | 知識への平等なアクセスは言語障壁によって阻害される。翻訳はこれを解消する手段 |
| **承認の正義** | 英語以外の言語で執筆された知識も「正当な知識」として承認される必要がある |
| **言語的資本** | 多言語対応は言語的資本の格差を縮小する |

---

### 9.3 認知負荷理論との接続

[LEARNING_FOUNDATIONS.md](./LEARNING_FOUNDATIONS.md) の認知負荷理論は、翻訳品質に以下の基準を与える：

| 認知負荷の観点 | 翻訳品質への含意 |
| :--- | :--- |
| **内在的負荷** | 概念の複雑さ自体は翻訳で変わらない |
| **外在的負荷** | 不自然な訳文、不統一な用語は外在的負荷を増大させる |
| **本来的負荷** | 学習を促進する「適切な困難さ」は翻訳で維持されるべき |

**実務的含意**: 翻訳品質評価において「流暢性」は、単なる読みやすさではなく**認知負荷の最適化**として位置づけられる。

---

## 10. スキーマへの反映

### 10.1 翻訳関連フィールド

| 理論 | スキーマフィールド |
| :--- | :--- |
| **等価論** | `translation.strategy`, `translation.equivalence_type` |
| **翻訳不可能性** | `translation.untranslatability_note`, `display.original_term` |
| **スコポス理論** | 翻訳ワークフロー全体の設計原則 |
| **翻訳規範論** | `translation.norms`（採用した規範を記録） |
| **NSM** | `display.intuition`（シンプルな表現を志向） |
| **語彙類型論** | `translation.cultural_adaptation` |
| **翻訳倫理** | `translation.notes`（翻訳者の判断を記録） |

---

## 11. 年表

| 年代 | 理論/著作 | 著者 |
| :--- | :--- | :--- |
| 1923 | 翻訳者の使命 | Benjamin |
| 1964 | Toward a Science of Translating | Nida |
| 1965 | A Linguistic Theory of Translation | Catford |
| 1971 | Möglichkeiten und Grenzen der Übersetzungskritik（テクストタイプ論） | Reiss |
| 1978 | 多元システム論（Polysystem Theory） | Even-Zohar |
| 1978 | スコポス理論 | Vermeer |
| 1979 | 等価の5類型 | Koller |
| 1988 | Translation Studies: An Integrated Approach | Snell-Hornby |
| 1992 | Siting Translation | Niranjana |
| 1992 | Translation, Rewriting, and the Manipulation of Literary Fame | Lefevere |
| 1993 | The Politics of Translation | Spivak |
| 1995 | Descriptive Translation Studies and Beyond | Toury |
| 1995 | The Translator's Invisibility | Venuti |
| 1996 | Semantics: Primes and Universals | Wierzbicka |
| 1996 | Thinking for Speaking | Slobin |
| 1997 | Translating as a Purposeful Activity | Nord |
| 1997 | Translation Quality Assessment | House |
| 2003 | LISA Localization Best Practices | LISA |
| 2004 | Dictionary of Untranslatables | Cassin et al. |
| 2010 | Exploring Translation Theories | Pym |
| 2010 | Semantic Typology and Spatial Cognition | Evans |
| 2012 | On Translator Ethics | Pym |
| 2015 | MQM Framework | DFKI/TAUS |

---

## 12. 推奨文献 (Further Reading)

### 翻訳理論入門

- **Munday, J. (2016).** *Introducing Translation Studies.* 4th ed. Routledge.
  - 翻訳学の標準的教科書。各理論を網羅的に解説。
- **Pym, A. (2010).** *Exploring Translation Theories.* Routledge.
  - 翻訳理論の歴史と批評的分析。

### 等価と翻訳可能性

- **Nida, E.A. (1964).** *Toward a Science of Translating.* Brill.
  - 形式的等価と動的等価の原典。
- **Cassin, B., et al. (Eds.) (2014).** *Dictionary of Untranslatables.* Princeton University Press.
  - 翻訳不可能な哲学概念の百科事典。

### 翻訳倫理と批評

- **Venuti, L. (1995).** *The Translator's Invisibility.* Routledge.
  - 同化と異化、翻訳者の不可視性。
- **Pym, A. (2012).** *On Translator Ethics.* John Benjamins.
  - 翻訳者の倫理的責任。

### 翻訳規範論

- **Toury, G. (1995).** *Descriptive Translation Studies and Beyond.* John Benjamins.
  - 翻訳規範理論の基礎文献。
- **Nord, C. (1997).** *Translating as a Purposeful Activity.* St. Jerome.
  - スコポス理論の実務的発展。

### ローカリゼーション

- **Esselink, B. (2000).** *A Practical Guide to Localization.* John Benjamins.
  - ローカリゼーションの実務ガイド。
- **Pym, A. (2004).** *The Moving Text: Localization, Translation, and Distribution.* John Benjamins.
  - ローカリゼーションと翻訳学の接点。

### ポストコロニアル翻訳論

- **Spivak, G.C. (1993).** The Politics of Translation. In *Outside in the Teaching Machine.* Routledge.
  - 翻訳と権力の関係。
- **Niranjana, T. (1992).** *Siting Translation.* University of California Press.
  - 植民地主義と翻訳。

### 機械翻訳

- **Forcada, M.L. (2017).** Making sense of neural machine translation. *Translation Spaces.*
  - NMTの仕組みと翻訳学への含意。

### 品質評価

- **House, J. (2015).** *Translation Quality Assessment: Past and Present.* Routledge.
  - 翻訳品質評価の体系的解説。
- **TAUS (2015).** *MQM Framework.* https://www.qt21.eu/mqm-definition/
  - 業界標準の品質評価フレームワーク。

### 対照言語学

- **Evans, N., & Levinson, S.C. (2009).** The myth of language universals. *Behavioral and Brain Sciences.*
  - 言語間の差異と普遍性への批判的視点。
- **Wierzbicka, A. (1996).** *Semantics: Primes and Universals.* Oxford University Press.
  - NSMの体系的解説。

