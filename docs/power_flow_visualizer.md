# power_flow_visualizer.html - 多手法潮流計算可視化プラットフォーム

## 📋 概要

`power_flow_visualizer.html`は、主要な潮流計算アルゴリズムの動作を直感的に理解できる教育的可視化ツールです。アルゴリズムの特徴、収束過程、計算精度を視覚的に比較学習できる設計となっています。

## 🎯 主要機能

### 実装アルゴリズム

#### 基本手法（5種）
1. **Newton-Raphson法**
   - **特徴**: 二次収束、最も一般的
   - **数式**: J·Δx = -f(x)
   - **適用**: 中小規模系統の標準解法

2. **Gauss-Seidel法**
   - **特徴**: 一次収束、単純実装
   - **数式**: V_i^(k+1) = (S_i*/V_i* - Σ Y_ij·V_j)/Y_ii
   - **適用**: 小規模系統、教育用途

3. **高速分離解法（Fast Decoupled）**
   - **特徴**: P-δ/Q-V分離、高速
   - **数式**: B'·Δδ = ΔP/V, B''·ΔV = ΔQ/V
   - **適用**: 大規模系統の実用解法

4. **DC潮流法**
   - **特徴**: 線形化、超高速
   - **数式**: P = B·δ
   - **適用**: 経済負荷配分、概算計算

5. **Gauss法（Jacobi）**
   - **特徴**: 同時更新、並列化可能
   - **数式**: 全要素同時更新版Gauss-Seidel
   - **適用**: 理論学習、並列処理研究

### 可視化機能

#### リアルタイム収束表示
- **収束曲線**: 各手法の誤差推移をリアルタイム描画
- **比較表示**: 複数手法の同時実行・比較
- **動的更新**: パラメータ変更の即座反映
- **アニメーション**: 段階的収束過程の視覚化

#### 性能分析チャート
- **散布図**: 収束速度vs計算精度
- **バーチャート**: アルゴリズム別性能比較
- **時系列**: 計算時間の推移表示
- **ヒートマップ**: 系統規模vs性能マップ

#### ネットワーク可視化
- **系統図**: 母線・送電線のグラフィカル表示
- **状態表示**: 各母線の電圧・電力状態
- **フロー表示**: 送電線電力潮流の矢印表示
- **ミスマッチ表示**: 収束過程での誤差分布

## 🖥️ ユーザーインターフェース

### メイン画面構成

```
┌─────────────────────────────────────────────────┐
│ ヘッダー：潮流計算アルゴリズム可視化             │
├─────────────────────────────────────────────────┤
│ コントロールパネル                               │
│ [系統] [手法] [パラメータ] [表示] [実行]         │
├─────────────────┬───────────────────────────────┤
│     主要表示    │        サイドパネル           │
│                 │                               │
│   収束曲線      │    詳細設定                   │
│   性能比較      │    数値結果                   │
│   系統図        │    統計情報                   │
│                 │    説明パネル                 │
└─────────────────┴───────────────────────────────┘
```

### コントロール要素

#### 系統選択
```javascript
const systemOptions = [
    "IEEE 2-bus",    // 最小テスト系統
    "IEEE 5-bus",    // 基本学習用
    "IEEE 9-bus",    // WSCC 3機系統
    "IEEE 14-bus",   // 標準テスト系統
    "IEEE 30-bus",   // 中規模系統
    "Random Network" // ランダム生成
];
```

#### アルゴリズム選択
- **単一実行**: 1手法の詳細分析
- **比較実行**: 2-3手法の同時比較
- **全手法実行**: 性能ベンチマーク
- **カスタム**: ユーザー定義組み合わせ

#### パラメータ設定
- **収束判定値**: 1e-3 ～ 1e-12
- **最大反復数**: 10 ～ 500
- **アニメーション速度**: 0.1秒 ～ 2.0秒
- **表示精度**: 3桁 ～ 8桁

## 🔬 技術実装

### アーキテクチャ

#### モジュール構成
```javascript
// 主要クラス
class PowerFlowVisualizer {
    // システム管理
    SystemManager
    AlgorithmEngine
    VisualizationController
    UIController
    
    // データ管理
    ResultsDatabase
    ConfigurationManager
    EventHandler
}
```

#### データフロー
```
User Input → Configuration → Algorithm Engine → Results → Visualization
     ↑                                                          ↓
     ←─────────────── Event Handler ←─────────────────────────
```

### 数値計算実装

