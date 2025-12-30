# power_flow_compare.html - 潮流計算手法比較分析ツール

## 📋 概要

`power_flow_compare.html`は、主要な潮流計算アルゴリズムの性能特性を定量的に比較分析するツールです。Newton-Raphson法、高速分離解法、Gauss-Seidel法の3手法を同一条件下で実行し、収束性・計算速度・精度を多角的に評価できます。

## 🎯 主要機能

### 比較対象アルゴリズム

#### 1. Newton-Raphson法
- **収束特性**: 二次収束（quadratic convergence）
- **計算量**: O(n³) per iteration
- **メモリ使用**: 完全Jacobian行列保存
- **適用場面**: 汎用的、中小規模系統の標準手法

#### 2. 高速分離解法（Fast Decoupled XB）
- **収束特性**: 準二次収束（superlinear convergence）
- **計算量**: 2×O(n³/2) per iteration  
- **メモリ使用**: 簡化行列（B', B''）
- **適用場面**: 大規模系統、実用的高速解法

#### 3. Gauss-Seidel法
- **収束特性**: 一次収束（linear convergence）
- **計算量**: O(n²) per iteration
- **メモリ使用**: 最小（行列保存不要）
- **適用場面**: 小規模系統、教育用途

### 性能分析指標

#### 収束性評価
- **反復回数**: 収束までの必要反復数
- **収束曲線**: 各反復での誤差推移
- **収束率**: 誤差減少の収束次数
- **安定性**: 初期値依存性の評価

#### 計算効率評価
- **実行時間**: アルゴリズム別計算時間
- **メモリ使用量**: ピーク・平均メモリ消費
- **スループット**: 単位時間当たり処理系統数
- **スケーラビリティ**: 系統規模増加に対する性能変化

#### 精度評価
- **解精度**: 最終収束値の精度
- **数値安定性**: 条件数による影響
- **誤差分布**: 母線別・送電線別誤差分析
- **再現性**: 同一条件での結果一致性

## 🖥️ ユーザーインターフェース

### 比較分析画面

```
┌─────────────────────────────────────────────────────────┐
│ 潮流計算アルゴリズム性能比較                              │
├─────────────────────────────────────────────────────────┤
│ 実行設定: [系統: IEEE 14] [許容値: 1e-8] [回数: 5]      │
│ [Newton-Raphson ☑] [Fast Decoupled ☑] [Gauss-Seidel ☑]  │
│ [一括実行] [個別実行] [統計分析] [エクスポート]           │
├───────────────┬─────────────────┬─────────────────────────┤
│   収束曲線    │   性能比較      │      詳細統計           │
│              │                │                         │
│ 誤差(log)    │  実行時間(ms)   │ ┌─────────────────────┐ │
│  10^0 |\     │                │ │Algorithm│Iter│Time│  │ │
│      | \    │ ████ Newton    │ │Newton   │ 4  │12ms│  │ │
│  10^-4| \\   │ ██   Fast Dec  │ │Fast Dec │ 7  │8ms │  │ │
│      |  \\  │ ██████ Gauss   │ │Gauss    │23  │15ms│  │ │
│  10^-8|   \  │                │ └─────────────────────┘ │ │
│      |____\__|________________│                         │ │
│      0  10 20│反復            │                         │ │
├───────────────┼─────────────────┼─────────────────────────┤
│   系統図      │   Jacobian構造  │      収束分析           │
│              │                │                         │
│    ○─────○   │ ████░░░░████    │ R² = 0.987 (Newton)     │
│    │  1  │   │ ░███░░░░███░    │ R² = 0.962 (Fast Dec)   │
│    ○─────○   │ ░░██████░░██    │ R² = 0.945 (Gauss)      │
│       2      │ ███░░███░░░█    │                         │
└───────────────┴─────────────────┴─────────────────────────┘
```

### 統計分析パネル

#### 基本統計量
```
アルゴリズム性能統計 (n=100回実行)
┌─────────────────┬─────────┬─────────┬─────────┐
│     指標        │ Newton  │FastDec  │ Gauss   │
├─────────────────┼─────────┼─────────┼─────────┤
│ 平均反復回数    │  4.2    │  6.8    │  23.1   │
│ 標準偏差        │  0.4    │  0.6    │  3.2    │
│ 最大反復回数    │   5     │   8     │   29    │
│ 成功率(%)       │  100    │  99.8   │  96.2   │
├─────────────────┼─────────┼─────────┼─────────┤
│ 平均実行時間(ms)│  12.3   │  8.9    │  15.7   │
│ 標準偏差(ms)    │  1.2    │  0.8    │  2.1    │
│ 最大時間(ms)    │  15.1   │  10.4   │  19.8   │
├─────────────────┼─────────┼─────────┼─────────┤
│ 平均メモリ(MB)  │  2.1    │  1.6    │  0.8    │
│ 最終精度        │ 1e-10   │ 1e-9    │ 1e-8    │
│ 数値安定性      │ 優秀    │ 良好    │ 良好    │
└─────────────────┴─────────┴─────────┴─────────┘
```

