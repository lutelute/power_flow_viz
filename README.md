# Power Flow Visualization Project

電力潮流計算アルゴリズムの可視化・教育プラットフォーム

## 🌐 オンライン版

**GitHub Pages**: https://lutelute.github.io/power_flow_viz/

全ての可視化ツールをブラウザで直接実行できます。ダウンロード不要！

## 📖 概要

このプロジェクトは、電力系統解析における潮流計算アルゴリズムを視覚的に理解できる教育ツール群です。Newton-Raphson法、Gauss-Seidel法、高速分離解法、DC潮流計算など、主要な潮流計算手法の動作過程、収束特性、精度特性を対話的に学習できます。

## 🎯 特徴

- **20以上の潮流計算アルゴリズム**の実装・比較
- **リアルタイム収束可視化**による学習効果の向上
- **IEEE標準系統**（2, 5, 9, 14, 30母線系統）のサポート
- **MATPOWER準拠**のデータ形式とアルゴリズム実装
- **段階的アルゴリズム解説**による直感的理解
- **数学的厳密性**と教育的分かりやすさの両立

## 🎯 可視化ツール一覧

| ツール名 | 主要機能 | オンライン版 | 詳細文書 |
|---|---|---|---|
| **包括的アルゴリズム比較** | 20+手法の実装・ベンチマーク | [🚀 起動](https://lutelute.github.io/power_flow_viz/power_flow_v5.html) | [📖 詳細](./docs/power_flow_v5.md) |
| **多手法可視化** | Newton-Raphson, Gauss-Seidel等 | [🚀 起動](https://lutelute.github.io/power_flow_viz/power_flow_visualizer.html) | [📖 詳細](./docs/power_flow_visualizer.md) |
| **計算過程ステップ表示** | 5手法の段階別解説 | [🚀 起動](https://lutelute.github.io/power_flow_viz/power_flow_process_visualizer.html) | [📖 詳細](./docs/power_flow_process_visualizer.md) |
| **MATPOWER準拠実装** | Newton-Raphson, 高速分離解法 | [🚀 起動](https://lutelute.github.io/power_flow_viz/power_flow_matpower_v2.html) | [📖 詳細](./docs/power_flow_matpower_v2.md) |
| **多手法比較分析** | 主要3手法の性能比較 | [🚀 起動](https://lutelute.github.io/power_flow_viz/power_flow_compare.html) | [📖 詳細](./docs/power_flow_compare.md) |
| **収束過程直感的理解** | 複素平面・誤差面可視化 | [🚀 起動](https://lutelute.github.io/power_flow_viz/power_flow_intuitive.html) | [📖 詳細](./docs/power_flow_intuitive.md) |
| **└ v5最新版** | 問題修正・UI改善版 | [🚀 v5起動](https://lutelute.github.io/power_flow_viz/power_flow_intuitive_v5.html) | 最新改善版 |
| **DC潮流精度検証** | DC近似 vs AC解析 | [🚀 起動](https://lutelute.github.io/power_flow_viz/dc_accuracy_analysis.html) | [📖 詳細](./docs/dc_accuracy_analysis.md) |

## 🚀 使用方法

### オンライン版の使用方法

1. **メインページアクセス**: https://lutelute.github.io/power_flow_viz/
2. **ツール選択**: 目的に応じた可視化ツールを選択
3. **系統設定**: IEEE標準系統（2, 5, 9, 14, 30母線）を選択
4. **アルゴリズム選択**: 使用する潮流計算手法を選択  
5. **パラメータ調整**: 収束判定値（ε）、最大反復回数を設定
6. **実行・可視化**: リアルタイムで収束過程を観察

### ローカル実行

```bash
# リポジトリクローン
git clone https://github.com/lutelute/power_flow_viz.git
cd power_flow_viz

# 任意のHTMLファイルをブラウザで開く
open power_flow_v5.html
```

### 推奨学習順序

1. **入門** [`power_flow_intuitive_v6.html`](https://lutelute.github.io/power_flow_viz/power_flow_intuitive_v6.html) - 基本概念の理解（大規模系統対応）
2. **基礎** [`power_flow_process_visualizer.html`](https://lutelute.github.io/power_flow_viz/power_flow_process_visualizer.html) - アルゴリズム詳細（修正版）  
3. **応用** [`power_flow_compare.html`](https://lutelute.github.io/power_flow_viz/power_flow_compare.html) - 手法間比較
4. **発展** [`power_flow_v5.html`](https://lutelute.github.io/power_flow_viz/power_flow_v5.html) - 高度な手法群
5. **実務** [`power_flow_matpower_v2.html`](https://lutelute.github.io/power_flow_viz/power_flow_matpower_v2.html) - 実用的実装

## 📊 実装アルゴリズム

### 数学的定式化

潮流計算は以下の非線形代数方程式系の数値解法です：

```
P_i = Re[V_i^* ∑_{j=1}^n Y_{ij} V_j] = P_{Gi} - P_{Di}     (1)
Q_i = Im[V_i^* ∑_{j=1}^n Y_{ij} V_j] = Q_{Gi} - Q_{Di}     (2)
```

ここで、V_i = |V_i|e^{jθ_i} は母線 i の複素電圧、Y_{ij} はアドミタンス行列要素です。

### 主要アルゴリズムの特性

| 手法 | 収束次数 | 計算複雑度 | 収束条件 | 適用場面 |
|---|---|---|---|---|
| **Newton-Raphson** | O(ε²) | O(n³)/iteration | J非特異 | 汎用潮流計算 |
| **Fast Decoupled** | O(ε^{1.6}) | O(n³/2)/iteration | 送電系統 | 大規模系統 |
| **Gauss-Seidel** | O(ε) | O(n²)/iteration | ρ(G) < 1 | 教育・小規模系統 |
| **DC Power Flow** | 線形 | O(n³) | B正則 | 経済負荷配分 |

### アルゴリズム詳細

#### Newton-Raphson法
反復式: **x^{(k+1)} = x^{(k)} - [J(x^{(k)})]^{-1}f(x^{(k)})**

Jacobian行列:
```
J = [∂P/∂θ   ∂P/∂|V|]
    [∂Q/∂θ   ∂Q/∂|V|]     (3)
```

#### 高速分離解法
分離近似: **P ≈ B'θ, Q ≈ B''|V|**

反復式:
```
Δθ = (B')^{-1}(ΔP/|V|)     (4)
Δ|V| = (B'')^{-1}(ΔQ/|V|)   (5)
```

#### DC潮流計算
線形化近似下での高速解法：

```
P = Bθ  ⟹  θ = B^{-1}P     (6)
```

ここで、B_{ij} = -1/X_{ij} （i ≠ j）, B_{ii} = ∑_{j≠i} 1/X_{ij}

### 先進手法

#### Levenberg-Marquardt法
制御パラメータ μ による収束性改善：
```
x^{(k+1)} = x^{(k)} - [J^T J + μI]^{-1} J^T f(x^{(k)})     (7)
```

#### 連続潮流法（Continuation Power Flow）
パラメータ λ を用いた経路追跡：
```
F(x,λ) = 0,  λ ∈ [0, λ_{critical}]     (8)
```

#### Holomorphic Embedding法
複素解析による非反復解法：
```
V(s) = ∑_{n=0}^∞ V_n s^n,  |s| < R     (9)
```

## 📁 HTMLファイル構造

各HTMLファイルは以下の技術スタックで実装されています：

### 技術仕様
- **フロントエンド**: HTML5 + CSS3 + Vanilla JavaScript
- **数値計算**: 独自実装（外部ライブラリ依存なし）
- **可視化**: Canvas API + Chart.js
- **UI**: CSS Grid + Flexbox レスポンシブデザイン
- **フォント**: Google Fonts (Noto Sans JP + JetBrains Mono)

### コード構造
```javascript
// 典型的な実装パターン
class PowerFlowSolver {
    constructor(systemData) {
        this.Ybus = this.buildAdmittanceMatrix(systemData);
        this.busTypes = this.classifyBuses(systemData);
    }
    
    solve(algorithm, tolerance = 1e-8) {
        switch(algorithm) {
            case 'newton': return this.newtonRaphson(tolerance);
            case 'fastdec': return this.fastDecoupled(tolerance);
            case 'gauss': return this.gaussSeidel(tolerance);
        }
    }
}
```

### 数値計算実装
- **複素数演算**: カスタム Complex クラス
- **行列演算**: スパース行列対応
- **線形解法**: LU分解 + 前進後進代入
- **収束判定**: 最大値ノルム ||f||_∞ < ε

## 🎓 教育機能

### 可視化要素

- **収束曲線**: 各反復における誤差推移
- **複素平面軌跡**: 電圧の収束経路
- **Jacobian構造**: 係数行列のスパース性
- **誤差分布**: 母線別ミスマッチ表示
- **性能比較**: アルゴリズム間ベンチマーク

### 学習支援

- **数式表示**: LaTeX形式での厳密な数学的定式化
- **段階表示**: アルゴリズムの各ステップを明示
- **パラメータ感度**: 設定値変更の影響を実時間表示
- **誤差分析**: 収束判定基準の理解促進

## 🔧 技術仕様

### 対応ブラウザ
- Chrome 80+
- Firefox 75+
- Safari 13+
- Edge 80+

### 必要な機能
- JavaScript ES6+
- Canvas API
- WebGL（一部可視化）
- Local Storage

### 外部ライブラリ
- **数値計算**: 独自実装（依存ライブラリなし）
- **可視化**: Canvas API + 独自描画エンジン
- **UI**: Vanilla JavaScript
- **フォント**: Google Fonts (Noto Sans JP, JetBrains Mono)

## 📈 系統データ

### IEEE標準系統

| 系統 | 母線数 | 枝数 | 発電機数 | 特徴 |
|---|---|---|---|---|
| IEEE 2-bus | 2 | 1 | 1 | 最小系統 |
| IEEE 5-bus | 5 | 7 | 2 | 基本系統 |
| IEEE 9-bus | 9 | 9 | 3 | WSCC 3-machine |
| IEEE 14-bus | 14 | 20 | 5 | 標準テスト系統 |
| IEEE 30-bus | 30 | 41 | 6 | 中規模系統 |

### MATPOWER互換性
- **完全準拠**: mpc.bus, mpc.gen, mpc.branch形式
- **データ変換**: 自動的な単位系変換
- **検証済み**: 公式MATPOWER結果との一致確認

## 📚 技術文書・学術資料

### 📖 技術文書
- [**潮流計算手法技術ノート**](./docs/power_flow_methods.md) - 数学的基礎理論と実装詳細
- [**各可視化ツール詳細文書**](./docs/) - 個別機能・アルゴリズム解説

### 🔬 学術的背景

#### 理論的基盤
本プロジェクトは以下の学術分野の理論に基づいています：

**電力系統解析理論**
- 非線形代数方程式の数値解法
- 複素電力の定式化理論
- アドミタンス行列の代数的性質

**数値解析理論**  
- Newton-Kantorovich定理による収束保証
- 不動点定理とBanach空間における縮小写像
- 分解原理による計算複雑度の削減

**最適化理論**
- 信頼領域法（Trust Region Methods）
- 連続変形法（Homotopy Continuation）  
- 非線形計画法における制約処理

#### 参考文献

**基本文献**
1. Bergen, A.R., Vittal, V. "Power Systems Analysis", 2nd ed., Prentice Hall, 2000
2. Kundur, P. "Power System Stability and Control", McGraw-Hill, 1994  
3. Grainger, J.J., Stevenson, W.D. "Power System Analysis", McGraw-Hill, 1994

**専門文献**  
4. Stott, B., Alsac, O. "Fast Decoupled Load Flow", IEEE Trans. Power App. Syst., 1974
5. Monticelli, A. "State Estimation in Electric Power Systems", Springer, 1999
6. Zimmerman, R.D., et al. "MATPOWER: Steady-State Operations, Planning and Analysis Tools", IEEE Trans. Power Syst., 2011

**最新研究**
7. Trias, A. "The Holomorphic Embedding Load Flow Method", IEEE Trans. Power Syst., 2012
8. Milano, F. "Continuous Newton's Method for Power Flow Analysis", IEEE Trans. Power Syst., 2009
9. Various recent papers in IEEE Transactions on Power Systems

## 🤝 対象ユーザー

### 学習者
- **電力系統工学専攻学生**: 理論と実践の橋渡し
- **新人エンジニア**: 実務的アルゴリズム理解
- **研究者**: 手法比較・性能評価

### 教育者
- **大学教員**: 講義での可視化教材
- **企業研修**: 実践的スキル向上
- **セミナー講師**: デモンストレーション

## 🔍 学習効果

### 理解促進要素
1. **視覚化学習**: 抽象的概念の具体化
2. **対話的操作**: パラメータ変更による理解深化
3. **比較学習**: 複数手法の同時比較
4. **段階的学習**: 基礎から応用への体系的習得

### 実践的スキル
1. **アルゴリズム選択**: 用途に応じた手法選定能力
2. **パラメータ調整**: 収束性改善技術
3. **結果解釈**: 計算結果の妥当性判断
4. **故障対応**: 非収束時の対処法習得

## 📝 ライセンス

このプロジェクトは教育目的で作成されています。

## 🔄 更新履歴

### 直感的理解ツール バージョン履歴
- **v6.0** (2024-12-31): **大規模系統対応・PVバス修正版**
  - IEEE 30バス・30バスランダム系統追加（最適化済み）
  - PVバスサポート修正（電圧制約の正しい処理）
  - 動的系統生成機能実装
  - スパース接続によるパフォーマンス最適化
- **v5.0** (2024-12-31): 問題修正・UI改善、数値安定性向上
- **v4.0** (2024-12-31): 最新機能統合版
- **v3.0** (2024-12-31): 遠い初期点設定、大規模系統対応
- **v2.0** (2024-12-30): 精度・収束可視化改善
- **v1.0** (2024-12-30): 初版リリース

### 計算過程可視化ツール
- **v2.0** (2024-12-31): **Newton-Raphson収束問題修正版**
  - 収束アルゴリズムの根本的修正
  - PVバス制約の適切な処理
  - 過度なダンピングの除去
- **v1.0** (2024-12-30): 初版リリース

### プロジェクト全体
- **v1.4** (2024-12-31): 30ノード系統最適化・安定性向上
- **v1.3** (2024-12-31): 大規模系統対応・計算過程修正
- **v1.2** (2024-12-31): 直感的理解ツールv5対応、問題修正
- **v1.1** (2024-12-30): 技術文書充実、GitHub Pages対応
- **v1.0** (2024-12-30): 初回リリース

## 🙏 謝辞

このプロジェクトは電力系統解析における教育の質向上を目的として開発されました。MATPOWERプロジェクト、IEEE標準系統データ、および電力系統解析分野の先駆的研究に深く感謝いたします。

---

**開発者**: [プロジェクト作成者]  
**更新日**: 2024年12月31日  
**バージョン**: 1.4.0