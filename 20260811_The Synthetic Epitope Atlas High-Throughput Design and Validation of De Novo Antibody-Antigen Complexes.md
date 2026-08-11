# The Synthetic Epitope Atlas: High-Throughput Design and Validation of De Novo Antibody-Antigen Complexes

## まず何の論文か

この論文は、de novo抗体設計モデルの訓練データ不足を、実験的に検証された「疑似構造」データを大規模に作ることで解決しようとする研究である。著者らは、抗体側、特にVHHのパラトープを固定し、そのパラトープに結合する小型のde novoタンパク質を設計するという逆向きの問題設定を採る。これらの小型タンパク質をsynthetic epitope proteins（SEPs）と呼び、VHHとSEPの予測複合体構造を、AlphaSeqという酵母ベースの高スループット結合測定で検証する。結合が強く、かつ標的VHHに特異的であれば、その予測複合体を抗体-抗原相互作用を学習するためのpositive pseudo-structureとして扱う。一方、構造予測や設計スコア上は有望に見えるが実験では結合しない候補は、hard negative pseudo-structureとして使える。

結果として、Synthetic Epitope Atlas（SEPIA）は45,430個の設計SEP、190個の親VHH、2,600万件を超えるオンターゲット/オフターゲット結合測定を含むデータ生成フレームワークになっている。著者らは1,161個の強く特異的なVHH-SEP positive pseudo-structureを検証し、さらにVHHおよびSEPの変異体で75,000件超の強い結合相互作用を得ている。重要なのは、これは単に「データをたくさん測った」論文ではなく、PDBに存在しない非結合例と、自然抗原とは異なるエピトープ幾何を意図的に作れるという点である。SEPIAを用いて訓練したABACUSというBoltz-2ベースの結合分類器は、de novo VHH設計候補のランキングでipSAEのような構造予測confidence指標を上回った。したがって本論文は、抗体生成モデルそのものというより、次世代の抗体設計モデルを訓練・評価するためのデータ生産パラダイムを提案した論文として読むべきである。

## 書誌情報

- URL/DOI: https://www.biorxiv.org/content/10.64898/2026.04.17.719295v2 / https://doi.org/10.64898/2026.04.17.719295
- 公開日/更新日: v1 posted 2026-04-18、v2 posted 2026-04-19。v2ではSupplementary Figure S24/S25の表示修正などが記載されている。
- 著者・所属: Nicholas Altieri, Joseph L. Harman, David Noble, Natasha Murakowska, Alexander Eng, Kerry L. McGowan, Davis Goodnight, Lucian DiPeso, Colleen Shikany, Emily Engelhart, Leah J. Homad, Miranda C. Lahman, Shyam Gandhi, Mackenzie Goodwin, Kendrick Herbst, Charles Lin, Margot McMurray, Juliana Barrett, Aditya A. Agarwal, James Harrang, Ryan O. Emerson, Randolph M. Lopez, David A. Younger, Adrian W. Lange; A-Alpha Bio, Seattle, WA, USA
- 掲載誌/プレプリントサーバー: bioRxiv preprint
- リサーチ日: 2026-08-11
- 分類: 新着 / ベンチマーク・データセット / モデル起点

## 背景と問題設定

抗体のde novo設計はRFdiffusion、Germinal、BoltzGen、Chai系モデル、Nabla BioやA-Alpha Bioの社内パイプラインなどで急速に進んでいるが、成功率は標的やエピトープに強く依存し、実験に出した候補の大半はまだ失敗する。著者らは、その根本要因の一つを抗体-抗原複合体の構造データ不足と見る。PDB全体の構造数は多いが、抗体を含む構造は2026年初頭で約10,763件、VHHを含むものは2,211件、抗原と複合体になったunique VHHは811件にすぎないとされる。さらにPDBは「結合したから構造が解けた」陽性例に偏っており、設計モデルが最も必要とする「もっともらしいが結合しない」hard negativeを含まない。

この欠落は抗体設計では特に痛い。構造予測モデルは、候補複合体が幾何的にもっともらしいかを高いconfidenceで出せても、その界面が実際に結合エネルギーや特異性を持つかまでは十分に保証しない。de novo binder designの実験では、少数の候補だけを合成・測定できるため、ランキングの品質がヒット率に直結する。陽性構造だけで学習したモデルは、実験的な失敗例から何を避けるべきかを学べず、過信した候補を上位に出しやすい。

