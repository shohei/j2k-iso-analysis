# IsotopeMixer_.java

## 概要

単一水槽への同位体混合を計算するシンプルなJAMSコンポーネントです。  
流入する水（体積・同位体濃度）を既存の貯留水に質量フラックスとして混合し、混合後の濃度と総質量を更新します。

- **作者:** Andrew Watson, Christian Birkel, Sven Kralisch
- **バージョン:** 1.0_0
- **作成日:** 2023-04-04
- **フレームワーク:** JAMS
- **注意:** クラス名は `IsotopeMixer_`（末尾にアンダースコア）で、後継の `IsotopeMixer.java` とは別コンポーネントです。

---

## 科学的背景

完全混合（Complete Mixing）の仮定のもと、流入水と既存貯留水を体積加重平均で混合します。  
この手法は流域水文モデルにおける各貯留層（土壌水層、地下水層など）の同位体トレーサー輸送に広く用いられます。

---

## 入力変数

| 変数名 | 単位 | 説明 |
|--------|------|------|
| `inVol` | L | 流入水の体積 |
| `inConc` | mol/L（‰） | 流入水の同位体濃度 |

---

## 入出力変数（READWRITE）

| 変数名 | 単位 | 説明 |
|--------|------|------|
| `actVol` | L | 現在の貯留水体積（混合後に更新） |
| `actConc` | mol/L（‰） | 現在の貯留水の同位体濃度（混合後に更新） |
| `actM` | - | 現在の同位体質量（= actConc × actVol） |

---

## 計算ロジック

### initメソッド（初期化）

```java
actVol  = inVol
actConc = inConc
```

モデル起動時に、貯留水の初期値を流入水の値で設定します。

### runメソッド（毎タイムステップ）

#### 1. 混合後の総体積
```
totalVolume = actVol + inVol
```

#### 2. 混合後の同位体濃度（体積加重平均）
```
newConcentration = (actConc × actVol + inConc × inVol) / totalVolume
```

#### 3. 値の更新
```
actVol  = totalVolume
actConc = newConcentration
actM    = newConcentration × totalVolume
```

---

## IsotopeMixer.java との違い

| 項目 | IsotopeMixer_.java | IsotopeMixer.java |
|------|--------------------|-------------------|
| 配列対応 | なし（単一セル） | あり（複数セル） |
| 双方向混合 | なし | `bidirectional` パラメータで制御 |
| 混合割合の調整 | なし | `mixingProportion` パラメータあり |
| 初期化 | `init()` で実行 | `init()` なし |
| バージョン | 1.0_0 | 1.0_1 |
