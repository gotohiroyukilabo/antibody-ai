# BoltzGen: Toward Universal Binder Design

## まず何の論文か

BoltzGen は、標的分子との複合体構造と、バインダーの配列・全原子構造を同時に生成する汎用拡散モデルである。タンパク質、nanobody、直鎖・環状ペプチドなどのバインダーを、タンパク質、核酸、低分子などに対して設計できる。単一モデルで構造予測と設計を共同学習し、生成後の inverse folding、Boltz-2 による refolding、物理・developability 指標、ranking、diversity selection までを一つのパイプラインにまとめた点が中心的な貢献である。

AI×抗体の観点では、実験的な根拠が最も強いのは通常型 IgG ではなく nanobody/VHH 設計である。最新版 v2 では、既知の bound homolog が乏しい 10 個の hard target に対し、各標的 60,000 個の nanobody を生成して最大 15 個を実験し、6/10 標的で screening hit を得た。ただし、これは候補配列の 60% が結合したという意味ではなく、少なくとも一つの hit が得られた標的の割合である。v2 は screening hit と confirmed binder を明確に区別し、6/10 の全てを確認済み 1:1 binder とは扱っていない。

したがって本論文は、BoltzGen を完成した抗体医薬生成器ではなく、大規模な in silico 候補生成から少数の実験候補へ絞り込む open-source の初期 hit 探索基盤として評価するのが妥当である。

## 書誌情報

- URL/DOI: https://doi.org/10.1101/2025.11.20.689494
- 最新版URL: https://www.biorxiv.org/content/10.1101/2025.11.20.689494v2.full
- 初版/更新日: v1 posted 2025-11-24、v2 posted 2026-06-16
- 著者・所属: Hannes Stark、Felix Faltings、MinGyu Choi、Yuxin Xie、Eunsu Hur、ほか計41名。MIT、Boltz、Open Athena、CTU Prague、IOCB Prague、NVIDIA、UCSF、UC Irvine、MPI、HHMI、Jameel Clinic など
- 掲載誌/プレプリントサーバー: bioRxiv preprint
- 査読状況: 査読前
- 論文ライセンス: CC BY 4.0
- リサーチ日: 2026-07-29 JST
- 分類: 新着 / モデル起点 / de novo binder design / nanobody design

v2 では、低分子・天然変性領域バインダー、選択性評価、追加実験データが加わり、screening hit と confirmed binder を区別する新しい分類が導入された。

## 背景と問題設定

従来の de novo binder design には、nanobody、miniprotein、peptide など特定の modality に特化した手法が多い。さらに、学習時に近縁な複合体を見た標的で評価されることがあり、未知標的への汎化能力が分かりにくい。実際の discovery campaign では、binding site、避ける領域、部分構造、二次構造、disulfide や staple などの共有結合、固定 framework といった条件も必要になる。

もう一つの問題は、生成モデルだけで実用パイプラインにならないことである。大量生成した候補に配列を割り当て、複合体として refold し、binder 単独でも fold するか確認し、界面・溶解性・疎水性を評価し、少数の非冗長候補へ絞る必要がある。BoltzGen は、構造予測と設計を同じ全原子拡散モデルのタスクとして扱い、この downstream funnel までを公開実装に含める。

## この論文のコアアイデア

BoltzGen のコアは、設計する amino-acid residue を離散ラベルではなく、固定長 14 原子の連続的な幾何表現として扱うことである。最初の 4 原子を backbone の N、Cα、C、O とし、残りの virtual atom を backbone atom 上に重ねる数と位置で residue type を符号化する。重ならなかった原子は side chain として解釈される。これにより、構造予測と配列・全原子構造生成に同じ diffusion objective を使える。

モデルは AlphaFold3/Boltz-2 系の trunk と diffusion module を継承する。PDB、AlphaFold DB、Boltz-1 distillation から、folding、binder design、motif scaffolding、unconditional design を共同学習する。推論時は design specification YAML を用い、標的構造、binding site、設計範囲、二次構造、共有結合などを条件として複合体を生成する。

nanobody では framework の構造・配列を固定し、CDR など指定領域だけを設計できる。現在の CLI には `nanobody-anything` と `antibody-anything` protocol があるが、本論文で強く prospective wet-lab validation されているのは nanobody 側である。

## 手法の詳細