SEPIAの問題設定は、この構造データ不足を従来のX線結晶構造解析やcryo-EMの蓄積だけで待つのではなく、設計と高スループット実験で能動的に作ることである。自然抗原を標的に抗体を設計する通常の方向ではなく、既知VHHに対して小型のsynthetic epitope proteinを設計すれば、minibinder設計の比較的高い成功率と小型ライブラリの実験容易性を使って、抗体パラトープが認識しうる多様なエピトープ場を探索できる。この発想が本論文の中心である。

## この論文のコアアイデア

コアアイデアは、「抗体-抗原複合体の実験構造が少ないなら、抗体に結合する人工エピトープを大量に設計し、結合実験で検証した予測複合体を構造付き教師データとして使う」というものだ。著者らは、実験的に構造決定されたVHHから本来の抗原を取り除き、VHHのパラトープに結合するde novo小型タンパク質SEPを設計する。設計候補は構造誘導パイプラインでスコアリング・フィルタリングされ、AlphaSeqでVHHライブラリとSEPライブラリの組合せ結合を測定される。

このとき、強く特異的に結合したVHH-SEP予測複合体をpositive pseudo-structure、設計上は有望だったが結合しなかったものをnegative pseudo-structureとみなす。pseudo-structureという言葉が重要で、これは原子分解能の実験構造ではない。しかし、de novo binderの先行研究では、強い特異的結合が後続の構造決定で予測界面とよく対応する例が多い。SEPIAはこの経験則を、抗体設計モデル用のスケーラブルなデータ生成戦略に拡張している。

さらに著者らは、SEPIAを訓練データとして使うABACUS（AntiBody Affinity Classifier Using pSeudo-structures）を構築する。ABACUSはBoltz-2の構造予測表現、PAE、global confidence、予測界面上のparatope/epitope残基表現を入力し、候補複合体が実験的に結合するかを分類する。つまり、Boltz-2のconfidenceをそのままランキングに使うのではなく、AlphaSeqで得た結合/非結合ラベルを用いて、抗体設計候補の実験ヒットを濃縮する分類器を学習する。

## 手法の詳細

- 入力データ: 実験構造があるVHH、設計されたSEP配列・予測構造、VHH-SEP予測複合体、AlphaSeqで測定されたVHH/SEP間の結合affinity、SAbDab-nano由来のVHH-抗原複合体、de novo VHH設計候補とAlphaSeq検証ラベル。
- 出力: VHH-SEP positive/negative pseudo-structure、変異体の結合ラベル、SAbDab-nano decoy分類タスク、de novo VHH候補の結合ランキング、ABACUSの結合/非結合予測。
- モデル/アルゴリズム: SEP設計には構造誘導de novo design pipelineを用いる。本文と謝辞からRFDiffusion、ProteinMPNN、Boltz-2、Foldseek等の利用が示唆される。Round 2ではBoltz-2のhard templatingと、3Di構造エンコーディングに基づくcustom RFDiffusion potentialが使われたと記載されている。詳細な設定値はSupplementary Materials B/Eにあるが、今回bioRxivのSupplementary本体は直接取得できず、完全な設定は未確認。
- 特徴量・表現学習: ABACUSはBoltz-2のpairformer表現であるsingle表現`s`とpair表現`z`、PAE行列、global confidence scoreを使う。Cα-Cα距離10 A以内の残基ペアをAb-Ag界面として定義し、paratope/epitope残基の単一埋め込みと、双方向のinterface pair embeddingを抽出する。
- 学習方法: SEPIA pseudo-structureとSAbDab-nano decoy datasetを用いて結合/非結合分類器を学習する。Decoy datasetの評価では5-fold cross-validationと100回のresamplingで学習曲線を比較している。
- 損失関数・目的関数: ABACUSの具体的な損失関数名は本文では明示されていない。二値分類headを持つため、標準的にはbinary cross entropy系と推測されるが、未確認として扱う。
- 推論方法: 候補VHH-抗原複合体をBoltz-2で予測し、予測構造と表現をABACUSに入力して結合確率を出す。候補をこのスコアで順位付けし、上位何%に実験ヒットが濃縮されるかを評価する。
- ベースライン: ipSAEなどの構造予測confidence metric、SAbDab-nano decoyのみで訓練したモデル、SEPIAのみまたはSAbDab-nanoのみのMLP分類器。
- 評価指標: AUROC、AUPRC、Enrichment@k、top rank accuracy。Enrichment@kは上位k%におけるprecisionを全体のヒット率で割った値。
- 実装上の重要点: ABACUSは全複合体をそのまま処理するのではなく、予測界面近傍に集約してからaxial transformerと軽量transformer encoderで処理する。これは界面情報に計算を集中し、Boltz-2の埋め込みを結合分類へ転用する設計である。

