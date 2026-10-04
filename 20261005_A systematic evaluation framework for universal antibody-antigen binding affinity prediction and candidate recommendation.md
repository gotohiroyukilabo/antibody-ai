# A systematic evaluation framework for universal antibody-antigen binding affinity prediction and candidate recommendation

> HTML版: [ビジュアル精読ノート](20261005_A%20systematic%20evaluation%20framework%20for%20universal%20antibody-antigen%20binding%20affinity%20prediction%20and%20candidate%20recommendation.html)

## まず何の論文か

この論文は、未知抗原に対する抗体結合親和性予測を、絶対的なKdの回帰ではなく「同じ抗原に対する2抗体のどちらが強く結合するか」という順位学習として捉え直した研究である。著者らは、候補群から実験すべき上位抗体を選ぶという創薬現場の用途に合わせ、pairwise accuracy、Top-K retrieval accuracy、Top-K retrieval precisionを組み合わせた評価枠組みを提案した。同時に、抗体配列と抗原配列だけを入力するMochiBindを開発した。MochiBindはESM-2 3Bの残基埋め込みを平均プーリングし、抗体–抗原表現を軽量な射影層とMLPで比較する。評価にはAlphaBindの4抗原系（TIGIT、PD-1、SARS-CoV-1 RBD、HER2）を用い、2抗原で学習、1抗原で検証、1抗原を完全に未見のテストとするローテーションを採用した。各抗原について20万ペアを評価し、MochiBindは4系すべてでGraphinity、Boltz-2のipLDDT、GeoDockを上回った。pairwise accuracyは59.89〜74.27%で、4抗原平均は69.20%だった。特にPEMではGraphinityの63.20%に対して72.35%、TRAでは68.55%に対して74.27%である。一方、これは4抗原・単一データ生成基盤・各抗原1つの親抗体周辺という限定された検証であり、「任意の新規抗原に普遍的に使える」ことを証明したわけではない。重要なのは最高精度の主張だけでなく、非結合体を含む大規模候補プール、抗原単位の分割、上位回収性能という評価設計を標準化した点にある。コード、サンプル済みデータ、4抗原用チェックポイントが公開され、再現性を確認しやすいことも価値が高い。

## 書誌情報