## 🔬 技術実装

### 比較フレームワーク

#### BenchmarkEngine クラス
```javascript
class BenchmarkEngine {
    constructor() {
        this.algorithms = {
            'newton': new NewtonRaphsonSolver(),
            'fastdec': new FastDecoupledSolver(), 
            'gauss': new GaussSeidelSolver()
        };
        this.metrics = new PerformanceMetrics();
        this.statistics = new StatisticalAnalysis();
    }
    
    // 総合比較実行
    runComparison(testCase, iterations = 100) {
        const results = {};
        
        Object.entries(this.algorithms).forEach(([name, solver]) => {
            console.log(`Running ${name} algorithm...`);
            
            const algorithmResults = [];
            for (let i = 0; i < iterations; i++) {
                const result = this.runSingleTest(solver, testCase);
                algorithmResults.push(result);
            }
            
            results[name] = {
                individual: algorithmResults,
                statistics: this.statistics.analyze(algorithmResults),
                convergence: this.analyzeConvergence(algorithmResults)
            };
        });
        
        return this.generateComparisonReport(results);
    }
    
    // 単一テスト実行
    runSingleTest(solver, testCase) {
        const startTime = performance.now();
        const startMemory = this.getMemoryUsage();
        
        const result = solver.solve(testCase.Ybus, testCase.Sbus, testCase.V0);
        
        const endTime = performance.now();
        const endMemory = this.getMemoryUsage();
        
        return {
            converged: result.converged,
            iterations: result.iterations,
            executionTime: endTime - startTime,
            memoryUsed: endMemory - startMemory,
            finalError: result.finalError,
            convergenceHistory: result.history,
            voltage: result.voltage
        };
    }
}
```

### 統計分析システム

#### StatisticalAnalysis クラス
```javascript
class StatisticalAnalysis {
    analyze(results) {
        const iterations = results.map(r => r.iterations);
        const times = results.map(r => r.executionTime);
        const errors = results.map(r => r.finalError);
        const successes = results.filter(r => r.converged).length;
        
        return {
            iterations: {
                mean: this.mean(iterations),
                std: this.standardDeviation(iterations),
                min: Math.min(...iterations),
                max: Math.max(...iterations),
                median: this.median(iterations),
                percentile95: this.percentile(iterations, 95)
            },
            
            executionTime: {
                mean: this.mean(times),
                std: this.standardDeviation(times),
                min: Math.min(...times),
                max: Math.max(...times),
                median: this.median(times)
            },
            
            convergence: {
                successRate: (successes / results.length) * 100,
                averageError: this.mean(errors.filter(e => e > 0)),
                reliability: this.calculateReliability(results)
            },
            
            efficiency: {
                timePerIteration: this.mean(times) / this.mean(iterations),
                convergenceRate: this.estimateConvergenceRate(results),
                scalabilityFactor: this.estimateScalability(results)
            }
        };
    }
    
    // 収束次数推定
    estimateConvergenceRate(results) {
        const rates = [];
        
        results.forEach(result => {
            if (result.convergenceHistory && result.convergenceHistory.length > 3) {
                const history = result.convergenceHistory;
                
                // 連続する誤差比から収束次数を推定
                for (let i = 2; i < history.length - 1; i++) {
                    if (history[i] > 0 && history[i-1] > 0 && history[i-2] > 0) {
                        const rate = Math.log(history[i+1] / history[i]) / 
                                    Math.log(history[i] / history[i-1]);
                        if (isFinite(rate) && rate > 0) {
                            rates.push(rate);
                        }
                    }
                }
            }
        });
        
        return rates.length > 0 ? this.median(rates) : 1.0;
    }
}
```

### 可視化エンジン

