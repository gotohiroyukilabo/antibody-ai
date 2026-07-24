# VHH / nanobody 配列–titer データセット調査

調査日: 2026-07-24

## 結論

公開情報を、データリポジトリ、論文のSource Data・Supplementary
Data、抗体データベース、GitHub、特許まで横断して調べたが、**VHH /
nanobodyの配列と、同一条件で測定された絶対生産titer（mg/Lなど）を大量に
収録した公開データセットは見つからなかった**。

現時点で最も大きく、配列と発現量を行単位で直接ダウンロードできる候補は
Overath et al. のデータである。ただし、目的変数はmg/Lではなく、
split-luciferase assayの**相対発現シグナル**である。

- 129 variant + 62 selected VHH
- シート間の重複を除くと **175 unique full-length VHH sequences**
- 全行に発現シグナルあり
- 追加でdeep sequencing 13,274行、AlphaFold3 prediction 258行を収録

絶対値については、小規模な論文表と特許に散在している。今回確認できた
有力な情報源を統合すると、重複や条件違いを含めて**約100の
construct/condition-level測定値**をキュレーションできる見込みがある。
ただし、宿主、培養スケール、細胞質・ペリプラズム、タグ、精製法、
monomer/fusion/Fc/bivalent、報告値の定義が異なるため、単一のtiterとして
無条件に混ぜるべきではない。

機械可読な調査結果は
[`vhh_expression_dataset_catalog.csv`](vhh_expression_dataset_catalog.csv)
に収録した。

## 用語と採用基準

この調査では、`titer`を次の優先順で扱った。

1. 培養液量あたりの回収量・生産量（mg/L、g/L、µg/mL）
2. 配列間で比較できる相対発現量または分泌量
3. 定性的なexpressed / not expressed

抗原結合ELISA titer、血清抗体価、neutralization titer、phage enrichment、
read countは、タンパク質生産量ではないため除外した。`yield`については、
論文が培養液量あたりの精製回収量として報告した場合を候補に含めたが、
未精製上清のtiterとは区別が必要である。

## 最有力の既存データセット

### 1. Overath et al. integrated VHH discovery dataset

