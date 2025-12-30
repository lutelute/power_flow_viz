# power_flow_matpower_v2.html - MATPOWER準拠潮流計算実装

## 📋 概要

`power_flow_matpower_v2.html`は、電力系統解析の標準ソフトウェアであるMATPOWERとの完全互換性を持つ潮流計算実装です。実用的な系統解析機能と教育的可視化機能を両立させた、プロフェッショナル向けツールです。

## 🎯 主要機能

### MATPOWER完全準拠

#### データ形式互換性
- **mpc.bus**: 母線データ完全対応
- **mpc.gen**: 発電機データ完全対応  
- **mpc.branch**: 送電線データ完全対応
- **mpc.gencost**: 費用データ対応（将来拡張用）

#### IEEE標準系統サポート
- **IEEE 9-bus (WSCC)**: 3機9母線系統
- **IEEE 14-bus**: 標準テスト系統
- **IEEE 30-bus**: 中規模系統
- **IEEE 57-bus**: 大規模系統（将来対応）
- **IEEE 118-bus**: 超大規模系統（将来対応）

### アルゴリズム実装

#### Newton-Raphson法（極座標）
```matlab
% MATPOWER互換実装
function [V, converged, i] = newtonpf(Ybus, Sbus, V0, mpopt)
    % 完全にMATPOWER newtonpf.m と同等の処理
    tolerance = mpopt.pf.tol;
    max_it = mpopt.pf.nr.max_it;
    
    % 反復計算
    for i = 1:max_it
        [mismatch, J] = evaluate(Ybus, Sbus, V);
        
        if norm(mismatch, inf) < tolerance
            converged = 1;
            return;
        end
        
        dx = -J \ mismatch;
        V = update_voltage(V, dx);
    end
```

#### 高速分離解法（Fast Decoupled）
- **XB方式**: P-δ → Q-V順序
- **BX方式**: Q-V → P-δ順序
- **定数行列**: B', B''行列の事前LU分解

#### Gauss-Seidel法
- **電圧更新式**: MATPOWER準拠の更新アルゴリズム
- **加速係数**: 収束性改善オプション

## 🗃️ データ構造

### MATPOWER互換データ形式

#### mpc.bus (母線データ)
```javascript
const mpc = {
    bus: [
        // BUS_I TYPE PD QD GS BS AREA VM VA BASE_KV ZONE VMAX VMIN
        [1, 3, 0,    0,   0, 0, 1, 1.06,  0,   138, 1, 1.1, 0.9],
        [2, 2, 21.7, 12.7,0, 0, 1, 1.045,-4.98,138, 1, 1.1, 0.9],
        [3, 1, 94.2, 19,  0, 0, 1, 1.01, -12.72,138,1, 1.1, 0.9],
        // ...
    ]
};

// 母線種別
// 1: PQ母線（負荷母線）
// 2: PV母線（発電機母線）  
// 3: Slack母線（基準母線）
```

#### mpc.gen (発電機データ)
```javascript
mpc.gen = [
    // GEN_BUS PG QG QMAX QMIN VG MBASE STATUS PMAX PMIN
    [1, 232.4, -16.9, 10,   0,   1.06, 100, 1, 332.4, 0],
    [2, 40,     42.4, 50,  -40,  1.045,100, 1, 140,   0],
    [3, 0,      23.4, 40,   0,   1.01, 100, 1, 100,   0],
    // ...
];
```

#### mpc.branch (送電線データ)
```javascript
mpc.branch = [
    // F_BUS T_BUS R     X     B    RATEA RATEB RATEC RATIO ANGLE STATUS ANGMIN ANGMAX
    [1,    2,    0.01938,0.05917,0.0528,0,   0,   0,   0,   0,   1,    -360, 360],
    [1,    5,    0.05403,0.22304,0.0492,0,   0,   0,   0,   0,   1,    -360, 360],
    [2,    3,    0.04699,0.19797,0.0438,0,   0,   0,   0,   0,   1,    -360, 360],
    // ...
];
```

### 計算結果データ