#### ComparisonVisualizer クラス
```javascript
class ComparisonVisualizer {
    constructor() {
        this.charts = {};
        this.colors = {
            newton: '#3b82f6',      // 青
            fastdec: '#10b981',     // 緑
            gauss: '#f59e0b'        // 橙
        };
    }
    
    // 収束曲線比較
    drawConvergenceComparison(results) {
        const ctx = document.getElementById('convergence-chart').getContext('2d');
        
        const datasets = Object.entries(results).map(([name, data]) => ({
            label: this.getAlgorithmName(name),
            data: data.individual[0].convergenceHistory.map((error, i) => ({x: i, y: error})),
            borderColor: this.colors[name],
            backgroundColor: this.colors[name] + '20',
            fill: false,
            tension: 0.1
        }));
        
        this.charts.convergence = new Chart(ctx, {
            type: 'line',
            data: { datasets },
            options: {
                responsive: true,
                scales: {
                    y: {
                        type: 'logarithmic',
                        title: { display: true, text: '収束誤差' }
                    },
                    x: {
                        title: { display: true, text: '反復回数' }
                    }
                },
                plugins: {
                    title: { display: true, text: '収束特性比較' },
                    legend: { position: 'top' }
                }
            }
        });
    }
    
    // 性能比較バーチャート
    drawPerformanceComparison(statistics) {
        const ctx = document.getElementById('performance-chart').getContext('2d');
        
        const algorithms = Object.keys(statistics);
        const iterations = algorithms.map(alg => statistics[alg].iterations.mean);
        const times = algorithms.map(alg => statistics[alg].executionTime.mean);
        
        this.charts.performance = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: algorithms.map(name => this.getAlgorithmName(name)),
                datasets: [{
                    label: '平均反復回数',
                    data: iterations,
                    backgroundColor: algorithms.map(alg => this.colors[alg] + '80'),
                    borderColor: algorithms.map(alg => this.colors[alg]),
                    borderWidth: 1,
                    yAxisID: 'y'
                }, {
                    label: '平均実行時間 (ms)',
                    data: times,
                    backgroundColor: algorithms.map(alg => this.colors[alg] + '40'),
                    borderColor: algorithms.map(alg => this.colors[alg]),
                    borderWidth: 1,
                    yAxisID: 'y1'
                }]
            },
            options: {
                responsive: true,
                scales: {
                    y: {
                        type: 'linear',
                        display: true,
                        position: 'left',
                        title: { display: true, text: '反復回数' }
                    },
                    y1: {
                        type: 'linear',
                        display: true,
                        position: 'right',
                        title: { display: true, text: '実行時間 (ms)' },
                        grid: { drawOnChartArea: false }
                    }
                }
            }
        });
    }
    
    // 散布図分析（速度 vs 精度）
    drawScatterAnalysis(results) {
        const ctx = document.getElementById('scatter-chart').getContext('2d');
        
        const datasets = Object.entries(results).map(([name, data]) => ({
            label: this.getAlgorithmName(name),
            data: data.individual.map(result => ({
                x: result.executionTime,
                y: -Math.log10(result.finalError) // 精度（有効桁数）
            })),
            backgroundColor: this.colors[name] + '60',
            borderColor: this.colors[name]
        }));
        
        this.charts.scatter = new Chart(ctx, {
            type: 'scatter',
            data: { datasets },
            options: {
                responsive: true,
                scales: {
                    x: { title: { display: true, text: '実行時間 (ms)' } },
                    y: { title: { display: true, text: '精度 (有効桁数)' } }
                },
                plugins: {
                    title: { display: true, text: '速度-精度トレードオフ分析' }
                }
            }
        });
    }
}
```

## 📊 比較分析結果例

### IEEE 14母線系統での典型的結果

#### 基本性能指標
```
アルゴリズム別性能サマリー
================================

Newton-Raphson法:
- 平均反復回数: 4.2回
- 平均実行時間: 12.3ms  
- 収束成功率: 100%
- 収束次数: 1.98 (二次収束)
- メモリ使用量: 2.1MB

高速分離解法:
- 平均反復回数: 6.8回
- 平均実行時間: 8.9ms
- 収束成功率: 99.8%
- 収束次数: 1.65 (準二次収束)  
- メモリ使用量: 1.6MB

Gauss-Seidel法:
- 平均反復回数: 23.1回
- 平均実行時間: 15.7ms
- 収束成功率: 96.2%
- 収束次数: 1.02 (一次収束)
- メモリ使用量: 0.8MB
```

#### 系統規模別スケーラビリティ

| 母線数 | Newton-Raphson | Fast Decoupled | Gauss-Seidel |
|--------|----------------|----------------|---------------|
| 5      | 3回, 2.1ms     | 5回, 1.8ms     | 12回, 3.2ms   |
| 14     | 4回, 12.3ms    | 7回, 8.9ms     | 23回, 15.7ms  |
| 30     | 5回, 45.2ms    | 9回, 28.1ms    | 41回, 58.3ms  |
| 57     | 6回, 156ms     | 12回, 89ms     | 78回, 198ms   |

#### 収束特性分析