- 公開先: [Zenodo record 18290348](https://zenodo.org/records/18290348)
- ファイル: `combined_data.xlsx`
- `split_luciferase`: 62行、62 unique sequences
- `split_luciferase_variants`: 129行、129 unique sequences
- 両シートのunion: 175 unique sequences（16配列が重複）
- 発現値の範囲:
  - selected VHH: 5,587–802,357 raw units
  - variants: 2,012–1,461,347 raw units
- 配列: full-length VHH amino-acid sequence
- 付随情報: antigen、結合、developability、deep-sequencing、
  AlphaFold3 prediction
- 注意: 絶対濃度ではなくassay-specificな相対シグナル。Zenodoの表示上、
  ライセンスを明瞭に確認できなかったため、再配布前に要確認。

これは、**配列からVHH発現性を学習するための公開データとしては今回の
調査で最も実用的**である。一方、mg/L予測モデルの教師データとして直接
扱うことはできない。

### 2. FLAb expression collection（重要なnegative result）

- 公開先: [Graylab/FLAb expression data](https://github.com/Graylab/FLAb/tree/main/data/expression)
- FLAb全体: 241 datasets、300万超のdata points
- expressionフォルダで実際に確認できた4データ:
  - Adams 2017: 10,970、Fab relative expression enrichment
  - Garbinski 2023: 94、conventional Fv、µg/mL
  - Jain 2017: 137、conventional antibody、HEK titer mg/L
  - Koenig 2017: 4,276、conventional antibody relative expression

expressionデータはいずれもFabまたは通常型抗体で、**VHH-specificな
titerデータはなかった**。抗体発現研究でよく参照されるJain 2017の
137抗体titerもVHHではない。

## 絶対titerを持つキュレーション候補

以下は「完成済みの単一データセット」ではなく、配列と測定値を原典から
対応付けて統合できる候補である。行数はconstructまたは測定条件単位の概数。

| 優先度 | 情報源 | 規模 | 値 | 配列の所在 | 主な注意点 |
|---|---|---:|---|---|---|
| A | [US 8,372,398 anti-vWF VHH patent](https://www.freepatentsonline.com/8372398.html) | 約34測定 | 2–48 mg/L | 本文の配列表・各Table | monomer、bivalent、humanized、tag有無が混在。特許のため再利用条件と権利関係を別途確認 |
| A | [Soler et al. 2016](https://pmc.ncbi.nlm.nih.gov/articles/PMC5056509/) | 16 constructs | 0–5 mg/L | 本文配列・mutation定義 | framework/CDR改変系列。CC BY 4.0 |
| A | [Ramon et al. 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12510460/) | 8 parent/optimized variants + 個別変異系列 | 0–19 mg/L（主要表） | Supplementary Fig. S2、Tables S1–S2 | 主要表とcell-free mutation assayを分離すべき |
| A | [CDR1 composition study](https://pmc.ncbi.nlm.nih.gov/articles/PMC8465892/) | 5 variants | 1.2–12.8 mg/L | 本文Table | 同一scaffoldの比較。CC BY 4.0 |
| A | [VEEV sdAbs](https://www.nature.com/articles/s41598-021-04434-x) | 8 monomers + optimized/fusions | 3.2–19.6 mg/L（主要8種） | Figure 1 | bivalent/optimized formsは別formatとして保持。CC BY 4.0 |
| A | [CHIKV CC3 stabilization](https://www.frontiersin.org/journals/medicine/articles/10.3389/fmed.2021.626028/full) | 11 constructs | 3.4–28 mg/L | Figure 2 | C-terminal cysteine formsは本文でrangeのみ。CC BY |
| A | [anti-TIGIT nanobodies](https://www.frontiersin.org/journals/immunology/articles/10.3389/fimmu.2023.1268900/full) | 9 Nbs | 0.75–15.1 mg/L | Supplementary Figure 1 | mouse/human TIGIT bindersが混在。CC BY |
| B | [anti-SEB nanobodies](https://www.mdpi.com/2072-6651/15/6/400) | 12 sequences、9数値 | 0.32–78 mg/L | alignment / Supplementary | 3配列はinsolubleまたはpoor expressionで定性的。CC BY |
| B | [anti-CRP nanobodies](https://d-nb.info/1256840262/34) | 8 candidates、6数値 | 0–11.1 mg/L | Figure 1 | 一部はnot produced。CC BY |
| B | [fusion protein expression system](https://pmc.ncbi.nlm.nih.gov/articles/PMC10045489/) | 3 VHH、10条件 | 0–7.686 mg/L | 本文・引用元 | 同一配列のhost/promoter/compartment条件比較。配列対応は要追加確認 |
| C | [visual secretion platform](https://www.mdpi.com/2218-273X/15/1/111) | 7 fusion constructs | 188–335 mg/L | 本文・引用元 | sfGFP fusionとmulti-VHHを含み、単独VHHの配列効果とは分離が必要 |

### 原典で確認した代表値

転記ミスの検出と候補評価に使えるよう、代表的な表の値を残す。

- US 8,372,398 Table 9:
  12A5 13、12B1 6、12B6 16、12D11 8、12E3 4、12C9 25、
  14F8 48 mg/L
- CDR1 composition:
  A10 wt 5.2、mutG0 6.9、mutKG 9.6、mutKG.1 12.8、
  mutKG.2 1.2 mg/L
- CHIKV CC3:
  CC3-hop 12.4、m1 28、m2 18、m3 9.9、m4 9.2、m5 20.5、
  m6 9.6、m7 11、m8 11、m9 5.8、m10 3.4 mg/L
- anti-TIGIT:
  16966 5.4、16972 4.3、16979 0.75、16988 1.7、16920 8.3、
  16925 3.8、17010 8.5、17018 15.1、17037 10.8 mg/L

これらの数値は調査用の索引であり、正式な学習データ化では各原典の
sequence ID、construct form、実験条件、単位、replicateを再確認する。

## 大規模だがtiterを持たないデータ

大規模なVHHデータが見つかっても、目的変数が結合、親和性、熱安定性、
polyreactivity、構造であるケースが大半だった。

| データセット | 規模・内容 | titerに使えない理由 |
|---|---|---|
| [ANDD](https://zenodo.org/records/18151718) | 48,683 antibody/nanobody sequences、9,557 affinity values | affinity中心で生産titerなし |
| [Harvey nanobody polyreactivity](https://huggingface.co/datasets/hugging-science/harvey-nanobody-polyreactivity) | 141,021 full VHH sequences | PSR polyreactivityのbinary label |
| AVIDa-hIL6 | 573,891 antigen–VHH pairs | binding / non-binding label |
| [TNP](https://github.com/oxpig/TNP) | 108 nanobodies、複数developability assays | HIC、AC-SINS、DLS等で、生産titer assayなし |
| NbBench | binding、thermostability、polyreactivity等 | expression taskなし |
| NanoMelt | 配列とmelting temperature | 一部定性的expressionのみでtiterなし |
| AbNatiV / AbNatiV2 | 大規模native-like sequence data | production labelなし |
| SAbDab / PLAbDab-nano / OAS | 構造・特許・repertoire sequence | structured production titerなし |
| Nanobodies.be | nanobody sequence/resource aggregation | structured titer fieldなし |
| [Rapid discovery Source Data](https://www.nature.com/articles/s41551-023-01093-3) | nanobody候補約63行、CDR・binding data | nanobody sheetにyield列なし。scFv sheetのみにyield |
| [Directed-evolution SARS-CoV-2 VHHs](https://pmc.ncbi.nlm.nih.gov/articles/PMC8223476/) | sequence・affinity・developability | matured VHH 27–110 mg/L等のrangeのみで、配列別ラベルなし |
| [Dryad source data](https://doi.org/10.5061/dryad.f4qrfj77j) | nanobody paperのELISA、imaging、WB source data | expression titerなし |

## 推奨するデータ化方針

### すぐ使えるもの

相対発現性モデルやrankingなら、Overath datasetの175 unique VHHを起点に
できる。`Expression_log`を用い、同一配列が両シートにある16件の扱いと、
assay batchの有無を確認してsplitを設計する。

### 絶対値モデルを作る場合

上記A/B候補を原典から手作業でキュレーションする。最低限、次の列を持つ
long-form tableにする。

- `source_id`, `source_url`, `table_or_figure`
- `construct_id`
- `vhh_domain_sequence`
- `expressed_construct_sequence`
- `construct_format`（monomer、bivalent、Fc、fusion等）
- `host_strain`, `expression_system`
- `compartment`（cytoplasm、periplasm、secreted、cell-free）
- `promoter`, `tag`, `culture_scale`, `culture_time`
- `purification_method`
- `reported_value`, `reported_unit`
- `normalized_mg_per_L`
- `value_definition`（supernatant titer、purified yield等）
- `replicate_n`, `sd_or_sem`
- `qualitative_expression`
- `license`, `curation_notes`

最初の解析では、E. coli periplasmic monomerだけに限定する、または
source/host/formatをgroupとして扱うのが安全である。ランダムなrow splitは
近縁variantや同一parentの情報漏洩を起こすため、parent/scaffold単位の
group splitを使う。

## 調査範囲と方法

以下を組み合わせて検索し、可能なものは本文だけでなく実ファイルや表を
開いて列・行数を確認した。

- 検索語:
  `VHH sequence expression yield dataset`、`nanobody sequence titer mg/L`、
  `single-domain antibody production yield sequence`、
  `VHH supplementary table expression`、`nanobody patent yield mg/L`
- リポジトリ:
  Zenodo、Dryad、GitHub、Hugging Face、NCBI GEO
- 抗体リソース:
  FLAb、SAbDab、PLAbDab-nano、OAS、ANDD、AbNatiV、NanoMelt、
  NbBench、Nanobodies.be
- 論文出版社・全文:
  PubMed Central、Nature Portfolio、Frontiers、MDPI
- 特許:
  sequence listingとproduction/yield tableが同一文書にある例を重点確認

「見つからなかった」は公開Webと取得可能な付属ファイルの範囲での結論で
あり、企業内データ、契約が必要なデータ、検索不能な古いSupplementary
Fileまでは保証しない。