#### Newton-Raphson実装例
```javascript
function newtonRaphson(Ybus, Sbus, V0, tolerance) {
    let V = [...V0];
    let iteration = 0;
    const maxIter = 100;
    const convergenceHistory = [];
    
    while (iteration < maxIter) {
        // 電力ミスマッチ計算
        const S_calc = calculatePowerInjection(V, Ybus);
        const mismatch = subtractComplex(Sbus, S_calc);
        
        // 収束判定
        const error = maxAbsolute(mismatch);
        convergenceHistory.push(error);
        
        if (error < tolerance) {
            return {
                converged: true,
                voltage: V,
                iterations: iteration,
                history: convergenceHistory
            };
        }
        
        // Jacobian行列構築
        const J = buildJacobian(V, Ybus);
        
        // 線形方程式求解: J·Δx = -f
        const delta = solveLinearSystem(J, mismatch);
        
        // 電圧更新
        V = updateVoltages(V, delta);
        iteration++;
    }
    
    return {
        converged: false,
        voltage: V,
        iterations: iteration,
        history: convergenceHistory
    };
}
```

#### 高速分離解法実装例
```javascript
function fastDecoupled(Ybus, Pbus, Qbus, V0, tolerance) {
    // B'およびB''行列構築（定数行列）
    const Bp = buildBprimeMatrix(Ybus);
    const Bpp = buildBdoubleprimeMatrix(Ybus);
    
    // LU分解（一回のみ）
    const LU_Bp = luDecomposition(Bp);
    const LU_Bpp = luDecomposition(Bpp);
    
    let V = [...V0];
    let theta = extractAngles(V);
    let vmag = extractMagnitudes(V);
    
    for (let iter = 0; iter < maxIterations; iter++) {
        // P-θ サブ問題
        const deltaP = calculatePowerMismatch(V, Pbus);
        const deltaTheta = solveLU(LU_Bp, deltaP);
        theta = addArrays(theta, deltaTheta);
        
        // Q-V サブ問題
        V = createComplexVoltage(vmag, theta);
        const deltaQ = calculateReactiveMismatch(V, Qbus);
        const deltaV = solveLU(LU_Bpp, deltaQ);
        vmag = addArrays(vmag, deltaV);
        
        // 収束判定
        V = createComplexVoltage(vmag, theta);
        if (checkConvergence(V, Pbus, Qbus, tolerance)) {
            return {converged: true, voltage: V, iterations: iter};
        }
    }
    
    return {converged: false, voltage: V, iterations: maxIterations};
}
```

### 可視化エンジン

#### Chart.js統合
```javascript
class ConvergenceChart {
    constructor(canvasId) {
        this.chart = new Chart(canvasId, {
            type: 'line',
            data: {
                datasets: []
            },
            options: {
                responsive: true,
                scales: {
                    y: {
                        type: 'logarithmic',
                        title: {
                            display: true,
                            text: '収束誤差'
                        }
                    },
                    x: {
                        title: {
                            display: true,
                            text: '反復回数'
                        }
                    }
                },
                plugins: {
                    legend: {
                        position: 'top'
                    },
                    tooltip: {
                        mode: 'index',
                        intersect: false
                    }
                }
            }
        });
    }
    
    addAlgorithm(name, data, color) {
        this.chart.data.datasets.push({
            label: name,
            data: data,
            borderColor: color,
            backgroundColor: color + '20',
            fill: false
        });
        this.chart.update();
    }
}
```

#### ネットワーク描画
```javascript
class NetworkRenderer {
    constructor(canvas) {
        this.ctx = canvas.getContext('2d');
        this.width = canvas.width;
        this.height = canvas.height;
    }
    
    drawNetwork(busData, branchData, voltages) {
        this.clearCanvas();
        
        // 母線描画
        busData.forEach(bus => {
            const voltage = voltages[bus.id - 1];
            const color = this.getVoltageColor(voltage);
            this.drawBus(bus.x, bus.y, bus.id, color);
        });
        
        // 送電線描画
        branchData.forEach(branch => {
            const flow = this.calculatePowerFlow(branch, voltages);
            this.drawBranch(branch, flow);
        });
    }
    
    getVoltageColor(voltage) {
        const magnitude = Math.abs(voltage);
        if (magnitude > 1.1) return '#ff4757';      // 過電圧
        if (magnitude < 0.9) return '#5352ed';     // 不足電圧
        return '#2ed573';                          // 正常電圧
    }
}
```

