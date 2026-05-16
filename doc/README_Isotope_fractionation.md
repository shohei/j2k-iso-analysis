# Isotope_fractionation.java

## 概要

同位体分別（Isotope Fractionation）を計算するJAMSコンポーネントです。  
土壌からの蒸発過程における液相-気相平衡同位体分別（Horita & Wesolowski式）と動力学的同位体分別を組み合わせて、蒸発水・土壌水の同位体組成を求めます。

- **作者:** Andrew Watson, Christian Birkel, Sven Kralisch
- **バージョン:** 1.0_0
- **作成日:** 2023-04-04
- **フレームワーク:** JAMS

---

## 科学的背景

水の蒸発時には軽い同位体（¹H, ¹⁶O）が優先的に蒸発するため、残留水は重い同位体（²H, ¹⁸O）が濃縮されます。  
このコンポーネントは以下の理論に基づいています：

- **液相-気相平衡分別:** Horita & Wesolowski (1994) の温度依存式
- **動力学的分別:** Merlivat (1978) に基づく運動論的フラクショネーション
- **Craig-Gordon モデル:** 蒸発水の同位体組成を計算する枠組み（Craig & Gordon 1965; Gibson 2016）

---

## 入力変数

| 変数名 | 単位 | 説明 |
|--------|------|------|
| `temp` | °C | 気温 |
| `rhum` | % | 相対湿度 |
| `pConc` | ‰ | 降水の水蒸気同位体濃度 |
| `init_concS` | ‰ | 初期土壌水の同位体組成 |
| `k` | - | 季節性係数（デフォルト: 1） |
| `x` | - | 交換係数（デフォルト: 0.9） |

---

## 出力・更新変数

| 変数名 | 単位 | 説明 |
|--------|------|------|
| `tempK` | K | 絶対温度（計算用） |
| `alphamas` | - | 液相-気相平衡同位体分別係数（α⁺） |
| `epsimas` | ‰ | 平衡分別係数（ε⁺）= (α⁺ − 1) × 1000 |
| `epsk_H` | ‰ | 動力学的分別係数（εk） |
| `enrichment_slope` | - | 濃縮スロープ（m） |
| `concA` | ‰ | 大気の同位体組成（降水-平衡仮定から算出） |
| `dstar` | ‰ | 限界同位体組成（δ*） |
| `concS` | ‰ | 残留土壌水の同位体組成 |
| `concE` | ‰ | 蒸発水の同位体組成 |

---

## 計算ロジック（runメソッド）

### 1. 絶対温度への変換
```
tempK = 273.13 + temp
```

### 2. 液相-気相平衡分別係数 α⁺（Horita & Wesolowski 1994）
```
alphamas = exp( (1/1000) × (
    1158.8 × T³/10⁹ − 1620.1 × T²/10⁶
    + 794.84 × T/10³ − 161.04 + 2.9992×10⁹/T³
) )   ※ T = tempK
```

### 3. 平衡分別係数（‰表記）
```
epsimas = (alphamas − 1) × 1000
```

### 4. 動力学的分別係数（Merlivat 1978）
```
epsk_H = 0.9755 × (1 − 0.9755) × 1000 × (1 − rhum)
```

### 5. 濃縮スロープ（m）（Gibson et al. 2016）
```
m = (rhum − 10⁻³ × (epsk_H + epsimas/alphamas))
    / (1 − rhum + 10⁻³ × epsk_H)
```

### 6. 大気同位体組成（Gibson et al. 2008）
```
concA = (pConc − k × epsimas) / (1 + epsimas × 10⁻³)
```

### 7. 限界同位体組成 δ*（Gonfiantini 1986）
```
dstar = (rhum × concA + epsk_H + epsimas/alphamas)
        / (rhum − 10⁻³ × (epsk_H + epsimas/alphamas))
```

### 8. 残留土壌水の同位体組成
```
concS = init_concS − dstar × (1 − x)^m + dstar
```

### 9. 蒸発水の同位体組成（Craig & Gordon 1965）
```
concE = ((concS − epsimas)/alphamas − rhum × concA − epsk_H)
        / (1 − rhum + 10⁻³ × epsk_H)
```

---

## 参考文献

- Horita, J. & Wesolowski, D.J. (1994). Liquid-vapor fractionation of oxygen and hydrogen isotopes of water. *Geochimica et Cosmochimica Acta*, 58(16), 3425–3437.
- Craig, H. & Gordon, L.I. (1965). Deuterium and oxygen 18 variations in the ocean and marine atmosphere.
- Merlivat, L. (1978). Molecular diffusivities of H₂¹⁶O, HD¹⁶O, and H₂¹⁸O in gases. *Journal of Chemical Physics*, 69(6), 2864–2871.
- Gibson, J.J. et al. (2008, 2016). Isotope-based studies of evaporation from lakes.
- Gonfiantini, R. (1986). Environmental isotopes in lake studies.
