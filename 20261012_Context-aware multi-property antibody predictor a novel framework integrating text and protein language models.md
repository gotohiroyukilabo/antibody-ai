# Context-aware multi-property antibody predictor: a novel framework integrating text and protein language models

> HTML版: [同じ論文のビジュアルノート](20261012_Context-aware%20multi-property%20antibody%20predictor%20a%20novel%20framework%20integrating%20text%20and%20protein%20language%20models.html)

## まず何の論文か

CA-MAP（Context-aware Multi-Property Antibody Predictor）は、抗体の配列だけを見て固定的な物性値を返すのではなく、同じ測定環境から得た少数の「抗体配列―物性値」例を文脈として受け取り、問い合わせ抗体のdevelopability特性を予測するモデルである。狙いは、ラボ、装置、実験日などで測定値がずれるバッチ効果へ、再学習なしで適応することにある。モデルは抗体重鎖配列、物性名、数値を別々の経路で埋め込み、Mambaベースの小型オーケストレータで統合する。さらにAB-context-aware trainingという学習法で、文脈と正解に同じランダム変換を施し、モデルが文脈を無視して配列だけから近道することを防ぐ。876,898本のユニークな抗体重鎖と、計算で付与された6種類のdevelopability特性を用いて評価した。強い模擬バッチ効果（加算バイアス0〜0.3）では、通常fine-tuningしたTxGemmaのhydrophobicity予測がSpearmanのρ=0.58まで低下した一方、AB-context-aware版は約0.99を維持した。4特性同時予測ではCA-MAPが全特性で数値を100%返し、ρ=0.66〜0.98を示した。学習時に目的変数として使わなかったimmunogenicityとpositive charge heterogeneityも、6特性を文脈に含めると双方ρ=0.73まで改善した。学習可能パラメータは約18.2万で、LoRA適用TxGemmaの4,000万より大幅に少なく、事前計算済み埋め込みを除く推論は約0.009秒/プロンプトだった。ただし教師ラベルは実測値ではなく既存計算ツールの出力であり、実ラボ間バッチや実験的developabilityを直接検証した論文ではない。したがって本研究の価値は「実用精度を確立した完成品」というより、抗体物性予測をin-context adaptationとして定式化し、文脈利用を強制する学習設計を示した点にある。

## 書誌情報

