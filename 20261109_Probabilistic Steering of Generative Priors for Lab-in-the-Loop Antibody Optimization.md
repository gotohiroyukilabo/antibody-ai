# Probabilistic Steering of Generative Priors for Lab-in-the-Loop Antibody Optimization

> HTMLビジュアルノート: [同名HTML版](20261109_Probabilistic%20Steering%20of%20Generative%20Priors%20for%20Lab-in-the-Loop%20Antibody%20Optimization.html)

## まず何の論文か

この論文は、抗体の「de novo発見」と「少数の実験データを用いた反復最適化」を別工程として扱わず、1つの生成・獲得ループへ滑らかに接続するCHAD（Calibrated Handoff via Acquisition-guided Diffusion）を提案する。出発点は、標的条件付きの配列・構造生成モデルProteina-Complexaであり、その学習済み重みは固定したまま使う。各ラウンドで蓄積した測定値からGaussian process（GP）サロゲートを学習し、予測平均だけでなく予測不確実性も含むlog probability of improvement（logPI）を報酬にする。生成途中の複数粒子をFeynman–Kac（FK）steeringで再標本化し、生成モデルが学んだ分子らしい領域を保ちながら有望なCDR配列へ集団を偏らせる。最終バッチは、予測平均と不確実性のPareto frontをqHSRIで順位付けして選ぶ。VHH–独自標的、scFv–SARS-CoV-2 peptide、trastuzumab–HER2の3キャンペーンを回顧的に再生し、各キャンペーンで3反復、1反復あたり初期32配列と10ラウンド×32配列を評価した。CHADは全3標的でbest quality、top-10平均、親配列を上回る比率が最良だった。特にPTでは最高5.1倍、SARS-CoV-2では119.3倍のoracle上の改善を報告する。一方、獲得関数でsteeringすると平均予測値だけでsteeringした場合より、バッチ選択前のunique variantが概ね2倍になり、CDR空間被覆のAUC比は1.20–1.51だった。ただし実際の新規配列をwet labで測った研究ではなく、完了済みキャンペーンから学習したoracleを実験の代理にしたin silicoシミュレーションである。したがって、CHADは有望な探索アルゴリズムだが、「実験で抗体を改善した」と読むべき論文ではない。

## 書誌情報