- URL/DOI: [iScience / DOI 10.1016/j.isci.2026.116522](https://doi.org/10.1016/j.isci.2026.116522)
- 論文ページ: [Amazon Science](https://www.amazon.science/publications/a-systematic-evaluation-framework-for-universal-antibody-antigen-binding-affinity-prediction-and-candidate-recommendation)
- PubMed/PMC: [PubMed PMID 42564542](https://pubmed.ncbi.nlm.nih.gov/42564542/) / [PMC13444443](https://pmc.ncbi.nlm.nih.gov/articles/PMC13444443/)
- 公開日/更新日: オンライン公開 2026-07-24、iScience 29(8)号日付 2026-08-21
- 著者・所属: Yunrui Li, Yue Zhao, Kemal Sonmez, Luca Giancardo, Lan Guo, Pengyu Hong, Nina Cheng, Melih Yilmaz。Brandeis UniversityおよびAmazon Web Services
- 掲載誌/プレプリントサーバー: iScience, Volume 29, Issue 8, Article 116522
- リサーチ日: 2026-10-05（JST）
- 分類: 新着 / ベンチマーク・データセット / 親和性予測

## 背景と問題設定

抗体探索で本当に必要なのは、数千〜数十万の候補から少数の実験候補を選ぶことである。しかし既存研究は、既知抗原に似たテスト集合、強結合体中心で非結合体が少ないデータ、絶対Kdの相関や誤差を使うことが多い。これでは、学習時に見ていない標的へ適用し、多数の非結合体の中から当たりを拾う実務能力を過大評価しうる。また、測定法や実験バッチが違えば絶対値は揺れやすく、論文間でMAEや相関を単純比較しにくい。

著者らは問題を「同一抗原に結合する候補AとBのどちらが強いか」に変換し、局所的な二者比較をTrueSkillで全体順位へ統合した。さらに、抗原を丸ごと分離したsplitと、Top-K回収指標を導入した。Table 1では先行7研究を、非結合体、大規模候補群、out-of-sample test、十分なtest規模、直接的な親和性評価の5条件で整理し、全条件を満たす評価が不足していると論じている。

## この論文のコアアイデア

MochiBindの核は「絶対Kdを当てるより、同じ抗原上で候補の順序を当てる」という設計である。入力は抗原配列と2つの抗体候補配列。各抗体–抗原組をESM-2（esm2_t36_3B_UR50D、残基埋め込み2560次元）で表現し、残基方向に平均プーリングする。抗体と抗原の表現を連結して5120次元にし、1024次元へ射影する。候補AとBの射影差をMLP（256→128→64）へ入れ、相対的な結合強度差を予測する。20万件の二者比較結果はTrueSkillへ渡し、各候補の潜在スキル平均を全体ランキングとして用いる。

この設計により、構造予測を全候補に実行せずに順位付けできる。論文報告では5,000抗体由来20万ペアのCPU推論が約13秒、TrueSkillの順位統合が約76秒である。ただしESM-2埋め込みの事前計算時間・メモリはこの13秒に含まれないと読むべきである。

## 手法の詳細

- 入力データ: 同一抗原に対する抗体候補2件のアミノ酸配列と抗原配列。AlphaBindで測定されたlog10(Kd[nM])を教師信号に使用。
- 出力: 2候補間の相対親和性差。符号で勝者を決め、TrueSkillで候補プール全体の順位へ変換。
- モデル/アルゴリズム: ESM-2 3B埋め込み、平均プーリング、5120→1024の射影、MLP [256, 128, 64]、TrueSkill。
- 特徴量・表現学習: ESM-2残基埋め込みを固定長へ平均化。ESM-2本体をfine-tuneしたかは本文記述から明確でなく、公開コードで事前計算埋め込みを使うため実質的に凍結特徴量として扱われる。
- 学習方法: 1構成につき2抗原40万ペアで学習、1抗原20万ペアで検証、1抗原20万ペアでテスト。4抗原が1回ずつテストになるようローテーション。
- 損失関数・目的関数: MSE。順位の符号だけでなく親和性差の大きさも学習。
- 最適化: Adam、学習率0.001、batch size 32、dropout 0.3、検証lossが5 epoch改善しなければ停止。
- ペア生成: 各抗原20万ペア。学習・検証は相対親和性差10%超だけを採用し、テストは閾値なし。
- 推論方法: ペアごとの勝敗を予測し、TrueSkill（初期値 μ=25、σ=25/3、1vs1更新）で全体順位を推定。
- ベースライン: Graphinity（AlphaBind上でfine-tune）、Boltz-2のipLDDT、GeoDockの構造信頼度由来指標、random guess。
- 評価指標: pairwise accuracy、Top-K retrieval accuracy（予測上位Kと実測上位Kの重なり/K）、Top-K retrieval precision（予測上位K中のlog10(Kd[nM])<2、すなわちKd<100 nMの割合）。
- 実装上の重要点: 全組合せは5,000候補で約2,500万ペアになるため20万ペアを層化サンプリング。公開READMEでは埋め込みを先に生成し、`model_src/main_mean.py`で学習する。

## データセットと評価設計

使用データは[AlphaBind](https://doi.org/10.1080/19420862.2025.2534626)。AlphaSeqで各抗原約3万variantの親和性を実験測定したデータから層化抽出している。

| 略称 | 親抗体・形式 | 抗原 | 使用候補数 | 分布上の特徴 |
|---|---|---|---:|---|
| PP489 | humanoid scFv | TIGIT | fine-tuning 5,000 + evaluation 5,000 | 強結合体が約40%と多い |
| PEM | Pembrolizumab-scFv | PD-1 | 5,000 + 5,000 | 二峰性、強結合体は少数 |
| VHH72 | camelid VHH | SARS-CoV-1 RBD | 5,000 + 5,000 | 右裾の長い単峰性、moderate中心 |
| TRA | Trastuzumab-scFv | HER2 | 5,000 | 主に弱結合/非結合、strong binderなし |

抗原間のglobal sequence similarityは−0.053〜0.034と低く、抗原名の暗記では解けない。2 train / 1 validation / 1 testを抗原単位で分けるため、同一抗原がsplitをまたぐ一般的なリークは避けられている。一方、全抗原が同じAlphaBind/AlphaSeq系から来るので、測定プラットフォームやライブラリ生成規則に由来する共通信号は残る。また各系は1つの親抗体を多点変異した局所ライブラリであり、完全de novo抗体や多様なframeworkへの一般化とは異なる。

## 主要結果

### Pairwise accuracy

| 抗原系 | GeoDock | Boltz-2 ipLDDT | Graphinity fine-tuned | MochiBind | 対Graphinity差 |
|---|---:|---:|---:|---:|---:|
| PEM | 47.11% | 57.93% | 63.20% | **72.35%** | +9.15 pt |
| PP489 | 50.17% | 55.52% | 67.01% | **70.29%** | +3.28 pt |
| TRA | 51.21% | 60.09% | 68.55% | **74.27%** | +5.72 pt |
| VHH72 | 49.58% | 57.90% | 58.63% | **59.89%** | +1.26 pt |
| 4抗原単純平均 | 49.52% | 57.86% | 64.35% | **69.20%** | +4.85 pt |

全比較で片側McNemar検定p<0.0001。ただし20万ペアは同じ抗体を繰り返し含むため、ペアを完全独立標本とみなしたp値の大きさは慎重に解釈したい。VHH72ではGraphinityとの差が1.26ポイントに縮まり、候補分布によって優位性が変わる。

### Retrieval

Figure 2（precision）とFigure 3（accuracy）では、4抗原全体でMochiBindが概ね最良、fine-tuned Graphinityが次点だった。Boltz-2/GeoDockの構造信頼度はrandom guessに近く、構造の「確からしさ」を未見抗原の結合強度proxyにする危険を示す。論文は曲線上の全K値を表として提示していないため、本ノートでは数値を推定しない。Figure 4では真の親和性差が大きいペアほどMochiBind accuracyが上がり、誤りが主に差の小さい曖昧なペアに集中した。付録CではTrueSkillを単純win-rateに置き換えても傾向はほぼ同じで、結論が順位統合法だけに依存しないことを確認している。

Graphinityは事前学習checkpointのままだと47.40〜50.90%とほぼランダムだったが、AlphaBindでfine-tuneすると58.63〜68.55%へ改善した。これは、約100万件のsimulated single mutationで学んだ幾何特徴が、独立にBoltz-2構造予測されたmulti-mutation variantへそのまま移らないことを示す重要な陰性結果である。

## 重要なFigure/Table

- Figure/Table番号: Figure 1
- 何を示しているか: AlphaBind候補群 → ESM-2埋め込み → pairwise比較 → TrueSkill全体順位 → Top-K推薦という全パイプライン。
- 読み取り方: モデル精度と推薦評価を分離し、二者比較を実験候補選定へ接続する流れを見る。
- 主要な数値・傾向: 各抗原20万ペア、5,000〜10,000候補。
- なぜ重要か: MochiBindの新規性は複雑なencoderより、タスク定義と評価接続にある。
- Markdown内での扱い: 独自の文章フロー
- HTML内での扱い: 独自の再構成フロー図
- 出典URL: [著者公開PDF](https://cdn.amazon.science/2a/a7/6ea54b214519b0a41eb952a0cc2e/260409-mochibind-manuscript.pdf)
- ライセンス確認: 原論文Figureの再利用条件は未確認。原図は転載しない。

- Figure/Table番号: Table 2
- 何を示しているか: 未見抗原上のpairwise accuracy比較。
- 読み取り方: 各行でMochiBindと3 baselineを比較し、抗原ごとの難易度差も見る。
- 主要な数値・傾向: MochiBind 59.89〜74.27%、4抗原すべて最良。
- なぜ重要か: 論文で最も再利用しやすい定量結果。
- Markdown内での扱い: 必要列だけを再構成した表
- HTML内での扱い: 数値ラベル付き横棒チャートと表
- 出典URL: [iScience DOI](https://doi.org/10.1016/j.isci.2026.116522)
- ライセンス確認: 表全体は転載せず、報告値を独自表・チャートに再構成。

- Figure/Table番号: Figure 5
- 何を示しているか: 4抗原のlog10(Kd)分布の違い。
- 読み取り方: TRAはstrong binderなし、PP489は約40% strong、VHH72はmoderate中心という母集団差を先に見る。
- 主要な数値・傾向: precisionはモデルだけでなくbinder prevalenceに強く依存する。
- なぜ重要か: 単一平均値だけでモデルを比較してはいけない理由を示す。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: 4枚の分布特性カード
- 出典URL: [著者公開PDF](https://cdn.amazon.science/2a/a7/6ea54b214519b0a41eb952a0cc2e/260409-mochibind-manuscript.pdf)
- ライセンス確認: 未確認。原図は転載しない。

## Code / License / Weights

- コード公開: あり
- GitHub URL: [amazon-science/ab-binding-eval](https://github.com/amazon-science/ab-binding-eval)
- GitHub以外のコードURL: [OSF 56azs（論文記載のview-only公開）](https://osf.io/56azs/overview?view_only=df049360b3684b7db76f567502b164ea)
- 実装種別: 公式（Amazon Science）
- GitHub確認: 確認済み
- リポジトリのライセンス: CC BY-NC 4.0（商用利用不可）
- ライセンス確認元: [GitHub LICENSEファイル](https://github.com/amazon-science/ab-binding-eval/blob/main/LICENSE)およびREADME
- Hugging Face URL: 見つからず
- Hugging Face種別: 該当なし
- モデル重み・チェックポイント公開: あり
- 重み公開URL: [GitHub checkpoint directory](https://github.com/amazon-science/ab-binding-eval/tree/main/checkpoint)（PEM、PP489、TRA、VHH72）/ [OSF](https://osf.io/56azs/overview?view_only=df049360b3684b7db76f567502b164ea)
- データセット公開: あり
- データセットURL: [GitHub data directory](https://github.com/amazon-science/ab-binding-eval/tree/main/data) / 原データ [AlphaBind](https://doi.org/10.1080/19420862.2025.2534626)
- 再現性メモ: 学習・評価コード、pairファイル、チェックポイントが揃う。完全学習には`generate_esm2_embedding.py`で事前計算したESM平均埋め込みが必要。READMEの実行例には`data_processed_esm_mean.py`と書かれている箇所があり、実ファイル名との不一致に注意する。コードのCC BY-NC 4.0と、READMEが説明するAlphaBind由来データのMIT条件は区別する必要がある。ESM-2本体の利用条件も別途確認が必要。

## 抗体研究・創薬への意味

MochiBindは、配列生成モデルや実験ライブラリが出した大量候補を安価にpre-screenする段階に向く。特に「Kdを正確に報告する」用途ではなく、「限られたwet-lab枠へ何を送るか」を決めるrankerである。Top-K precisionを使えばfalse positiveを減らす選抜能力、Top-K accuracyを使えば真の最上位をどれだけ回収できたかを評価できる。生成モデルの論文にも、同一抗原内ランダムsplitではなく抗原holdoutとbinder/non-binder混在プールを要求するというベンチマーク上の示唆がある。

一方、抗体医薬で必要な特異性、交差反応性、発現、凝集、粘度、免疫原性は別問題である。MochiBind上位だけでleadを決めず、構造・liability・developability・negative target・実験検証と組み合わせる必要がある。

## 限界と注意点

### 著者が述べている限界

- 大規模で高品質、かつ商用・オープン利用可能な抗体–抗原親和性データが不足している。
- 抗原ごとに親和性分布が大きく異なり、同じprecisionでもタスク難易度が違う。
- Graphinityのsimulated single-mutation学習分布は、実測multi-mutationデータや独立構造予測の座標揺らぎへ移りにくい。
- より良いretrieval/ranking戦略と、さらに多様なデータが必要。

### 読んで気づいた限界

- 抗原は4種だけで、すべてAlphaBind/AlphaSeq由来。真のcross-lab、cross-assay、cross-format一般化は未検証。
- 各抗原系は単一seed抗体の変異近傍で、de novo生成抗体の広いsequence/framework空間とは異なる。
- ESM-2平均プーリングはCDR、paratope、epitope、chain pairing、界面局所性を明示的に扱わない。どの残基が予測を駆動したかも分かりにくい。
- Boltz-2/GeoDockは構造信頼度を親和性proxyとして用いており、親和性専用headとの比較ではない。これは「構造モデル一般が劣る」証明ではない。
- GraphinityだけがAlphaBindへfine-tuneされるため、各baselineの利用条件は非対称。ただし事前学習のままではほぼランダムという結果も併記されている。
- 20万pairは候補を共有するため統計的に独立ではない。McNemar検定の極小p値より、抗原別effect sizeと再現性を重視すべき。
- CPU 13秒は埋め込み生成を除いたhead推論と考えられ、end-to-end処理時間ではない。
- 学習時は差10%以下を除外し、テストには含める。実用的なstress testだが、近接順位の学習能力は意図的に弱くなりうる。
- prospectiveな新規抗原キャンペーンでのhit rate改善は本論文単独では実証されていない。
- 公式コードはCC BY-NC 4.0で、商用創薬へそのまま組み込めない。

## この論文を読む上での前提知識

- **Kdとlog10(Kd)**: Kdが小さいほど結合が強い。本論文はnM単位のlog10(Kd)を使い、log10(Kd)<2、すなわちKd<100 nMをgood binderとする。
- **Pairwise ranking**: 絶対値を回帰せず、2候補の順序を学ぶ。測定スケール差に頑健な一方、全体順位へ統合するアルゴリズムが必要。
- **ESM-2**: 大量タンパク質配列で事前学習されたprotein language model。残基ごとの文脈表現を下流タスクの特徴量に使える。
- **TrueSkill**: 対戦結果から各候補の潜在能力と不確実性を更新するBayesian rating法。本論文では抗体同士の仮想対戦を全体順位へ変換する。
- **Antigen holdout**: 抗体単位ではなく抗原全体をtestへ隔離する分割。未知標的への汎化を測るには重要。
- **Retrieval precision / accuracy**: 前者は推薦上位の純度、後者は真の上位Kとの重なりを見る。本論文のaccuracyは一般的な分類accuracyとは定義が異なる。
- **AlphaSeq / AlphaBind**: yeast-displayと次世代シーケンスを使う大規模親和性測定系と、それに基づく抗体–抗原データ・モデル。

## 今回この1報を選んだ理由

2026-09-28以降に広く紹介されたAmazon Bio Discoveryの3研究を候補にし、MochiBind論文を選んだ。新着のpeer-reviewed原著であり、抗体生成より手薄だった「未見抗原で候補をどう順位付けし、どう公平に評価するか」を正面から扱う。4抗原・各数千候補・非結合体を含むcross-antigen splitは学習価値が高く、Table 2とFigure 1〜5から評価設計を具体的に学べる。さらに公式GitHub、データ、4 checkpoint、OSFがあり、再現性確認の入口が明確である。Agent-guided nanobody designは実験結果が魅力的だが既存ノートのde novo設計テーマと重なりが大きく、今回は知識ベースのバランスを優先した。

## 読む優先度

**High**。親和性予測モデルそのものより、抗原holdout、非結合体を含む候補プール、Top-K回収という評価設計が今後の論文を読む基準になる。4抗原という限界を理解した上で、生成モデルや構造モデルの評価表と比較して読む価値が高い。

## 自分用メモ

- 次はAlphaBind原著を読み、AlphaSeq測定・ライブラリ設計・MITデータ範囲を確認する。
- 公開checkpointでTable 2を再現し、pair sampling seedとTrueSkill処理順への感度を測る。
- MochiBindにCDR-aware pooling、antigen residue attention、hard-negative targetを加えたときの改善を試す。
- Boltz-2 affinity headを使った公平な再比較が可能か確認する。
- Obsidianリンク候補: [[AlphaBind]], [[ESM-2]], [[Graphinity]], [[Boltz-2]], [[抗体親和性予測]], [[cross-antigen split]], [[retrieval evaluation]]

## 関連キーワード

- antibody–antigen binding affinity
- pairwise ranking
- candidate retrieval
- out-of-distribution generalization
- ESM-2
- TrueSkill
- AlphaBind / AlphaSeq
- non-binder
- antigen holdout

## 検索ログ

- 検索したデータベース/クエリ: PubMed、Amazon Science、iScience/Cell、bioRxiv、arXiv、GitHub、Hugging Face、OSF。`antibody AI machine learning September 2026`、`MochiBind GitHub`、`MochiBind Hugging Face`、論文完全タイトル、DOIで検索。
- 候補にした論文: 本論文、`Agent-guided de novo design of nanobody binders against a novel cancer target`、`Context-aware multi-property antibody predictor`、`Frozen Protein Foundation-Model Embeddings Improve Antibody–Antigen Binding Affinity Prediction`、AI抗体設計レビュー。
- 最終的にこの1報を採用した理由: peer-reviewed新着、抗体に直接関係し、評価設計の汎用性、定量結果、公式コード・データ・重みの公開度が最も高かった。
- 新着論文と基盤論文のバランス: 前回に続き新着だが、今回は生成研究ではなく評価基盤を選び、知識ベースの偏りを抑えた。次回はAbLang、DiffAb、MEANなど基盤論文を優先候補に残す。
- PDF/HTML/Supplementaryを確認できたか: 著者公開23ページPDFを全文確認。iScience HTMLはアクセス制限で直接取得できなかったが、DOI、PubMed/PMC、Amazon Science、著者PDFで書誌・本文を照合。Figure 1〜5、Table 1〜2、Appendix A〜Cを確認。
- Figure/Tableを確認したURLやライセンス確認状況: [著者公開PDF](https://cdn.amazon.science/2a/a7/6ea54b214519b0a41eb952a0cc2e/260409-mochibind-manuscript.pdf)。原論文Figureの明示的な再利用ライセンスは確認できず、原図を保存・転載しなかった。
- GitHub/コード検索で使ったクエリ: 完全タイトル、`MochiBind GitHub`、`amazon-science ab-binding-eval`。
- 確認したGitHub URL: [amazon-science/ab-binding-eval](https://github.com/amazon-science/ab-binding-eval)、README、LICENSE、checkpoint、data。
- 確認したHugging Face URL: サイト検索を実施したがMochiBind公式model/datasetは見つからず。
- 確認したその他コード/重み/データURL: [OSF 56azs](https://osf.io/56azs/overview?view_only=df049360b3684b7db76f567502b164ea)、[AlphaBind DOI](https://doi.org/10.1080/19420862.2025.2534626)。
- リポジトリライセンスの確認元: GitHub LICENSEとREADMEでCC BY-NC 4.0を確認。
- モデル重み・チェックポイントの確認先: GitHub `checkpoint/`にPEM、PP489、TRA、VHH72を確認。OSFも論文Resource Availabilityに記載。