- 入力データ: タンパク質、RNA、DNA、低分子などの molecular entity、配列、原子種、電荷、共有結合、標的構造または配列、設計対象 residue、binding site、structure group、二次構造条件。
- 出力: 標的と設計 binder を含む複合体の全原子 3D 座標、設計 residue の amino-acid type、refold 構造、confidence metric、物理・developability 指標、filtered/ranked designs。
- residue 表現: 設計 residue は 14 原子の固定長表現。virtual atom の backbone への重なり方で residue type を表す。
- trunk: protein は residue、RNA/DNA は nucleotide、small molecule は atom を token とする。token/pair representationを PairFormer で更新する。
- diffusion module: atom-level 3 層、token-level 24 層、atom-level 3 層。trunk は一度、diffusion module は反復実行する。
- sampler: デフォルト 300 function evaluations。residue type が決まりやすい生成区間を引き延ばす dilated schedule を使う。
- 学習 objective: denoising MSE、bond-length loss、smooth lDDT loss。
- 学習タスク: folding、protein-chain/interface binder design、non-protein interface design、motif scaffolding、unconditional protein design。
- crop: folding は最大 768 residue、生成タスクは最大 512 residue。
- design specification: target chain/residue、binding/not-binding site、structure visibility、design mask、長さ範囲、secondary structure、disulfide/staple/ligand bond、固定配列、対称性を YAML で指定できる。
- generation: BoltzGen diffusion で大量の複合体・binder候補を生成する。
- inverse folding: BoltzIF で配列を任意に再設計し、refold 可能性や溶解性を改善する。
- folding: Boltz-2 で binder–target complex を MSA なしで再予測し、生成構造との RMSD、pTM、ipTM、pAE を計算する。globular protein では binder 単独の refolding も行う。
- affinity: protein–small molecule 設計では Boltz-2 affinity module を使用する。
- analysis: 水素結合、salt bridge、buried surface area、solubility、surface hydrophobic patch などを計算する。
- filtering: 複数指標の worst weighted rank と quality–diversity selection で少数の非冗長候補を選ぶ。
- 推奨規模: まず数十 designs で設定を点検し、本番では 10,000–60,000 designs を生成する。

## データセットと評価設計

学習データ処理は主に Boltz-2 を継承している。

- PDB の実験構造
  - cutoff: 2023-06-01
  - Biological Assembly 1 を使用
  - AlphaFold3 と同様に結晶化補助分子、衝突 chain、極端に短い chain などを除外
- AlphaFold DB self-distillation
  - 約 500 万 protein monomer
  - global lDDT 0.5 以上
- Boltz-1 distillation
  - BindingDB/ChEMBL 由来 protein–ligand
  - Rfam 由来 RNA
  - JASPAR/SELEX 由来 protein–DNA

Boltz-2 で upsample していた antibody/TCR データは、生成多様性が低下したため BoltzGen では使用していない。nanobody 性能は、大量の抗体専用データだけで学習した結果というより、広い構造・相互作用データからの転移と解釈される。

wet-lab validation は 8 種類の design campaign、26 標的に及ぶ。未知標的評価の中心は 10 hard protein targets で、PDB 中の bound protein に対して 30% 以上の sequence identity を持たないよう選ばれた。実際の最大 identity は 1–27.9% だった。各標的について nanobody と miniprotein をそれぞれ 60,000 個生成し、各 modality の上位最大 15 個を SPR/BLI で評価した。加えて、既知 bound structure を持つ 5 easy benchmark targets、bioactive peptides、small molecules、Rag GTPase、天然変性領域、GyrA interface を扱った。

評価指標には、screening hit / confirmed binder、Kd、発現、SEC、secondary structure、functional neutralization、growth inhibition、HSA off-target、human proteome microarray、refolding RMSD、pTM/ipTM/pAE、界面水素結合、buried surface area、solubility、hydrophobic patch、diversity が含まれる。

## 主要結果

