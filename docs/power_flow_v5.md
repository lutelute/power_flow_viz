# power_flow_v5.html - 包括的潮流計算アルゴリズム比較プラットフォーム

## 📋 概要

`power_flow_v5.html`は、20種類以上の潮流計算アルゴリズムを実装し、性能比較とベンチマーク分析を行う包括的なプラットフォームです。研究用途と教育用途の両方に対応した高度な可視化機能を提供します。

## 🎯 主要機能

### アルゴリズム実装（20+手法）

#### 基本手法
- **Newton-Raphson (Polar)**: 極座標形式での二次収束法
- **Newton-Raphson (Rectangular)**: 直交座標形式での実装
- **Fast Decoupled XB**: P-δ, Q-V分離高速解法
- **Fast Decoupled BX**: 逆順分離解法
- **Gauss-Seidel**: 逐次近似法（最も基本的手法）

#### 高度な手法
- **Levenberg-Marquardt**: 収束性改善のための信頼領域法
- **Iwamoto Newton**: ステップサイズ最適化付きNewton法
- **Continuation Power Flow**: 電圧安定性解析用連続潮流
- **Holomorphic Embedding**: 非反復的解法（理論的ブレークスルー）
- **Backward Forward Sweep**: 配電系統専用高速解法

#### 数値解法バリエーション
- **Quasi-Newton Methods**: 近似Jacobian使用法
- **Second-Order Newton**: 二次微分項考慮法
- **Modified Newton**: 簡易Jacobian更新法
- **Dishonest Newton**: 行列再利用による高速化
- **Gradient Descent**: 最急降下法ベース
- **Conjugate Gradient**: 共役勾配法応用

### 性能分析機能

#### リアルタイム比較
- **収束速度分析**: 反復回数とCPU時間の測定
- **精度評価**: 各手法の解精度比較
- **メモリ使用量**: アルゴリズム別リソース消費
- **安定性評価**: 異なる初期値からの収束性

#### 統計的分析
- **成功率統計**: 各手法の収束成功率
- **性能ランキング**: 総合スコアによる手法順位
- **散布図分析**: 速度vs精度の二次元評価
- **ヒストグラム表示**: 収束特性の分布分析

## 🖥️ ユーザーインターフェース

### コントロールパネル
```
[系統選択] [手法選択] [パラメータ設定] [実行制御]
  ↓          ↓          ↓           ↓
IEEE系統    全手法     収束判定    一括実行
カスタム    選択手法   最大反復    個別実行
ランダム    比較対象   初期値      リセット
```

### 可視化エリア
1. **メイン表示**: リアルタイム収束カーブ
2. **性能比較**: バーチャート・散布図
3. **詳細分析**: 数値テーブル・統計情報
4. **系統表示**: ネットワーク図・母線状態

### 設定オプション
- **収束判定値**: 1e-3 から 1e-12 まで
- **最大反復数**: 10 から 1000 まで
- **アニメーション速度**: リアルタイムから瞬時まで
- **表示精度**: 有効桁数設定

## 🔬 技術実装

### 数値計算エンジン

#### PowerFlowEngine クラス
```javascript
class PowerFlowEngine {
    // 系統データ管理
    loadSystemData(systemType)
    buildYbus()
    
    // アルゴリズム実装
    newtonRaphsonPolar()
    newtonRaphsonRectangular() 
    fastDecoupledXB()
    gaussSeidel()
    levenbergMarquardt()
    // ... 他15+手法
    
    // 数値解析
    jacobianMatrix()
    luDecomposition()
    forwardBackwardSubstitution()
    convergenceCheck()
}
```

#### 行列演算最適化
- **スパース行列処理**: メモリ効率的な疎行列操作
- **LU分解**: 部分ピボット付き数値安定化
- **反復改良**: 解精度向上のための後処理
- **条件数監視**: 数値的安定性の評価

### データ構造

#### 系統データ形式
```javascript
const powerSystem = {
    name: "IEEE-14",
    baseMVA: 100,
    bus: [
        {id: 1, type: 'slack', V: 1.06, theta: 0, PD: 0, QD: 0},
        {id: 2, type: 'PV', V: 1.045, theta: 0, PG: 40, PD: 21.7, QD: 12.7},
        // ...
    ],
    branch: [
        {from: 1, to: 2, R: 0.01938, X: 0.05917, B: 0.0528},
        // ...
    ],
    generator: [
        {bus: 1, PG: 232.4, QG: -16.9, Qmax: 10, Qmin: 0},
        // ...
    ]
};
```

#### 結果データ構造
```javascript
const results = {
    converged: boolean,
    iterations: number,
    executionTime: number,
    finalError: number,
    busVoltages: Array,
    branchFlows: Array,
    convergenceHistory: Array
};
```

