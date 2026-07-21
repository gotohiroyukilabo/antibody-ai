# CHIMERA-Bench: A Benchmark Dataset for Epitope-Specific Antibody Design

## まず何の論文か

CHIMERA-Bench は、抗体の CDR 配列・構造をエピトープ条件付きで設計するための、標準化されたベンチマークデータセットと評価プロトコルを提案する論文である。抗体設計分野では DiffAb、MEAN、RefineGNN、AbDockGen など多様な深層生成モデルが登場しているが、各論文が異なる SAbDab スナップショット、異なるフィルタ、異なるテストセット、異なる RMSD や contact 定義を使っており、数字を横並びにしにくい。本論文はこの混乱を、抗原構造、標的エピトープ、抗体フレームワークを入力し、CDR 残基のアミノ酸、座標、局所フレームを設計するという単一タスクにまとめる。データは SAbDab から構築され、最終的に 2,922 個の抗体-抗原複合体、エピトープ・パラトープ注釈、IMGT/Chothia 番号付け、contact map、構造特徴を含む。評価分割は random split ではなく、未知エピトープ、未知抗原フォールド、時間的な将来データという 3 種類で、特に「見たことのないエピトープに本当に条件付けできるか」を問う設計になっている。評価指標も AAR だけでなく、contact AAR、RMSD、TM-score、Fnat、iRMSD、DockQ、Epitope F1、liability motif 数を含むため、単なる配列復元ではなく、標的エピトープに正しく接触する設計かを見られる。著者らは 11 種類の既存手法を同一条件で再学習・再評価し、配列復元が高い手法と界面品質が高い手法が一致しないことを示した。特に MEAN/RAAD は AAR と backbone RMSD で強く、RefineGNN は Fnat、DockQ、EpiF1 など界面指標で強い。結論として、この論文は「抗体設計モデルの性能」を 1 つの数字で語るのは危険であり、エピトープ特異性、界面再現、長い CDR-H3 での破綻を分けて見るべきだと主張している。モデルそのものを提案する論文ではないが、AI 抗体設計の評価基盤として、今後の生成モデル論文を読む際の物差しになる。

## 書誌情報

- URL/DOI: https://arxiv.org/abs/2603.13431 / https://doi.org/10.48550/arXiv.2603.13431
- 公開日/更新日: arXiv v1 2026-03-13、v3 2026-06-05。OpenReview は Published 2026-03-02、Last Modified 2026-05-26。
- 著者・所属: Mansoor Ahmed, Nadeem Taj, Imdad Ullah Khan, Hemanth Venkateswara, Murray Patterson。所属は Georgia State University、Georgia Institute of Technology、University of Engineering and Technology Lahore、Lahore University of Management Sciences。
- 掲載誌/プレプリントサーバー: arXiv、ICLR 2026 Workshop on Generative and Experimental Perspectives for Biomolecular Design (GEM) / OpenReview。
- リサーチ日: 2026-06-29
- 分類: ベンチマーク・データセット / 新着 / モデル評価基盤

## 背景と問題設定

抗体医薬の計算設計では、抗原上のどのエピトープを狙うか、どの CDR を変えるか、配列だけを変えるか構造も同時に生成するか、既存の抗体フレームワークを固定するかまで、問題設定が論文ごとに異なる。SAbDab は抗体構造の中心的データベースだが、そのままでは冗長性、品質差、番号付け、エピトープ注釈、標準 train/test split を持たない。既存の RAbD benchmark は代表的だが 60 複合体規模で、現代の生成モデルを体系的に比較するには小さく、エピトープ特異性の評価も弱い。

この状況では、ある手法が AAR で高い数字を出しても、それが同一エピトープに対する一般化なのか、近い抗原や似た CDR を覚えただけなのか、界面を正しく作ったのかが判断しにくい。さらに contact cutoff や RMSD の alignment 方法も論文ごとに違い、同じ名前の指標でも比較できないことがある。抗体設計では、もっとも医薬的に重要なのは「もっともらしい CDR」ではなく「指定した抗原部位に、意図した界面で結合しうる CDR」である。したがって、エピトープを条件として与え、その部位への接触と off-target 的な接触を評価するベンチマークが必要になる。

