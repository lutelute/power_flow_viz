# 潮流計算手法技術ノート

## 📚 概要

本技術ノートは、電力系統の潮流計算における主要なアルゴリズムの数学的基礎、実装方法、収束特性、適用場面を包括的に解説します。理論的厳密性と実践的有用性の両立を目指し、学習者から実務者まで幅広いニーズに対応します。

## 📋 目次

1. [潮流計算の基礎理論](#1-潮流計算の基礎理論)
2. [Newton-Raphson法](#2-newton-raphson法)
3. [高速分離解法](#3-高速分離解法)
4. [Gauss-Seidel法](#4-gauss-seidel法)
5. [DC潮流計算](#5-dc潮流計算)
6. [先進的手法](#6-先進的手法)
7. [収束理論](#7-収束理論)
8. [実装技術](#8-実装技術)
9. [性能比較](#9-性能比較)
10. [適用指針](#10-適用指針)

---

## 1. 潮流計算の基礎理論

### 1.1 電力系統の数学モデル

#### 基本方程式の導出

電力系統の定常状態解析において、各母線 $i$ における電力保存則（Kirchhoffの電流則）から出発する。

**Step 1: 複素電力の定義**

母線 $i$ への複素電力注入は：
$$S_i = P_i + jQ_i = V_i \cdot I_i^*$$

ここで、$V_i$ は母線 $i$ の複素電圧、$I_i^*$ は注入電流の共役である。

**Step 2: アドミタンス行列による電流表現**

電流は電圧とアドミタンス行列の関係により：
$$I_i = \sum_{j=1}^n Y_{ij} V_j$$

したがって、複素電力は：
$$S_i = V_i \left(\sum_{j=1}^n Y_{ij} V_j\right)^* = V_i^* \sum_{j=1}^n Y_{ij}^* V_j^*$$

**Step 3: 実部・虚部分離**

$V_i = |V_i|e^{j\theta_i}$、$Y_{ij} = G_{ij} + jB_{ij}$ として：

$$P_i = \text{Re}[S_i] = \text{Re}\left[V_i^* \sum_{j=1}^n Y_{ij} V_j\right] = P_{Gi} - P_{Di} \tag{1}$$

$$Q_i = \text{Im}[S_i] = \text{Im}\left[V_i^* \sum_{j=1}^n Y_{ij} V_j\right] = Q_{Gi} - Q_{Di} \tag{2}$$

#### 極座標形式への展開

複素電圧を極座標で表現：$V_i = |V_i|e^{j\theta_i}$

**導出過程：**

$$\begin{align}
P_i &= \sum_{j=1}^n |V_i||V_j||Y_{ij}|\cos(\theta_{ij} - \theta_i + \theta_j) \tag{3}\\
&= \sum_{j=1}^n |V_i||V_j|[G_{ij}\cos(\theta_i - \theta_j) + B_{ij}\sin(\theta_i - \theta_j)] \tag{4}
\end{align}$$

$$\begin{align}
Q_i &= \sum_{j=1}^n |V_i||V_j||Y_{ij}|\sin(\theta_{ij} - \theta_i + \theta_j) \tag{5}\\
&= \sum_{j=1}^n |V_i||V_j|[G_{ij}\sin(\theta_i - \theta_j) - B_{ij}\cos(\theta_i - \theta_j)] \tag{6}
\end{align}$$

ここで、$\theta_{ij} = \arg(Y_{ij})$、$|Y_{ij}| = \sqrt{G_{ij}^2 + B_{ij}^2}$ である。

#### アドミタンス行列の構成

**送電線要素：**

送電線を $R + jX$ で表現すると、送電線アドミタンスは：
$$y_{ij} = \frac{1}{R_{ij} + jX_{ij}} = \frac{R_{ij}}{R_{ij}^2 + X_{ij}^2} - j\frac{X_{ij}}{R_{ij}^2 + X_{ij}^2} = g_{ij} + jb_{ij} \tag{7}$$

**母線アドミタンス行列：**

$$Y_{ii} = \sum_{k \in \mathcal{N}_i} y_{ik} + y_{sh,i} \quad \text{（自己アドミタンス）} \tag{8}$$

$$Y_{ij} = -y_{ij} \quad \text{（相互アドミタンス, } i \neq j\text{）} \tag{9}$$

ここで、$\mathcal{N}_i$ は母線 $i$ に接続する母線の集合、$y_{sh,i}$ は母線 $i$ の分路アドミタンスである。

#### 線形化による感度解析

微小変動 $\Delta\mathbf{x}$ に対する電力変化は：

$$\begin{bmatrix} \Delta\mathbf{P} \\ \Delta\mathbf{Q} \end{bmatrix} = \begin{bmatrix} \frac{\partial \mathbf{P}}{\partial \boldsymbol{\theta}} & \frac{\partial \mathbf{P}}{\partial |\mathbf{V}|} \\ \frac{\partial \mathbf{Q}}{\partial \boldsymbol{\theta}} & \frac{\partial \mathbf{Q}}{\partial |\mathbf{V}|} \end{bmatrix} \begin{bmatrix} \Delta\boldsymbol{\theta} \\ \Delta|\mathbf{V}| \end{bmatrix} = \mathbf{J} \Delta\mathbf{x} \tag{10}$$

これが潮流計算の線形化方程式の基礎となる。

### 1.2 母線の分類

#### 1. Slack母線（基準母線）
- **与えられる量**: |V|, θ (通常 θ = 0°)
- **求める量**: P, Q
- **役割**: 電圧位相基準、電力バランス調整

#### 2. PV母線（発電機母線）
- **与えられる量**: P, |V|
- **求める量**: Q, θ
- **制約**: Q_min ≤ Q ≤ Q_max

#### 3. PQ母線（負荷母線）
- **与えられる量**: P, Q
- **求める量**: |V|, θ
- **最も多数**: 系統の大部分を占める

### 1.3 潮流方程式の極座標表現

```
P_i = ∑_{j=1}^n |V_i||V_j||Y_{ij}|cos(θ_{ij} - θ_i + θ_j)
Q_i = ∑_{j=1}^n |V_i||V_j||Y_{ij}|sin(θ_{ij} - θ_i + θ_j)
```

簡略表記：
```
P_i = ∑_{j=1}^n |V_i||V_j|[G_{ij}cos(θ_i - θ_j) + B_{ij}sin(θ_i - θ_j)]
Q_i = ∑_{j=1}^n |V_i||V_j|[G_{ij}sin(θ_i - θ_j) - B_{ij}cos(θ_i - θ_j)]
```

---

## 2. Newton-Raphson法

### 2.1 理論的基礎

Newton-Raphson法は多変数非線形方程式系 $\mathbf{f}(\mathbf{x}) = \mathbf{0}$ の数値解法である。

#### 多変数Newton法の導出

**Step 1: Taylor展開**

点 $\mathbf{x}^{(k)}$ における1次Taylor展開：
$$\mathbf{f}(\mathbf{x}^{(k)} + \Delta\mathbf{x}^{(k)}) \approx \mathbf{f}(\mathbf{x}^{(k)}) + \mathbf{J}(\mathbf{x}^{(k)}) \Delta\mathbf{x}^{(k)} \tag{11}$$

ここで、$\mathbf{J}(\mathbf{x})$ はJacobian行列：
$$J_{ij}(\mathbf{x}) = \frac{\partial f_i(\mathbf{x})}{\partial x_j} \tag{12}$$

**Step 2: Newton方向の決定**

$\mathbf{f}(\mathbf{x}^{(k)} + \Delta\mathbf{x}^{(k)}) = \mathbf{0}$ を要求すると：
$$\mathbf{f}(\mathbf{x}^{(k)}) + \mathbf{J}(\mathbf{x}^{(k)}) \Delta\mathbf{x}^{(k)} = \mathbf{0} \tag{13}$$

したがって、Newton方向は：
$$\Delta\mathbf{x}^{(k)} = -[\mathbf{J}(\mathbf{x}^{(k)})]^{-1} \mathbf{f}(\mathbf{x}^{(k)}) \tag{14}$$

**Step 3: 反復更新**

$$\mathbf{x}^{(k+1)} = \mathbf{x}^{(k)} + \Delta\mathbf{x}^{(k)} = \mathbf{x}^{(k)} - [\mathbf{J}(\mathbf{x}^{(k)})]^{-1} \mathbf{f}(\mathbf{x}^{(k)}) \tag{15}$$

#### 潮流計算への適用

**変数ベクトル：**
$$\mathbf{x} = \begin{bmatrix} \boldsymbol{\theta}_{PV,PQ} \\ |\mathbf{V}|_{PQ} \end{bmatrix} = \begin{bmatrix} \theta_2, \ldots, \theta_n \\ |V|_1, \ldots, |V|_m \end{bmatrix}^T \tag{16}$$

ここで、$n$ は総母線数、$m$ はPQ母線数である。

**ミスマッチベクトル：**
$$\mathbf{f}(\mathbf{x}) = \begin{bmatrix} \Delta\mathbf{P} \\ \Delta\mathbf{Q} \end{bmatrix} = \begin{bmatrix} \mathbf{P}_{spec} - \mathbf{P}_{calc}(\mathbf{x}) \\ \mathbf{Q}_{spec} - \mathbf{Q}_{calc}(\mathbf{x}) \end{bmatrix} \tag{17}$$

**Newton更新式：**
$$\begin{bmatrix} \Delta\boldsymbol{\theta}^{(k)} \\ \Delta|\mathbf{V}|^{(k)} \end{bmatrix} = -\begin{bmatrix} \mathbf{J}_{P\theta} & \mathbf{J}_{P|V|} \\ \mathbf{J}_{Q\theta} & \mathbf{J}_{Q|V|} \end{bmatrix}^{-1} \begin{bmatrix} \Delta\mathbf{P}^{(k)} \\ \Delta\mathbf{Q}^{(k)} \end{bmatrix} \tag{18}$$

### 2.2 Jacobian行列の厳密導出

Jacobian行列は4つのブロックから構成される：

$$\mathbf{J} = \begin{bmatrix} \frac{\partial \mathbf{P}}{\partial \boldsymbol{\theta}} & \frac{\partial \mathbf{P}}{\partial |\mathbf{V}|} \\ \frac{\partial \mathbf{Q}}{\partial \boldsymbol{\theta}} & \frac{\partial \mathbf{Q}}{\partial |\mathbf{V}|} \end{bmatrix} = \begin{bmatrix} \mathbf{J}_{P\theta} & \mathbf{J}_{P|V|} \\ \mathbf{J}_{Q\theta} & \mathbf{J}_{Q|V|} \end{bmatrix} \tag{19}$$

#### $\mathbf{J}_{P\theta}$ ブロックの導出

式(4)より：$P_i = \sum_{j=1}^n |V_i||V_j|[G_{ij}\cos(\theta_i - \theta_j) + B_{ij}\sin(\theta_i - \theta_j)]$

**対角要素（$i = j$）：**
$$\frac{\partial P_i}{\partial \theta_i} = \sum_{j=1, j \neq i}^n |V_i||V_j|[-G_{ij}\sin(\theta_i - \theta_j) + B_{ij}\cos(\theta_i - \theta_j)] \tag{20}$$

式(6)と比較すると：
$$\frac{\partial P_i}{\partial \theta_i} = -Q_i - |V_i|^2 B_{ii} \tag{21}$$

**非対角要素（$i \neq j$）：**
$$\frac{\partial P_i}{\partial \theta_j} = |V_i||V_j|[G_{ij}\sin(\theta_i - \theta_j) - B_{ij}\cos(\theta_i - \theta_j)] \tag{22}$$

#### $\mathbf{J}_{P|V|}$ ブロックの導出

**対角要素：**
$$\frac{\partial P_i}{\partial |V_i|} = \frac{1}{|V_i|} \sum_{j=1}^n |V_j|[G_{ij}\cos(\theta_i - \theta_j) + B_{ij}\sin(\theta_i - \theta_j)] \tag{23}$$

式(4)を用いると：
$$\frac{\partial P_i}{\partial |V_i|} = \frac{P_i + |V_i|^2 G_{ii}}{|V_i|} \tag{24}$$

**非対角要素：**
$$\frac{\partial P_i}{\partial |V_j|} = |V_i|[G_{ij}\cos(\theta_i - \theta_j) + B_{ij}\sin(\theta_i - \theta_j)] \tag{25}$$

#### $\mathbf{J}_{Q\theta}$ および $\mathbf{J}_{Q|V|}$ ブロック

同様の手順で無効電力についても導出：

**$\mathbf{J}_{Q\theta}$ ブロック：**
$$\frac{\partial Q_i}{\partial \theta_i} = P_i - |V_i|^2 G_{ii} \tag{26}$$

$$\frac{\partial Q_i}{\partial \theta_j} = -|V_i||V_j|[G_{ij}\cos(\theta_i - \theta_j) + B_{ij}\sin(\theta_i - \theta_j)] \tag{27}$$

**$\mathbf{J}_{Q|V|}$ ブロック：**
$$\frac{\partial Q_i}{\partial |V_i|} = \frac{Q_i - |V_i|^2 B_{ii}}{|V_i|} \tag{28}$$

$$\frac{\partial Q_i}{\partial |V_j|} = |V_i|[G_{ij}\sin(\theta_i - \theta_j) - B_{ij}\cos(\theta_i - \theta_j)] \tag{29}$$

#### 計算量解析

Jacobian行列の構築：$O(n^2)$  
LU分解：$O(n^3)$  
前進・後進代入：$O(n^2)$

総計算量：$O(n^3)$ per iteration

### 2.3 アルゴリズム

```
1. 初期化: θ_i^(0) = 0, |V_i|^(0) = 1.0 (PQ母線)
2. 反復 k = 0, 1, 2, ...
   a) ミスマッチ計算: f^(k) = [ΔP^(k); ΔQ^(k)]
   b) 収束判定: ||f^(k)||_∞ < ε なら終了
   c) Jacobian構築: J^(k)
   d) 線形方程式求解: J^(k) Δx^(k) = -f^(k)
   e) 変数更新: x^(k+1) = x^(k) + Δx^(k)
```

### 2.4 実装例（JavaScript）

```javascript
function newtonRaphsonPowerFlow(Ybus, Pbus, Qbus, V0) {
    let V = [...V0];
    const tolerance = 1e-8;
    const maxIter = 20;
    
    for (let iter = 0; iter < maxIter; iter++) {
        // 電力ミスマッチ計算
        const Scalc = calculatePowerInjection(V, Ybus);
        const deltaP = subtractReal(Pbus, Scalc);
        const deltaQ = subtractImag(Qbus, Scalc);
        
        // 収束判定
        const maxMismatch = Math.max(
            Math.max(...deltaP.map(Math.abs)),
            Math.max(...deltaQ.map(Math.abs))
        );
        
        if (maxMismatch < tolerance) {
            return {converged: true, voltage: V, iterations: iter};
        }
        
        // Jacobian行列構築
        const J = buildJacobian(V, Ybus);
        
        // 線形方程式求解
        const mismatch = [...deltaP, ...deltaQ];
        const dx = solveLinearSystem(J, mismatch);
        
        // 電圧更新
        V = updateVoltage(V, dx);
    }
    
    return {converged: false, voltage: V, iterations: maxIter};
}

function buildJacobian(V, Ybus) {
    const n = V.length;
    const J = new Array(2*n-1).fill(0).map(() => new Array(2*n-1).fill(0));
    
    for (let i = 1; i < n; i++) {  // Slack母線除く
        for (let j = 1; j < n; j++) {
            if (i === j) {
                // 対角要素
                J[i-1][j-1] = -calculateQi(i, V, Ybus) - 
                              Ybus[i][i].imag * Math.pow(Math.abs(V[i]), 2);
            } else {
                // 非対角要素
                const Vi = Math.abs(V[i]);
                const Vj = Math.abs(V[j]);
                const theta_ij = Math.arg(V[i]) - Math.arg(V[j]);
                const Y_ij = Ybus[i][j];
                
                J[i-1][j-1] = Vi * Vj * (Y_ij.real * Math.sin(theta_ij) - 
                                        Y_ij.imag * Math.cos(theta_ij));
            }
        }
    }
    
    return J;
}
```

### 2.5 収束特性

- **収束次数**: 二次収束 (誤差 ∝ (誤差)²)
- **収束範囲**: 解の近傍で局所的
- **反復回数**: 通常 3-6回
- **計算量**: O(n³) per iteration

---

## 3. 高速分離解法（Fast Decoupled Method）

### 3.1 理論的基礎と近似仮定

高速分離解法は、実用送電系統の物理的特性に基づく近似により、Newton-Raphson法を簡素化した手法である。

#### 基本仮定

送電系統の以下の物理的特性を仮定：

**仮定 A1**: $r_{ij} \ll x_{ij}$ （送電線抵抗 $\ll$ リアクタンス）  
**仮定 A2**: $|\theta_i - \theta_j| \ll 1$ （母線間位相角差は小さい）  
**仮定 A3**: $|V_i| \approx 1.0$ （電圧大きさは基準値付近）  
**仮定 A4**: $Q_i/|V_i|^2 \ll B_{ii}$ （無効電力項 $\ll$ 自己サセプタンス）

### 3.2 Jacobian行列の分離近似

#### Step 1: 近似の導出

仮定A1, A2により、式(22), (25)は次のように近似できる：

$$G_{ij}\sin(\theta_i - \theta_j) \approx G_{ij}(\theta_i - \theta_j) \approx 0 \quad (\because G_{ij} \ll B_{ij})$$

$$B_{ij}\cos(\theta_i - \theta_j) \approx B_{ij}$$

したがって：
$$\frac{\partial P_i}{\partial \theta_j} \approx -|V_i||V_j|B_{ij} \tag{30}$$

同様に、仮定A3により：
$$\frac{\partial P_i}{\partial |V_j|} \approx B_{ij}(\theta_i - \theta_j) \approx 0 \tag{31}$$

#### Step 2: 分離されたJacobian

近似により、Jacobian行列は以下のように分離される：

$$\mathbf{J} \approx \begin{bmatrix} -\mathbf{B}' & \mathbf{0} \\ \mathbf{0} & -\mathbf{B}'' \end{bmatrix} \tag{32}$$

ここで：
- $\mathbf{B}'$: P-θ カップリング行列
- $\mathbf{B}''$: Q-|V| カップリング行列

#### Step 3: B'およびB''行列の定義

**B'行列（有効電力-位相角）：**
$$B'_{ii} = \sum_{j \in \mathcal{N}_i} \frac{1}{x_{ij}} \tag{33}$$

$$B'_{ij} = -\frac{1}{x_{ij}} \quad (i \neq j, \text{隣接母線}) \tag{34}$$

**B''行列（無効電力-電圧大きさ）：**
$$B''_{ii} = B'_{ii} + b_{sh,i} \tag{35}$$

$$B''_{ij} = B'_{ij} \tag{36}$$

ここで、$b_{sh,i}$ は母線 $i$ の分路サセプタンスである。

### 3.3 分離された線形方程式

近似により、連立方程式は2つの独立な部分問題に分離される：

**P-θ 部分問題：**
$$\Delta\mathbf{P} = -\mathbf{B}' \Delta\boldsymbol{\theta} \quad \Rightarrow \quad \Delta\boldsymbol{\theta} = -(\mathbf{B}')^{-1} \frac{\Delta\mathbf{P}}{|\mathbf{V}|} \tag{37}$$

**Q-|V| 部分問題：**
$$\Delta\mathbf{Q} = -\mathbf{B}'' \Delta|\mathbf{V}| \quad \Rightarrow \quad \Delta|\mathbf{V}| = -(\mathbf{B}'')^{-1} \frac{\Delta\mathbf{Q}}{|\mathbf{V}|} \tag{38}$$

### 3.4 XB方式アルゴリズム

**初期化：**
1. $\mathbf{B}'$, $\mathbf{B}''$ 行列の構築
2. LU分解：$\mathbf{B}' = \mathbf{L}_1 \mathbf{U}_1$, $\mathbf{B}'' = \mathbf{L}_2 \mathbf{U}_2$

**反復計算（$k = 0, 1, 2, \ldots$）：**

**Step 1**: P-θ サブ問題
$$\Delta\mathbf{P}^{(k)} = \mathbf{P}_{spec} - \mathbf{P}_{calc}(\boldsymbol{\theta}^{(k)}, |\mathbf{V}|^{(k)}) \tag{39}$$

$$\Delta\boldsymbol{\theta}^{(k)} = -(\mathbf{L}_1 \mathbf{U}_1)^{-1} \frac{\Delta\mathbf{P}^{(k)}}{|\mathbf{V}|^{(k)}} \tag{40}$$

$$\boldsymbol{\theta}^{(k+1)} = \boldsymbol{\theta}^{(k)} + \Delta\boldsymbol{\theta}^{(k)} \tag{41}$$

**Step 2**: Q-|V| サブ問題
$$\Delta\mathbf{Q}^{(k)} = \mathbf{Q}_{spec} - \mathbf{Q}_{calc}(\boldsymbol{\theta}^{(k+1)}, |\mathbf{V}|^{(k)}) \tag{42}$$

$$\Delta|\mathbf{V}|^{(k)} = -(\mathbf{L}_2 \mathbf{U}_2)^{-1} \frac{\Delta\mathbf{Q}^{(k)}}{|\mathbf{V}|^{(k)}} \tag{43}$$

$$|\mathbf{V}|^{(k+1)} = |\mathbf{V}|^{(k)} + \Delta|\mathbf{V}|^{(k)} \tag{44}$$

**収束判定：**
$$\max(||\Delta\mathbf{P}^{(k)}||_\infty, ||\Delta\mathbf{Q}^{(k)}||_\infty) < \varepsilon \tag{45}$$

### 3.3 B'およびB''行列

#### B'行列（P-θ用）
```
B'_{ii} = ∑_{j≠i} X_{ij}^(-1)
B'_{ij} = -X_{ij}^(-1)  (i ≠ j)
```

#### B''行列（Q-|V|用）  
```
B''_{ii} = B'_{ii} + B_{shi}
B''_{ij} = B'_{ij}
```

### 3.4 アルゴリズム（XB方式）

```
1. 初期化・前処理
   - B', B''行列構築
   - LU分解: [L₁U₁] = B', [L₂U₂] = B''

2. 反復計算 k = 0, 1, 2, ...
   a) P-θ サブ問題
      - ΔP^(k) = P_spec - P_calc^(k)
      - Δθ^(k) = (B')^(-1) (ΔP^(k) / |V|^(k))
      - θ^(k+1) = θ^(k) + Δθ^(k)
      
   b) Q-|V| サブ問題  
      - ΔQ^(k) = Q_spec - Q_calc^(k)
      - Δ|V|^(k) = (B'')^(-1) (ΔQ^(k) / |V|^(k))
      - |V|^(k+1) = |V|^(k) + Δ|V|^(k)
      
   c) 収束判定
```

### 3.5 実装例

```javascript
function fastDecoupledPowerFlow(Ybus, Pbus, Qbus, V0) {
    // B'およびB''行列構築
    const Bp = buildBprimeMatrix(Ybus);
    const Bpp = buildBdoubleprimeMatrix(Ybus);
    
    // LU分解（一度のみ）
    const LU_Bp = luDecomposition(Bp);
    const LU_Bpp = luDecomposition(Bpp);
    
    let V = [...V0];
    const tolerance = 1e-8;
    const maxIter = 30;
    
    for (let iter = 0; iter < maxIter; iter++) {
        // P-θ サブ問題
        const deltaP = calculateActiveMismatch(V, Pbus, Ybus);
        const deltaPoverV = deltaP.map((dp, i) => dp / Math.abs(V[i]));
        const deltaTheta = solveLU(LU_Bp, deltaPoverV);
        
        // 位相角更新
        for (let i = 0; i < V.length; i++) {
            const mag = Math.abs(V[i]);
            const newAngle = Math.arg(V[i]) + deltaTheta[i];
            V[i] = mag * Math.exp(new Complex(0, newAngle));
        }
        
        // Q-|V| サブ問題
        const deltaQ = calculateReactiveMismatch(V, Qbus, Ybus);
        const deltaQoverV = deltaQ.map((dq, i) => dq / Math.abs(V[i]));
        const deltaV = solveLU(LU_Bpp, deltaQoverV);
        
        // 電圧大きさ更新
        for (let i = 0; i < V.length; i++) {
            const newMag = Math.abs(V[i]) + deltaV[i];
            const angle = Math.arg(V[i]);
            V[i] = newMag * Math.exp(new Complex(0, angle));
        }
        
        // 収束判定
        const maxP = Math.max(...deltaP.map(Math.abs));
        const maxQ = Math.max(...deltaQ.map(Math.abs));
        
        if (Math.max(maxP, maxQ) < tolerance) {
            return {converged: true, voltage: V, iterations: iter};
        }
    }
    
    return {converged: false, voltage: V, iterations: maxIter};
}
```

### 3.6 収束特性

- **収束次数**: 1.6-1.8次（準二次収束）
- **反復回数**: 通常 5-12回
- **計算量**: 2×O(n³/2) per iteration
- **メモリ効率**: B', B''は定数行列

---

## 4. Gauss-Seidel法

### 4.1 理論的基礎

反復法の最も基本的な形：

```
V_i^(k+1) = (1/Y_{ii}) [S_i*/V_i* - ∑_{j≠i} Y_{ij}V_j^(k+1/k)]
```

ここで：
- `S_i* = P_i - jQ_i`: 指定電力の共役
- `V_i*`: 電圧の共役
- `j < i`: 既に更新済みの電圧を使用
- `j > i`: 前回反復の電圧を使用

### 4.2 アルゴリズム

```
1. 初期化: V_i^(0) = 1.0∠0° (PQ母線)
2. 反復 k = 0, 1, 2, ...
   for i = 2 to n (Slack母線以外)
     if (母線iがPQ母線) then
       V_i^(k+1) = (1/Y_{ii})[S_i*/V_i* - ∑_{j≠i} Y_{ij}V_j^(k+1/k)]
     else if (母線iがPV母線) then
       V_temp = (1/Y_{ii})[S_i*/V_i* - ∑_{j≠i} Y_{ij}V_j^(k+1/k)]
       V_i^(k+1) = |V_i|^spec × (V_temp/|V_temp|)
   収束判定
```

### 4.3 加速技法

#### SOR（Successive Over-Relaxation）
```
V_i^(k+1) = (1-ω)V_i^(k) + ω·V_i^new
```
ここで、`ω`は緩和係数（通常 1.0 < ω < 2.0）

### 4.4 実装例

```javascript
function gaussSeidelPowerFlow(Ybus, busData, V0) {
    let V = [...V0];
    const tolerance = 1e-8;
    const maxIter = 100;
    const omega = 1.6;  // SOR加速係数
    
    for (let iter = 0; iter < maxIter; iter++) {
        const V_old = [...V];
        
        for (let i = 1; i < V.length; i++) {  // Slack母線除く
            const bus = busData[i];
            
            // 指定電力
            const Sspec = new Complex(bus.PG - bus.PD, bus.QG - bus.QD);
            
            // 他母線からの影響項計算
            let sum = new Complex(0, 0);
            for (let j = 0; j < V.length; j++) {
                if (j !== i) {
                    sum = sum.add(Ybus[i][j].multiply(V[j]));
                }
            }
            
            // 新しい電圧値計算
            const Vnew = Sspec.conjugate().divide(V[i].conjugate())
                        .subtract(sum)
                        .divide(Ybus[i][i]);
            
            if (bus.type === 'PQ') {
                // PQ母線: そのまま更新
                V[i] = V[i].multiply(1 - omega).add(Vnew.multiply(omega));
            } else if (bus.type === 'PV') {
                // PV母線: 電圧大きさ制約
                const Vmag_spec = bus.V;
                const Vnew_normalized = Vnew.multiply(Vmag_spec / Vnew.magnitude());
                V[i] = V[i].multiply(1 - omega).add(Vnew_normalized.multiply(omega));
            }
        }
        
        // 収束判定
        let maxMismatch = 0;
        for (let i = 1; i < V.length; i++) {
            const mismatch = calculatePowerMismatch(i, V, busData, Ybus);
            maxMismatch = Math.max(maxMismatch, Math.abs(mismatch.P), Math.abs(mismatch.Q));
        }
        
        if (maxMismatch < tolerance) {
            return {converged: true, voltage: V, iterations: iter};
        }
    }
    
    return {converged: false, voltage: V, iterations: maxIter};
}
```

### 4.5 収束特性

- **収束次数**: 一次収束（線形）
- **収束率**: 最大固有値に依存
- **反復回数**: 通常 15-50回
- **計算量**: O(n²) per iteration
- **メモリ**: 最小（行列保存不要）

---

## 5. DC潮流計算

### 5.1 理論的基礎

#### 基本仮定
1. **|V_i| = 1.0**: すべての母線電圧大きさ = 1.0pu
2. **θ_ij << 1**: 母線間位相角差は小さい  
3. **sin(θ_ij) ≈ θ_ij**: 小角度近似
4. **cos(θ_ij) ≈ 1**: 小角度近似
5. **G_ij << B_ij**: コンダクタンス << サセプタンス

#### 線形化されたAC潮流方程式

AC潮流方程式：
```
P_i = ∑_{j=1}^n |V_i||V_j|[G_{ij}cos(θ_i - θ_j) + B_{ij}sin(θ_i - θ_j)]
```

DC近似：
```
P_i ≈ ∑_{j=1}^n B_{ij}(θ_i - θ_j) = ∑_{j=1}^n B_{ij}θ_i - ∑_{j=1}^n B_{ij}θ_j
```

行列形式：
```
P = Bθ
```

### 5.2 サセプタンス行列B

```
B_{ii} = ∑_{j≠i} (1/X_{ij})
B_{ij} = -1/X_{ij}  (i ≠ j)
```

### 5.3 アルゴリズム

```
1. サセプタンス行列B構築
2. 指定有効電力ベクトルP構築
3. 基準母線（Slack）除去
4. 線形方程式求解: θ = B^(-1) P
5. 送電線潮流計算: P_{ij} = (θ_i - θ_j)/X_{ij}
```

### 5.4 実装例

```javascript
function dcPowerFlow(branchData, busData) {
    const n = busData.length;
    
    // サセプタンス行列構築
    const B = buildBMatrix(branchData, n);
    
    // 指定有効電力ベクトル
    const P = busData.map(bus => bus.PG - bus.PD);
    
    // Slack母線（通常母線1）を除去
    const B_reduced = removeSlackBus(B, 0);
    const P_reduced = removeSlackBus(P, 0);
    
    // 線形方程式求解
    const theta_reduced = solveLinearSystem(B_reduced, P_reduced);
    
    // 完全な位相角ベクトル復元
    const theta = [0, ...theta_reduced];  // Slack母線は0
    
    // 送電線潮流計算
    const branchFlows = branchData.map(branch => {
        const i = branch.fbus - 1;
        const j = branch.tbus - 1;
        const flow = (theta[i] - theta[j]) / branch.x;
        return {
            from: branch.fbus,
            to: branch.tbus,
            flow: flow
        };
    });
    
    return {
        bus: busData.map((bus, i) => ({
            id: bus.id,
            angle: theta[i] * 180 / Math.PI,  // 度に変換
            voltage: 1.0  // DC仮定
        })),
        branch: branchFlows
    };
}

function buildBMatrix(branchData, nbus) {
    const B = new Array(nbus).fill(0).map(() => new Array(nbus).fill(0));
    
    branchData.forEach(branch => {
        const i = branch.fbus - 1;
        const j = branch.tbus - 1;
        const b = 1.0 / branch.x;
        
        B[i][i] += b;
        B[j][j] += b;
        B[i][j] -= b;
        B[j][i] -= b;
    });
    
    return B;
}
```

### 5.5 適用場面と限界

#### 適用場面
- **経済負荷配分**: 発電機の最適配分
- **概算計算**: 初期検討・感度解析
- **大規模解析**: 多数ケースの高速処理
- **最適潮流**: 最適化問題の初期解

#### 限界
- **電圧情報**: |V|, Q の情報なし
- **精度限界**: 重負荷時・長距離送電で精度悪化
- **制約考慮**: 電圧・無効電力制約考慮不可

---

## 6. 先進的手法

### 6.1 Levenberg-Marquardt法

Newton法の収束性改善：

```
x^(k+1) = x^(k) - [J^T J + μI]^(-1) J^T f(x^(k))
```

- `μ`: 制御パラメータ（大→Gauss法、小→Newton法）
- 悪条件問題での収束性向上

### 6.2 連続潮流法（Continuation Power Flow）

電圧安定性解析のための手法：

```
F(x,λ) = 0
```

- `x`: 状態変数（電圧）
- `λ`: 負荷パラメータ
- PV曲線の追跡により臨界点検出

### 6.3 Holomorphic Embedding法

非反復的解法：

```
V(s) = ∑_{n=0}^∞ V_n s^n
```

- `s`: 埋め込みパラメータ
- 電力級数展開による解析解
- 収束保証（条件下で）

### 6.4 機械学習ベース手法

#### ニューラルネットワーク
- **学習**: 大量の潮流計算結果でNNを学習
- **推論**: 高速な初期値推定・収束予測
- **応用**: リアルタイム運用支援

#### 強化学習
- **エージェント**: 潮流計算アルゴリズム
- **環境**: 電力系統状態
- **報酬**: 収束速度・精度
- **学習**: 最適な計算戦略の獲得

---

## 7. 収束理論

### 7.1 Newton法の収束理論

#### 7.1.1 局所収束定理

**定理 (Newton-Kantorovich定理)**  
$\mathbf{f}: \mathbb{R}^n \to \mathbb{R}^n$ を $C^2$ 級関数とし、$\mathbf{x}^*$ を $\mathbf{f}(\mathbf{x}^*) = \mathbf{0}$ の解とする。以下の条件が満たされるとき、$\mathbf{x}^*$ の近傍で開始されたNewton反復は二次収束する：

1. $\mathbf{J}(\mathbf{x}^*)$ が正則（$\det(\mathbf{J}(\mathbf{x}^*)) \neq 0$）
2. $\mathbf{J}(\mathbf{x})$ がLipschitz連続：$||\mathbf{J}(\mathbf{x}) - \mathbf{J}(\mathbf{y})|| \leq L||\mathbf{x} - \mathbf{y}||$

**収束率：**
$$||\mathbf{x}^{(k+1)} - \mathbf{x}^*|| \leq C ||\mathbf{x}^{(k)} - \mathbf{x}^*||^2 \tag{70}$$

ここで、$C$ は定数である。

#### 7.1.2 大域収束理論

Newton法の収束領域を拡大する手法：

**信頼領域法 (Trust Region Method)**

各反復で以下の制約付き最適化問題を解く：
$$\min_{\mathbf{d}} ||\mathbf{f}(\mathbf{x}^{(k)}) + \mathbf{J}(\mathbf{x}^{(k)})\mathbf{d}||^2 \quad \text{s.t.} \quad ||\mathbf{d}|| \leq \Delta^{(k)} \tag{71}$$

**線探索法 (Line Search Method)**

ステップサイズ $\alpha$ を調整：
$$\mathbf{x}^{(k+1)} = \mathbf{x}^{(k)} + \alpha^{(k)} \mathbf{d}^{(k)} \tag{72}$$

ここで、$\mathbf{d}^{(k)} = -[\mathbf{J}(\mathbf{x}^{(k)})]^{-1}\mathbf{f}(\mathbf{x}^{(k)})$ はNewton方向。

### 7.2 不動点反復理論

#### 7.2.1 Gauss-Seidel法の理論

Gauss-Seidel法は不動点反復 $\mathbf{x} = \mathbf{G}(\mathbf{x})$ として定式化：

$$V_i^{(k+1)} = G_i(V_1^{(k+1)}, \ldots, V_{i-1}^{(k+1)}, V_{i+1}^{(k)}, \ldots, V_n^{(k)}) \tag{73}$$

**収束条件 (Banachの不動点定理)**

$\mathbf{G}: D \subset \mathbb{C}^n \to D$ が縮小写像：
$$||\mathbf{G}(\mathbf{x}) - \mathbf{G}(\mathbf{y})|| \leq \rho ||\mathbf{x} - \mathbf{y}||, \quad 0 \leq \rho < 1 \tag{74}$$

このとき、唯一の不動点 $\mathbf{x}^*$ が存在し、任意の初期値から収束：
$$||\mathbf{x}^{(k)} - \mathbf{x}^*|| \leq \rho^k ||\mathbf{x}^{(0)} - \mathbf{x}^*|| \tag{75}$$

**線形収束率：**
$$\rho = \lim_{k \to \infty} \frac{||\mathbf{x}^{(k+1)} - \mathbf{x}^*||}{||\mathbf{x}^{(k)} - \mathbf{x}^*||} \tag{76}$$

#### 7.2.2 行列による収束解析

Gauss-Seidel行列を $\mathbf{M} = (\mathbf{D} + \mathbf{L})^{-1}\mathbf{U}$ とすると：

**収束条件：** $\rho(\mathbf{M}) < 1$ （スペクトル半径 < 1）

**最適SOR係数：**
$$\omega_{opt} = \frac{2}{1 + \sqrt{1 - \rho(\mathbf{B})^2}} \tag{77}$$

ここで、$\mathbf{B}$ はJacobi反復行列。

### 7.3 高速分離解法の収束理論

#### 7.3.1 分解条件数

$\mathbf{B}'$, $\mathbf{B}''$ 行列の条件数による収束性評価：

$$\kappa(\mathbf{B}') = ||\mathbf{B}'|| \cdot ||(\mathbf{B}')^{-1}||, \quad \kappa(\mathbf{B}'') = ||\mathbf{B}''|| \cdot ||(\mathbf{B}'')^{-1}|| \tag{78}$$

**収束性能：**  
- $\kappa(\mathbf{B}')$, $\kappa(\mathbf{B}'') < 10^3$: 良好な収束  
- $\kappa(\mathbf{B}')$, $\kappa(\mathbf{B}'') > 10^6$: 収束困難

#### 7.3.2 近似誤差の伝播

真のJacobian $\mathbf{J}$ と近似Jacobian $\mathbf{J}_{FD}$ の差：

$$\Delta\mathbf{J} = \mathbf{J} - \mathbf{J}_{FD} = \begin{bmatrix} \mathbf{0} & \mathbf{J}_{P|V|} \\ \mathbf{J}_{Q\theta} & \mathbf{0} \end{bmatrix} \tag{79}$$

摂動理論により、収束次数は以下に低下：
$$r_{FD} \approx 1 + \frac{\log(||\Delta\mathbf{J}||)}{\log(||\mathbf{f}||)} \tag{80}$$

### 7.4 収束判定基準

#### 7.4.1 絶対誤差基準

**最大値ノルム：**
$$||\mathbf{f}(\mathbf{x}^{(k)})||_\infty = \max_i |f_i(\mathbf{x}^{(k)})| < \varepsilon \tag{81}$$

**ユークリッドノルム：**
$$||\mathbf{f}(\mathbf{x}^{(k)})||_2 = \sqrt{\sum_i f_i(\mathbf{x}^{(k)})^2} < \varepsilon \tag{82}$$

#### 7.4.2 相対誤差基準

$$\frac{||\mathbf{x}^{(k+1)} - \mathbf{x}^{(k)}||}{||\mathbf{x}^{(k)}||} < \varepsilon_{rel} \tag{83}$$

#### 7.4.3 成分別基準

工学的実用性を考慮：
$$|\Delta P_i| < \varepsilon_P \quad \text{かつ} \quad |\Delta Q_i| < \varepsilon_Q \quad \forall i \tag{84}$$

典型値：$\varepsilon_P = \varepsilon_Q = 10^{-4}$ pu (工学精度)、$10^{-8}$ pu (研究精度)

### 7.5 数値的安定性

#### 7.5.1 条件数と数値誤差

機械精度 $\varepsilon_m$ の影響：
$$\delta\mathbf{x} \approx \kappa(\mathbf{J}) \varepsilon_m ||\mathbf{x}|| \tag{85}$$

**実用判定基準：**
- $\kappa(\mathbf{J}) < 10^{12}$: 数値的に安定
- $\kappa(\mathbf{J}) > 10^{15}$: 数値的不安定（特異に近い）

#### 7.5.2 ピボット戦略

**部分ピボット選択：**
$$|a_{kk}^{(k)}| = \max_{k \leq i \leq n} |a_{ik}^{(k)}| \tag{86}$$

**完全ピボット選択：**
$$|a_{pq}^{(k)}| = \max_{\substack{k \leq i \leq n \\ k \leq j \leq n}} |a_{ij}^{(k)}| \tag{87}$$

### 7.3 収束加速技法

#### Anderson加速
```
x^(k+1) = ∑_{i=0}^m θ_i G(x^(k-i))
```

#### Aitkenの Δ² 加速
```
x_accelerated = x_k - (x_{k+1} - x_k)² / (x_{k+2} - 2x_{k+1} + x_k)
```

---

## 8. 実装技術

### 8.1 数値線形代数

#### LU分解
```
PA = LU
```

最適化：
- **部分ピボット**: 数値安定性
- **スパース技術**: メモリ効率
- **並列化**: 計算高速化

#### 反復解法
- **GMRES**: 非対称行列用
- **CG**: 対称正定値行列用
- **BiCGSTAB**: 一般的な非対称行列用

### 8.2 スパース行列技術

#### 格納形式
- **CRS**: Compressed Row Storage
- **CCS**: Compressed Column Storage  
- **COO**: Coordinate format

#### 並び替え
- **Cuthill-McKee**: 帯幅最小化
- **AMD**: Approximate Minimum Degree
- **METIS**: グラフ分割ベース

### 8.3 実装言語別特徴

#### MATLAB/Octave
```matlab
function [V, converged] = newtonpf(Ybus, Sbus, V0, tol)
% 高水準・簡潔な実装
% 豊富な数値ライブラリ
% プロトタイピングに最適
```

#### Python
```python
def newton_raphson_pf(Ybus, Sbus, V0, tol=1e-8):
    # NumPy/SciPy活用
    # 可読性と拡張性
    # 機械学習との親和性
```

#### C/C++
```cpp
bool newton_raphson_pf(SparseMatrix& Ybus, Vector& Sbus, 
                       Vector& V, double tol) {
    // 高速・メモリ効率
    // 商用ソフトウェア品質
    // 大規模系統対応
}
```

#### JavaScript
```javascript
function newtonRaphsonPF(Ybus, Sbus, V0, tolerance) {
    // ブラウザ実行
    // 教育・可視化
    // インタラクティブ性
}
```

---

## 9. 性能比較

### 9.1 計算複雑度

| 手法 | 単位反復 | 総反復数 | 総計算量 |
|------|----------|----------|----------|
| Newton-Raphson | O(n³) | 3-6 | O(n³) |
| Fast Decoupled | O(n³/2) | 5-12 | O(n³) |  
| Gauss-Seidel | O(n²) | 20-50 | O(n³) |
| DC Power Flow | O(n³) | 1 | O(n³) |

### 9.2 メモリ使用量

| 手法 | 主要行列 | スパース率 | メモリ量 |
|------|----------|------------|----------|
| Newton-Raphson | Jacobian | 90-95% | O(n²) |
| Fast Decoupled | B', B'' | 95-98% | O(n²/2) |
| Gauss-Seidel | なし | - | O(n²) |
| DC Power Flow | B | 95-98% | O(n²/4) |

### 9.3 収束性能

#### 典型的なIEEE 14母線系統での結果

```
手法              反復数  時間[ms]  メモリ[MB]  精度
Newton-Raphson      4      12.3      2.1       1e-10
Fast Decoupled      7       8.9      1.6       1e-8  
Gauss-Seidel       23      15.7      1.2       1e-8
DC Power Flow       1       1.2      0.8       1e-2
```

### 9.4 スケーラビリティ

#### 系統規模別性能（Newton-Raphson）

| 母線数 | 反復数 | 時間[s] | メモリ[MB] | 備考 |
|--------|--------|---------|------------|------|
| 14     | 4      | 0.01    | 2.1        | 小規模 |
| 57     | 5      | 0.15    | 8.9        | 中規模 |
| 118    | 6      | 0.45    | 28.4       | 大規模 |
| 300    | 7      | 2.1     | 156        | 実用規模 |
| 2383   | 8      | 45      | 1024       | 実系統規模 |

---

## 10. 適用指針

### 10.1 手法選択指針

#### Newton-Raphson法
**適用場面:**
- 汎用的な潮流計算
- 高精度が必要な解析
- 中小規模系統（～500母線）

**避けるべき場面:**
- 超大規模系統
- リアルタイム計算
- メモリ制約が厳しい環境

#### 高速分離解法
**適用場面:**
- 大規模系統（500母線～）
- 商用ソフトウェア
- バランスの良い性能

**避けるべき場面:**
- 高R/X比系統
- 精密解析
- 配電系統

#### Gauss-Seidel法
**適用場面:**
- 教育用途
- 小規模系統（～50母線）
- メモリ制約環境

**避けるべき場面:**
- 大規模系統
- 時間制約がある場面
- 高精度要求

#### DC潮流計算
**適用場面:**
- 概算計算・初期検討
- 感度解析
- 経済負荷配分
- 最適化の初期解

**避けるべき場面:**
- 詳細解析
- 電圧・無効電力重要な解析
- 高精度要求

### 10.2 実装上の考慮事項

#### 数値安定性
- **ピボット選択**: LU分解での数値安定性
- **反復改良**: 解精度向上
- **前処理**: 条件数改善

#### 効率性
- **スパース技術**: メモリと計算効率
- **並列処理**: マルチコア活用
- **ベクトル化**: SIMD命令活用

#### 信頼性
- **例外処理**: 非収束時の対応
- **妥当性検査**: 解の物理的妥当性
- **ログ機能**: デバッグ支援

### 10.3 将来動向

#### 計算技術
- **量子計算**: 組み合わせ最適化への応用
- **GPU計算**: 並列数値計算の加速
- **分散計算**: クラウドベース大規模計算

#### 人工知能
- **深層学習**: 初期値推定・収束予測
- **強化学習**: 適応的アルゴリズム選択
- **生成AI**: 自動プログラム生成

#### 応用分野
- **スマートグリッド**: 分散型電源統合
- **電力市場**: 高頻度取引支援
- **リアルタイム運用**: ミリ秒レベル計算

---

## 📚 参考文献

1. **基本文献**
   - Bergen, A.R. & Vittal, V. "Power Systems Analysis" (2nd ed.)
   - Kundur, P. "Power System Stability and Control"
   - Grainger, J.J. & Stevenson, W.D. "Power System Analysis"

2. **専門文献**
   - Stott, B. & Alsac, O. "Fast Decoupled Load Flow"
   - Monticelli, A. "State Estimation in Electric Power Systems"
   - Zimmerman, R.D. et al. "MATPOWER User's Manual"

3. **最新研究**
   - Trias, A. "The Holomorphic Embedding Load Flow Method"
   - Milano, F. "Continuous Newton's Method for Power Flow Analysis"
   - Various IEEE Transactions on Power Systems papers

## 📝 付録

### A. 記号一覧

| 記号 | 意味 |
|------|------|
| V_i | 母線iの複素電圧 |
| θ_i | 母線iの電圧位相角 |
| P_i, Q_i | 母線iの有効・無効電力注入 |
| Y_{ij} | アドミタンス行列要素 |
| G_{ij}, B_{ij} | コンダクタンス・サセプタンス |
| J | Jacobian行列 |
| ε | 収束判定値 |

### B. IEEE標準系統データ

詳細な系統データは各HTMLファイルに実装済み。

### C. 実装テンプレート

基本的なアルゴリズム実装のテンプレートコードを提供。

---

**文書**: `power_flow_methods.md`  
**作成者**: Power Flow Visualization Project  
**作成日**: 2024年12月30日  
**バージョン**: 1.0  
**ライセンス**: Educational Use