| 設計課題 | 計算・実験規模 | v2 の主要結果 | 解釈上の注意 |
|---|---|---|---|
| nanobody / miniprotein → 10 hard protein targets | 各標的・各 modality 60,000 生成、最大 15 候補を試験 | nanobody は 6/10 標的、miniprotein は 5/10 標的で screening hit | 標的単位の hit。全てが confirmed 1:1 binder ではない |
| nanobody / miniprotein → 5 easy benchmark targets | 各標的最大 15 候補 | nanobody は 4/5 標的で hit、3/5 で confirmed binder。miniprotein は 3/5 で hit | 標的が PDB に 100% 同一配列の bound 構造を持つ |
| protein → 3 bioactive peptides | 各標的 1,000 生成、目視を経て 6 候補を試験 | protegrin で 1.2、7.2 µM、indolicidin で nM binder、melittin で 0.41、4.4 µM。機能中和も確認 | 目視選抜を含み、凝集・oligomer 化候補もある |
| protein → 3 small molecules | brilacidin 9,999→5、heme 20,000→3、rucaparib 10,000→6 | brilacidin 最良 `Kdiss ≤ 3.9 µM`、heme 最良 `Kd = 697 ± 663 nM`、rucaparib 43–151.5 µM | rucaparib は専用手法の既報 `Kd < 5 nM` より弱い。専門家選抜あり |
| linear / disulfide peptide → Rag GTPase | linear 27、cyclic 26 を試験 | linear 8/27、cyclic 23/26 で濃度依存 binding。最良は 11 µM、6 µM | 高 hit 率でも多くは µM affinity |
| peptide → NPM1 / NUP98 IDR | NPM1 10、NUP98 9 候補 | NPM1 は 3 候補が nucleolar enrichment、2 候補を BLI で確認。NUP98 は 1 候補が condensate を溶解する挙動 | 全候補の定量的 affinity はない |
| antimicrobial peptide → GyrA interface | 1,808 designs と 1,788 interface mutants | 352/1,808（19.5%）が強い growth inhibition。interface 依存は厳格基準 54（3.0%）、緩い基準 99（5.5%） | 一般的 inhibition と意図した site への specific binding を分ける必要がある |

hard target の `6/10` は、少なくとも一つの screening hit が得られた target-level success である。計算上の分母は各標的 60,000 designs、wet-lab の分母は最大 15 selected designs であり、配列単位 hit rate ではない。v2 では primary SPR/BLI を複数回行い、nanobody hit を Adaptyv Bio と Sino Biological の orthogonal assay で確認しようとしている。nanobody designs は HSA への検出可能な結合を示さず、1 つの IDI2 miniprotein は 21,000 超の human protein microarray で off-target を示さなかった。

v1 や初期の紹介記事では、9 novel targets に対して nanobody、protein とも 6/9 標的で nM binder、すなわち 66% success と報告された。最新版 v2 は 10 hard targets に増え、nanobody 6/10、miniprotein 5/10 を screening hit と表現し、一部を confirmed 1:1 binder としている。今後は旧版の「66%で nM binder」を最新版の confirmed-binder率として引用しない方がよい。

同じ 10 hard targets を扱う後続研究 BoltzProt-1 は、BoltzGen の nanobody design 単位 confirmed-binder hit rate を 3.3%、新しい BoltzPPI ranking を用いた BoltzProt-1 を 8.0% と報告する。target-level の 6/10 とは定義が異なるが、BoltzGen では生成器だけでなく ranking が主要なボトルネックであることを示す。

## 重要なFigure/Table

- Figure/Table番号: Figure 1
- 何を示しているか: BoltzGen の全体像、design specification で指定できる条件、26 標的に対する wet-lab validation の一覧。
- 読み取り方: 数万候補の生成から、ranking/filtering/diversity selection を経て数十候補へ絞る funnel と、対応できる target/binder modality の幅を見る。
- 主要な数値・傾向: 8 design campaigns、26 targets。
- なぜ重要か: 「汎用 binder design」という論文全体の主張を最も短く把握できる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.1101/2025.11.20.689494v2.full.pdf
- ライセンス確認: 確認済み。CC BY 4.0。

- Figure/Table番号: Figure 2
- 何を示しているか: 10 hard targets と 5 easy benchmark targets に対する nanobody/miniprotein の実験結果、target sequence/interface novelty。
- 読み取り方: screening hit と confirmed binder を区別し、target-level success と個々の affinity を混同しない。
- 主要な数値・傾向: hard targets では nanobody 6/10、miniprotein 5/10 で screening hit。easy targets では nanobody 4/5、miniprotein 3/5 で hit。
- なぜ重要か: BoltzGen の nanobody 設計能力と未知標的への汎化を支える中心図である。
- Markdown内での扱い: 要約表
- 出典URL: https://www.biorxiv.org/content/10.1101/2025.11.20.689494v2.full.pdf
- ライセンス確認: 確認済み。CC BY 4.0。