## データセットと評価設計

- 使用データセット名: SEPIA、SAbDab-nano decoy dataset、de novo VHH holdout dataset。
- データの規模: SEPIAではRound 1で25,448 SEP、Round 2で19,982 SEPを実験測定し、合計45,430設計SEPを扱う。Round 2では19,982 SEPを11,425 total VHH（180 parent VHHと各50-100程度の点変異体）に対して測定し、2,600万件超のPPI測定を得ている。
- 抗体、抗原、タンパク質、複合体など対象の内訳: 主対象はVHHとde novo設計SEPの複合体である。親VHHは合計190 unique VHHと説明され、Round 1は48 VHH、Round 2は180 VHH（うち38はRound 1から再利用、142は新規）を対象にする。評価には自然抗原を持つSAbDab-nano VHH-抗原複合体と、22抗原に対するde novo VHH設計候補4,930件も使われる。
- train/validation/test の分け方: Decoy datasetでは5-fold cross-validationを用いる。de novo VHH datasetは、SEPIAやSAbDab-nanoとは別のhold-outとして扱われ、22抗原に対する4,930候補のうち37 hitを含む。
- リーク対策やクラスタ分割の有無: NTXモデルのpre-trainingでは、PDB抗体を除いたAFDB由来のmonomer protein structureのみを使い、Ab-Ag binding情報へのリークを避けたと説明される。de novo VHH holdoutでは、抗体bound structureがPDBにない22抗原を用い、Foldseek解析でSEPIA複合体やPDBとの構造的重複が小さいことを確認している。クラスタ分割の詳細はSupplementary依存で未確認。
- 評価指標: Decoy taskではtop rank accuracy、de novo design rankingではAUROC、AUPRC、Enrichment@k。
- ベースラインや比較対象: ipSAE、SAbDab-nanoのみ、SEPIAのみ、SAbDab-nano + SEPIA。
- この評価設計が妥当かどうか: 妥当性は高い。特に、自然VHH-抗原複合体のdecoy detectionと、実際のde novo VHH設計キャンペーンのholdout rankingを分けている点がよい。一方で、著者企業内のデータ生成・モデル訓練・評価が一体で、SEPIA本体やABACUS重みが公開されていないため、外部から完全な再現性を確認できない。

## 主要結果

Round 1では、48親VHHに対して約600,000個のSEP候補をin silicoで作り、そのうち25,448個をAlphaSeqで測定した。組合せ測定は120万件超で、928個のVHH-SEP相互作用が強く特異的なオンターゲット結合のhit基準を満たした。これは48 VHH中34 VHH、つまり70.8%のVHHで少なくとも1つのSEP hitを得たことに相当し、SEP単位では3.6%のhit率である。VHHごとの成功率には大きなばらつきがあり、抗体設計の成功率が標的パラトープに強く依存することも示している。

Round 2では、38個の再利用VHHに142個の新規VHHを加え、180親VHHに対象を広げた。19,982 SEPを11,425 total VHHに対して測り、2,600万件超のPPI測定を得た。親VHH-SEPのpositive pseudo-structureは233個で、180 VHH中75 VHH、つまり41.7%でhitを得た。SEP単位のhit率は1.2%に下がったが、これはより広いVHH空間を対象にしたこと、各VHHに等量の上位SEPを配分したこと、Round 2の設計パイプライン変更が影響した可能性があると説明されている。重要なのは、hit率が下がっても、カバーされるVHHの絶対数と多様性は拡大した点である。

変異体データも大きい。Round 2では11,425個の単一点変異VHHを含め、36,803件のhit interactionが得られた。これは3,855個の変異VHH、1,435個のSEPに関わる。Round 3では、VHH 7XLDに結合するSEPに対して4,779個のsingle-site mutagenesis variantを作り、そのうち3,496個、73.2%が強い結合を保持した。また96個のVHH-SEP pseudo-structureに対する両側alanine scanでは、界面残基のalanine置換が非界面残基より有意に結合を壊しやすいことが示された（Mann-Whitney U test, p < 10^-30）。これは、予測pseudo-structureの界面が単なる絵ではなく、変異効果と対応する機能的構造仮説であることを支持する。

