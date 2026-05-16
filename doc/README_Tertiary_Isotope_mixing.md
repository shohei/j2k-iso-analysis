# Tertiary_Isotope_mixing.java

## 概要

三成分同位体混合モデル（Tertiary Isotope Mixing Model）のJAMSコンポーネントです。  
降水・土壌水・地下水の同位体濃度と、シミュレートされた流出成分（RD1・RD2・RG1+RG2）を組み合わせ、  
河川ハイドログラフの同位体組成を計算します。

- **作者:** Andrew Watson (awatson@sun.ac.za)
- **バージョン:** 1.0_0
- **作成日:** 2022-03-16
- **フレームワーク:** JAMS

---

## 科学的背景

流域水文モデル（J2000等）では、河川流量を以下の成分に分解します：

| 成分 | 説明 |
|------|------|
| RD1 | 直接表面流出（降水起源） |
| RD2 | 中間流（土壌水起源） |
| RG1 | 上部地下水からの基底流出 |
| RG2 | 深層地下水からの基底流出 |

このコンポーネントは、各成分の同位体濃度を重み付け（各流量成分の比）で足し合わせ、  
観測された河川同位体値と比較可能な合成値を生成します。

---

## 入力変数

| 変数名 | 単位 | 説明 |
|--------|------|------|
| `isotopeRain` | ‰ | 降水の同位体濃度 |
| `isotopeStream` | ‰ | 河川水の同位体濃度 |
| `isotopeSw[]` | ‰ | 土壌水の同位体濃度（月別配列、12要素） |
| `isotopeGw` | ‰ | 地下水の同位体濃度 |
| `catchmentRD1` | ‰ | 流域全体のRD1（直接流出）成分量 |
| `catchmentRD2` | ‰ | 流域全体のRD2（中間流）成分量 |
| `catchmentRG1RG2` | ‰ | 流域全体のRG1+RG2（地下水）成分量 |
| `catchmentSimRunoff` | ‰ | 流域の総シミュレーション流出量 |
| `time` | Calendar | 現在の計算時刻（月の取得に使用） |

---

## 出力変数

| 変数名 | 単位 | 説明 |
|--------|------|------|
| `compA` | ‰ | 河川同位体濃度 × 総流出量（観測値の同位体フラックス） |
| `compB` | ‰ | 各成分を重み付けしたシミュレーション同位体フラックス |

---

## 計算ロジック（runメソッド）

### 1. 観測フラックス（compA）の計算
```
compA = isotopeStream × catchmentSimRunoff
```
観測された河川同位体濃度を総流出量でスケーリングします。

### 2. 月別土壌水同位体の取得
```
curIsotopeSw = isotopeSw[currentMonth]
```
`time.get(MONTH)` で現在の月インデックス（0〜11）を取得し、該当月の土壌水同位体値を参照します。

### 3. シミュレーションフラックス（compB）の計算

**降水データあり（`isotopeRain ≠ -99`）の場合:**
```
compB = isotopeGw × catchmentRG1RG2
      + curIsotopeSw × catchmentRD2
```
※ このケースでは降水項（RD1）を含めないことに注意（条件が逆の可能性あり → コード確認推奨）

**降水データなし（`isotopeRain == -99`）の場合:**
```
compB = isotopeRain × catchmentRD1
      + isotopeGw   × catchmentRG1RG2
      + curIsotopeSw × catchmentRD2
```

---

## 欠損値の扱い

`isotopeRain == -99` は降水の同位体データが欠損していることを示します。  
このフラグで降水項の有無を切り替えています。

---

## Binary_isotope_mixing.java との違い

| 項目 | Binary_isotope_mixing.java | Tertiary_Isotope_mixing.java |
|------|---------------------------|------------------------------|
| 分解成分数 | 2成分（雨水・地下水） | 3成分（RD1・RD2・RG1+RG2） |
| 土壌水の扱い | 外部入力（compSW） | 月別配列（isotopeSw[]） |
| 計算方式 | 同位体比から割合を逆算 | 各成分フラックスの重み付け和 |
| 用途 | 各成分割合の推定 | 観測・シミュレーションの比較 |

---

## 参考文献

Watson, A., Vystavna, Y., Kralisch, S., Helmschrot, J., van Rooyen, J., & Miller, J.  
*Development of an isotope-enabled rainfall-runoff model: Improving the capability to capture hydrological and anthropogenic change.*
