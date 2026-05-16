# IsotopeMixer.java

## 概要

複数セルに対応した同位体混合コンポーネントです（`IsotopeMixer_` の改良版）。  
配列形式のボリューム・濃度を受け取り、体積加重平均で混合します。双方向混合と混合割合の調整が可能です。

- **作者:** Andrew Watson, Christian Birkel, Sven Kralisch
- **バージョン:** 1.0_1
- **更新日:** 2026-05-15
- **フレームワーク:** JAMS

---

## 科学的背景

流域モデルでは、複数の空間セルや貯留層間の水・同位体交換を並列処理する必要があります。  
このコンポーネントは、グリッドセル配列に対して完全混合仮定のもと同位体濃度を更新します。  
`bidirectional` フラグにより、混合結果を送り元（A）にも反映させるか制御できます。

---

## パラメータ・変数

### 制御パラメータ（READ）

| 変数名 | 型 | デフォルト | 説明 |
|--------|----|-----------|------|
| `bidirectional` | Boolean | true | trueの場合、混合後濃度をconcAにも書き戻す |
| `mixingProportion` | Double | 1.0 | 混合割合（0〜1）。volBをこの値で割ることでA側への混合量を調整 |

### 入力変数（READ）

| 変数名 | 単位 | 説明 |
|--------|------|------|
| `volA[]` | L | 水源A（各セル）の体積配列 |
| `volB[]` | L | 水源B（各セル）の体積配列 |

### 入出力変数（READWRITE）

| 変数名 | 単位 | 説明 |
|--------|------|------|
| `concA[]` | ‰ | 水源Aの同位体濃度配列（bidirectional=trueなら更新される） |
| `concB[]` | ‰ | 水源Bの同位体濃度配列（常に更新される） |

---

## 計算ロジック（runメソッド）

各セル `i` について以下を実行します：

### 1. 実効ボリュームBの計算
```
volB_eff = volB[i] / mixingProportion
```
`mixingProportion` によりBのうちAと混合される割合を制御します。

### 2. 混合後濃度の計算
```
if (volA[i] + volB[i] == 0):
    x = 0
else:
    x = (concA[i] × volA[i] + concB[i] × volB_eff) / (volA[i] + volB_eff)
```

### 3. 結果の書き戻し
```
if (bidirectional == true):
    concA[i] = x   // Aにも混合後濃度を反映
concB[i] = x       // Bは常に更新
```

---

## IsotopeMixer_.java との主な違い

| 項目 | IsotopeMixer_.java | IsotopeMixer.java |
|------|--------------------|-------------------|
| 処理セル数 | 単一 | 配列（複数セル） |
| 双方向更新 | なし | `bidirectional` で制御 |
| 混合割合 | なし | `mixingProportion` で調整可 |
| init()処理 | あり（初期値設定） | なし |
| バージョン | 1.0_0 | 1.0_1（2026-05-15更新） |

---

## 使用上の注意

- `volA[i] + volB[i] == 0` の場合（どちらも空）は濃度を 0 にします。
- `mixingProportion = 0` になると除算エラーが発生するため、0以外の値を設定してください。