CHIMERA-Bench はこの問題に対して、エピトープ条件付き CDR sequence-structure co-design を標準タスクとして定義する。これにより、inverse folding、structure prediction、docking、CDR redesign、de novo generation のような断片化したタスクを、抗原、エピトープ、抗体フレームワークを条件に CDR を設計するという枠組みに寄せる。直接のモデル提案ではないが、抗体 AI の研究で「何をもって良い設計と呼ぶか」を定義するため、後続研究の評価設計に大きく効く論文である。

## この論文のコアアイデア

コアアイデアは、抗体 CDR 設計を「標的エピトープを明示した条件付き生成問題」として再定義し、そのためのデータ、分割、指標、baseline 変換器をまとめて公開することである。入力は抗原構造、標的エピトープ、抗体フレームワークであり、出力は設計対象 CDR 残基のアミノ酸タイプ、C alpha 座標、局所フレーム方向を含む。設計した CDR は、単に天然 CDR と配列が一致すればよいのではなく、標的エピトープに contact し、構造的に破綻せず、界面の native contact を回復し、manufacturability 上の liability motif を過剰に含まないことが求められる。

データセットとしての工夫は、SAbDab から出発して、タンパク質/ペプチド抗原、paired VH/VL、分解能 4.0 A 以下、MMseqs2 による 95% sequence identity クラスタリング、ANARCI 番号付け可能性、保存残基、CDR 完全性、backbone integrity を通して絞り込む点にある。これにより 20,509 件の SAbDab entry から、最終的に 2,922 複合体を作る。エピトープとパラトープは 4.5 A heavy-atom contact cutoff で定義され、各複合体には IMGT/Chothia の CDR mask、contact pair、atom14 座標、抗原表面点と化学特徴などが付与される。

評価としての工夫は、3 つの split にある。epitope-group split は同じ抗原上の同じエピトープ残基集合をクラスタ化し、test では未知エピトープパターンを出す。antigen-fold split は抗原 identity/トポロジーをまたいだ一般化を見る。temporal split は PDB deposition date に基づき、将来に出る構造への prospective evaluation を模擬する。これらは CDR-H3 長、エピトープサイズ、抗原サイズの分布が大きくずれないよう設計され、単なる難易度差ではなく一般化軸の差を評価しようとしている。

## 手法の詳細

- 入力データ: 抗体-抗原複合体構造、抗体 heavy/light chain 配列、抗原配列、各 chain の座標、CDR mask、標的エピトープ、抗体フレームワーク、contact pair、抗原表面特徴。
- 出力: ベンチマークタスク上では設計対象 CDR のアミノ酸、C alpha 座標、局所フレーム方向。データセット配布物としては metadata、split JSON、PDB 構造、PyTorch tensor 特徴量、評価コード。
- モデル/アルゴリズム: CHIMERA-Bench 自体はモデルではなく、データ構築 pipeline と評価 protocol。baseline として RAAD、MEAN、dyMEAN、DiffAb、AbMEGD、RADAb、AbFlowNet、dyAb、RefineGNN、AbDockGen、AbODE を同一条件で評価する。
- 特徴量・表現学習: 各 complex feature は heavy/light/antigen の atom14 座標、C alpha 座標、IMGT/Chothia numbering、CDR mask、epitope/paratope residue、contact pair、抗原表面点 128 個と hydropathy、charge、H-bond、aromaticity、polarity などの表面化学特徴を含む。
- 学習方法: 各 baseline は著者公開コードと default hyperparameter を使って CHIMERA-Bench 上で再学習された。手法ごとの損失や sampling は baseline 側に依存する。
- 損失関数・目的関数: CHIMERA-Bench 自体には単一の学習損失はない。評価タスクとしては、条件付き分布のもとで CDR 残基を設計し、標的エピトープ contact と precision を満たすことを目的とする。
- 推論方法: 各 baseline の生成方式に依存する。GNN、diffusion、flow matching、autoregressive、hierarchical equivariant network、neural ODE などを比較する。
- ベースライン: 11 手法、6 generative paradigm。主なものは equivariant GNN 系の RAAD/MEAN/dyMEAN、diffusion 系の DiffAb/AbMEGD/RADAb/AbFlowNet、flow matching の dyAb、autoregressive の RefineGNN、hierarchical refinement の AbDockGen、conjoined ODE の AbODE。
- 評価指標: AAR、CAAR、PPL、C alpha RMSD、TM-score、Fnat、iRMSD、DockQ、Epitope precision/recall/F1、liability motif 数。dataset annotation と CAAR には 4.5 A heavy-atom cutoff、interface/epitope metrics には 8 A C alpha cutoff を使う。
- 実装上の重要点: 既存手法ごとに入力フォーマットが違うため、CHIMERA-Bench は 5 種類のデータカテゴリへの converter と評価パイプラインを提供する。Hugging Face dataset と Zenodo、GitHub でデータとコードが公開されている。