- Figure/Table番号: Figure 8–10 / Method schematic
- 何を示しているか: pipeline、14 原子 residue encoding、Boltz-2 系 trunk + diffusion architecture。
- 読み取り方: residue identity を幾何学的に連続表現し、folding と design を同じモデルで扱う接続を見る。
- 主要な数値・傾向: 14 atoms/design residue、atom-level 3層 + token-level 24層 + atom-level 3層、デフォルト 300 function evaluations。
- なぜ重要か: BoltzGen が単なる backbone generator ではなく、配列・全原子構造・複合体を統合生成する仕組みを理解できる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.1101/2025.11.20.689494v2.full.pdf
- ライセンス確認: 確認済み。CC BY 4.0。

- Figure/Table番号: Figure 16
- 何を示しているか: protein/small-molecule target ごとに生成構造が変わるかを Vendi score で評価した target conditioning analysis。
- 読み取り方: 標的と無関係に同じ scaffold を出す model collapse が起きていないかを見る。
- 主要な数値・傾向: 110 targets に対して、BoltzGen は RFdiffusion/RFdiffusionAA より標的依存の構造多様性を示す。
- なぜ重要か: 生成器が target condition を実際に使っているという計算的根拠になる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.1101/2025.11.20.689494v2.full.pdf
- ライセンス確認: 確認済み。CC BY 4.0。

- Figure/Table番号: Figure 19
- 何を示しているか: 73–76 residue 付近で ubiquitin 様配列へ diversity collapse する memorization issue。
- 読み取り方: target conditioning と全体 diversity が良好でも、特定の長さで training-data frequency bias が表面化することを見る。
- 主要な数値・傾向: length 73 の 156 designs が全て ubiquitin 配列と 97% 以上一致。
- なぜ重要か: 実運用で length distribution と nearest-neighbor identity を監視すべき直接的根拠になる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.1101/2025.11.20.689494v2.full.pdf
- ライセンス確認: 確認済み。CC BY 4.0。

## Code / License / Weights

- コード公開: あり
- GitHub URL: https://github.com/HannesStark/boltzgen
- 実装種別: 公式 / 著者実装
- GitHub確認: 確認済み
- リポジトリのライセンス: MIT
- ライセンス確認元: GitHub LICENSE / repository metadata
- Hugging Face URL: https://huggingface.co/boltzgen/boltzgen-1
- Hugging Face種別: model weights / checkpoints
- Hugging Faceライセンス: MIT
- モデル重み・チェックポイント公開: あり
- model download: 推論時に約 6 GB を cache へ取得
- 学習データ公開: 一部あり
- 学習データURL: https://huggingface.co/datasets/boltzgen/boltzgen1_train
- inference data URL: https://huggingface.co/datasets/boltzgen/inference-data
- v2 nanobody sensorgrams: https://huggingface.co/datasets/boltzgen/update_sensograms/resolve/main/nanobody_sensograms.xlsx?download=true
- v2 protein sensorgrams: https://huggingface.co/datasets/boltzgen/update_sensograms/resolve/main/protein_sensograms.xlsx?download=true
- インストール: Python 3.11 以上で `pip install boltzgen`。Docker example あり。
- protocol: `protein-anything`、`peptide-anything`、`protein-small_molecule`、`nanobody-anything`、`antibody-anything`、`protein-redesign`。
- 再現性メモ: 公開 checkpoint で推論可能。training code と small model 用データ取得手順もある。ただし README は large model が追加 distillation datasets を必要とし、それらは今後公開予定と記載する。large model をゼロから完全再学習する再現性は現時点で限定される。

## 抗体研究・創薬への意味

BoltzGen は、VHH の CDR を標的構造へ直接条件付けして de novo 設計し、framework を固定したまま CDR 配列と複合体構造を同時生成できる。target sequence だけから folding と binder design を同時に行え、binding site や避ける領域も指定できるため、未知標的の初期 hit generation、epitope-directed nanobody design、intrabody、診断・研究用 VHH の探索に接続しやすい。

open-source で code、weights、training/inference 手順、sensorgram が公開されている点も重要である。closed model の headline success rate だけでなく、generated pool、filter funnel、実験分母、raw sensorgram を検討できる。

一方、nanobody の結果を full-length IgG や Fab へそのまま外挿できない。CLI に antibody protocol があっても、本論文の strongest prospective validation は VHH である。affinity maturation、humanization、immunogenicity低減、Fc engineering、発現・凝集・粘度・薬物動態の最適化は別工程になる。

AI×抗体論文で使う場合は、次のような表現が妥当である。