SAbDab-nano decoy datasetは、646個の既知on-target VHH-抗原複合体と14,521個のoff-target decoy、合計15,167 PPIからなる。960 unique VHH、438 unique antigenを含み、decoy比は22.5:1である。SEPIAだけでも学習信号を持つが、SAbDab-nanoとSEPIAを併用すると、既存PDB由来データだけで訓練した場合よりtop rank accuracyが改善し、PDB由来データの上限をSEPIAが補完することが示された。

de novo VHH holdoutでは、複数のopen-source de novo VHH設計モデルで22抗原に対する4,930候補を生成し、AlphaSeqで37 high-quality hitsを確認した。hitは8抗原にまたがり、抗原単位では36.4%、設計候補単位では0.8%のhit率である。Foldseek解析では、validated de novo VHH-抗原複合体のPDBとのmedian complex TM-scoreは0.455であり、SEPIAやPDBと混ざった構造クラスタは観察されなかった。これはholdoutが既存構造やSEPIAの単純な近傍検索では解けないことを示す。

ABACUSはこのholdout rankingでipSAEを上回った。本文Figure 5では、上位10%の候補選択でABACUSがEnrichment >= 4を示し、ipSAEは最上位percentileでランダム以下になる傾向が示される。これは構造予測confidenceが「明らかに悪いものを落とす」negative filterとしては使えても、実験ヒットを上位に並べるranking metricとしては過信を起こしうることを示す。SEPIAで得た結合/非結合ラベルを使うことで、ランキングは単なる幾何confidenceから、実験結果に校正された結合分類に近づく。

## 重要なFigure/Table

- Figure/Table番号: Figure 1
- 何を示しているか: SEPIA全体のworkflow。VHH構造からSEPを設計し、AlphaSeqで測定し、positive/negative pseudo-structureを作り、ABACUS訓練に使う流れを示す。
- 読み取り方: 上段はVHHに対してSEPを設計する「逆向き」データ生成、下段は通常の抗原に対するde novo VHH設計を示す。結合した候補だけでなく、結合しなかった高confidence候補もnegativeとして使う点を見る。
- 主要な数値・傾向: Figure自体は概念図だが、本文では45,430 SEP、190 VHH、1,161 positive pseudo-structure、2,600万件超測定につながる。
- なぜ重要か: 本論文の新規性はモデル構造よりも、このデータ生成サイクルにある。Figure 1を理解すると、SEPIAがなぜPDBの補完になるのかが分かる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.64898/2026.04.17.719295v2.full
- ライセンス確認: 確認済み。bioRxivページおよびPDFでCC-BY-NC-ND 4.0。ND条件を含むため画像保存・埋め込みはしない。

- Figure/Table番号: Table 1 / Table 2
- 何を示しているか: Wild-type VHHに対するSEP設計検証の規模とhit率、変異体データセットの規模を示す。
- 読み取り方: Round 1は狭いVHH集合で高hit率、Round 2は広いVHH集合でhit率は下がるがカバー範囲が広がる。Table 2は変異体データが単なる補助ではなく、界面の局所変異効果を学習できる規模であることを示す。
- 主要な数値・傾向:

| データ | 規模 | 主な結果 |
| --- | ---: | --- |
| Round 1 | 48 VHH、25,448 SEP | 34 VHHでhit、928 positive pseudo-structure、SEP hit率3.6% |
| Round 2 parent | 180 VHH、19,982 SEP | 75 VHHでhit、233 positive pseudo-structure、SEP hit率1.2% |
| Round 2 VHH point mutant | 11,425 mutant VHH | 36,803 mutant hit interactions |
| Round 3 SEP SSM | 4,779 SEP variants | 3,496 variantsが強い結合を保持 |
| Round 3 VHH+SEP alanine scan | 57 VHH variants、7,040 SEP variants | 35,010 mutant hit interactions |

- なぜ重要か: SEPIAが「1回のデモ」ではなく、複数roundで拡張できるデータ生成基盤であることを定量的に示す。
- Markdown内での扱い: 要約表
- 出典URL: https://www.biorxiv.org/content/10.64898/2026.04.17.719295v2.full
- ライセンス確認: 確認済み。CC-BY-NC-ND 4.0。表は必要な列だけを再構成した。