## データセットと評価設計

- 使用データセット名: CHIMERA-Bench v1.0
- データの規模: 論文本文では 2,922 antibody-antigen complexes、2,721 unique PDB。GitHub README では pre-computed feature 2,922 `.pt` files、Hugging Face dataset card では 2,941 `.pt` files と記載があり、複合体数と feature file 数に表記差があるため注意が必要。
- 抗体、抗原、タンパク質、複合体など対象の内訳: 2,485 protein antigen、437 peptide antigen。median resolution は 2.72 A。X-ray が 1,929、cryo-EM が 989。平均エピトープ残基数 17.9、平均パラトープ残基数 20.5、平均 contact pair 数 45.6。
- train/validation/test の分け方: epitope-group は 2,338/292/292、antigen-fold は 2,338/292/292、temporal は 2,337/292/293。
- リーク対策やクラスタ分割の有無: MMseqs2 による 95% sequence identity clustering と、split ごとの cluster-level separation を使う。epitope-group はエピトープ残基集合、antigen-fold は抗原 identity/トポロジー、temporal は PDB deposition date に基づく。
- 評価指標: sequence quality、structural quality、binding interface quality、epitope specificity、designability の 5 群。OpenReview abstract では seven metric groups と記載されているが、arXiv v3 本文は five metric groups としているため、最終的な精読では arXiv v3 を優先する。
- ベースラインや比較対象: 11 手法を同一データ・同一 split・同一 metric で評価する。RFAntibody は full PDB で学習済み重みの split が不明なため漏洩制御下で評価できず、ProteinMPNN も将来課題として扱われる。
- この評価設計が妥当かどうか: 妥当性は高い。特に random split では隠れる「未知エピトープへの一般化」を独立 split と EpiF1 で評価する点が重要である。一方、実験的親和性、発現性、熱安定性、免疫原性、抗原の conformational flexibility は直接評価されないため、医薬候補としての成功を保証する評価ではない。

## 主要結果

CDR-H3 の epitope-group split では、配列復元では MEAN が AAR 0.42、RAAD が 0.38、dyMEAN が 0.35 と高く、diffusion/flow 系の DiffAb、AbFlowNet、AbMEGD、RADAb は 0.22 前後、dyAb は 0.27 だった。構造 RMSD でも RAAD が 1.95 A、MEAN が 2.01 A と良く、DiffAb は 2.64 A、AbFlowNet は 2.70 A、AbMEGD は 2.76 A、dyAb は 3.31 A であった。これは、少なくともこのベンチマークの CDR-H3 では、大きな生成モデルや diffusion 系が必ずしも小型の equivariant GNN を上回らないことを示す。

しかし interface quality を見ると順位は変わる。RefineGNN は AAR 0.23 と高くないが、Fnat 0.62、iRMSD 1.55 A、DockQ 0.71、EpiF1 0.76 と、界面・エピトープ指標ではもっとも強い。RAAD と MEAN は AAR では上位だが Fnat は 0.49/0.48、DockQ は 0.64/0.63、EpiF1 はどちらも 0.67 で、RefineGNN に劣る。つまり、天然配列に似ていることと、標的エピトープに良い界面を作ることは分離して評価すべきである。