> BoltzGen は、構造予測と全原子生成を統合し、固定 VHH framework 上の CDR 設計を含む複数の binder modality を単一パイプラインで扱う open-source model である。2026年の v2 では、bound homolog が乏しい 10 標的のうち 6 標的で nanobody の screening hit を得ており、未知標的への構造条件付き設計の可能性を prospective wet-lab validation で示した。ただし、この 6/10 は target-level screening-hit 率であり、confirmed-binder の配列単位 hit rate ではない。

## 限界と注意点

著者が述べる限界:

- high-affinity binding は therapeutic development の最初の一歩にすぎず、selectivity、developability、標的固有の性質を別途評価する必要がある。
- 73–76 residue の binder では ubiquitin memorization により generation diversity が collapse する場合がある。
- BoltzGen は zero-shot、plug-and-play、失敗しない手法とは主張しておらず、小規模試行、構造点検、設定変更、再実行を推奨している。

追加で重要な注意点:

- v2 は 2026-06-16 公開の preprint で、査読前である。
- 主要な 6/10 は confirmed-binder 率ではなく target-level screening-hit 率である。
- hard target では各標的・各 modality 60,000 候補を生成して 15 候補へ絞る。計算量、filter、候補選択が結果に大きく寄与する。
- bioactive peptide と small-molecule campaign は専門家の目視・化学知識による選抜を含み、完全自動ではない。
- BoltzGen 生成構造を同系統の Boltz-2 で refold/rank するため、生成器と評価器の誤差が相関する可能性がある。
- 一部で expression、SEC、hydrophobic patch、HSA off-target を調べているが、全 design の熱安定性、polyreactivity、aggregation、粘度、免疫原性、血中安定性を網羅していない。
- target sequence identity <30% は厳しい novelty criterion だが、全ての局所構造 motif、interface pattern、化学的相互作用が学習データから独立であることを保証しない。
- BoltzGenv0 には fixed design residue を target として template 処理する bug があり、refolding filter がほぼ無効になった。別 campaign で 12/12 designs が発現しなかったことを受け、binder 単独 refolding が追加された。実装 version と論文結果を対応付ける必要がある。
- training code は公開されたが、large model の完全再学習に必要な distillation datasets は一部未公開である。

## この論文を読む上での前提知識

- de novo binder design: 既知 binder の単純改変ではなく、標的構造などから新しい結合分子を設計すること。
- nanobody/VHH: camelid heavy-chain-only antibody の可変ドメインに由来する単一ドメイン抗体。通常型 IgG より小さく、単一 chain で扱える。
- diffusion model: データに noise を加える過程の逆過程を学習し、noise から構造を生成するモデル。
- all-atom model: backbone だけでなく side chain、ligand、核酸などの原子を扱うモデル。
- inverse folding: backbone/structure を条件に、その構造を取りやすい amino-acid sequence を設計する問題。
- pTM、ipTM、pAE: 構造予測の confidence metric。binder design では interface の信頼度が重要になる。
- SPR/BLI: 分子間相互作用を測定する label-free assay。sensorgram の quality により screening hit と kinetic affinity を確定できる binder を分ける必要がある。
- Kd: dissociation constant。小さいほど一般に強い結合を表す。
- developability: 発現、安定性、凝集、溶解性、polyreactivity、製造性など、薬剤・試薬候補として開発可能かを左右する性質。
- target-level success / design-level hit rate: 少なくとも一つ hit を得た標的の割合と、試験した配列のうち hit になった割合。直接比較できない。

## 今日この1報を選んだ理由

このプロジェクトでは、CHIMERA-Bench、AgForce、OpenGerminal など、抗原条件付き抗体設計の benchmark、生成モデル、実装パイプラインを順に調べている。BoltzGen はこの流れに対し、特定の抗体モデルに閉じず、構造予測と全原子 binder generation を統合し、nanobody を含む複数 modality で prospective wet-lab validation を行った基盤的研究として位置付けられる。

特に、v1 で広まった「9標的中6標的、66%で nM binder」という数字が、v2 では 10標的中6標的の screening hit へ再分類されている点は、AI抗体論文の成功率を比較する上で重要である。公開 sensorgram と後続 BoltzProt-1 により、target-level success、screening hit、confirmed binder、ranking 性能を切り分けて読めるため、単なるモデル紹介以上の価値がある。

## 読む優先度

High。AIによる抗体・nanobody設計を論じるなら、open-source の全原子 generative model、未知標的への prospective validation、screening hit と confirmed binder の区別、大規模生成後の ranking funnel を一度に学べる。通常型抗体への直接的証拠は弱いが、VHH設計と汎用 binder design の接点を理解する基盤論文として優先度が高い。