- URL/DOI: [Nature論文ページ](https://www.nature.com/articles/s41540-026-00723-1) / [10.1038/s41540-026-00723-1](https://doi.org/10.1038/s41540-026-00723-1)
- 公開日/更新日: 2026-04-25（Received 2026-01-16、Accepted 2026-04-11）
- 著者・所属: Luca Giancardo, Melih Yilmaz, Edward Lee, Ke Ren, Yue Zhao, Gordon Trang, Kemal Sonmez, Lan Guo, Nina Cheng / Life Sciences, Amazon Web Services, Seattle, WA, USA
- 掲載誌/プレプリントサーバー: npj Systems Biology and Applications 12, Article 95 (2026)
- リサーチ日: 2026-10-12（JST）
- 分類: 新着 / モデル起点

## 背景と問題設定

抗体候補は、標的への結合だけでなく、溶解性、安定性、疎水性、凝集傾向、電荷不均一性、免疫原性など複数のdevelopability軸を満たす必要がある。従来の予測器は、1特性ごとの回帰ヘッドや特定アッセイ用のfine-tuningを要求し、別ラボや別測定系に移るたびに再学習が必要になりやすい。また、同じ分子でも装置、試薬、実験日、担当者による系統的な測定ずれが入り、学習データと実運用データのバッチ差が性能を壊す。

LLM的なプロンプトへ少数の例を入れれば適応できそうに見えるが、学習時の全データが同じ尺度なら、モデルは例を読まず配列だけで答えるshortcut learningを起こしうる。さらに、抗体配列、自然言語の物性名、連続数値という異種モダリティを一つの語彙空間へ無理に押し込むと、巨大モデルと高い計算費用が必要になる。本研究は、(1) 文脈を使わないと正解できない訓練問題を人工的に作る、(2) 各モダリティを専用encoder/projectorで32次元へ揃える、(3) Mambaで可変長の文脈を統合する、という三段構えでこの問題に取り組む。

## この論文のコアアイデア

AB-context-aware trainingでは、各プロンプトごと、かつ物性ごとにランダム潜在変数を引き、文脈中の全物性値と正解値へ同じ変換を適用する。たとえば元の物性値へ同じランダムな加算オフセットを加える。問い合わせ配列だけを見てもそのオフセットは分からず、文脈例の配列―値対応を読んだときだけ正しい尺度を復元できる。そのため、モデルは `P(y | query, context)` を `P(y | query)` へ退化させにくくなる。

CA-MAPの入力は、複数のcontext antibody（重鎖配列と任意数の物性名・値）と、1つのquery antibody（重鎖配列と予測したい物性名）である。配列は凍結ESM-2 `esm2_t6_8M_UR50D`、物性名は凍結Sentence-BERT `paraphrase-multilingual-MiniLM-L12-v2`、数値は線形層で埋め込む。4経路（配列、物性名、数値、特殊token）を32次元へ揃え、6層Mambaへ順番に入力する。最後のEOS token表現を単一の線形headで0〜1の物性値へ変換するので、特性ごとの専用headは不要である。

## 手法の詳細

- 入力データ: 10〜15本を基本とするcontext抗体重鎖配列、各配列に付随する可変個の物性名・0〜1正規化値、query抗体重鎖配列、問い合わせ物性名。
- 出力: 問い合わせ物性の単一連続値。CA-MAPは構造上必ず数値を返す。
- モデル/アルゴリズム: 6層Mamba（hidden state 32、local convolution width 4、block expansion 2）と単一回帰head。
- 特徴量・表現学習: ESM-2の最終hidden stateを残基方向に平均し、線形projectorで32次元化。物性名はSentence-BERTのmean pooling後に32次元化。数値と特殊tokenも32次元化する。
- 学習方法: ESM-2とSentence-BERTは凍結。Mamba、projector、特殊token、回帰headを学習する。公開実装の初期設定はbatch size 128、learning rate 0.001、weight decay 0.01、最大20 epochs、模擬bias最大0.3。
- 損失関数・目的関数: 連続値回帰。本文と公開コードからMSELossの使用を確認できるが、論文本文では損失式を詳細展開していない。
- 推論方法: 同じ測定条件を代表する抗体例をcontextへ入れ、query配列と物性名を末尾へ置く。再学習なしでcontextに含まれる尺度・バッチへ適応する。
- ベースライン: TxGemma 9B（実効5B、LoRAで学習可能40M）、ESM-2埋め込みに対するordinary least-squares線形回帰、AB-context-awareを使わない通常fine-tuning。
- 評価指標: Spearman順位相関ρ、数値としてparse可能な出力率、推論時間。
- 実装上の重要点: CA-MAPの報告推論時間は配列・物性名埋め込みの事前計算時間を含まない。Mambaは後方文脈を見るため予測headを末尾EOSへ置く。

## データセットと評価設計

元データはBashour et al.のdevelopability profilingデータで、OAS由来1,738,091配列と実験生成298,698配列を含む。そこから本研究はユニークな重鎖876,898本だけを選んだ。対象はhuman heavy chain（IgD/IgM/IgG/IgA/IgE）とmouse heavy chain（IgM/IgG）を含むが、軽鎖はCA-MAP評価から除外された。

6特性は、Hydrophobicity（Peptides R）、Solubility（SoluProt v1.0）、Stability/aliphatic index（Peptides R）、Immunogenicity（netMHCIIpan 4.0）、positive/negative charge heterogeneity（Therapeutic Antibody Profiler）で計算された。つまり「実験生成配列」を含んでも、CA-MAPの教師ラベル自体は計算値である。

配列をsetA 70%、setB 15%、setC 15%へ先に分け、順にtrain/validation/testへ使用した。プロンプト生成もset内だけで行い、一つのプロンプトへ異なるsetの配列を混ぜていない。test contextもsetC配列から作るため、配列単位の直接リークは抑制されている。一方、配列類似度クラスタやgermline/CDR identityによる分割は記載されず、近縁配列を跨いだ一般化の厳しさは未確認である。

Hydrophobicity単一特性実験はtrain 70,156、validation 15,043、test 15,043プロンプト。4特性実験もtrain 70,156、validation 15,043、test約15,000で、contextに10〜15抗体を用いた。模擬バッチは物性ごとに0〜0.1または0〜0.3の一様分布から加算した。実ラボ、実装置、実日付をまたぐ外部検証はない。

## 主要結果

### 1. 文脈を使わせる学習法は模擬バッチに強い

Hydrophobicity予測でバッチ効果なしの場合、通常fine-tuningとAB-context-awareはいずれもρ≈0.99だった。0〜0.1の加算biasでは通常法がρ=0.95へ低下したが、context-aware法は約0.99を維持した。0〜0.3では通常法がρ=0.58へ崩れ、context-aware法は約0.99を保った。数値出力率もcontext-awareはほぼ100%だったのに対し、通常法は強いbiasで約88%まで落ちた。

### 2. 小型CA-MAPは4特性を安定して数値化する

Table 1のbias 0〜0.3条件で、CA-MAPはHydrophobicity ρ=0.98、NegCh heterogeneity 0.84、Solubility 0.83、Stability 0.97を示し、全て出力率100%だった。biasなしでも順に0.95、0.66、0.81、0.97である。対照のTxGemmaは出力自体が少なく、最大でもSolubilityの2.62%で、Stabilityは数値出力0件だった。ESM-2線形回帰は順に0.42、0.14、0.49、0.47で、CA-MAPの改善幅はρで0.34〜0.70だった。

### 3. 未学習特性は、相関する他特性を文脈に足すと改善する

ImmunogenicityとPosCh heterogeneityを学習・validationの目的特性から外した実験で、contextにこの2特性だけを入れるとρは0.25と0.08だった。6特性すべてをcontextへ含めると、双方ρ=0.73へ上昇した。これは未知物性名を理解しただけではなく、既知物性間の相関と少数の目的物性例を組み合わせた推定である。Figure 2ではcontext数を増やすほど性能が上がるが、およそ100抗体で改善が頭打ちになる。

### 4. 計算量は軽いが、埋め込み生成は別勘定

総パラメータは125Mだが大半は凍結encoderで、学習可能なのは約182K。TxGemmaはLoRAでも約40Mを学習する。単一NVIDIA L40S上でTxGemmaは1.83秒/prompt、CA-MAPは0.009秒/promptと約200倍速い。ただしCA-MAP値はESM-2とtext embeddingの事前計算を除くため、end-to-end速度として直接比較してはいけない。

## 重要なFigure/Table

### Figure 1: 通常fine-tuningとAB-context-awareのバッチ耐性

- Figure/Table番号: Figure 1
- 何を示しているか: 模擬加算biasを強くしたときのTxGemma hydrophobicity予測の順位相関と出力安定性。
- 読み取り方: biasなし、0〜0.1、0〜0.3の3条件で、通常法とcontext-aware法を比較する。
- 主要な数値・傾向: 通常法はρ≈0.99→0.95→0.58、context-aware法は全条件で約0.99。
- なぜ重要か: 本論文の学習アイデアが「例を与えただけ」ではなく、モデルに例を使わせる仕掛けであることを示す。
- Markdown内での扱い: 独自の再構成表（本文数値）
- HTML内での扱い: 独自の横棒比較図
- 出典URL: [論文PDF](https://www.nature.com/articles/s41540-026-00723-1.pdf)
- ライセンス確認: 確認済み（CC BY-NC-ND 4.0。改変図の共有不可なので原図は不使用）

### Table 1: 4特性予測とESM-2 ablation

- Figure/Table番号: Table 1
- 何を示しているか: TxGemma、CA-MAP、ESM-2線形回帰の4物性に対するρと数値出力率。
- 読み取り方: ρだけでなく、モデルが有効な数値を返した割合を見る。出力率が低いTxGemmaのρをCA-MAPと同列に扱わない。
- 主要な数値・傾向: bias 0〜0.3のCA-MAPはρ=0.84〜0.98、出力率100%。TxGemmaは最大出力率2.62%。
- なぜ重要か: 小さい専用multimodalモデルが、巨大な生成LLMより数値回帰へ適する可能性を示す。
- Markdown内での扱い: 必要列だけの再構成表
- HTML内での扱い: ρの横棒チャートと出力率注記
- 出典URL: [論文ページ](https://www.nature.com/articles/s41540-026-00723-1)
- ライセンス確認: 確認済み（原表は転載せず独自表現）

### Figure 4 / Table 2: アーキテクチャと未学習特性

- Figure/Table番号: Figure 4、Table 2
- 何を示しているか: 4種類の埋め込み経路をMambaへ統合する構成と、相関特性をcontextへ加える効果。
- 読み取り方: 配列、物性名、数値、特殊tokenを一つのtext token列へ変換せず、それぞれ32次元化して順序列として統合する。Table 2は目的2特性だけのcontextと全6特性contextを比較する。
- 主要な数値・傾向: Immunogenicity 0.25→0.73、PosCh 0.08→0.73。
- なぜ重要か: モデル設計と「安価な測定から高価な測定を補助する」という応用の接点を示す。
- Markdown内での扱い: 独自フローと要約値
- HTML内での扱い: 独自アーキテクチャ図とpaired bar chart
- 出典URL: [論文PDF](https://www.nature.com/articles/s41540-026-00723-1.pdf)
- ライセンス確認: 確認済み（原図は不使用）

## Code / License / Weights

- コード公開: あり
- GitHub URL: [amazon-science/ca-map](https://github.com/amazon-science/ca-map)
- GitHub以外のコードURL: 該当なし
- 実装種別: 公式 / 著者実装
- GitHub確認: 確認済み
- リポジトリのライセンス: CC BY-NC 4.0（商用利用不可）
- ライセンス確認元: GitHubのLICENSEファイル、README、各source header
- Hugging Face URL: 該当なし
- Hugging Face種別: 該当なし
- モデル重み・チェックポイント公開: なし
- 重み公開URL: 該当なし。READMEに「weights are not provided」と明記。
- データセット公開: あり
- データセットURL: [developability.csv](https://github.com/csi-greifflab/developability_profiling/blob/main/data/native/developability.csv) / [元データrepository](https://github.com/csi-greifflab/developability_profiling)
- 再現性メモ: 学習、tokenizer、dataset、Mamba本体、推論実験、例示出力は公開されている。学習済みCA-MAP重みと論文のraw prediction CSVは含まれず、再現には元CSVからの再学習が必要。公開READMEは`models/`を示すが、提供物はplaceholderで、推論scriptも学習済み重みを要求する。ESM-2とSentence-BERTの外部checkpoint取得、CUDA環境も必要。公式Hugging Face model/dataset/SpaceはGitHub README、論文Data/Code availability、Hugging Face検索で見つからなかった。

## 抗体研究・創薬への意味

CA-MAPが効くのは、生成モデルで候補を作った後のmulti-parameter triageである。新しいキャンペーンで少数の実測例を取得し、それをcontextとして大規模候補群の溶解性・安定性などを順位づけできれば、アッセイごとにモデルをfine-tuneする運用を減らせる。特にラボ固有のoffsetをcontextへ含める発想は、装置更新、site transfer、assay protocol変更が頻繁な創薬現場と相性がよい。

また、安価な物性を多数測り、高価な物性を一部だけ測って文脈に混ぜる「測定カスケード」へつながる。単一headが欠損値を許容するため、全候補で同じpanelを揃えにくい初期探索にも向く。一方で、本論文は結合親和性、特異性、off-target、発現量を検証しておらず、抗体最適化の万能scorerではない。実用化には実測データでの校正、抗原/軽鎖/format条件の追加、uncertainty推定が必要である。

## 限界と注意点

### 著者が述べる限界

- 教師となる6物性は実験測定ではなく計算予測値で、実験的妥当性は未確認。
- contextの品質と代表性へ強く依存する。
- 6特性だけで、binding affinity、specificity、off-target bindingへの一般化は未検証。
- 複数ラボ・施設の実データによる外部検証が必要。
- 特性間相関を利用できる理由は未解釈で、projector別ablationや表現解析が不足。

### この精読で気づいた限界

- splitはユニーク配列単位であり、sequence identity cluster、germline、CDR familyを跨ぐ厳格分割ではない。近縁配列がtrain/testに分かれる可能性は残る。
- 重鎖のみで、軽鎖とのpairing、抗原、format、buffer、濃度といったdevelopabilityの条件依存性を直接表現しない。
- 模擬batchは主に加算offsetで、heteroscedastic noise、非線形drift、欠測機構、ラボごとのlabel definition差を網羅しない。
- 未学習物性実験でもtest contextには目的物性のラベル例が入る。完全なzero-shotではなくfew-shot calibrationと解釈すべき。
- TxGemma比較は数値parse率が極端に低く、prompt設計・decoding改善余地が大きい。一方、CA-MAPのタスク適合性が高いこと自体は変わらない。
- 推論速度は埋め込み事前計算を除き、end-to-end比較ではない。
- コードはCC BY-NC 4.0、論文はCC BY-NC-ND 4.0で、商用利用や改変物の共有に制約がある。
- 学習済み重みがないため、「すぐ使えるモデル」ではなく再学習可能な研究実装である。

## この論文を読む上での前提知識

- Developability: 抗体が製造、保存、投与、体内利用に耐えられるかを表す複数特性。結合力が高くても凝集・低溶解性・不安定なら医薬候補になりにくい。
- バッチ効果: 生物差ではなく、測定日、装置、施設、試薬などが生む系統的な値のずれ。順位を保つ単純offsetから複雑な非線形差まである。
- In-context learning / conditioning: モデルの重みを更新せず、入力中の例を使って新しい尺度やタスクへ適応する考え方。
- Protein language model: 大量のアミノ酸配列から残基・配列表現を学ぶモデル。CA-MAPは小型ESM-2を凍結encoderとして使う。
- Mamba / state space model: 長い系列を線形時間で扱うことを狙う系列モデル。CA-MAPでは異種embedding列のオーケストレータとなる。
- Spearmanのρ: 予測値と真値の順位一致度。絶対誤差の小ささではなく、候補rankingが合うかを見る指標。
- Shortcut learning: 本来使わせたい情報を無視し、訓練データ中の簡単な相関だけで正解する現象。

## 今回この1報を選んだ理由

前回は抗体―抗原結合の汎用ranking評価（MochiBind）だったため、今回は創薬の別ボトルネックであるmulti-property developabilityへ範囲を広げた。CA-MAPは2026年の査読済み新着論文で、抗体に直接関係し、モデル、学習戦略、データ、速度、失敗点までFigure/Tableから具体的に学べる。さらに公式コードと元データが公開され、重みが非公開である点も含めて再現性を現実的に評価できる。AlphaBind原著は重要な基盤候補だが、前回MochiBindノートで4つのAlphaBind datasetを詳しく扱った直後なので、知識ベースの重複を避けCA-MAPを優先した。

## 読む優先度

**High**。抗体物性予測そのものよりも、「同じラボの少数例を使って尺度へ適応させる」「文脈を無視できないよう訓練する」という設計が汎用的である。実測検証は弱いが、社内アッセイを複数持つ研究者や、生成―評価loopを設計する人には実装上の示唆が大きい。

## 自分用メモ

- 後で深掘りしたい点: 実測multi-assay datasetでのsite/batch holdout、配列cluster split、uncertainty calibration、context選択法。
- 関連して読むべき論文: Bashour et al. 2024 developability landscape、TxGemma、PROPERMAB、Therapeutic Antibody Profiler、前回のMochiBind、AlphaBind。
- 実装を触る場合の入口: `train.py`の`propToUseLst`、`abNumRangePerPrompt`、`maxBias`を確認し、まず公開CSVの小規模subsetで学習。次に自施設の物性名・正規化範囲を`datasetDevProp.py`へ対応させる。
- Obsidianでリンクしたいキーワード: [[抗体developability]]、[[バッチ効果]]、[[in-context learning]]、[[ESM-2]]、[[Mamba]]、[[multi-property optimization]]

## 関連キーワード

- antibody developability
- multi-property prediction
- batch effect
- in-context learning
- protein language model
- ESM-2
- Sentence-BERT
- Mamba
- multimodal learning
- shortcut learning

## 検索ログ

- 検索したデータベース/クエリ: Web検索、Nature、PubMed/PMC相当、Amazon Science、GitHub、Hugging Face。`antibody artificial intelligence machine learning 2026 design prediction`、`Context-aware multi-property antibody predictor`、`CA-MAP antibody GitHub`、`CA-MAP Hugging Face`、`AlphaBind official paper GitHub`を検索。
- 候補にした論文: CA-MAP、AlphaBind原著、AB-Panda、Benchmarking antigen-aware inverse folding methods、ADiT、Frozen Protein Foundation-Model Embeddings。
- 最終的にこの1報を採用した理由: 新着・査読済み、抗体developabilityに直接関係、公式コードとデータあり、前回のbinding affinity benchmarkと補完的、Figure/Tableの教育価値が高い。
- 新着論文と基盤論文のバランス: 今回は新着を採用。AlphaBindを基盤候補として比較したが、前回とのデータ重複を避けた。
- PDF/HTML/Supplementaryを確認できたか: Natureの9ページPDF全文、HTML metadata、Supplementaryの存在を確認。Supplementary本文の個別値は本ノートの主張へ使っていない。
- Figure/Tableを確認したURLやライセンス確認状況: [Nature PDF](https://www.nature.com/articles/s41540-026-00723-1.pdf)でFigure 1〜4、Table 1〜2を確認。論文ページでCC BY-NC-ND 4.0を確認。原図は保存・転載せず、事実と報告値だけを独自構成で再可視化。
- GitHub/コード検索で使ったクエリ: `site:github.com "Context-aware multi-property antibody predictor"`、論文Code availability、Amazon ScienceのGitHubリンク。
- 確認したGitHub URL: [CA-MAP公式実装](https://github.com/amazon-science/ca-map)、[元developability dataset](https://github.com/csi-greifflab/developability_profiling)。
- 確認したHugging Face URL: 公式CA-MAP model/dataset/Spaceは見つからず。依存checkpointは[ESM-2](https://huggingface.co/facebook/esm2_t6_8M_UR50D)と[Sentence-BERT](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2)。
- 確認したその他コード/重み/データURL: [developability.csv](https://github.com/csi-greifflab/developability_profiling/blob/main/data/native/developability.csv)、[Amazon Science publication page](https://www.amazon.science/publications/context-aware-multi-property-antibody-predictor-a-novel-framework-integrating-text-and-protein-language-models)。
- リポジトリライセンスの確認元: CA-MAPのLICENSE、README、source headerでCC BY-NC 4.0を確認。論文はNature rights statementでCC BY-NC-ND 4.0。
- モデル重み・チェックポイントの確認先: CA-MAP README、`models/`、inference説明、GitHub Releases、Hugging Face検索。READMEに重み非提供と明記され、release/Hugging Faceにも公式学習済みCA-MAP重みは見つからなかった。