#### results構造体
```javascript
const results = {
    // 基本結果
    success: boolean,        // 収束成功フラグ
    iterations: number,      // 反復回数
    et: number,             // 計算時間
    
    // 母線結果
    bus: [
        // BUS_I VM VA PD QD
        [1, 1.060, 0,    0,    0   ],
        [2, 1.045, -4.98, 21.7, 12.7],
        [3, 1.010, -12.72,94.2, 19  ],
        // ...
    ],
    
    // 発電機結果  
    gen: [
        // GEN_BUS PG QG
        [1, 232.4, -16.9],
        [2, 40,    42.4],
        [3, 0,     23.4],
        // ...
    ],
    
    // 送電線結果
    branch: [
        // F_BUS T_BUS PF QF PT QT
        [1, 2, 156.88, 75.55, -152.6, -61.89],
        [1, 5, 75.52,  18.58, -72.18, -2.23],
        // ...
    ]
};
```

## 🖥️ ユーザーインターフェース

### プロフェッショナル仕様UI

#### ヘッダー部
```
┌─────────────────────────────────────────────────────┐
│ ⚡ MATPOWER準拠 潮流計算                              │
│                                      [MATPOWER v7] │
├─────────────────────────────────────────────────────┤
│ 系統選択: [IEEE 14-bus ▼] アルゴリズム: [Newton-Raphson ▼] │
│ 許容値: [1e-8] 最大反復: [10] [実行] [リセット] [エクスポート] │
└─────────────────────────────────────────────────────┘
```

#### メイン表示エリア
```
┌─────────────────┬───────────────────────────────────┐
│   系統図表示    │        計算結果表示               │
│                 │                                   │
│   [ネットワーク] │  ┌─ 収束履歴 ─────────────────┐  │
│                 │  │ 反復 │ 最大誤差 │ 計算時間  │  │
│   [母線情報]    │  │  1   │ 0.234   │ 2.3ms    │  │
│                 │  │  2   │ 0.045   │ 1.8ms    │  │
│   [送電線情報]  │  │  3   │ 0.008   │ 1.5ms    │  │
│                 │  │  4   │ 0.001   │ 1.2ms    │  │
│                 │  └─────────────────────────────┘  │
├─────────────────┼───────────────────────────────────┤
│   詳細パネル    │        アルゴリズム情報           │
│                 │                                   │
│ ┌─ Jacobian ─┐ │  ┌─ 数式表示 ──────────────────┐  │
│ │ 15.2 -5.1  │ │  │ J·Δx = -f                   │  │
│ │ -5.1 12.7  │ │  │                             │  │
│ │ 2.1  -0.9  │ │  │ f = [ΔP] = [Pspec - Pcalc] │  │
│ │ -0.7  1.2  │ │  │     [ΔQ]   [Qspec - Qcalc] │  │
│ └───────────┘ │  └─────────────────────────────────┘  │
└─────────────────┴───────────────────────────────────┘
```

## 🔬 技術実装

### PowerFlowEngine クラス

#### 主要メソッド構成
```javascript
class PowerFlowEngine {
    constructor() {
        this.tolerance = 1e-8;
        this.maxIterations = 10;
        this.verbose = 1;
    }
    
    // MATPOWER互換メインAPI
    runpf(casedata, mpopt = {}) {
        this.loadCase(casedata);
        
        switch(mpopt.pf.alg) {
            case 'NR':  return this.newtonpf();
            case 'FDXB': return this.fdpf();
            case 'GS':  return this.gaussseidel(); 
            default:    return this.newtonpf();
        }
    }
    
    // 系統データ処理
    loadCase(casedata) {
        this.baseMVA = casedata.baseMVA || 100;
        this.bus = this.processBusData(casedata.bus);
        this.gen = this.processGenData(casedata.gen);
        this.branch = this.processBranchData(casedata.branch);
        
        this.buildYbus();
        this.setupBusTypes();
    }
    
    // アドミタンス行列構築
    buildYbus() {
        const nb = this.bus.length;
        this.Ybus = new ComplexMatrix(nb, nb);
        
        // 送電線アドミタンス
        this.branch.forEach(branch => {
            const f = branch.f_bus - 1;  // 0-indexed
            const t = branch.t_bus - 1;
            
            const r = branch.r;
            const x = branch.x; 
            const b = branch.b;
            
            const y = new Complex(r, x).inverse();
            const ysh = new Complex(0, b/2);
            
            // 自己アドミタンス
            this.Ybus.add(f, f, y.add(ysh));
            this.Ybus.add(t, t, y.add(ysh));
            
            // 相互アドミタンス
            this.Ybus.subtract(f, t, y);
            this.Ybus.subtract(t, f, y);
        });
        
        // 分路アドミタンス
        this.bus.forEach((bus, i) => {
            const ysh = new Complex(bus.gs, bus.bs);
            this.Ybus.add(i, i, ysh);
        });
    }
}
```