- Figure/Table番号: Figure 5
- 何を示しているか: ABACUSのモデル構成と、de novo VHH holdout datasetでのランキング性能。
- 読み取り方: Boltz-2の予測構造と埋め込みから界面特徴を抽出し、ABACUSがbind/non-bindを分類する。Enrichment@kでは、上位候補を何%選ぶとヒット率がランダムより何倍濃縮されるかを見る。
- 主要な数値・傾向: 本文では、ABACUSが上位10%でEnrichment >= 4を示し、AUROCでもipSAEを大きく上回るとされる。ipSAEは最上位quantileではbindingと負に相関するような挙動、つまり過信を示す。
- なぜ重要か: SEPIAの価値が「データセットを作った」だけでなく、実際のde novo VHH候補の実験ヒット濃縮に効くことを示す中心図である。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.64898/2026.04.17.719295v2.full
- ライセンス確認: 確認済み。CC-BY-NC-ND 4.0。画像保存・埋め込みはしない。

## Code / License / Weights

- コード公開: 見つからず
- GitHub URL:
- GitHub以外のコードURL:
- 実装種別: 該当なし
- GitHub確認: 確認済み
- リポジトリのライセンス: 該当なし
- ライセンス確認元: 該当なし
- Hugging Face URL: https://huggingface.co/aalphabio
- Hugging Face種別: organization。関連するA-Alpha Bioのopen AlphaSeq dataset organizationは確認したが、SEPIA/ABACUSのmodelまたはdataset公開は見つからず。
- モデル重み・チェックポイント公開: 見つからず
- 重み公開URL:
- データセット公開: 見つからず
- データセットURL:
- 再現性メモ: bioRxiv本文とA-Alpha Bioの解説記事は確認できたが、SEPIA本体、ABACUSコード、ABACUS重み、設計候補構造、AlphaSeq全測定テーブルの公開リンクは見つからなかった。Hugging FaceのA-Alpha Bio organizationには`aalphabio/open-alphaseq`があり、AlphaSeq由来の既存公開データセットは確認できるが、README上のdataset sourcesはAlphaSeq technology、Engelhart et al. 2022、AlphaBind 2025であり、本SEPIA preprintは含まれていなかった。A-Alpha Bioの解説記事末尾にはデータ生成について問い合わせ先が示されており、現時点ではSEPIAを完全再現する公開パッケージではなく、研究発表・社内基盤に近いと解釈するのが妥当である。

## 抗体研究・創薬への意味

抗体設計で最も重要な意味は、VHHのパラトープが許容する結合界面を、大量の人工エピトープで測れる点である。通常の抗体-抗原構造データは、自然抗原と成功例に偏っているため、モデルは「どのような界面が結合しないか」「どの変異が局所的に結合を壊すか」を学びにくい。SEPIAは、positiveだけでなくhard negativeと変異効果を含むため、抗体候補のランキング、false positive低減、zero-shot affinity optimizationの基盤データになりうる。

創薬パイプラインでは、de novo設計候補を何千から何万も生成したあと、実験に出せる数に絞る必要がある。ここでipSAEやipTMのような構造confidenceに頼ると、幾何的には自信があるが実際には結合しない候補を上位に置く危険がある。ABACUSのように、構造予測表現を実験ラベルで校正した分類器は、候補選抜のヒット率を改善する実用的な層になる。

またSEPIAは、抗原に対する抗体設計だけでなく、抗体パラトープの「エピトープ場」を学ぶという見方を導入している。これは、あるVHHがどのような化学・形状・電荷分布を認識できるかを、多様な人工エピトープを通じて探索する発想である。将来的には、既存抗体の交差反応性予測、エピトープ模倣、paratope annotation、抗体humanization後の結合保持評価、抗体developabilityとの同時最適化にも接続しうる。

## 限界と注意点

著者が述べている限界:

- pseudo-structureは実験的な共結晶構造ではなく、機能的結合実験で支持された予測構造である。強く特異的な結合は構造仮説を支持するが、原子レベルの正確性を保証しない。
- AlphaSeqは酵母display系であり、治療用抗体の全ての生物物理的文脈、発現、folding、哺乳類細胞でのpost-translational modificationを完全には反映しない。
- 酵母displayや発現不良により、実際には結合可能な相互作用がfalse negativeになる可能性がある。
- SEPIAは高解像度構造生物学を置き換えるものではなく、構造情報を持つ訓練データを補完するものと位置付けられる。