## 自分用メモ

- 後で深掘りしたい点: v2 nanobody/protein sensorgram を標的別に再判定し、`screening_hit`、`confirmed_binder`、`Kd`、assay orientation、replicate を表にする。
- tested 配列を取得し、training cutoff 前の PDB、OAS、特許配列との identity を再計算したい。
- nanobody framework、CDR 長、germline proximity、humanization 可能性を比較したい。
- BoltzGen と BoltzProt-1 で、同じ generated pool に ranking だけを変えた head-to-head 結果を確認したい。
- confirmed binder の発現量、Tm、SEC、HIC、polyreactivity を配列単位で集めたい。
- `antibody-anything` protocol の Fab/full antibody prospective evidence は別タスクで調査する。
- 実装を触る場合は target chain/residue numbering を `label_asym_id` 基準で確認し、`boltzgen check` と Mol* で design specification を可視化する。
- 73–76 residue は避けるか、ubiquitin identity filter を追加する。
- Obsidianでリンクしたいキーワード: [[Boltz-2]], [[nanobody design]], [[de novo binder design]], [[VHH]], [[screening hit]], [[confirmed binder]], [[developability]], [[BoltzProt-1]]

## 関連キーワード

- BoltzGen
- Boltz-2
- all-atom generative model
- diffusion model
- de novo binder design
- nanobody design
- VHH CDR design
- antibody design
- protein binder
- peptide binder
- small-molecule binder
- inverse folding
- BoltzIF
- structure prediction
- target conditioning
- design specification language
- screening hit
- confirmed binder
- prospective wet-lab validation
- developability
- quality-diversity selection
- ubiquitin memorization
- BoltzProt-1

## 検索ログ

- 検索したデータベース/クエリ:
  - Web検索: `BoltzGen paper protein binder design`
  - Web検索: `BoltzGen bioRxiv paper`
  - Web検索: `BoltzGen GitHub`
  - Web検索: `BoltzGen documentation binder design`
  - Web検索: `"BoltzGen: Toward Universal Binder Design" DOI authors journal`
  - Web検索: `2025.11.20.689494v2 "ten novel targets"`
  - bioRxiv API: `https://api.biorxiv.org/details/biorxiv/10.1101/2025.11.20.689494/na/json`
  - 公式 v2 JATS XML と PDF を取得し、本文、補足方法、training cutoff、実験結果、license を確認
  - GitHub README で install、protocol、pipeline step、推奨生成数、training instructions、未公開 distillation data を確認
  - Hugging Face organization/model/dataset で weights、license、training data、sensorgram を確認
- 確認した論文:
  - BoltzGen: Toward Universal Binder Design, bioRxiv v2
  - BoltzProt-1: Towards Efficient De Novo Binder Design with Good Developability
  - v1 を収載する PubMed/PMC は、版差分の確認にのみ使用
- 最終的にこの1報を採用した理由: nanobody を含む汎用 all-atom binder design を open-source で実装し、未知性を意識した標的群で prospective wet-lab validation を行っているため。v2 の再分類により、AI抗体研究の成功率を批判的に読む材料にもなる。
- 新着論文と基盤論文のバランスをどう考えたか: v2 は 2026-06-16 の新着更新だが、Boltz-1/Boltz-2、AlphaFold3、inverse folding、diffusion binder design を統合する基盤性が高い。前回までの抗体専用モデル・benchmarkを、汎用 binder design の流れへ接続できる。
- PDF/HTML/Supplementaryを確認できたか: bioRxiv v2 full HTML、78ページPDF、JATS XML、Supplementary Methods を確認。
- Figure/Tableを確認したURLやライセンス状況: https://www.biorxiv.org/content/10.1101/2025.11.20.689494v2.full.pdf 、CC BY 4.0。
- GitHub/コード検索で確認したURL: https://github.com/HannesStark/boltzgen
- Hugging Faceで確認したURL:
  - https://huggingface.co/boltzgen/boltzgen-1
  - https://huggingface.co/datasets/boltzgen/boltzgen1_train
  - https://huggingface.co/datasets/boltzgen/inference-data
  - https://huggingface.co/datasets/boltzgen/update_sensograms
- リポジトリライセンスの確認元: GitHub LICENSE / repository metadata。MIT。
- model weights license の確認元: Hugging Face model card metadata。MIT。
- 実装上の再現性メモ: Python 3.11以上、GPU推奨、約6 GB model download。small model training instructions は公開されるが、large model の追加 distillation datasets は未公開。