### Newton-Raphson実装

#### MATPOWER準拠アルゴリズム
```javascript
newtonpf() {
    let V = this.initializeVoltage();
    let converged = false;
    let i = 0;
    
    // 指定電力ベクトル
    const Sbus = this.makeSbus();
    
    // 反復計算
    while (!converged && i < this.maxIterations) {
        // 電力ミスマッチとJacobian評価
        const [mis, J] = this.evaluate(Ybus, Sbus, V);
        
        // 収束判定
        const normf = Math.max(...mis.map(Math.abs));
        if (normf < this.tolerance) {
            converged = true;
            break;
        }
        
        // Newton更新: Δx = -J\f
        const dx = this.solveLinearSystem(J, mis.map(x => -x));
        
        // 電圧更新
        V = this.updateVoltage(V, dx);
        i++;
        
        // 進捗表示（MATPOWER互換）
        if (this.verbose > 1) {
            console.log(`iter: ${i}, max P & Q mismatch: ${normf.toExponential(3)}`);
        }
    }
    
    if (this.verbose > 0) {
        if (converged) {
            console.log(`Newton's method power flow converged in ${i} iterations.`);
        } else {
            console.log(`Newton's method power flow did not converge in ${i} iterations.`);
        }
    }
    
    return this.packageResults(V, converged, i);
}
```

### 高速分離解法実装

#### XB方式（P-δ → Q-V）
```javascript
fdpf_XB() {
    // B'（P-δ用）とB''（Q-V用）行列構築
    const Bp = this.makeBp();    // [∂P/∂θ] 近似
    const Bpp = this.makeBpp();  // [∂Q/∂|V|] 近似
    
    // 一度だけLU分解
    const LU_Bp = this.luDecompose(Bp);
    const LU_Bpp = this.luDecompose(Bpp);
    
    let V = this.initializeVoltage();
    let converged = false;
    let i = 0;
    
    while (!converged && i < this.maxIterations) {
        // P-δ サブ問題求解
        const dP = this.calculateActiveMismatch(V);
        const da = this.luSolve(LU_Bp, dP);  // Δθ
        
        // 位相角更新
        V = this.updateAngles(V, da);
        
        // Q-V サブ問題求解  
        const dQ = this.calculateReactiveMismatch(V);
        const dVm = this.luSolve(LU_Bpp, dQ);  // Δ|V|
        
        // 電圧大きさ更新
        V = this.updateMagnitudes(V, dVm);
        
        // 収束判定
        const [dPmax, dQmax] = this.checkMismatch(V);
        if (Math.max(dPmax, dQmax) < this.tolerance) {
            converged = true;
        }
        
        i++;
    }
    
    return this.packageResults(V, converged, i);
}
```

## 📊 可視化機能

### MATPOWER風結果表示

#### 母線結果
```
================================================================================
|     Bus Data                                                                 |
================================================================================
 Bus      Voltage          Generation             Load        
  #   Mag(pu) Ang(deg)   P (MW)   Q (MVAr)   P (MW)   Q (MVAr)
---- ------- --------  --------  --------  --------  --------
  1   1.060    0.000    232.39   -16.85      0.00      0.00   
  2   1.045   -4.982     40.00    42.40     21.70     12.70   
  3   1.010  -12.725      0.00    23.44     94.20     19.00   
  4   1.019  -10.313      0.00     0.00     47.80    -3.90   
  5   1.020   -8.774      0.00     0.00      7.60      1.60   
---- ------- --------  --------  --------  --------  --------
              Total:    272.39    49.00     171.30     29.40
