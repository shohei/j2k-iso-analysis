# Binary_isotope_mixing.java

## 概要

二成分同位体混合モデル（Binary Isotope Mixing Model）のJAMSコンポーネントです。  
降水・河川水・地下水の同位体濃度を使って、ハイドログラフにおける各水源の寄与割合を計算します。

- **作者:** Andrew Watson (awatson@sun.ac.za)
- **バージョン:** 1.0_0
- **作成日:** 2022-09-20
- **フレームワーク:** JAMS (Java-based Analysing and Modelling System)

---

## 科学的背景

二成分同位体混合は、河川流量を「地表流出（雨水起源）」と「地下水」の2成分に分解する手法です。  
以下の質量保存則に基づきます：

```
δ_stream = f_rain × δ_rain + f_gw × δ_gw
```

- `f_rain + f_gw = 1`（各成分の割合の合計は1）
- `δ` は同位体比（‰、パーミル表記）

---

## 入力変数

| 変数名 | 単位 | 説明 |
|--------|------|------|
| `isotopeRain` | ‰ | 降水の同位体濃度 |
| `isotopeStream` | ‰ | 河川水の同位体濃度 |
| `isotopeGw` | ‰ | 地下水の同位体濃度 |
| `compSW` | fraction | 土壌水の寄与割合（シミュレーション済み RD2） |
| `minCompRain` | fraction | 地表流出寄与の最小キャリブレーション値 |
| `minCompGw` | fraction | 地下水寄与の最小キャリブレーション値 |

---

## 出力変数

| 変数名 | 単位 | 説明 |
|--------|------|------|
| `compRain` | fraction | 雨水成分（地表流出）の割合 |
| `compGw` | fraction | 地下水成分の割合 |
| `compRainN` | fraction | 正規化後の雨水割合 |
| `compGwN` | fraction | 正規化後の地下水割合 |
| `FracCompRainGw` | fraction | RD1を考慮した利用可能な流出・地下水割合 |

---

## 計算ロジック（runメソッド）

### 1. 各成分割合の計算

```
compRain = (isotopeStream - isotopeRain) / (isotopeRain - isotopeGw)
compGw   = (isotopeStream - isotopeRain) / (isotopeGw  - isotopeRain)
```

### 2. 最小値補正

`compRain` または `compGw` が 0 の場合、それぞれ `minCompRain`、`minCompGw` に置き換えます。

### 3. 土壌水成分を考慮した正規化

```
fracCompRainGw = 1 - compSW
compRainN = compRain × fracCompRainGw
compGwN   = compGw  × fracCompRainGw
```

`compSW`（土壌水の寄与 RD2）を差し引いた残りを、雨水と地下水で分配します。

---

## 参考文献

Watson, A., Vystavna, Y., Kralisch, S., Helmschrot, J., van Rooyen, J., & Miller, J.  
*Development of an isotope-enabled rainfall-runoff model: Improving the capability to capture hydrological and anthropogenic change.*  
（査読中 / Publication status: Under review, 2022）