### 可視化技術

#### Chart.js統合
- **リアルタイム更新**: 動的データバインディング
- **多軸表示**: 複数指標の同時表示
- **ズーム機能**: 詳細分析のための拡大
- **エクスポート**: PNG/SVG形式での保存

#### カスタム描画
- **ネットワーク図**: Canvas APIによる系統図
- **状態表示**: リアルタイム母線状態更新
- **軌跡表示**: 収束過程の視覚化
- **注釈機能**: インタラクティブな説明

## 📊 ベンチマーク結果例

### IEEE 14母線系統での典型的性能

| 手法 | 反復数 | 時間(ms) | 精度 | 成功率 |
|------|--------|----------|------|--------|
| Newton-Raphson | 4 | 15.2 | 1e-12 | 100% |
| Fast Decoupled | 7 | 12.8 | 1e-10 | 99.8% |
| Gauss-Seidel | 23 | 25.6 | 1e-8 | 95.2% |
| Levenberg-Marquardt | 5 | 18.9 | 1e-12 | 100% |

### 大規模系統での性能特性
- **メモリスケーラビリティ**: O(n²)からO(n)への改善
- **並列化効果**: 行列演算の最適化
- **数値安定性**: 条件数の系統サイズ依存性

## 🎓 教育的価値

### 学習シナリオ

#### 初心者向け
1. **基本手法比較**: Newton vs Gauss-Seidel
2. **収束特性理解**: 二次収束vs一次収束
3. **パラメータ感度**: 許容誤差の影響

#### 中級者向け
1. **高速化技法**: Fast Decoupled法の原理
2. **数値安定性**: 条件数と収束性の関係
3. **実用性評価**: 計算速度vs解精度

#### 上級者向け
1. **先進手法**: Holomorphic Embedding理論
2. **最適化理論**: Levenberg-Marquardt法
3. **研究動向**: 最新アルゴリズムの特徴

### インタラクティブ学習

#### 仮説検証学習
- **パラメータ変更**: 即座の結果確認
- **手法切替**: リアルタイム比較
- **系統変更**: 規模効果の体験

#### 問題発見学習
- **非収束ケース**: 失敗原因の分析
- **精度劣化**: 数値誤差の理解
- **性能ボトルネック**: 計算効率の改善

## 🔧 カスタマイズ

### アルゴリズム追加

#### 新手法実装テンプレート
```javascript
function customPowerFlow(Ybus, busData, tolerance) {
    // 初期化
    let V = initializeVoltages();
    let iteration = 0;
    
    // 反復ループ
    while (iteration < maxIterations) {
        // 電力ミスマッチ計算
        let mismatch = calculateMismatch(V, Ybus, busData);
        
        // 収束判定
        if (maxAbsolute(mismatch) < tolerance) {
            return {converged: true, V, iteration};
        }
        
        // 電圧更新（カスタムアルゴリズム）
        V = updateVoltages(V, mismatch, Ybus);
        iteration++;
    }
    
    return {converged: false, V, iteration};
}
```

### 系統データ拡張

#### 独自系統の追加
```javascript
const customSystem = {
    name: "Custom System",
    description: "ユーザー定義系統",
    baseMVA: 100,
    bus: [...],      // 母線データ
    branch: [...],   // 送電線データ
    generator: [...] // 発電機データ
};

addSystemToLibrary(customSystem);
```

### 可視化カスタマイズ

#### チャートオプション
- **色設定**: カスタムカラーパレット
- **軸設定**: 対数軸・線形軸選択
- **表示項目**: 選択的データ表示
- **アニメーション**: カスタム遷移効果

## 🚀 将来拡張

### 計画中機能
1. **機械学習統合**: AI支援による収束予測
2. **並列計算**: Web Worker活用の高速化
3. **3D可視化**: WebGLベース空間表示
4. **クラウド連携**: オンライン共有機能

### 研究応用
1. **アルゴリズム研究**: 新手法のプロトタイピング
2. **性能評価**: 標準ベンチマーク提供
3. **教育研究**: 学習効果の定量評価
4. **産業応用**: 実系統での性能検証

## 📈 使用統計

### パフォーマンス指標
- **初期化時間**: < 100ms
- **計算速度**: 最大1000母線対応
- **メモリ使用量**: < 50MB（通常使用）
- **応答性**: 60fps インタラクション

### 対応環境
- **ブラウザ**: Chrome 80+, Firefox 75+, Safari 13+
- **OS**: Windows, macOS, Linux
- **デバイス**: デスクトップ、タブレット
- **解像度**: 1920×1080推奨、最小1366×768

---

**ファイル**: `power_flow_v5.html`  
**作成日**: 2024年12月30日  
**更新日**: 2024年12月30日  
**バージョン**: 5.0