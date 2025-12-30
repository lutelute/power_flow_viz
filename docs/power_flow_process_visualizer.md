# power_flow_process_visualizer.html - 潮流計算過程ステップ可視化

## 📋 概要

`power_flow_process_visualizer.html`は、潮流計算アルゴリズムの各計算ステップを段階的に可視化する教育特化型ツールです。初学者が潮流計算の流れを理解できるよう、計算過程を分解して表示し、各段階での数値変化を詳細に観察できます。

## 🎯 主要機能

### ステップバイステップ可視化

#### 計算過程の分解表示
1. **初期化段階**
   - 初期電圧設定（通常1.0∠0°）
   - 母線データ読み込み
   - アドミタンス行列構築

2. **ミスマッチ計算段階**
   - 指定電力 vs 計算電力
   - P, Q ミスマッチの算出
   - 誤差ベクトルの表示

3. **Jacobian行列段階**
   - 偏微分要素の計算
   - 行列構造の可視化
   - スパースパターン表示

4. **線形方程式求解段階**
   - LU分解過程
   - 前進・後進代入
   - 解ベクトル算出

5. **電圧更新段階**
   - Δθ, ΔV の適用
   - 新電圧値の計算
   - 更新前後の比較

6. **収束判定段階**
   - 収束基準チェック
   - 次反復への移行判断
   - 最終結果確認

### 実装アルゴリズム（5手法）

#### 1. Newton-Raphson法
- **特徴**: 最も一般的、二次収束
- **表示**: 完全Jacobian行列の構築過程
- **学習価値**: 偏微分概念の理解

#### 2. Gauss-Seidel法  
- **特徴**: 単純、逐次更新
- **表示**: 母線ごとの順次計算過程
- **学習価値**: 反復法の基本理解

#### 3. 高速分離解法
- **特徴**: P-δ/Q-V分離
- **表示**: 2つのサブ問題への分解
- **学習価値**: 実用的高速化手法

#### 4. DC潮流法
- **特徴**: 線形化、瞬時計算
- **表示**: 近似過程の可視化
- **学習価値**: 線形近似の理解

#### 5. Gauss法（Jacobi法）
- **特徴**: 同時更新、並列化可能
- **表示**: 全要素同時更新過程
- **学習価値**: 並列計算概念

## 🖥️ ユーザーインターフェース

### プロセス表示パネル

```
┌─────────────────────────────────────────────────┐
│ 現在のステップ: [3/6] Jacobian行列構築          │
├─────────────────────────────────────────────────┤
│ ◯ 1. 初期化完了     ◯ 2. ミスマッチ計算完了     │
│ ● 3. Jacobian構築中  ◯ 4. 線形方程式求解        │
│ ◯ 5. 電圧更新       ◯ 6. 収束判定             │
├─────────────────────────────────────────────────┤
│ [前のステップ] [次のステップ] [自動実行] [リセット] │
└─────────────────────────────────────────────────┘
```

### 詳細表示エリア

#### 数値表示パネル
```
現在の電圧 (反復2回目)
┌─────┬──────────┬──────────┬──────────┐
│母線 │ 電圧大きさ │  位相角   │ミスマッチ │
├─────┼──────────┼──────────┼──────────┤
│  1  │  1.060    │   0.00°  │  0.000   │
│  2  │  1.043    │  -4.25°  │ -0.086   │
│  3  │  1.011    │ -12.35°  │  0.034   │
└─────┴──────────┴──────────┴──────────┘
```

#### 行列表示
```
Jacobian行列 (4×4)
┌─────────────────────────────────────┐
│  15.2  -5.1   2.3  -0.8  │ ∂P/∂θ  │
│  -5.1  12.7  -3.2   1.1  │         │
│  ├─────────────────────────┤         │
│   2.1  -0.9   8.4  -2.3  │ ∂Q/∂θ  │
│  -0.7   1.2  -2.1   6.8  │         │
└─────────────────────────────────────┘
      ∂P/∂θ     ∂Q/∂V
```

### ネットワーク図

#### リアルタイム状態表示
- **母線色分け**: 種類別（Slack/PV/PQ）
- **ミスマッチ表示**: 誤差の大きさを色の濃淡で表現
- **更新アニメーション**: 電圧変化の動的表示
- **フロー表示**: 送電線電力潮流の矢印

## 🔬 技術実装

### ステップ管理システム

#### ProcessController クラス
```javascript
class ProcessController {
    constructor() {
        this.currentStep = 0;
        this.maxSteps = 6;
        this.stepData = {};
        this.isRunning = false;
        this.autoMode = false;
    }
    
    nextStep() {
        if (this.currentStep < this.maxSteps) {
            this.executeStep(this.currentStep + 1);
            this.currentStep++;
            this.updateDisplay();
        }
    }
    
    executeStep(stepNumber) {
        switch(stepNumber) {
            case 1: this.initializeSystem(); break;
            case 2: this.calculateMismatch(); break;
            case 3: this.buildJacobian(); break;
            case 4: this.solveLinearSystem(); break;
            case 5: this.updateVoltages(); break;
            case 6: this.checkConvergence(); break;
        }
    }
    
    // 各ステップの詳細実装...
}
```