未知エピトープへの一般化では、epitope-group split が特に厳しい。RAAD の AAR は 3 split でほぼ 0.38 と変わらないが、Fnat は antigen-fold の 0.62 から epitope-group の 0.49 に下がり、EpiF1 も 0.78 から 0.67 に下がる。著者らは、sequence recovery だけを見ると一般化しているように見えるが、interface と epitope metrics を見ると未知エピトープで落ちる、と解釈している。これは抗体生成モデルが抗原やエピトープ条件を本当に使っているかを問ううえで重要である。

All-CDR design では、H3 が最も難しく、H1 が比較的容易という明確な難易度差が出る。例えば RAAD は H1 AAR 0.72、H2 AAR 0.67、H3 AAR 0.38 で、H3 の難しさが目立つ。DiffAb は全 6 CDR を設計でき、H1 AAR 0.57、H2 0.29、H3 0.22、L1 0.55、L2 0.50、L3 0.43 という結果である。light chain は平均で paratope contact の 35% に関わるため、H3 だけでなく全 CDR を扱う評価が必要だという示唆もある。

計算コストの分析も重要である。MEAN は 0.7M parameters、RAAD は 6.8M parameters で、epitope-group の総学習時間はそれぞれ 3.5 GPU-hours、2.8 GPU-hours、推論は 1 split あたり 1 分未満である。一方 RADAb は total 661M parameters、trainable 10.1M parameters、推論が epitope-group 119.6 分、temporal 204.4 分と重いが、DiffAb に対して明確な性能改善を示さない。著者らは、性能を決めるのは単純なモデル容量ではなく、構造条件付きの inductive bias と評価タスクへの適合だと結論づける。

## 重要なFigure/Table

- Figure/Table番号: Figure 1
- 何を示しているか: SAbDab 20,509 entries から、品質フィルタ、95% identity の重複除去、IMGT/Chothia annotation、contact/epitope/paratope 作成、validation、feature 化、split 作成までの pipeline。
- 読み取り方: 単なるデータ収集ではなく、抗体設計ベンチマークに必要な「番号付け可能」「CDR 完全」「backbone が破綻しない」「contact が定義できる」構造だけを残している。
- 主要な数値・傾向: 20,509 SAbDab entries から 9,171 件に filter され、2,981 件に deduplicate され、validation 後 2,922 complexes になる。
- なぜ重要か: benchmark の信頼性はモデルよりもデータ構築に依存する。ここが不透明だと、後続モデルの比較が成り立たない。
- Markdown内での扱い: リンクのみ
- 出典URL: https://arxiv.org/html/2603.13431
- ライセンス確認: 確認済み。arXiv HTML は CC BY 4.0 表示あり。ただし今回は画像保存せず要約のみ。

- Figure/Table番号: Table 2
- 何を示しているか: 3 種類の評価 split の train/validation/test 数と一般化ターゲット。
- 読み取り方: epitope-group は未知エピトープ、antigen-fold は未知抗原トポロジー、temporal は将来構造への一般化を問う。いずれも約 80/10/10 に近い分割である。
- 主要な数値・傾向:

| Split | Train | Val | Test | Generalization target |
|---|---:|---:|---:|---|
| Epitope-group | 2,338 | 292 | 292 | Unseen epitope patterns |
| Antigen-fold | 2,338 | 292 | 292 | Unseen antigen topologies |
| Temporal | 2,337 | 292 | 293 | Prospective, post-2023 |

- なぜ重要か: 抗体設計では同じ抗原・近いエピトープが train/test にまたがると過大評価になりやすい。split の設計そのものがこの論文の価値である。
- Markdown内での扱い: 要約表
- 出典URL: https://arxiv.org/html/2603.13431
- ライセンス確認: 確認済み。arXiv HTML は CC BY 4.0 表示あり。

