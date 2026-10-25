# Sequence-only antigen context improves modeling of antibody hypervariable regions

> HTML版: [図解ノートを開く](20261026_Sequence-only%20antigen%20context%20improves%20modeling%20of%20antibody%20hypervariable%20regions.html)

## まず何の論文か

抗体のCDRを設計するなら本来は「何に結合させたいか」を条件にすべきだが、既存の抗体言語モデルの多くは抗体配列だけを読み、抗原を入力しない。一方、抗原条件付き設計法の主流は抗体–抗原複合体の3D構造、ドッキング姿勢、または既知エピトープを必要とする。本研究はこの隙間を埋めるため、抗体と抗原の一次配列だけからCDR残基を予測する masked language model「LangAAI」を提案した。凍結した2つのESM-2で抗体と抗原を別々に埋め込み、軽量adapterと4層のjoint transformerで両配列を同時に自己注意させる。重要なのは、単に回復率が高いだけでなく、正しい抗原を非結合decoyへ置換したときCDR予測が悪化するかを `ΔNLL` で直接検査した点である。この抗原依存性はCDR3に最も強く、ag_novel・未見CDR3条件では ΔNLL 0.170（95% CI 0.140–0.201）だった。CHIMERA-Benchでは配列のみの入力で6 CDR中4領域のAARが最高、残る2領域も2位だったが、学習集合とのCDR-H3重複を除くとH3 AARは0.386で2位となる。AlphaSeqのPD-1、CoV2-RBD変異体ではモデル確率と実測親和性改善が弱いながら有意に相関し、AbBiBenchでも16系の平均Spearman ρ=0.188で全sequence-only法を上回った。ただし新規配列を合成・結合測定した研究ではなく、AARと尤度による候補順位付けの評価である。したがってLangAAIは「構造なしで抗原を意識した候補を絞る」実用的な前段フィルターであり、物理的結合を保証する生成器ではない。

## 書誌情報