```

#### 送電線結果
```
================================================================================
|     Branch Data                                                             |
================================================================================
Brnch   From   To    From Bus Injection   To Bus Injection     Loss (MVA)
  #     Bus   Bus    P (MW)  Q (MVAr)     P (MW)  Q (MVAr)   P (MW)  Q (MVAr)
----  -----  -----  --------  --------   --------  --------  --------  --------
  1      1      2    156.88     75.55    -152.60    -61.89     4.28     13.66
  2      1      5     75.52     18.58     -72.18     -2.23     3.34     16.35
  3      2      3     73.24     16.87     -70.91     -4.17     2.33     12.70
  4      2      4     56.13      5.04     -53.32      8.04     2.81     13.08
  5      2      5     41.51     17.63     -41.05    -16.75     0.46      0.88
```

### インタラクティブ系統図

#### Canvas描画システム
```javascript
class NetworkRenderer {
    drawMATLABStyle(results) {
        // MATLAB/MATPOWER風の系統図描画
        this.ctx.fillStyle = '#ffffff';
        this.ctx.fillRect(0, 0, this.width, this.height);
        
        // 母線描画（MATPOWER準拠色分け）
        results.bus.forEach((bus, i) => {
            const color = this.getBusColor(bus.type, bus.voltage);
            this.drawBus(bus.x, bus.y, bus.id, color);
            
            // 電圧表示
            this.drawText(`${bus.vm.toFixed(3)}∠${bus.va.toFixed(1)}°`, 
                         bus.x, bus.y - 20);
        });
        
        // 送電線描画
        results.branch.forEach(branch => {
            this.drawBranch(branch.from, branch.to);
            
            // 電力フロー表示
            this.drawPowerFlow(branch.pf, branch.qf, 
                             branch.x_mid, branch.y_mid);
        });
    }
    