- Figure/Table番号: Table 3 / Figure 6
- 何を示しているか: CDR-H3 design の主要 11 baseline 比較。特に epitope-group test split における AAR、RMSD、Fnat、DockQ、EpiF1 など。
- 読み取り方: AAR だけで見ると MEAN/RAAD が強いが、界面品質では RefineGNN が強い。AbODE は CAAR が高めでも RMSD が壊れており、配列と構造の decoupling が見える。
- 主要な数値・傾向:

| Method | AAR | RMSD | Fnat | DockQ | EpiF1 |
|---|---:|---:|---:|---:|---:|
| MEAN | 0.42 | 2.01 | 0.48 | 0.63 | 0.67 |
| RAAD | 0.38 | 1.95 | 0.49 | 0.64 | 0.67 |
| RefineGNN | 0.23 | 3.07 | 0.62 | 0.71 | 0.76 |
| DiffAb | 0.22 | 2.64 | 0.48 | 0.58 | 0.56 |
| AbODE | 0.31 | 16.40 | 0.08 | 0.34 | 0.20 |

- なぜ重要か: 「高い sequence recovery = 良い抗体設計」ではないことを最も端的に示す表である。
- Markdown内での扱い: 要約表
- 出典URL: https://arxiv.org/html/2603.13431
- ライセンス確認: 確認済み。arXiv HTML は CC BY 4.0 表示あり。

## Code / License / Weights

- コード公開: あり
- GitHub URL: https://github.com/mansoor181/chimera-bench
- GitHub以外のコードURL: OpenReview https://openreview.net/forum?id=PyZvVIJbSy
- 実装種別: 公式 / 著者実装
- GitHub確認: 確認済み
- リポジトリのライセンス: Code は MIT License。Data は CC-BY 4.0。
- ライセンス確認元: GitHub の LICENSE ファイル、LICENSE-DATA ファイル、README/repository metadata、Hugging Face dataset card。
- Hugging Face URL: https://huggingface.co/datasets/mansoorbaloch/chimera-bench
- Hugging Face種別: dataset
- モデル重み・チェックポイント公開: 該当なし
- 重み公開URL: 該当なし
- データセット公開: あり
- データセットURL: https://huggingface.co/datasets/mansoorbaloch/chimera-bench / https://zenodo.org/records/20598827
- 再現性メモ: GitHub には package、evaluation、baseline converter、leaderboard.csv、sample_data、tests がある。README によると smoke test は `pytest tests/ -q`、データ取得は Hugging Face CLI または Zenodo。Hugging Face dataset card は total file size 3.26 GB、Zenodo は data volume 4.4 GB と表示されており、取得元やバージョンでサイズ表記が違う可能性がある。モデル重みを出す論文ではなく、既存 baseline の再学習結果と評価基盤が主成果である。

## 抗体研究・創薬への意味

抗体設計では、候補配列の humanness や天然様性だけでなく、目的エピトープへの結合、off-target 的な界面形成の回避、長い CDR-H3 の形状制御が重要になる。CHIMERA-Bench は、まさにこの点を評価できるようにしたベンチマークである。抗体医薬開発では、例えばウイルス抗原の保存エピトープ、受容体結合部位、escape mutation を避ける部位など、狙いたいエピトープが先に決まることが多い。そのため、抗原全体に何となく結合しそうな CDR ではなく、指定エピトープに contact する設計を評価する枠組みは実用に近い。

また、この論文はモデル選定にも示唆を与える。大きな PLM や diffusion model を組み込めば必ず良いわけではなく、RefineGNN のような autoregressive/structure-conditioned な inductive bias が interface metrics で強い場合がある。抗体設計プロジェクトで生成モデルを評価するなら、AAR、perplexity、RMSD だけでなく、Fnat、DockQ、EpiF1、長い H3 での性能、peptide/protein antigen 別性能を見るべきである。将来的に自前の抗体設計モデルを作る場合にも、CHIMERA-Bench は train/test 分割、入力形式、baseline 比較、評価コードの出発点になる。

## 限界と注意点

著者が述べている限界:

- 静的な結晶構造や cryo-EM 構造に基づく評価であり、抗原・抗体の conformational flexibility、solvent effect、誘導適合を直接扱わない。
- experimental binding affinity を評価していないため、ベンチマークスコアが高くても実験的に強く結合するとは限らない。
- SAbDab/PDB 由来のバイアスを引き継ぐ。特に viral surface antigens、human-derived antibodies、X-ray crystallography への偏りがある。
- RFAntibody のように full PDB で学習済み重みの split が不透明な手法や、ProteinMPNN のような一般 inverse folding 手法は、今回の漏洩制御下では完全に評価されていない。

読んで気づいた限界:

- データセット構築が構造既知の抗体-抗原複合体に依存するため、構造が解かれやすい抗原、成功した抗体、研究コミュニティで注目された病原体に偏る。
- エピトープは contact cutoff で定義されるため、機能的エピトープ、競合エピトープ、escape mutation site、glycan を含む複雑な抗原表面とは一致しない場合がある。
- affinity、developability、aggregation、expression、thermal stability、immunogenicity は評価外であり、創薬の downstream filter としては別データが必要である。
- H鎖・L鎖の全可変領域設計や scaffold selection ではなく、主に既存 framework 上の CDR co-design にフォーカスしている。
- Hugging Face と GitHub README の feature file 数に 2,922 と 2,941 の表記差があり、利用時は dataset version と metadata の整合性を確認したい。
- baseline retraining は著者公開コードと default hyperparameter に依存するため、各手法を最適チューニングした場合の上限性能とは限らない。

## この論文を読む上での前提知識

- SAbDab: Structural Antibody Database。抗体構造研究で広く使われるデータベースだが、機械学習用には冗長性除去、品質フィルタ、分割設計、注釈付けが必要になる。
- CDR と framework: 抗体可変領域のうち抗原認識に強く関わるループが CDR、比較的保存された骨格部分が framework。特に CDR-H3 は長さと配列多様性が高く、結合特異性の中心になりやすい。
- エピトープ/パラトープ: 抗原側で抗体が接触する領域がエピトープ、抗体側で抗原に接触する領域がパラトープ。構造データでは距離 cutoff に基づいて定義することが多い。
- AAR と CAAR: Amino Acid Recovery は設計配列が天然配列をどの程度復元したかを見る。CAAR は contact residue に限定した AAR で、界面に近い残基の復元を重視する。
- RMSD と DockQ: RMSD は構造座標のずれを測る。DockQ は Fnat、iRMSD、ligand RMSD を組み合わせた docking quality 指標で、タンパク質複合体の界面再現に向く。
- Equivariant GNN / diffusion / flow matching: 3D 分子構造を扱う生成モデルの主要系統。回転・並進対称性を保つ設計や、ノイズ除去過程で構造と配列を生成する設計が多い。
- データリークと cluster split: 似た抗体・抗原・エピトープが train/test にまたがると、未知標的への一般化を過大評価する。配列やエピトープ単位でクラスタ分割することが重要。

## 今日この1報を選んだ理由

前回は AbAffinity という新着の親和性予測モデルを読んだため、今日は個別モデルではなく、抗体設計モデルを評価するための基盤論文を選んだ。CHIMERA-Bench は 2026 年 3 月投稿、6 月改訂の新しいベンチマークで、コード、データ、Hugging Face、Zenodo、ライセンスが明確に公開されている。今後 AgForce、AbMEGD、IgCraft、RFAntibody、DiffAb 系の論文を読むとき、AAR だけでなく EpiF1、Fnat、DockQ、split 設計を見る視点を与えてくれる。特に「配列復元と界面品質は一致しない」「未知エピトープ split で性能が落ちる」という結果は、AI 抗体設計の評価で非常に重要である。候補には AgForce もあったが、AgForce 自体が CHIMERA-Bench を使っているため、先にベンチマークを理解するほうが長期的な学習価値が高いと判断した。

## 読む優先度

High。モデル提案論文ではないが、抗体 CDR 生成・構造設計の性能評価を読むための共通言語になる。特に生成モデルの leaderboard 的な数字を鵜呑みにせず、split、contact 定義、interface metrics、長い CDR-H3 の失敗を確認する習慣を作るために優先度が高い。