- URL/DOI: [bioRxiv](https://www.biorxiv.org/content/10.64898/2026.10.19.760314v1) / [10.64898/2026.10.19.760314](https://doi.org/10.64898/2026.10.19.760314)
- 公開日/更新日: 2026-10-24（v1）
- 著者・所属: Faisal Bin Ashraf, Stefano Lonardi; Department of Computer Science and Engineering, University of California, Riverside
- 掲載誌/プレプリントサーバー: bioRxiv（Bioinformatics、査読前）
- リサーチ日: 2026-10-26
- 分類: 新着 / モデル起点 / 抗体–抗原条件付き配列モデル
- 論文ライセンス: CC BY-NC 4.0

## 背景と問題設定

抗体の結合特異性は主に重鎖・軽鎖の6本のCDR、とくに多様性の高いCDR-H3で決まる。抗体配列はOASなどに大量に存在する一方、PDBの抗体–抗原複合体は8,000件未満で、未知標的について正確な複合体構造を常に用意できるわけではない。AntiBERTy、AbLang2、IgLMのような抗体言語モデルは自然らしい抗体配列を学ぶが、抗原を見ないため、標的を替えても原理上同じCDR分布を返しうる。MEAN、DiffAb、dyMEAN、RAADなどは抗原条件付きだが、3D座標や接触領域に依存する。

さらに「抗原配列をモデルへ渡した」という事実だけでは、モデルがその情報を使った証拠にならない。抗体の局所配列だけで復元できる保存領域が多いため、抗原を無視しても高いrecoveryを得られるからである。本論文はこのshortcutを問題の中心に置き、抗原を正しいものからdecoyへ交換したときの損失差を条件付けの診断指標にした。

## この論文のコアアイデア

入力は、CDRの一部をmaskした抗体配列（重鎖、存在すれば軽鎖を連結）と抗原配列である。両者を別々の凍結ESM-2（12層、35M parameters、hidden width 480）で埋め込み、各adapterで共通空間へ射影する。その後 `[CLS] + antibody tokens + antigen tokens` を連結し、segment IDを付け、4層・8 headのunrestricted self-attentionへ通す。抗体残基と抗原残基が同じattention stack内で直接情報交換できることが、抗原条件付けの経路である。出力headはmask位置ごとに20アミノ酸の確率を返し、候補残基の提案、全CDR設計、変異体順位付けに使える。

学習時はCDR3の40%、CDR1/2と隣接flankの15%を選び、BERT式80/10/10 corruptionを適用する。CDR3を高率で隠すことで、残ったCDR3配列だけから復元するshortcutを弱め、抗原情報を使わせる設計である。モデル選択では通常のmasked lossだけでなく、同一抗体・同一maskに正しい抗原とshuffled抗原を与え、`ΔNLL = NLL(shuffled) − NLL(true)` を計算する。正なら正しい抗原が復元を助けたことを意味する。

## 手法の詳細

- 入力データ: 重鎖（必要に応じて軽鎖を連結）と抗原の一次配列。CDR spanはデータ側のannotationに依存する。
- 出力: mask位置の20アミノ酸確率、設計配列、抗体・抗原・pairの埋め込み、attention。
- モデル/アルゴリズム: 2系統の凍結ESM-2埋め込み、identity初期化adapter、4層joint transformer、tied LM head。
- 特徴量・表現学習: 抗体・抗原の位置依存埋め込みを共通480次元空間に置き、segment embeddingでheavy/light/antigenを区別する。joint indexの絶対位置encodingは加えない。
- 学習方法: ASD由来positive pairでmasked-residue reconstruction。trainable 15,476,193 parameters、モデルobject全体48,745,714 parameters。
- 損失関数・目的関数: mask位置のみのcross-entropy。CDR3 NLLとpaired-shuffle ΔNLLを評価・モデル選択に利用。
- 推論方法: maskした位置のtop-k確率を取得する。whole-loop designも可能だが、公開実装ではCDR spanを利用者が指定する。
- ベースライン: AbLang2、AntiBERTy、ESM-2、MINT、PLM-interact。外部比較としてCHIMERA-Benchの構造法、AlphaSeq、AbBiBenchの19法。
- 評価指標: top-1 recovery、AAR、NLL、ΔNLL、Spearman ρ、ROC-AUC、improver enrichment。
- 実装上の重要点: 抗原が長い場合は抗体を保持し、残りのlength budgetで抗原を連続windowへcropする。学習時はrandom crop、評価時はcenter cropであり、真のepitopeがwindow外なら情報を失う。

## データセットと評価設計

主学習データはASD（Antigen-Specific Antibody Database）から得たpositive-labeled抗体–抗原172,537 pairである。内訳はpatent 136,524 pair（79.13%）、COVID-19 13,074（7.58%）、HIV 11,957（6.93%）で、上位3群だけで93.63%を占める。抗体・抗原を別々にMMseqs2の90% identityでcluster化し、cluster単位でtrain/validation/testへ分けた。

| split | regime | pairs | unique antibodies | unique antigens |
|---|---|---:|---:|---:|
| train | — | 139,644 | 31,675 | 7,410 |
| validation | ab/ag/both novel合計 | 14,617 | — | — |
| test | ab_novel | 8,551 | 1,887 | 2,030 |
| test | ag_novel | 8,804 | 5,154 | 474 |
| test | both_novel | 628 | 277 | 170 |

全体配列clusterは分離しているが、held-out集合の66.6%でCDR-H3そのものが学習中に出現していたため、著者はseen/unseen CDR3を分けて報告した。これは適切な追加解析だが、「90% identity cluster splitだけで局所loopの重複を防げない」ことも示す。外部評価には、CHIMERA-Bench 292 complex、AlphaSeqのPD-1（single 432 / double 4,460）とCoV2-RBD（single 607 / double 1,870）、AbBiBench 16 system・213,685 variantを用いた。

## 主要結果

1. **未見CDR3でもrecoveryが向上**: unseen CDR3に限定したtop-1 recoveryはCDR1 0.751、CDR2 0.758、CDR3 0.526、flank 0.893で、最良baselineの0.689、0.678、0.465、0.843をすべて上回った。whole-sequenceでは6条件中5条件で最高だったが、ag_novel・unseen CDR3ではLangAAI 0.645、AntiBERTy/AbLang2 0.658で有意差なしだった。
2. **抗原を本当に使っている信号**: ag_novel・unseen CDR3で ΔNLL 0.170（95% CI 0.140–0.201、P=7.3×10^-26）。near-match（training配列から2 edit以内）を除くと0.194へ上がった。ただしperplexity ratioは1.19で、効果は統計的に明確でも絶対量は大きくない。both_novel・unseen CDR3は0.005（95% CI −0.032–0.042）でゼロと区別できず、n=628の厳しい条件では抗原依存性を確認できなかった。
3. **CHIMERA-Bench**: LangAAIはH1 0.731、H2 0.670、H3 0.422、L1 0.627、L2 0.703、L3 0.620。6領域中4領域で最高、2領域で2位だった。whole-chain exact overlap 13/292を除いてもH3 0.418だが、CDR-H3 exact overlapを除いた147 complexでは0.386となり、11法中2位。軽鎖比較は強いheavy-only法が未報告のため競合が弱く、L2/L3の大差は慎重に読むべきである。
4. **AlphaSeq affinity landscape**: double mutantのモデル確率と親和性改善はPD-1でρ=0.243、CoV2-RBDでρ=0.232。改善variant率はPD-1でbottom→top quintile 1.7%→13.5%、CoV2-RBDで6.1%→16.0%。AUCは0.665、0.616で「有用な濃縮」だが単独判定器としては中程度である。top-k提案はPD-1 k=4で1.62×、CoV2-RBD k=3で2.28×のenrichmentだった。
5. **AbBiBench**: 16 system平均ρ=0.188でsequence-only法中1位、全体4位。AntiFold（0.207）とは統計的に同等（Δρ=−0.019、P=0.56）だがProteinMPNN（0.299）には有意に劣った。系ごとのばらつきは大きく、4fqi_h1 0.598に対し1mlc −0.304、aayl49_ML −0.161だった。
6. **抗原依存性はCDR3へ局在**: antigen swapによるΔNLLはCDR3 0.044 nats、CDR2 0.017、CDR1 0.016、flank 0.005。layer ablationでも第1層のCDR3 penalty 0.565 natsが最大で、標的情報が一様な背景信号ではなくCDR3予測へ選択的に効くことを支持した。

## 重要なFigure/Table

- Figure/Table番号: Figure 1
- 何を示しているか: LangAAIの二入力architecture、抗体/抗原clusterの2次元holdout、paired-shuffle ΔNLL。
- 読み取り方: 「モデル構造」「一般化条件」「抗原を使ったかの反実仮想検査」を一枚で追う。
- 主要な数値・傾向: train 139,644 pair、test 3 regime、ΔNLL>0ならcognate antigenが復元を改善。
- なぜ重要か: 本論文の新規性はモデルだけでなく、conditional generationを検証する評価設計にある。
- Markdown内での扱い: 独自の再構成図・要約表
- HTML内での扱い: 独自の再構成フロー図
- 出典URL: [bioRxiv本文](https://www.biorxiv.org/content/10.64898/2026.10.19.760314v1.full.pdf)
- ライセンス確認: 確認済み（CC BY-NC 4.0）。原図は保存せず独自再構成。

- Figure/Table番号: Figure 2 / Figure 6
- 何を示しているか: baselineとのrecovery比較、shuffled-antigen specificity、CDR領域別の抗原依存性。
- 読み取り方: 回復率とΔNLLを分けて読む。高回復だけでは条件付けの証拠にならない。
- 主要な数値・傾向: unseen CDR3 recovery 0.526 vs 0.465、CDR3 swap ΔNLL 0.044 vs flank 0.005。
- なぜ重要か: 抗原の影響が生物学的に妥当なCDR3へ局在する。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: 独自の比較bar chart
- 出典URL: [bioRxiv本文](https://www.biorxiv.org/content/10.64898/2026.10.19.760314v1.full.pdf)
- ライセンス確認: 確認済み（CC BY-NC 4.0）。原図は保存せず独自再構成。

- Figure/Table番号: Figure 4 / Figure 5
- 何を示しているか: AlphaSeq変異体の親和性順位付けとAbBiBench 16系の外部評価。
- 読み取り方: 相関の有意性だけでなくAUC、enrichment、系ごとの負相関も確認する。
- 主要な数値・傾向: double-mutant ρ=0.243/0.232、AUC 0.665/0.616、AbBiBench平均ρ=0.188。
- なぜ重要か: 復元率が高いだけでなく、実測親和性に候補順位付けが接続する一方、効果量と系依存性の限界も見える。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: enrichment chartとbenchmark順位表
- 出典URL: [bioRxiv本文](https://www.biorxiv.org/content/10.64898/2026.10.19.760314v1.full.pdf)
- ライセンス確認: 確認済み（CC BY-NC 4.0）。原図は保存せず独自再構成。

## Code / License / Weights

- コード公開: あり
- GitHub URL: [fbabd/LangAAI](https://github.com/fbabd/LangAAI)
- GitHub以外のコードURL: 該当なし
- 実装種別: 公式 / 著者実装
- GitHub確認: 確認済み（README、tree、LICENSE、repository metadata）
- リポジトリのライセンス: MIT
- ライセンス確認元: GitHubのLICENSEファイル / repository metadata
- Hugging Face URL: [faisalashraf/langaai](https://huggingface.co/faisalashraf/langaai)
- Hugging Face種別: model
- モデル重み・チェックポイント公開: あり（`langaai.pt`、約195 MB）
- 重み公開URL: [langaai.pt](https://huggingface.co/faisalashraf/langaai/resolve/main/langaai.pt)
- データセット公開: 一部は元データ・benchmarkとして公開、論文で用いた統合済み172,537 pairとsplitは公式repo内に見つからず
- データセットURL: [ASD](https://opig.stats.ox.ac.uk/webapps/asd/)、[CHIMERA-Bench](https://github.com/mansoor181/chimera-bench)、[AlphaBind data/code](https://github.com/A-Alpha-Bio/alphabind)、[AbBiBench](https://github.com/XPRESS-BIO/AbBiBench)
- 再現性メモ: 公開packageは重みをHFから自動取得し、mask予測、whole-CDR design、5種embedding、attention取得を提供する。CDR spanは利用者がANARCI/IMGT等で与える必要がある。公式repository treeにはtraining script、統合済みtraining data、固定split、論文全benchmark再現scriptは見当たらず、end-to-end再学習の再現性は限定的。READMEは `score_sequence` を「affinity predictorではない」と明記している。

## 抗体研究・創薬への意味

最も直接的な用途は、複合体構造がない標的でのaffinity maturation候補の優先順位付けである。親抗体と抗原配列を与え、各CDR位置で候補残基を確率順に絞れば、実験libraryのサイズを縮小できる。とくに抗原構造予測やdockingが不安定な柔軟蛋白、膜蛋白、複数conformation標的で「構造パイプラインに入れる前の配列フィルター」になりうる。

方法論的にはpaired-shuffle ΔNLLが重要である。抗原条件付き、エピトープ条件付き、developability条件付きなどを名乗る生成モデルに対し、条件を入れ替えて出力分布が妥当に変わるかを検査する考え方は広く転用できる。GenCDRのような抗原を直接encodeしないalignment法と組み合わせ、LangAAIの尤度をrewardまたはproposal priorとして用いる構成も考えられる。

## 限界と注意点

**著者が述べる限界**

- 新規設計配列を合成・発現・結合測定していない。実証はrecoveryと既存affinity landscape上のretrospective rankingである。
- AARはnative sequenceの再構成であり、foldingや標的結合を保証しない。
- CHIMERA-Benchの軽鎖比較は、強いheavy-chain-only構造法の多くがlight CDRを報告しておらず、競合が弱い。
- both_novel・unseen CDR3ではΔNLLがゼロと区別できず、最難条件の抗原依存性は未確立。
- long antigenはcropされ、epitope位置を知らないままcenter cropするため、関連領域が落ちる可能性がある。

**精読して気づく限界**

- 学習pairの79.1%がpatent由来で、近縁variant群と特定scaffoldへの偏りが大きい。全体cluster split後もCDR-H3 exact overlapが66.6%残った。
- positive pairだけのmasked reconstructionなので、非結合pairとの識別やoff-target specificityを直接学習していない。shuffled antigenは診断に使うが、主損失のnegativeではない。
- ΔNLL 0.170は統計的には強いがperplexity ratio 1.19であり、実用上の条件効果は控えめ。高いrecoveryの一部は抗体priorによる。
- double mutant scoreは単一位置conditionalの和でepistasisを無視する。多変異最適化へそのまま拡張しにくい。
- AbBiBench平均ρ=0.188は候補triageには意味があるが、系ごとに負相関もあり、assay-specific calibrationなしの自動選抜は危険。
- 論文はCC BY-NC 4.0、コード・重みはMITで条件が異なる。商用利用では学習元データ、ESM-2、benchmark dataの各条件を別途確認すべきである。

## この論文を読む上での前提知識

- **CDR / CDR-H3**: 抗体可変領域にある6本のloop。H3はV(D)J junction由来で最も多様性が高く、標的特異性への寄与が大きい。
- **masked language model**: 入力配列の一部を隠し、周辺文脈から元残基を予測する。確率分布は候補変異の順位付けにも使える。
- **ESM-2**: 大規模蛋白配列で事前学習したTransformer。本研究では凍結feature extractorとして使う。
- **co-attention / joint self-attention**: 抗体tokenと抗原tokenを同じattention計算へ入れ、相互の文脈を直接参照させる仕組み。
- **AAR**: native配列と予測配列が一致する残基割合。設計benchmarkで使いやすいが、native以外の優れたbinderを低く評価しうる。
- **Spearman ρ**: 予測順位と実測順位の単調関係。絶対親和性の誤差ではなく、候補を正しい順に並べられるかを見る。
- **entity-holdout**: 配列clusterをtrain/testで分離し、近縁配列の暗記を減らす評価設計。

## 今回この1報を選んだ理由

2026-10-24公開で前回リサーチ後の明確な新着であり、「抗原配列のみを条件にCDRを設計する」というAI×抗体の中心課題を直接扱う。公式コード、MIT license、Hugging Face重みが同時公開され、すぐ試せる点も大きい。前回採用したGenCDRはframework-first生成とreward alignmentが主眼で抗原を直接encodeしなかったのに対し、LangAAIは抗原配列をjoint attentionへ入れ、paired-shuffleで条件利用を検証するため、連続して読む教育的価値が高い。HCAbLMやaffinity maturationの新着候補も有力だが、今回は公開日の新しさ、評価設計の新規性、実装可能性を優先した。

## 読む優先度

**High**。sequence-only antigen conditioningを主張するだけでなく、条件をshufflingして本当に使ったかを検証している点が重要である。数値は過度に強くなく、both_novel失敗やdata overlapも明示されているため、モデルの可能性と限界を同時に学べる。

## 自分用メモ

- 後で深掘りしたい点: ASDの90% cluster splitを再構成し、antigen family holdoutとCDR3 edit-distance holdoutをさらに厳しくした再評価。
- 関連して読むべき論文: GenCDR、CALM、PALM-H3、CHIMERA-Bench、AbBiBench、AntiFold、AlphaBind。
- 実装を触る場合の入口: GitHubの `examples/predict_cdr3.ipynb` と `candidate_cdr3_search.ipynb`。ANARCIでCDR spanを付与してから `langaai.load()` を使う。
- Obsidianでリンクしたいキーワード: [[抗原条件付き抗体設計]]、[[CDR-H3]]、[[paired-shuffle評価]]、[[affinity maturation]]、[[ESM-2]]、[[CHIMERA-Bench]]、[[AbBiBench]]。

## 関連キーワード

- antigen-conditioned antibody language model
- sequence-only antibody design
- co-attention
- masked language modeling
- CDR reconstruction
- affinity maturation
- entity holdout
- contrastive intervention / paired shuffle

## 検索ログ

- 検索したデータベース/クエリ: bioRxiv recent / `antibody machine learning October 2026`、`antibody language model antigen context`、`LangAAI`、候補名の完全一致検索。arXiv、GitHub、Hugging Faceも確認。
- 候補にした論文: 本論文、*A foundation model learns the sequence and functional grammar of fully human heavy-chain-only antibodies*（HCAbLM）、*Affinity maturation of a single-domain antibody via rational mutagenesis informed by artificial intelligence*、*Revisiting Structure-Based Filtering for AI-Designed Antibodies*。
- 最終的にこの1報を採用した理由: 前回後の10月24日新着、抗原配列のみでの条件付け、paired-shuffle評価、公式実装と重みの即時利用性。
- 新着論文と基盤論文のバランス: 今回は新着枠。基盤的なESM-2、AntiBERTy、DiffAb、MEANは背景・baselineとして接続した。
- PDF/HTML/Supplementaryを確認できたか: bioRxiv 22ページPDF、abstract HTML、5ページSupplementary PDFを確認済み。
- Figure/Tableを確認したURLやライセンス確認状況: Figure 1–7、Table 1、Supplementary Figure 1–2、Table 1–5。bioRxiv HTML metadataと各PDF上部でCC BY-NC 4.0を確認。
- GitHub/コード検索で使ったクエリ: `LangAAI GitHub`、論文記載のSource Code URL、GitHub API tree。
- 確認したGitHub URL: https://github.com/fbabd/LangAAI
- 確認したHugging Face URL: https://huggingface.co/faisalashraf/langaai
- 確認したその他コード/重み/データURL: bioRxiv Supplementary、ASD、CHIMERA-Bench、AlphaBind/AlphaSeq、AbBiBench。
- リポジトリライセンスの確認元: GitHub `LICENSE` とrepository metadata（MIT）。HF model metadataもMIT。
- モデル重み・チェックポイントの確認先: HF APIのsiblingsで `langaai.pt`、usedStorage 195,193,841 bytesを確認。GitHub READMEの自動download URLとSHA-256検証記載も確認。