### 段階別実装詳細

#### ステップ1: 初期化
```javascript
initializeSystem() {
    // 初期電圧設定
    this.voltages = this.busData.map(bus => {
        if (bus.type === 'slack') {
            return {magnitude: bus.V, angle: 0};
        } else if (bus.type === 'PV') {
            return {magnitude: bus.V, angle: 0};
        } else {
            return {magnitude: 1.0, angle: 0};
        }
    });
    
    // アドミタンス行列構築
    this.Ybus = this.buildAdmittanceMatrix();
    
    // 表示更新
    this.displayVoltages();
    this.displayYbusMatrix();
    this.highlightCurrentOperation("システム初期化完了");
}
```

#### ステップ2: ミスマッチ計算
```javascript
calculateMismatch() {
    const mismatch = [];
    
    this.busData.forEach((bus, i) => {
        if (bus.type !== 'slack') {
            // 計算電力
            const S_calc = this.calculatePowerInjection(i);
            
            // 指定電力
            const S_spec = {P: bus.PG - bus.PD, Q: bus.QG - bus.QD};
            
            // ミスマッチ
            const deltaP = S_spec.P - S_calc.P;
            const deltaQ = S_spec.Q - S_calc.Q;
            
            mismatch.push({bus: i+1, deltaP, deltaQ});
        }
    });
    
    this.mismatchData = mismatch;
    this.displayMismatch();
    this.highlightMismatchBuses();
}
```

#### ステップ3: Jacobian構築
```javascript
buildJacobian() {
    const n = this.busData.length;
    const J = new Array(2*n).fill(0).map(() => new Array(2*n).fill(0));
    
    // 各要素を順次計算・表示
    for (let i = 0; i < n; i++) {
        for (let j = 0; j < n; j++) {
            // ∂P/∂θ 要素
            J[i][j] = this.calculateDPDtheta(i, j);
            this.animateJacobianElement(i, j, J[i][j]);
            
            // ∂P/∂V 要素  
            J[i][j+n] = this.calculateDPDV(i, j);
            this.animateJacobianElement(i, j+n, J[i][j+n]);
            
            // ∂Q/∂θ 要素
            J[i+n][j] = this.calculateDQDtheta(i, j);
            this.animateJacobianElement(i+n, j, J[i+n][j]);
            
            // ∂Q/∂V 要素
            J[i+n][j+n] = this.calculateDQDV(i, j);
            this.animateJacobianElement(i+n, j+n, J[i+n][j+n]);
        }
    }
    
    this.jacobianMatrix = J;
    this.displayJacobianStructure();
}
```

### 教育的表示機能

#### 数式表示システム
```javascript
class EquationDisplay {
    showCurrentEquation(stepNumber) {
        const equations = {
            1: "V₀ = 1.0∠0° (初期電圧)",
            2: "ΔP = P_spec - P_calc = P_spec - Re(V*·(YV)*)",
            3: "J = [∂P/∂θ  ∂P/∂|V|]\n    [∂Q/∂θ  ∂Q/∂|V|]",
            4: "J·Δx = -f(x) → LU分解 → 前進後進代入",
            5: "θ_new = θ_old + Δθ\n|V|_new = |V|_old + Δ|V|",
            6: "max|f(x)| < ε ? 収束 : 次反復"
        };
        
        this.displayEquation(equations[stepNumber]);
    }
    
    displayEquation(latexString) {
        // MathJax または独自レンダラーで数式表示
        const mathElement = document.getElementById('current-equation');
        mathElement.innerHTML = this.renderLatex(latexString);
    }
}
```

#### アニメーション制御
```javascript
class AnimationController {
    animateVoltageUpdate(busIndex, oldV, newV) {
        const duration = 1000; // 1秒
        const steps = 50;
        const stepDuration = duration / steps;
        
        for (let step = 0; step <= steps; step++) {
            setTimeout(() => {
                const ratio = step / steps;
                const currentV = this.interpolateVoltage(oldV, newV, ratio);
                this.updateBusDisplay(busIndex, currentV);
            }, step * stepDuration);
        }
    }
    
    highlightMatrixElement(row, col, value) {
        const cell = document.getElementById(`matrix-${row}-${col}`);
        cell.classList.add('highlight');
        cell.textContent = value.toFixed(4);
        
        setTimeout(() => {
            cell.classList.remove('highlight');
        }, 1500);
    }
}
```