## 自分用メモ

- 後で深掘りしたい点: GitHub の `leaderboard.csv` と `evaluation/` を見て、EpiF1、Fnat、DockQ の実装を確認する。
- 関連して読むべき論文: DiffAb、MEAN、RefineGNN、AbDockGen、RAAD、RFAntibody、AbBiBench、SAbDab、RAbD。
- 実装を触る場合の入口: GitHub を clone し、`sample_data/` で `pytest tests/ -q`、次に Hugging Face dataset を取得して `scripts/build_leaderboard.py` を試す。
- Obsidianでリンクしたいキーワード: [[SAbDab]], [[CDR-H3]], [[Epitope-conditioned antibody design]], [[DockQ]], [[DiffAb]], [[MEAN]], [[RefineGNN]], [[Antibody benchmark]]

## 関連キーワード

- antibody design
- epitope-conditioned generation
- CDR sequence-structure co-design
- SAbDab
- benchmark dataset
- leakage prevention
- DockQ
- Epitope F1
- equivariant GNN
- diffusion model

## 検索ログ

- 検索したデータベース/クエリ:
  - Web/arXiv: `2026 antibody design machine learning arXiv antibody antigen binding deep learning`
  - Web/arXiv: `2025 AI antibody design generation structure prediction paper GitHub Hugging Face antibody`
  - Web/arXiv: `site:arxiv.org antibody design diffusion model GitHub 2025 antibody`
  - Web: `"CHIMERA-Bench" antibody`
  - Web: `"AgForce Enables Antigen-conditioned Generative Antibody Design"`
- 候補にした論文:
  - CHIMERA-Bench: A Benchmark Dataset for Epitope-Specific Antibody Design
  - AgForce Enables Antigen-conditioned Generative Antibody Design
  - Antibody Design and Optimization with Multi-scale Equivariant Graph Diffusion Models for Accurate Complex Antigen Binding
  - IgCraft: A versatile sequence generation framework for antibody discovery and engineering
  - AbGPT: De Novo Antibody Design via Generative Language Modeling
- 最終的にこの1報を採用した理由: CHIMERA-Bench は新着性があり、抗体設計モデル評価の基盤になる。コード、データ、Hugging Face、Zenodo、ライセンスが揃っており、後続の AgForce などを読む前提として重要。
- 新着論文と基盤論文のバランスをどう考えたか: 2026 年の新着だが、役割はベンチマーク基盤。前回が個別モデルだったため、今回は評価基盤を選んで知識ベースの土台を補強した。
- PDF/HTML/Supplementaryを確認できたか: arXiv abstract、arXiv HTML、OpenReview、GitHub README、LICENSE、LICENSE-DATA、Hugging Face dataset card、Zenodo record を確認。PDF は arXiv から利用可能だが、本文確認は主に HTML。
- Figure/Tableを確認したURLやライセンス確認状況: arXiv HTML https://arxiv.org/html/2603.13431 を確認。ページ上に CC BY 4.0 表示あり。画像は保存せず、Figure/Table 番号と要約のみ記載。
- GitHub/コード検索で使ったクエリ:
  - `CHIMERA-Bench GitHub`
  - `mansoor181 chimera-bench LICENSE`
  - `CHIMERA-Bench Hugging Face`
- 確認したGitHub URL: https://github.com/mansoor181/chimera-bench
- 確認したHugging Face URL: https://huggingface.co/datasets/mansoorbaloch/chimera-bench
- 確認したその他コード/重み/データURL: https://zenodo.org/records/20598827 / https://openreview.net/forum?id=PyZvVIJbSy
- リポジトリライセンスの確認元: GitHub `LICENSE` は MIT、`LICENSE-DATA` は CC-BY 4.0。Hugging Face dataset card も Data: CC-BY 4.0、Code: MIT と記載。
- モデル重み・チェックポイントの確認先: GitHub、Hugging Face dataset、Zenodo。CHIMERA-Bench はベンチマーク/データセットであり、専用モデル重みは該当なし。