読んで気づいた限界:

- SEPIA本体、ABACUSコード、モデル重みが公開されていないため、外部研究者が同じsplit、同じ特徴量、同じ評価を再現することは難しい。
- 親VHHは既知PDB構造から選ばれており、完全に未知の抗体formatや低品質予測構造に対する拡張性は今後の検証が必要である。
- SEPは自然抗原ではないため、SEPIAで学んだ界面特徴が全ての天然抗原、膜タンパク質、糖鎖抗原、構造可塑性の高い抗原に転移するとは限らない。
- ABACUS評価のholdoutは社内または著者側で生成・測定されたデータであり、独立第三者ベンチマークでの検証が必要である。
- FigureやデータはCC-BY-NC-ND 4.0のpreprintで説明されているが、SEPIAデータ自体の利用ライセンスは公開物が見つからないため未確認である。

## この論文を読む上での前提知識

- VHH / nanobody: 重鎖抗体に由来する単一ドメイン抗体。通常のIgGより小さく、単一chainで抗原認識できるため、構造設計・酵母display・de novo設計の実験に使いやすい。
- Paratopeとepitope: paratopeは抗体側の抗原認識面、epitopeは抗原側の認識される部位である。本論文では、VHHのparatopeに人工epitopeであるSEPを結合させる。
- de novo binder design: 既存の自然配列を少し変えるのではなく、標的に結合する新規タンパク質構造・配列を計算的に設計する手法。RFdiffusion、ProteinMPNN、Boltz系モデルなどが関連する。
- AlphaSeq: 酵母matingを利用して、多数のタンパク質ペアの結合を同時に測るA-Alpha Bio系の高スループットPPI測定技術。配列ペアと定量的affinityラベルを大量に得られる。
- Pseudo-structure: 実験的に構造決定されたものではなく、計算設計・構造予測により提案された複合体構造を、結合実験で機能的に支持したもの。構造学的真値ではないが、ML訓練データとしては有用な中間表現になる。
- Hard negative: モデルや人間の目にはもっともらしいが、実験では結合しない例。分類器がfalse positiveを減らすために重要で、PDBのような成功例中心のデータベースにはほぼ存在しない。
- ipSAE: AlphaFold/Boltz系の複合体予測confidenceから派生した界面品質指標。候補フィルタとして使われるが、結合affinityそのものを測る指標ではない。
- Enrichment@k: 候補上位k%を選んだときのヒット率が、全候補をランダムに選ぶ場合に比べて何倍良いかを表す。低ヒット率のde novo設計では実用上重要な指標である。

## 今日この1報を選んだ理由

前回までにBoltzGenやSaProtを読んでおり、生成モデル・構造表現モデル側の理解は進んでいる。一方で、抗体設計で実際にボトルネックになるのは、モデルだけでなく、結合/非結合を含む構造付き訓練データの不足である。SEPIAはこの問題を正面から扱い、PDBに依存しないデータ生成、hard negative、変異効果、実験ヒット濃縮まで一つの物語でつないでいる。

新着性も高く、2026年4月のbioRxiv preprintで、A-Alpha BioのAlphaSeq基盤を背景にした実験規模が大きい。コードやデータが完全公開されていない点は再現性面の弱点だが、むしろ「現在の産業寄り抗体AIでどこが公開され、どこが非公開になりがちか」を理解する上でも重要である。抗体AIの知識ベースとしては、個別の生成モデル論文の間に、このようなデータ生成・評価設計の論文を入れておく価値が高い。

## 読む優先度

High。抗体de novo設計の性能向上を、モデルアーキテクチャではなくデータ生成の観点から説明しており、今後の抗体AI論文を評価する基準になる。特に、hard negative、pseudo-structure、実験ラベルで校正したランキングという3点は、生成モデル論文を読む際のチェックリストとして使える。

## 自分用メモ

- 後で深掘りしたい点: Supplementary Materials B/E/Fを取得し、SEP設計パイプライン、ABACUS architecture、training split、label定義、hit criteriaを詳細確認する。
- 関連して読むべき論文: AlphaSeq 2017 PNAS、AlphaBind 2025 mAbs、Smorodina et al. 2026 confidence score limits、Overath et al. 2025 de novo binder design meta-analysis、RFdiffusion antibody design、Germinal、Boltz-2。
- 実装を触る場合の入口: SEPIA/ABACUS実装は未公開。公開実装としてはBoltz-2、RFDiffusion、ProteinMPNN、Foldseekを個別に触り、A-Alpha Bioの`aalphabio/open-alphaseq`でAlphaSeq形式のデータスキーマに慣れる。
- Obsidianでリンクしたいキーワード: [[VHH]], [[nanobody]], [[AlphaSeq]], [[hard negative]], [[pseudo-structure]], [[de novo antibody design]], [[Boltz-2]], [[ipSAE]], [[SAbDab-nano]], [[ABACUS]]