## 📊 可視化例

### 収束過程の段階表示

```
反復1回目:
ステップ2: ミスマッチ計算
┌─────────────────────────────┐
│ 母線2: ΔP = -0.086 MW       │
│ 母線3: ΔP =  0.034 MW       │
│ 母線2: ΔQ = -0.023 MVAR     │
│ 母線3: ΔQ =  0.012 MVAR     │
│ 最大誤差: 0.086 MW          │
└─────────────────────────────┘

ステップ3: Jacobian構築中...
[∂P₂/∂θ₂] = -12.35  ←計算中
[∂P₂/∂θ₃] =   4.21  
[∂P₂/∂V₂] =   8.94  
[∂P₂/∂V₃] =  -2.18  
```

### ネットワーク状態変化

```
反復前:                反復後:
  ┌─[1]─┐                ┌─[1]─┐
  │1.06∠0°│              │1.06∠0°│
  └─────┘                └─────┘
      │                      │
  ┌─[2]─┐                ┌─[2]─┐
  │1.00∠0°│ → 更新 →      │1.04∠-4°│
  └─────┘                └─────┘
      │                      │
  ┌─[3]─┐                ┌─[3]─┐
  │1.00∠0°│              │1.01∠-12°│
  └─────┘                └─────┘

色分け: 緑=収束, 黄=更新中, 赤=大誤差
```

## 🎓 教育効果

### 学習目標

#### 基礎理解レベル
1. **概念理解**: 潮流計算の目的と意義
2. **流れ理解**: 計算手順の全体像把握
3. **用語理解**: 専門用語の正確な理解

#### 応用理解レベル
1. **数学理解**: 数式の物理的意味
2. **アルゴリズム理解**: 各手法の特徴と使い分け
3. **実装理解**: プログラム化の考え方

#### 発展理解レベル
1. **理論理解**: 収束理論の数学的背景
2. **最適化理解**: 計算効率改善方法
3. **応用理解**: 実系統への適用課題

### インタラクティブ学習

#### 能動的学習機能
- **予測機能**: 次の値を予想してから確認
- **比較機能**: 異なる手法の同一ステップ比較
- **実験機能**: パラメータ変更の影響確認

#### 段階的習得
1. **観察段階**: 計算過程の単純観察
2. **理解段階**: 各ステップの意味理解
3. **予測段階**: 次の結果の予想
4. **応用段階**: 他系統への適用

## 🔧 カスタマイズ機能

### 表示設定

#### 詳細レベル設定
```javascript
const displayLevels = {
    beginner: {
        showEquations: false,    // 数式非表示
        showMatrix: false,       // 行列詳細非表示
        animationSpeed: 'slow',  // ゆっくり
        explanationLevel: 'detailed'  // 詳細説明
    },
    intermediate: {
        showEquations: true,     // 数式表示
        showMatrix: true,        // 行列構造表示
        animationSpeed: 'medium', // 中速
        explanationLevel: 'moderate'  // 適度説明
    },
    advanced: {
        showEquations: true,     // 完全数式
        showMatrix: true,        // 全行列要素
        animationSpeed: 'fast',  // 高速
        explanationLevel: 'minimal'  // 最小説明
    }
};
```

### 学習支援機能

#### クイズモード
```javascript
function generateQuiz(currentStep) {
    const quizzes = {
        2: "次のミスマッチが最も大きい母線は？",
        3: "Jacobian行列の(2,3)要素は何を表す？",
        4: "LU分解を使う理由は？",
        5: "電圧位相角の更新量が負の意味は？"
    };
    
    return {
        question: quizzes[currentStep],
        options: generateOptions(currentStep),
        correctAnswer: calculateCorrectAnswer(currentStep)
    };
}
```

#### ヒント機能
```javascript
function provideHint(stepNumber, difficulty) {
    const hints = {
        beginner: "この段階では電力の不足分を計算しています",
        intermediate: "ΔP = P指定 - P計算 = P指定 - Re(V*・(Y・V)*)",
        advanced: "ミスマッチベクトルf(x) = [ΔP; ΔQ]の構築"
    };
    
    return hints[difficulty];
}
```

## 🚀 今後の拡張

### 予定機能
1. **3D可視化**: 複素電圧平面での軌跡表示
2. **音声ガイド**: ステップの音声説明
3. **VR対応**: 没入型学習環境
4. **AI指導**: 個人適応型学習支援

### 技術改良
1. **WebGL活用**: 高速描画とスムーズアニメーション
2. **WebWorker**: バックグラウンド計算
3. **PWA対応**: オフライン使用可能
4. **モバイル最適化**: スマホ・タブレット対応

---

**ファイル**: `power_flow_process_visualizer.html`  
**作成日**: 2024年12月30日  
**更新日**: 2024年12月30日  
**バージョン**: 1.5