## 📊 可視化例

### 収束特性比較

```
Newton-Raphson vs Gauss-Seidel (IEEE 14-bus)

収束誤差
10^0  |*
      | *
10^-2 |  *
      |   **
10^-4 |     **
      |       ***
10^-6 |          ****
      |             ◇◇◇◇
10^-8 |                ◇◇◇◇
      |                    ◇◇◇
10^-10|________________________
      0    5    10    15    20
                反復回数

* Newton-Raphson (4回で収束)
◇ Gauss-Seidel (18回で収束)
```

### 性能比較

| アルゴリズム | 反復数 | 時間(ms) | 精度 | 使用メモリ |
|-------------|--------|----------|------|-----------|
| Newton-Raphson | 4 | 12.3 | 1e-10 | 2.1MB |
| 高速分離解法 | 7 | 8.9 | 1e-8 | 1.8MB |
| Gauss-Seidel | 18 | 15.7 | 1e-8 | 1.2MB |
| DC潮流 | 1 | 1.2 | 1e-2 | 0.8MB |

## 🎓 教育機能

### 学習モード

#### 初心者向け
1. **基本概念**: 潮流計算とは何か
2. **手法比較**: 各アルゴリズムの特徴
3. **収束理解**: なぜ反復計算するのか
4. **パラメータ影響**: 設定値の意味と効果

#### 中級者向け
1. **数学的背景**: 数式の導出過程
2. **数値解析**: Jacobian行列の意味
3. **実装詳細**: アルゴリズムの実装方法
4. **性能最適化**: 計算効率の改善

#### 上級者向け
1. **理論的考察**: 収束理論の詳細
2. **実用応用**: 実系統での適用例
3. **研究動向**: 最新手法の紹介
4. **カスタマイズ**: 独自アルゴリズム実装

### インタラクティブ学習

#### 仮説検証
- **パラメータ変更**: 即座の結果確認
- **系統変更**: 規模効果の体験
- **初期値変更**: 収束性の違い確認

#### 問題解決
- **非収束ケース**: 原因分析と対処法
- **精度問題**: 数値誤差の理解
- **計算効率**: ボトルネック特定

## 🔧 カスタマイズ

### 設定オプション

#### 表示設定
```javascript
const displayConfig = {
    showGrid: true,           // グリッド表示
    showLegend: true,         // 凡例表示
    showTooltip: true,        // ツールチップ
    animationDuration: 1000,  // アニメーション時間
    colorTheme: 'dark',       // カラーテーマ
    fontSize: 14              // フォントサイズ
};
```

#### アルゴリズム設定
```javascript
const algorithmConfig = {
    tolerance: 1e-6,          // 収束判定値
    maxIterations: 100,       // 最大反復数
    dampingFactor: 1.0,       // ダンピング係数
    initialVoltage: 1.0,      // 初期電圧
    accelerationFactor: 1.6   // 加速係数（Gauss-Seidel）
};
```

### 拡張機能

#### 新システム追加
```javascript
function addCustomSystem(systemData) {
    const newSystem = {
        name: systemData.name,
        description: systemData.description,
        bus: parseSystemData(systemData.bus),
        branch: parseSystemData(systemData.branch),
        generator: parseSystemData(systemData.generator)
    };
    
    systemLibrary.push(newSystem);
    updateSystemSelector();
}
```

#### 新可視化追加
```javascript
function addCustomVisualization(name, renderFunction) {
    visualizationModes[name] = {
        render: renderFunction,
        controls: generateControls(name),
        description: getDescription(name)
    };
    
    updateVisualizationSelector();
}
```

## 🚀 応用例

### 教育現場での活用

#### 大学講義
- **理論説明**: 数式と実動作の対応
- **演習**: パラメータ変更による学習
- **試験**: 理解度確認ツール

#### 企業研修
- **新人研修**: 基礎概念の理解
- **技術向上**: 実用的手法習得
- **問題解決**: 実際の課題解決

### 研究開発

#### アルゴリズム研究
- **新手法検証**: プロトタイプ実装
- **性能評価**: 既存手法との比較
- **理論検証**: 収束理論の確認

#### 教育研究
- **学習効果**: 可視化の効果測定
- **理解度調査**: 概念理解の分析
- **教材開発**: 効果的教材作成

---

**ファイル**: `power_flow_visualizer.html`  
**作成日**: 2024年12月30日  
**更新日**: 2024年12月30日  
**バージョン**: 2.0