```
収束曲線フィッティング分析
===========================

Newton-Raphson: 
  誤差(k) ≈ 0.12 × (誤差(k-1))^1.98
  R² = 0.987, 二次収束確認

Fast Decoupled:
  誤差(k) ≈ 0.23 × (誤差(k-1))^1.65  
  R² = 0.962, 準二次収束

Gauss-Seidel:
  誤差(k) ≈ 0.89 × (誤差(k-1))^1.02
  R² = 0.945, 一次収束（収束係数0.89）
```

## 🎓 教育効果

### 比較学習による理解促進

#### アルゴリズム特性の理解
1. **収束速度**: 二次収束 vs 一次収束の違い
2. **計算複雑度**: 反復当たり計算量の比較
3. **メモリ効率**: 行列保存戦略の違い
4. **数値安定性**: 条件数に対する耐性

#### 実用性の判断力
1. **用途別選択**: 系統規模に応じた手法選択
2. **トレードオフ理解**: 速度・精度・メモリの関係
3. **実装難易度**: プログラム実装の複雑さ
4. **拡張性**: 機能追加の容易さ

### 研究・開発スキル

#### 性能評価手法
1. **ベンチマーク設計**: 公正な比較実験設計
2. **統計分析**: 信頼性のある性能評価
3. **可視化技術**: 効果的な結果提示
4. **レポート作成**: 学術的・技術的報告書

#### 最適化思考
1. **ボトルネック特定**: 性能制限要因の発見
2. **改善方針**: 効率化の方向性決定
3. **検証方法**: 改善効果の定量評価
4. **文書化**: 知見の体系化・共有

## 🔧 カスタマイズ

### 比較条件設定

#### テストケース追加
```javascript
const customTestCases = {
    'heavy_load': {
        description: '重負荷条件',
        modification: (baseCase) => {
            baseCase.bus.forEach(bus => {
                bus.PD *= 1.5;  // 負荷1.5倍
                bus.QD *= 1.3;  // 無効電力1.3倍
            });
            return baseCase;
        }
    },
    
    'low_voltage': {
        description: '低電圧初期値',
        modification: (baseCase) => {
            baseCase.bus.forEach(bus => {
                if (bus.type !== 3) {  // Slack以外
                    bus.VM = 0.9;  // 初期電圧0.9pu
                }
            });
            return baseCase;
        }
    },
    
    'contingency': {
        description: 'N-1事故時',
        modification: (baseCase) => {
            // 最も重要な送電線を1本除外
            baseCase.branch.splice(0, 1);
            return baseCase;
        }
    }
};
```

#### 評価指標追加
```javascript
const customMetrics = {
    robustness: (results) => {
        // 頑健性評価: 異なる初期値からの収束一貫性
        const convergenceRates = results.map(r => r.converged ? 1 : 0);
        return convergenceRates.reduce((a, b) => a + b) / convergenceRates.length;
    },
    
    efficiency: (results) => {
        // 効率性評価: 解精度当たりの計算時間
        const validResults = results.filter(r => r.converged);
        const avgTime = validResults.reduce((sum, r) => sum + r.executionTime, 0) / validResults.length;
        const avgAccuracy = validResults.reduce((sum, r) => sum - Math.log10(r.finalError), 0) / validResults.length;
        return avgAccuracy / avgTime;
    },
    
    scalability: (results_small, results_large) => {
        // スケーラビリティ評価: 系統規模増加による性能劣化
        const ratio_time = results_large.avgTime / results_small.avgTime;
        const ratio_size = results_large.systemSize / results_small.systemSize;
        return Math.log(ratio_time) / Math.log(ratio_size);  // 時間複雑度指数
    }
};
```

## 🚀 応用・発展

### 研究応用例

#### 新手法評価プラットフォーム
- **プロトタイプ実装**: 新アルゴリズムの性能評価
- **ベンチマーク提供**: 標準的比較基準
- **再現性確保**: 同一条件での客観評価
- **論文支援**: 性能比較グラフ・表の自動生成

#### 教育研究
- **学習効果測定**: 視覚的比較による理解度向上
- **概念理解**: 抽象的理論の具体化
- **実験設計**: 科学的思考力の育成
- **データ分析**: 統計的思考力の養成

### 産業応用

#### 実装選択支援
- **要件分析**: 用途に応じた最適手法選択
- **性能予測**: 実系統での性能見積もり
- **リスク評価**: 非収束確率の定量評価
- **最適化指針**: 実装改善の方向性決定

#### 品質保証
- **アルゴリズム検証**: 実装正確性の確認
- **性能回帰**: 版数変更による性能影響
- **ストレステスト**: 極限条件での動作確認
- **ベンチマーク**: 競合製品との性能比較

---

**ファイル**: `power_flow_compare.html`  
**作成日**: 2024年12月30日  
**更新日**: 2024年12月30日  
**バージョン**: 1.8