    getBusColor(type, voltage) {
        if (voltage < 0.95) return '#ff4757';      // 低電圧（赤）
        if (voltage > 1.05) return '#ffa502';     // 高電圧（橙）
        
        switch(type) {
            case 3: return '#2ed573';  // Slack（緑）
            case 2: return '#1e90ff';  // PV（青）
            case 1: return '#57606f';  // PQ（灰）
            default: return '#000000';
        }
    }
}
```

## 🔧 検証機能

### MATPOWER結果比較

#### 精度検証
```javascript
function validateResults(matlabResults, jsResults) {
    const tolerance = 1e-6;
    let isValid = true;
    
    // 母線電圧比較
    for (let i = 0; i < matlabResults.bus.length; i++) {
        const vm_diff = Math.abs(matlabResults.bus[i].vm - jsResults.bus[i].vm);
        const va_diff = Math.abs(matlabResults.bus[i].va - jsResults.bus[i].va);
        
        if (vm_diff > tolerance || va_diff > tolerance) {
            console.warn(`Bus ${i+1} voltage mismatch: 
                         VM: ${vm_diff}, VA: ${va_diff}`);
            isValid = false;
        }
    }
    
    // 送電線潮流比較
    for (let i = 0; i < matlabResults.branch.length; i++) {
        const pf_diff = Math.abs(matlabResults.branch[i].pf - jsResults.branch[i].pf);
        const qf_diff = Math.abs(matlabResults.branch[i].qf - jsResults.branch[i].qf);
        
        if (pf_diff > tolerance || qf_diff > tolerance) {
            console.warn(`Branch ${i+1} flow mismatch: 
                         PF: ${pf_diff}, QF: ${qf_diff}`);
            isValid = false;
        }
    }
    
    return isValid;
}
```

### 単体テストスイート

#### 各アルゴリズムの検証
```javascript
const testSuite = {
    // IEEE 14母線系統での既知解テスト
    testNewtonRaphson() {
        const result = runpf('case14', {pf: {alg: 'NR'}});
        assert(result.success, 'Newton-Raphson should converge');
        assert(result.iterations <= 4, 'Should converge in 4 iterations');
        this.validateVoltages(result, ieee14_reference);
    },
    
    testFastDecoupled() {
        const result = runpf('case14', {pf: {alg: 'FDXB'}});
        assert(result.success, 'Fast Decoupled should converge');
        assert(result.iterations <= 7, 'Should converge in 7 iterations');
        this.validateVoltages(result, ieee14_reference);
    },
    
    // 既知解との比較
    validateVoltages(result, reference) {
        const tolerance = 1e-6;
        for (let i = 0; i < result.bus.length; i++) {
            const vm_error = Math.abs(result.bus[i].vm - reference.bus[i].vm);
            const va_error = Math.abs(result.bus[i].va - reference.bus[i].va);
            
            assert(vm_error < tolerance, `Bus ${i+1} VM error: ${vm_error}`);
            assert(va_error < tolerance, `Bus ${i+1} VA error: ${va_error}`);
        }
    }
};
```

## 🚀 実用機能

### データエクスポート

#### MATPOWER形式出力
```javascript
function exportMATLABCode(results) {
    const mcode = `
function mpc = case_result
%% Power flow results generated by MATPOWER-compatible solver
%% ${new Date().toISOString()}

mpc.version = '2';
mpc.baseMVA = ${results.baseMVA};

%% bus data
%	bus_i	type	Pd	Qd	Gs	Bs	area	Vm	Va	baseKV	zone	Vmax	Vmin
mpc.bus = [
${results.bus.map(bus => `\t${bus.id}\t${bus.type}\t${bus.pd}\t${bus.qd}\t${bus.gs}\t${bus.bs}\t${bus.area}\t${bus.vm.toFixed(6)}\t${bus.va.toFixed(4)}\t${bus.base_kv}\t${bus.zone}\t${bus.vmax}\t${bus.vmin};`).join('\n')}
];

%% generator data
mpc.gen = [
${results.gen.map(gen => `\t${gen.bus}\t${gen.pg.toFixed(4)}\t${gen.qg.toFixed(4)}\t${gen.qmax}\t${gen.qmin}\t${gen.vg.toFixed(6)}\t${gen.mbase}\t${gen.status}\t${gen.pmax}\t${gen.pmin};`).join('\n')}
];

%% branch data
mpc.branch = [
${results.branch.map(branch => `\t${branch.fbus}\t${branch.tbus}\t${branch.r.toFixed(6)}\t${branch.x.toFixed(6)}\t${branch.b.toFixed(6)}\t${branch.rateA}\t${branch.rateB}\t${branch.rateC}\t${branch.ratio}\t${branch.angle}\t${branch.status};`).join('\n')}
];

return;
`;
    
    return mcode;
}
```

### CSV出力
```javascript
function exportCSV(results, filename) {
    const csvData = [
        // 母線データ
        'Bus Results',
        'Bus,Type,Vm(pu),Va(deg),P(MW),Q(MVAr)',
        ...results.bus.map(bus => 
            `${bus.id},${bus.type},${bus.vm.toFixed(4)},${bus.va.toFixed(2)},${bus.pg.toFixed(2)},${bus.qg.toFixed(2)}`
        ),
        '',
        // 送電線データ
        'Branch Results', 
        'From,To,P_from(MW),Q_from(MVAr),P_to(MW),Q_to(MVAr)',
        ...results.branch.map(branch =>
            `${branch.fbus},${branch.tbus},${branch.pf.toFixed(2)},${branch.qf.toFixed(2)},${branch.pt.toFixed(2)},${branch.qt.toFixed(2)}`
        )
    ].join('\n');
    
    const blob = new Blob([csvData], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    a.click();
}
```

## 🎓 教育価値

### プロフェッショナル開発

#### 実務スキル習得
1. **業界標準理解**: MATPOWER形式の習得
2. **実用実装**: 産業品質のコード理解
3. **検証手法**: 結果妥当性確認方法
4. **最適化技術**: 計算効率改善手法

#### 実系統応用
1. **データ処理**: 実系統データの扱い方
2. **スケーラビリティ**: 大規模系統への対応
3. **エラー処理**: 非収束ケースの対応
4. **パフォーマンス**: 実用的計算速度

### 研究応用

#### 学術研究支援
- **再現性**: 標準データでの結果再現
- **比較研究**: 既存手法との性能比較
- **新手法評価**: 提案手法の性能評価基準
- **ベンチマーク**: 標準テストケースでの評価

---

**ファイル**: `power_flow_matpower_v2.html`  
**作成日**: 2024年12月30日  
**更新日**: 2024年12月30日  
**バージョン**: 2.1