- URL/DOI: [arXiv:2611.03844](https://arxiv.org/abs/2611.03844) / [10.48550/arXiv.2611.03844](https://doi.org/10.48550/arXiv.2611.03844)
- 公開日/更新日: 2026-11-02（v1、UTC）
- 著者・所属: Anisha Parsan（MIT、研究実施時はBigHat Biosciences）、Sebastian Ober、Calvin McCarter、Nick Bhattacharya、Hunter Elliott（BigHat Biosciences）
- 掲載誌/プレプリントサーバー: arXiv（NeurIPS 2026 GEM workshop / NeurIPS 2026 AI4DD workshop採択）
- リサーチ日: 2026-11-09（JST）
- 分類: 新着 / 抗体最適化 / 生成モデル×Bayesian optimization

## 背景と問題設定

抗体AIの実運用には、測定値がない段階で候補を出す「発見」と、少数の結合・安定性測定を積み上げて候補を改良する「最適化」がある。前者は構造条件付き生成モデルが得意だが、実験hit率、親和性、developabilityは十分でないことが多い。後者はGPなどのサロゲートとBayesian optimization（BO）が得意だが、データがほとんどない初期ラウンドでは予測器が弱く、探索空間も巨大である。

既存工程では生成モデルが出したhitを、別の最適化系へ「手渡す」。この境界では、生成モデルが持つ構造・配列の事前知識をどこまで残し、何件の測定後にデータ駆動探索へ重心を移すかが明示されない。CHADはこの問題を、生成分布を獲得関数で徐々に傾ける確率的推論として定式化する。狙いは、低Nでは生成priorを頼り、データが増えてGPの不確実性が縮むほど、観測データに基づくexploitationへ自然に移ることである。

## この論文のコアアイデア

最適化したい抗体配列を `x`、固定した生成priorを `pθ(x)`、ラウンドrの獲得関数を `a_r(x)` とすると、CHADが狙う分布は `p_r(x) ∝ pθ(x) exp(λ a_r(x))` である。λはsteering強度で、0なら生成priorのまま、強くすると高報酬側へ集中する。

Proteina-Complexaは400 stepのflow matchingで、各残基のCα座標と8次元潜在表現を同時に生成する。CHADは複数粒子を並列にdenoiseし、後半step 320/340/360/380で、その時点のclean estimateを配列へdecodeする。各配列のlogPIをGPで計算し、`exp(λ logPI)` に比例して粒子を再標本化する。微分可能な報酬を要求しないため、将来的には実測値から作った任意のサロゲートや非微分フィルタを使える。

毎ラウンド、GPは全測定データから再学習される。候補poolからはqHSRIを用い、予測平均μと不確実性σのPareto frontを考慮して32配列を選ぶ。直前ラウンドの上位20%からCDR Hamming距離が離れたincumbentを選び、次の生成seedにする。これにより「良い系統を深掘る」ことと「異なる系統を残す」ことを同時に狙う。

## 手法の詳細

- 入力データ: 親抗体配列、抗原構造、設計対象CDR、ラウンドごとのoracle測定値
- 出力: frameworkを固定し、指定CDRだけを変更した抗体候補バッチ
- モデル/アルゴリズム: Proteina-Complexa + GP（one-hot CDR表現、Tanimoto kernel）+ FK steering + qHSRI
- 特徴量・表現学習: 各残基のCα座標と8次元latentをflow modelで生成。GPには設計対象CDRのone-hot表現を使用
- 学習方法: 生成priorは固定。GPを各ラウンドで全観測値から再fit
- 損失関数・目的関数: CHAD自体は追加学習をせず、logPIをFK potentialとして使用。目的はoracle fitnessの最大化
- 推論方法: warm-start fraction γ=0.75、400 step中step 300からdenoise、320/340/360/380で再標本化
- ベースライン: Best-of-N（同じ生成prior、再標本化なし）、OberBO（配列のみのGA portfolio BO）、LaMBO（denoising autoencoder潜在空間のBO）
- 評価指標: best quality、top-10平均、親超え率、survival curve、pairwise CDR Hamming、#Circles AUC、pI、GRAVY、OASis humanness percentile、Protenix V2 ipTM
- 実装上の重要点: 非CDR残基は毎step親配列へclampする。late-step clean estimateを完全rolloutの近似に使い計算を節約する。baselineはおおむねper-round wall-clockを合わせる

## データセットと評価設計

| キャンペーン | 形式・長さ | 抗原 | 再設計範囲 | oracle |
|---|---:|---|---:|---|
| VHH–PT | VHH、117 aa | 独自標的PT | 3 CDR、31残基 | 658,000 phage-display配列でpretrain、1,335 BLI測定でfine-tuneした10 CARP/ByteNet ensemble、CV Spearman 0.71 |
| scFv–SARS-CoV-2 | scFv、264 aa | SARS-CoV-2 peptide | VL 3 CDR、30残基 | Ab-14系の59,019 unique VL、random 80/10/10。test n=5,901、MAE 0.557、Spearman 0.858 |
| trastuzumab–HER2 | Fv、227 aa | HER2 subdomain IV | CDR-H3、10残基 | 367,042 CDR-H3 variantのbinder classifier、ROC-AUC 0.995、PR-AUC 0.990 |

各methodは3 seedで評価され、1 seedあたり32配列のcold startと10ラウンド×32配列、合計352 oracle queryである。Table 1の1,056 designsは3 seed分の合計である。PT oracleは学習分布から離れた設計に弱いため、CDR edit distanceが9を超える候補をbatch選択前に除く。HER2はwild-typeが `p(binder)≈1` で改善余地がないため、CDR-H3の50%をrandomizeした `p(binder)≈0.74` の人工的に弱いparentから開始する。

評価は同じoracleを最適化にも最終評価にも使うため、oracle exploitationを完全には検出できない。SARSのsplitもrandom 80/10/10で、近縁配列clusterを跨いだリーク対策は記載されていない。さらに、完了済みキャンペーン由来のparentを使うため、de novo hitをそのまま最適化へ引き継ぐ本来のユースケースは再現していない。この3点が評価設計の核心的な注意点である。

## 主要結果

### 3キャンペーンの最終性能

| 標的 | 方法 | best quality | top-10平均 | 親超え率 |
|---|---|---:|---:|---:|
| PT | CHAD/FK | **5.1×** | **4.8×** | **91±1%** |
| PT | Best-of-N / OberBO / LaMBO | 4.1× / 4.0× / 4.9× | 4.0× / 3.7× / 4.6× | 83±3 / 69±20 / 77±4% |
| SARS-CoV-2 | CHAD/FK | **119.3×** | **115.4×** | **94±1%** |
| SARS-CoV-2 | Best-of-N / OberBO / LaMBO | 118.5× / 100.7× / 60.7× | 110.7× / 88.4× / 48.1× | 92±2 / 80±3 / 77±4% |
| HER2 | CHAD/FK | 1.000 | 1.000 | **86±2%** |
| HER2 | Best-of-N / OberBO / LaMBO | 1.000 / 1.000 / 1.000 | 1.000 / 1.000 / 1.000 | 68±3 / 60±10 / 39±7% |

CHADは全標的で最良だが、SARSの最高値はBest-of-N 118.5×に対して119.3×で差が小さい。HER2はclassifierが1.0で飽和し、best/top-10では全方法が同点であるため、86%という親超え率の差を過大解釈してはいけない。PTではCHADの改善が最も明瞭だが、評価対象は独自oracleで、距離filterも適用される。

### 不確実性を使う意味

logPIでsteeringした場合、GP posterior meanだけでsteeringするより、バッチ選択前に得られるunique variant数がおおむね2倍だった。#Circlesで測ったCDR空間被覆AUCは、3キャンペーンでacquisition/mean比1.20–1.51である。一方、fitnessのラウンド推移は両者で近い。つまり不確実性は最高スコアを大幅に押し上げるというより、同程度の高スコアを保ったまま候補集団のcollapseを遅らせる役割を果たした。

### developabilityと探索

上位10%候補について、全method・全標的でGRAVY<0が100%だった。pI 7–9の比率やOASis humannessはCHADが常に最良ではなく、SARSのOASisはOberBO 0.502±0.045、CHAD 0.451±0.032である。CHADは平均CDR diversityでも3標的のどれでも首位ではない。したがって「高fitnessと一般的developabilityを維持した」は支持されるが、「多様性やhumannessまで最良」は支持されない。

## 重要なFigure/Table

- Figure/Table番号: Figure 1 + Table 1
- 何を示しているか: ラウンド別fitness、quality survival、CDR diversity、および3標的の集約結果
- 読み取り方: orangeのFK/CHADが早期ラウンドから高fitness側へ上がる一方、diversityは常に最大ではない。HER2はp(binder)が1.0で飽和する
- 主要な数値・傾向: PT 5.1×、SARS 119.3×、親超え率はPT 91%、SARS 94%、HER2 86%
- なぜ重要か: 論文の主張と、oracle ceiling・diversity trade-offを同時に確認できる
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: 独自の比較チャートと要約表
- 出典URL: [arXiv HTML Figure 1周辺](https://arxiv.org/html/2611.03844v1#S3.SS1)
- ライセンス確認: arXiv non-exclusive distribution license。原図再利用許諾とは扱わず転載しない

- Figure/Table番号: Figure 3
- 何を示しているか: logPI acquisition steeringとposterior mean steeringの候補数・CDR空間被覆の比較
- 読み取り方: 同じfitness thresholdに対して紫のacquisition側が多くの候補を残し、CDR distance thresholdを上げても#Circlesが高い
- 主要な数値・傾向: unique high-scoring variantsは概ね2倍、#Circles AUC比1.20–1.51
- なぜ重要か: CHADでGP不確実性を使う理由を直接検証したablation
- Markdown内での扱い: 数値要約
- HTML内での扱い: 独自の「quality維持・coverage増加」模式図
- 出典URL: [arXiv HTML Figure 3周辺](https://arxiv.org/html/2611.03844v1#S3.SS2)
- ライセンス確認: arXiv non-exclusive distribution license。原図は転載しない

- Figure/Table番号: Figure 4 + Figures 11–12
- 何を示しているか: warm-start fraction、λ、粒子数、再標本化頻度がrewardとCDR diversityへ与える影響
- 読み取り方: warm-startを低くし粒子を増やすと探索が広がり、λと再標本化頻度を上げると高rewardへ集中する
- 主要な数値・傾向: λとwarm-startの組合せでexploration–exploitationが大きく変化する
- なぜ重要か: CHADが単一の「自動引き継ぎ」ではなく、複数の要調整hyperparameterを持つことが分かる
- Markdown内での扱い: 読み解きのみ
- HTML内での扱い: 独自の制御レバー図
- 出典URL: [arXiv HTML Figure 4周辺](https://arxiv.org/html/2611.03844v1#S3.SS3)
- ライセンス確認: arXiv non-exclusive distribution license。原図は転載しない

## Code / License / Weights

- コード公開: CHAD固有コードは見つからず。基盤priorのProteina-Complexaは公開あり
- GitHub URL: [NVIDIA-BioNeMo/Proteina-Complexa](https://github.com/NVIDIA-BioNeMo/Proteina-Complexa)
- GitHub以外のコードURL: [Proteina-Complexa project page](https://research.nvidia.com/labs/genair/proteina-complexa/)
- 実装種別: 関連公式実装（CHADそのものではなく生成prior）
- GitHub確認: 確認済み
- リポジトリのライセンス: Proteina-Complexa codeはApache-2.0。datasetはCC BY 4.0。weightsはNVIDIA Open Model License。CHAD固有実装はライセンス該当なし
- ライセンス確認元: GitHubのLICENSEと `licenses/` 内の個別ライセンス
- Hugging Face URL: 該当なし（CHAD名・論文名・著者でmodel/dataset/spaceを検索したが見つからず）
- Hugging Face種別: 該当なし
- モデル重み・チェックポイント公開: 基盤priorはあり、CHAD固有checkpointは該当なし
- 重み公開URL: [Proteina-Complexa protein-binder checkpoints on NGC](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/clara/models/proteina_complexa)
- データセット公開: 一部あり。SARS-CoV-2 datasetは公開、PT campaignとそのoracle学習データは非公開
- データセットURL: [SARS-CoV-2 antibody binding dataset](https://doi.org/10.1038/s41597-022-01779-4)、[Proteina-Complexa/Teddymer案内](https://github.com/NVIDIA-BioNeMo/Proteina-Complexa#data)
- 再現性メモ: CHADのFK loop、oracle、campaign splits、生成設定をまとめた公式repoは見つからない。Proteina-Complexa本体と重みは利用可能だが、PT oracle、3 campaignの完全な評価環境、CHAD固有コードがないためTable 1のend-to-end再現はできない

## 抗体研究・創薬への意味

CHADの重要点は、de novo modelを「初回候補作成器」で終わらせず、実験ラウンド中もpriorとして残すことである。抗体では1ラウンドの合成・発現・結合測定が高価なため、初期数十件でサロゲートだけへ全面移行するより、構造生成priorと測定不確実性を併用する設計は合理的である。CDRだけを変更しframeworkを固定する実装も、既存leadのaffinity maturationに合う。

また、予測平均ではなくacquisition functionを生成途中のrewardにする考えは、affinityだけでなく、発現量、非特異結合、HIC、熱安定性などへ拡張できる。将来的にPareto-aware acquisitionを使えば、多目的抗体最適化へ接続できる。一方、現在の結果は単一fitness oracleの最適化であり、developabilityは後付けの計算指標で確認しただけである。創薬パイプラインに入れるには、wet-lab multi-objective campaignでの前向き検証が必要である。

## 限界と注意点

### 著者が述べている限界

- campaign由来parentから開始しており、de novo hitから最適化へ渡す本来の設定を再現していない
- 改善値はwet-lab affinityではなく学習済みin silico oracleに対する値で、分布外ではoracle exploitationが起こりうる
- 検証には前向きwet-lab実験が必要
- 使用priorは抗体特異的でなく、代替生成architectureの検討が必要
- 不確実性σがラウンドとともにどう収縮したかを追跡していない

### 精読して気づいた限界

- タイトルの “Lab-in-the-Loop” は想定用途を表し、実験はlabを固定oracleで置き換えた回顧的simulationである
- PTは独自標的・独自データで完全再現不能。距離>9のfilterもこの標的だけに入り、公平性の解釈が難しい
- SARS splitはrandomで、clonal familyやsequence cluster単位の分割ではない。近縁配列間の汎化性能を過大評価する可能性がある
- HER2 oracleはROC-AUC 0.995と高いが、強binderで1.0へ飽和するため、best qualityを識別できない
- λ、粒子数、warm-start、再標本化頻度などの調整余地が大きく、「不確実性だけで自動handoffする」という説明より運用は複雑
- CHAD固有コード、3 oracle、完全な評価データが未公開で、論文数値のend-to-end再現性は低い
- 抗原に対する真の特異性、off-target、発現、凝集、polyreactivityを実測していない

## この論文を読む上での前提知識

- **Bayesian optimization**: 高価な評価関数を少数queryで最大化する枠組み。サロゲートの予測平均と不確実性を使って次の測定点を選ぶ。
- **Gaussian process**: 入力間のkernelから予測分布を作る確率モデル。平均μだけでなく標準偏差σを返せるため、探索と活用の均衡を取れる。
- **acquisition function**: 次に測る価値を表す関数。logPIは「現bestを改善する確率」をlogで表し、CHADでは生成途中の報酬になる。
- **flow matching / diffusion sampling**: noiseから段階的に構造・配列を復元する生成法。CHADは途中状態を複数持ち、良い粒子を残す。
- **Feynman–Kac steering**: 粒子集団を報酬に応じて再標本化し、元の生成分布を目的側へ傾ける推論時制御。
- **CDRとframework**: CDRは主に抗原認識を担う可変loop、frameworkは抗体foldを支える比較的保存された部分。本研究はframeworkを固定する。
- **oracle exploitation**: 予測モデルの誤差を突く配列が高得点になり、実物では改善しない現象。in silico optimizationの中心的リスクである。

## 今回この1報を選んだ理由

前回はPythia-1によるfull-length IgGのde novo設計と実験screenを扱った。今回は、その後段にある「少数の測定値からどう改良ラウンドを回すか」を補完する新着論文を選んだ。2026-11-02公開で新規性が高く、VHH、scFv、Fab/Fvにまたがる3 campaign、比較baseline、ablation、公開された生成priorがあり、方法論を学ぶ価値が高い。iGC、agentic antibody design、multi-objective GFlowNetも有力だったが、CHADは生成priorとBayesian active learningの境界を明確に定式化し、実装上の制御レバーと失敗リスクまで読み取れる点を優先した。

## 読む優先度

**High**。de novo生成とlab-in-the-loop最適化を一体として考える設計図が得られ、低N・不確実性・多様性の関係を具体的に学べる。ただし結果はwet-lab検証ではないため、性能値を実験的改善として引用せず、前向き検証待ちの方法論論文として読むべきである。

## 自分用メモ

- 後で深掘りしたい点: σのround別収縮、λ自動調整、multi-objective EHVI、oracle外挿検出
- 関連して読むべき論文: Proteina-Complexa、Ober et al.の抗体BO、LaMBO、CloneBO、iGC
- 実装を触る場合の入口: Proteina-Complexaのprotein-binder pipelineとFK search実装を確認し、公開SARS datasetで独自loopを再構成する
- Obsidianでリンクしたいキーワード: [[Bayesian optimization]]、[[Feynman–Kac steering]]、[[active learning]]、[[antibody affinity maturation]]、[[oracle exploitation]]

## 関連キーワード

- CHAD
- Proteina-Complexa
- antibody optimization
- lab-in-the-loop
- Bayesian optimization
- Gaussian process
- Feynman–Kac steering
- probability of improvement
- CDR redesign
- exploration–exploitation

## 検索ログ

- 検索したデータベース/クエリ: arXiv、bioRxiv、PubMed、Web検索。`antibody design machine learning November 2026`、`antibody optimization arXiv 2611`、`site:biorxiv.org antibody artificial intelligence`、論文タイトル完全一致
- 候補にした論文: CHAD、iGC、Can agents reliably design therapeutic antibodies?、The Quagmire of Immunogenicity-Guided Antibody Design、Multi-Objective Amortized CDRH3 Sequence Sampling、HCAbLM
- 最終的にこの1報を採用した理由: 前回以降の新着で、生成モデルと反復最適化を接続する明確な方法論、3 campaign、baseline、ablationが揃い、抗体創薬フローへの接続が強い
- 新着論文と基盤論文のバランス: 直近は新着中心だが、今回はProteina-Complexa、BO、FK steeringという基盤技術への導線が多い。HCAbLMは次回候補に残す
- PDF/HTML/Supplementaryを確認できたか: arXiv v1 PDF 12ページとHTMLを確認。本文Figure 1–4、Table 1、Appendix Figure 5–12、Table 2–5を全確認
- Figure/Tableを確認したURLやライセンス確認状況: [arXiv HTML](https://arxiv.org/html/2611.03844v1)、[PDF](https://arxiv.org/pdf/2611.03844)。arXiv non-exclusive distribution licenseで、Figure再利用可能なCC licenseは確認できず、原図は保存・転載しない
- GitHub/コード検索で使ったクエリ: 論文タイトル、`CHAD antibody Parsan Ober McCarter`、著者GitHub、BigHat Biosciences organization、GitHub API repository search
- 確認したGitHub URL: [Proteina-Complexa](https://github.com/NVIDIA-BioNeMo/Proteina-Complexa)、[BigHat-Biosciences organization](https://github.com/BigHat-Biosciences)、[aparsanMIT](https://github.com/aparsanMIT)
- 確認したHugging Face URL: CHAD/論文タイトル/著者でmodels、datasets、spacesをAPI検索。該当公開なし
- 確認したその他コード/重み/データURL: [Proteina-Complexa project](https://research.nvidia.com/labs/genair/proteina-complexa/)、[NGC weights](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/clara/models/proteina_complexa)、[SARS dataset](https://doi.org/10.1038/s41597-022-01779-4)
- リポジトリライセンスの確認元: Proteina-Complexa root `LICENSE` と `licenses/license_code.txt`、`license_datasets.txt`、`license_weights.txt`
- モデル重み・チェックポイントの確認先: Proteina-Complexa README、`complexa download` reference、NVIDIA NGC。CHAD固有checkpointは存在しない