## 関連キーワード

- Synthetic Epitope Atlas
- SEPIA
- ABACUS
- synthetic epitope protein
- VHH
- nanobody
- pseudo-structure
- AlphaSeq
- hard negative
- de novo antibody design
- antibody-antigen complex
- Boltz-2
- ipSAE
- SAbDab-nano

## 検索ログ

- 検索したデータベース/クエリ:
  - Web検索: `"Frozen Protein Foundation-Model Embeddings Improve Antibody-Antigen"`
  - Web検索: `"AI antibody" "bioRxiv" "2026" "machine learning"`
  - Web検索: `site:biorxiv.org antibody antigen machine learning antibody 2026 deep learning affinity`
  - Web検索: `"The Synthetic Epitope Atlas"`
  - Web検索: `"Synthetic Epitope Atlas" SEPIA ABACUS GitHub`
  - Web検索: `"10.64898/2026.04.17.719295"`
  - Web検索: `site:github.com/A-Alpha-Bio SEPIA OR abacus antibody`
  - Web検索: `site:huggingface.co A-Alpha-Bio ABACUS antibody`
  - Web検索: `site:zenodo.org "Synthetic Epitope Atlas" "VHH"`
- 候補にした論文:
  - The Synthetic Epitope Atlas: High-Throughput Design and Validation of De Novo Antibody-Antigen Complexes
  - Frozen Protein Foundation-Model Embeddings Improve Antibody-Antigen ΔΔG Ranking
  - DiffAb / MEAN / dyMEAN / RefineGNN / AbLangなどの基盤寄り候補
- 最終的にこの1報を採用した理由: 新着preprintであり、抗体de novo設計の中心課題である構造付き訓練データ不足、hard negative不足、ランキング指標の過信を一度に扱っているため。前回のBoltzGen/SaProtと接続しやすく、今後のモデル論文を読む基準にもなる。
- 新着論文と基盤論文のバランスをどう考えたか: 前回はBoltzGen、SaProtとモデル/基盤表現を読んだため、今回は新しめの抗体データ生成・評価設計論文を優先した。
- PDF/HTML/Supplementaryを確認できたか: bioRxiv v2 PDFを取得し本文を確認した。bioRxiv検索結果でv2ページ、公開日、ライセンス、Supplementary materialリンクの存在は確認した。Supplementary本体はbioRxiv側の429制限で直接取得できず、詳細設定は未確認。
- Figure/Tableを確認したURLやライセンス確認状況: bioRxiv v2 PDFおよびv2ページ。ライセンスはCC-BY-NC-ND 4.0と確認。ND条件を含むためFigure画像は保存・埋め込みしない。
- GitHub/コード検索で使ったクエリ: `"Synthetic Epitope Atlas" "github.com"`、`"ABACUS" "github.com" "aalphabio" antibody`、`site:github.com "AntiBody Affinity Classifier Using pSeudo-structures"`、`site:github.com/A-Alpha-Bio SEPIA OR abacus antibody`
- 確認したGitHub URL: 公式SEPIA/ABACUS repositoryは見つからず。関連既存データとしてAlphaSeq Antibody Dataset `https://github.com/mit-ll/AlphaSeq_Antibody_Dataset` は確認したが本論文の実装ではない。
- 確認したHugging Face URL: https://huggingface.co/aalphabio 、https://huggingface.co/datasets/aalphabio/open-alphaseq
- 確認したその他コード/重み/データURL: A-Alpha Bio解説記事 https://aalphabio.substack.com/p/the-synthetic-epitope-atlas-scaling
- リポジトリライセンスの確認元: SEPIA/ABACUS公式repoが見つからないため該当なし。Hugging Face `aalphabio/open-alphaseq`は本論文データではないため、SEPIAのライセンス確認元にはしない。
- モデル重み・チェックポイントの確認先: GitHub検索、Hugging Face organization/model検索、Zenodo検索。SEPIA/ABACUSの重みは見つからず。
