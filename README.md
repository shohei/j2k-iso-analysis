# j2k-iso-analysis
J2000-ISOのコード解析

各ファイルの内容まとめ

  ### 各ファイルの内容まとめ

| Javaファイル | 説明 |
| :--- | :--- |
| `Binary_isotope_mixing` | 河川水・降水・地下水の2成分分解。同位体比から雨水・地下水の寄与割合を逆算。 |
| `Isotope_fractionation` | 蒸発時の液相-気相同位体分別（Horita & Wesolowski式 + Craig-Gordonモデル）で土壌水・蒸発水の同位体組成を計算。 |
| `IsotopeMixer_` | 単一セル向けシンプル混合器。体積加重平均で流入水と既存貯留水を混合。 |
| `IsotopeMixer` | 複数セル対応の改良版混合器。双方向混合フラグ・混合割合パラメータ付き。 |
| `IsotopeMultiMixer` | 単一水源Aを複数セルB配列に体積比で分配して混合。 |
| `Tertiary_Isotope_mixing` | RD1・RD2・RG1+RG2の3成分フラックスを月別土壌水同位体で重み付けし、観測値と比較可能な同位体フラックスを生成。 |