# A blinded, prospective benchmark of in silico antibody discovery anchored to experimental affinity and developability

## まず何の論文か

この論文は、AIによる抗体発見・抗体最適化の実力を、後ろ向きの既存データ評価ではなく、ブラインドかつ前向きの実験評価で測ったAIntibody Challengeの結果報告である。対象抗原はSARS-CoV-2 RBDで、29組織から提出されたAI設計またはAI選抜抗体511配列を、同一条件でIgGとして発現し、HT-SPR、KinExA、5種類のdevelopability assayで評価している。課題は、既存の選択出力からのin silico affinity maturation、HCDR3クラスタ内での高親和性クローン選抜、選択データに存在しないCDR設計の3つである。結論は、AIは「十分な実験選択データがあり、設計空間が局所的に定義された親和性成熟」では有効な場合がある一方、クラスタ内ランキングやout-of-library設計では性能が不安定で、単純な実験的・統計的ベースラインに負ける場面も多い、というかなり冷静なものだった。特にchallenge 1ではAurekaのAuraBindが95 pM程度の抗体を設計し、親抗体から約2,000倍の親和性改善を示した。反面、challenge 2では多くのAI選抜がランダムピックより悪く、抗体配列の「言語モデル的もっともらしさ」だけでは親和性順位を十分に読めないことが示された。challenge 3ではXencorの2.9 pM抗体がルール上の勝者になったが、HICで溶出しないなど臨床候補としては致命的になりうるdevelopability問題を抱えており、スコア設計そのものの限界も明らかになった。この論文の価値は、特定モデルの性能誇示ではなく、AI抗体設計の成功条件、失敗条件、評価設計上の落とし穴を、実験データ付きで示した点にある。

## 書誌情報

- URL/DOI: https://www.nature.com/articles/s41587-026-03238-6 / https://doi.org/10.1038/s41587-026-03238-6
- PubMed: https://pubmed.ncbi.nlm.nih.gov/42618805/
- 公開日/更新日: 2026-08-19 published online / version of record 2026-08-19
- 著者・所属: M. Frank Erasmus, Daniel Bedinger, Elizabeth Hopkins, Ginger Ferguson, Justine Strickler, Christilyn P. Graff, Samantha R. Summers, Joshua D. Slocum, Jixian Zhang, Shuangjia Zheng, Wei Lu, Gregory L. Moore, Huaiyu Sun, Bryan Briney, Andrew B. Ward, Paolo Marcatili, Jeffrey J. Gray, Andrew R. M. Bradbury ほか。主な所属は Specifica/IQVIA, Carterra, Sapidyne Instruments, Mosaic Biosciences, GENEWIZ/Azenta, Aureka Biotechnologies, Xencor, Scripps Research, UCSD, WashU, Cambridge, BMS, Merck, Incyte, Tencent, Novo Nordisk, Johns Hopkins, UTHealth など。
- 掲載誌/プレプリントサーバー: Nature Biotechnology
- リサーチ日: 2026-09-07
- 分類: 新着 / ベンチマーク・データセット

## 背景と問題設定

抗体設計AIの論文は、既存データを分割して性能を示すものが多い。しかし抗体発見では、真に重要なのは未知配列を実際に作ったときに、結合するか、親和性が上がるか、発現・安定性・非特異結合などのdevelopabilityが壊れないかである。後ろ向き評価では、類似配列のリーク、既知抗原への過剰適合、測定条件のばらつき、企業内データの不可視性によって、モデルの実力を過大評価しやすい。

この論文は、CASPがタンパク質構造予測にもたらしたような、前向き・ブラインド・実験付きベンチマークを抗体発見に持ち込む試みである。AIntibody Challengeでは、参加者に同じ抗体選択データを配り、提出配列を主催側が合成・発現・精製し、同じ測定系で評価する。したがって「どの論文データセットで良かったか」ではなく、「同じ湿実験条件で実際に候補になる抗体を出せたか」を比較できる。

対象抗原はSARS-CoV-2 RBDである。これは抗体構造・配列・親和性データが非常に豊富な抗原なので、著者ら自身も、この結果は任意抗原に対する一般性能ではなく、現在の計算抗体設計能力の上限寄りに解釈すべきだと注意している。それでもなお、多くのモデルが不安定だったことは重要である。AI抗体設計で今必要なのは、華やかな生成例ではなく、どのデータ状況・どの設計タスクなら実験工数を減らせるのかを切り分ける評価であり、この論文はその基準点になる。

## この論文のコアアイデア

コアアイデアは、抗体設計AIを3つの実務的タスクに分け、各タスクで提出配列を実験的に測ることで、AIの使いどころを分解して評価する点にある。challenge 1は、親抗体とCDRサブライブラリ選択出力を与え、HCDR1、HCDR2、LCDR1、LCDR2、LCDR3を組み合わせたり変異させたりして、HCDR3とフレームワークは固定したまま高親和性・developableな抗体を設計する課題である。これは実験パイプラインで言えば、一次選択後に組み合わせライブラリを作って再選択する工程を、計算で置き換えられるかを見る。

challenge 2は、HCDR3でクラスタ化された既存選択出力の中から、各クラスタで最も高親和性かつdevelopableな抗体を選ぶ課題である。参加者には一部の実測親和性も与えられており、配列、NGS abundance、既知親和性の組み合わせから、クラスタ内順位を推定する。これは「既に大量の候補があるとき、どれを作るべきか」というhit triageに近い。

challenge 3は、challenge 2の全選択出力をもとに、データ内に存在しないCDR組み合わせまたは変異配列を作る課題である。完全なde novo抗体設計というより、既存ライブラリの局所近傍やCDR組み合わせ空間を使ったout-of-library設計に近い。各課題の勝者は異なり、Aureka、UCSD/Scripps、Xencor、WashUなどがそれぞれ一部で強かったが、単一モデルが全課題で一貫して強いわけではなかった。ここから、抗体AIは汎用万能モデルというより、タスクと利用可能データに応じたポートフォリオとして使うべきだ、という主張が出てくる。

## 手法の詳細

- 入力データ: SARS-CoV-2 RBDに対する抗体選択出力、CDRサブライブラリのNGS配列、HCDR3クラスタごとの配列、クラスタ内の一部実測親和性、親抗体配列、抗原配列。challenge 1では親抗体以外の親和性は与えられず、challenge 2/3では一部の親和性データが与えられた。
- 出力: 参加者が提出した抗体VH/VL配列。ルール上、課題ごとに許される変異領域が制限され、フレームワーク変異などは失格対象になった。
- モデル/アルゴリズム: 論文全体として単一モデルを提案したのではなく、29組織の多様な手法を比較している。手法クラスは、統計・ヒューリスティック、古典的ML、PLM/attention系、構造認識系に整理された。代表例としてAurekaのAuraBind、ScrippsのAF3 ensemble ssRMSD、UCSDのGaussian process、Xencorのft-ESM embedding + AutoML/attention pooling、WashUのAlignNet/DMS系が説明されている。
- 特徴量・表現学習: AurekaはProtenix由来のpairformer/structure-aware表現とDPOによるfitness rankingを使った。Xencorは650Mパラメータのft-ESM最終層残基embeddingを使い、challenge 2ではmean pooling + sort abundanceをAutoGluonに入力し、challenge 3ではattention poolingで配列全体の重要位置を学習した。UCSDはIMGT整列後のPFASUM-62符号化、abundance/enrichment、Gaussian processを用いた。
- 学習方法: 参加者ごとに異なる。Aurekaは実験的enrichmentの相対順位に合わせるDPO、Xencorは142本の実測VH-VL配列に対する80:20 train-test split、MSE、R2/Pearson/Spearmanで選択、UCSDはGaussian processの周辺尤度最大化を用いた。
- 損失関数・目的関数: Aurekaは絶対KD回帰ではなく、実験enrichment由来の相対fitness順位を学ぶDPO。Xencor challenge 3はMSEで親和性予測を学習し、その予測値をCDR組み合わせ・MCMC/Gibbs探索の目的関数にした。UCSDはGaussian process regressionとsequence likelihood/enrichmentを組み合わせた。
- 推論方法: 参加者は配列候補を生成またはランキングし、少数を提出する。Aurekaは10,000変異体をサンプリングし、AuraBind fitnessで順位付けした。XencorはCDR候補を組み合わせて27Fで129,298、28Fで505,049の新規VH-VLペアを列挙し、さらにMCMC 100,000 stepsやGibbs 20,000 iterationsも使った。
- ベースライン: 親抗体、最頻クローン、実験的FACS/combination library由来抗体、ランダムピック、ProBioGenのCDR consensusのような非AI統計手法。
- 評価指標: SPR KD、ka、kd、KinExA KD、developability composite score。developabilityはHIC-HPLC retention time、BVP polyreactivity、AC-SINS Δλmax、Tm、Taggを0/1/2で評価し、合計score <=3をdevelopable、>3を不合格相当とした。
- 実装上の重要点: 実験はscFv由来データを使いつつ、評価時にはfull-length IgGとして発現・精製している。SPRはCarterra LSA-XT、KinExAはSapidyne、developabilityはMosaic BiosciencesがJain et al.基準に基づく閾値で測定した。これにより、配列ランキングだけでなく、実際の治療抗体候補としての物性が同時に見られる。

## データセットと評価設計

- 使用データセット名: AIntibody Challenge datasets / Supplementary Data 1-3
- データの規模: 29組織、511 AI-designed or predicted antibodies。challenge 1は25組織・165 submissions、challenge 2は各クラスタ26組織で27F 58 submissions、28F 58 submissions、47F 61 submissions、challenge 3は23組織・168 submissions。
- 対象の内訳: SARS-CoV-2 RBD結合抗体。challenge 2のHCDR3クラスタは27F 2,524 unique sequences、28F 3,554 unique sequences、47F 410 unique sequences。
- train/validation/test の分け方: ベンチマーク全体は参加者提出後に実験評価する前向き設計。個別参加者モデルの内部splitは手法ごとに異なる。例としてXencorは142本のpaired experimental affinityを持つVH-VL配列を80:20に分け、5つの親和性quantileでstratifyした。
- リーク対策やクラスタ分割の有無: 提出はブラインド評価だが、対象抗原がSARS-CoV-2 RBDで公開データが非常に豊富なため、著者は「任意抗原への汎化ではなく上限寄りの見積もり」と明示している。challenge 2はHCDR3クラスタで明示的に分けられ、challenge 3はデータ内に存在しない配列が要求されたが、上位設計の多くは実験データ内HCDR3を使っていた。
- 評価指標: HT-SPR、single-point KinExA、standard KinExA、5-assay developability composite score。
- ベースラインや比較対象: 親抗体、cluster control、実験的affinity maturation、best experimental antibody、ランダムピック、CDR consensus。
- この評価設計が妥当かどうか: 前向き・ブラインド・実験付きという点で、現在の抗体AI評価として非常に強い。一方、単一抗原、RBDという過剰に研究された標的、主催者と報告者が同一、challenge 2/3で通常より豊富な親和性情報が与えられている点は、外的妥当性を制限する。developability composite scoreが単一の重大欠陥を相殺できてしまう点も、著者自身が問題として扱っている。

## 主要結果

challenge 1では、AIが最も実用的に機能した。25組織・165 submissionsのうち、118本、つまり71.5%がRBD bindersで、22本がsingle-digit nM、5本がsub-nM、1本がsub-100 pMだった。86.1%がdevelopability基準を通過し、63.0%がbinderかつdevelopableだった。Aurekaの勝者はKinExAで約95 pM、親抗体から約2,000倍改善し、best experimental antibodyの113 pMと統計的に区別できない水準だった。SPRでもtop AI designは340 pMで、best experimental antibodyの517 pMを上回った。これは、一次選択後の局所的な親和性成熟なら、十分に良いAIが実験組み合わせライブラリ工程を2-3週間程度短縮しうることを示す。

ただし、AIだけが勝ったわけではない。ProBioGenの非AI consensus approachは、各CDRライブラリのsort populationで最頻残基を組み合わせた統計的配列で、KinExA 540 pM、developability score 0、全体3位相当だった。これは、NGS選択データの質が高い場合、シンプルな統計処理でも非常に強い候補を出せることを意味する。AIの価値は、単に「MLを使った」ことではなく、統計的シグナルをどれだけうまく構造・相互作用・実験ランキングに結びつけるかにある。

challenge 2は厳しい結果だった。クラスタ27Fでは58 submissions中57本がbinder、14本がsub-100 pMで、UCSDとScrippsが同一9.2 pM抗体を予測し、cluster control 36.6 pMから4.0倍改善した。28FではXencorとWashUが105-106 pM binderで同率勝者になったが、cluster control 177 pMからの改善は1.7倍にとどまった。47FではWashUが50 pM抗体を選び、cluster control 378 pMから7.6倍改善した。しかし全体として、cluster controlを上回る提出は27F 10.3%、28F 13.8%、47F 9.8%で、ランダムに選んだクローンの39%がcluster controlを上回るという比較に対して、多くのAIは明確に不利だった。WashUだけが50%でランダムを上回ったが、それも全クラスタに安定して移るものではなかった。

challenge 3では、168 submissions中117本、69.6%がRBD binders、57本、33.9%がsub-nM、24本、14.3%がsub-100 pMで、数値だけ見れば強い設計も出ている。Xencorの2.9 pM抗体がルール上の勝者で、best experimental antibodyの9.2 pMより高親和性だった。しかしこの抗体はHICカラムから溶出せず、specific activityも低く、positive Hill coefficient 1.41の協同性を示しており、臨床候補としては問題が大きい。次点のdevelopable antibodyは8.69 pMで、best experimental antibody 9.2 pMと統計的に同等だった。さらに上位設計はHCDR3が実験データ内配列と同一で、CDR全体でも近傍の1-5 substitutions程度に寄る傾向があり、真に遠い新規設計というより、選択ライブラリ近傍の保守的改変が成功していた。

## 重要なFigure/Table

- Figure/Table番号: Figure 1
- 何を示しているか: AIntibody Challenge全体の設計、参加組織、提出数、配列重複、SPR/KinExAの相関、developability scoreの閾値をまとめた図。
- 読み取り方: まずa-cで3課題の違いを把握し、j-lでSPRとKinExAが順位評価として十分相関しているかを確認し、m-oでdevelopability判定の意味を読む。SPRとsingle-point KinExAのSpearman rhoは0.94、single-point KinExAとstandard KinExAは0.97、standard KinExAとSPRは0.92で、順位評価の一貫性は高い。
- 主要な数値・傾向: 29組織、511 submissions。developabilityはHIC、BVP、AC-SINS、Tm、Taggの5項目を0/1/2で採点し、合計score <=3がdevelopable。
- なぜ重要か: この論文の主張は、モデルそのものより評価系の信頼性に依存している。Figure 1は、その評価がどの程度統一され、どの基準で合否判定されたかを確認する入口になる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.nature.com/articles/s41587-026-03238-6/figures/1
- ライセンス確認: 確認済み。論文本体はCreative Commons Attribution 4.0 International License。

- Figure/Table番号: Figure 2
- 何を示しているか: challenge 1、つまり一次選択後のin silico affinity maturationの結果。
- 読み取り方: isoaffinity plotでka/kdのどちらが改善しているかを見て、organizationごとのbox plotで、上位組織だけでなく分布全体が実験ベースラインより広く不安定な点を見る。Extended Data Fig. 1と合わせると、Aurekaの勝者が親抗体からCDR Levenshtein distance 19、実験データ最近傍からも12 substitutions離れていることが分かる。
- 主要な数値・傾向: Aurekaの勝者は約95 pM、親抗体から約2,000倍改善。top five AI antibodiesは95-984 pM、top five experimental antibodiesは113-1,230 pM。ProBioGen consensusは540 pMでdevelopability score 0。
- なぜ重要か: AI抗体設計が実験工程を短縮しうる、最もポジティブな証拠である。同時に、非AI consensusがかなり強いという、過剰なAI礼賛を抑える対照にもなっている。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.nature.com/articles/s41587-026-03238-6/figures/2
- ライセンス確認: 確認済み。論文本体はCreative Commons Attribution 4.0 International License。

- Figure/Table番号: Figure 3 / Figure 4
- 何を示しているか: Figure 3はHCDR3クラスタ内ranking、Figure 4はout-of-library CDR designの結果。
- 読み取り方: Figure 3ではcluster controlを上回る割合とランダムピック比較を見る。Figure 4では高親和性だけでなくdevelopability pass/fail、HIC問題、実験データからのLevenshtein distanceを見る。
- 主要な数値・傾向: challenge 2ではcluster controlを上回ったAI submissionsが9.8-13.8%で、ランダムピック39%を下回った。challenge 3では2.9 pMの勝者が出たが、重大なHIC問題を持ち、次点developable antibodyは8.69 pMだった。上位は実験データ内HCDR3に強く寄っていた。
- なぜ重要か: AIの弱点が最も見える図である。特に、ランキングタスクは「binderを見つける」より「既にbindする候補群から最良を選ぶ」方が難しく、developabilityとの同時最適化はさらに難しい。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.nature.com/articles/s41587-026-03238-6/figures/3 / https://www.nature.com/articles/s41587-026-03238-6/figures/4
- ライセンス確認: 確認済み。論文本体はCreative Commons Attribution 4.0 International License。

## Code / License / Weights

- コード公開: あり
- GitHub URL:
  - Aureka challenge 1: https://github.com/AurekaBio/Aureka-AIntibody-Challenges
  - Scripps/Ward-Briney ssRMSD: https://github.com/brineylab/ssrmsd
  - UCSD: https://github.com/Wang-lab-UCSD/aintibody_competition_code
  - UCSD dependency AntPack: https://github.com/Wang-lab-UCSD/AntPack
  - UCSD dependency xGPR: https://github.com/jlparkI/xGPR
  - Xencor challenge 2/3: https://github.com/XenInc/Xencor-AIntibody-Challenges
  - WashU: https://github.com/kaszubat/AIntibody_Competition_WashU_Uploads
- GitHub以外のコードURL: Aureka READMEからZenodo checkpoint URL: https://zenodo.org/records/17784985/files/Covid-design-10.pt?download=1
- 実装種別: 公式 / 著者実装。論文のCode availabilityで各勝者の実装として列挙されている。
- GitHub確認: 確認済み
- リポジトリのライセンス:
  - AurekaBio/Aureka-AIntibody-Challenges: MIT
  - brineylab/ssrmsd: MIT
  - Wang-lab-UCSD/aintibody_competition_code: MIT
  - XenInc/Xencor-AIntibody-Challenges: ライセンス未記載を確認
  - kaszubat/AIntibody_Competition_WashU_Uploads: ライセンス未記載を確認
- ライセンス確認元: GitHubのLICENSEファイル / repository metadata / README。XencorとWashUはGitHubページ上でLICENSE表示が見つからず。
- Hugging Face URL: 見つからず
- Hugging Face種別: 該当なし
- モデル重み・チェックポイント公開: あり
- 重み公開URL:
  - Aureka: https://zenodo.org/records/17784985/files/Covid-design-10.pt?download=1
  - Xencor: https://github.com/XenInc/Xencor-AIntibody-Challenges の `regressor/model_c2_residue_embedding_regression_with_attention_pooling_20250202020607.pth`
  - WashU: https://github.com/kaszubat/AIntibody_Competition_WashU_Uploads の `ModelWeights/`
  - UCSD: https://github.com/Wang-lab-UCSD/aintibody_competition_code の `models/`
- データセット公開: あり
- データセットURL:
  - Nature Supplementary Data 1-3: https://www.nature.com/articles/s41587-026-03238-6#Sec25
  - Xencor repository data: https://github.com/XenInc/Xencor-AIntibody-Challenges/tree/main/data
  - UCSD repository datasets: https://github.com/Wang-lab-UCSD/aintibody_competition_code/tree/main/datasets
- 再現性メモ: 論文はSupplementary DataとしてSPR kinetic parameters、KinExA、developability scores、challenge input sequencing datasets、sequence-to-result mappingを機械可読形式で提供している。AurekaはREADMEにインストール、checkpoint取得、`predict.sh`実行、上位10配列抽出までを書いている。Xencorはコード、データ、regressor weightを含むがライセンス未記載なので再利用時は注意が必要。Hugging Face上の公式モデル・datasetは検索で見つからなかった。

## 抗体研究・創薬への意味

この論文は、AI抗体設計の「どこで使えるか」を現実的に切り分ける点で重要である。challenge 1の結果は、既に親抗体と良質なCDR選択データがある場合、AIが次の実験ライブラリ作成・ソーティング工程を短縮し、少数合成で高親和性候補に到達できる可能性を示した。これは抗体医薬のlead optimization、variant prioritization、感染症抗体の迅速改変に直接効く。

一方、challenge 2の失敗は、抗体配列データが大量にあっても、クラスタ内の親和性順位を読むことはかなり難しいことを示す。抗体探索では「binderかどうか」よりも、既に結合する候補の中から最も良いKD、低いoff-rate、良いdevelopabilityを持つものを選ぶ場面が多い。このタスクでランダムピックに負けるなら、in silico triageを信じて実験数を大きく削るのは危険である。

challenge 3は、生成モデルやPLMの使い方に関する示唆が大きい。成功した上位配列は、完全に新しいHCDR3を作るよりも、実験データ内のHCDR3を保持し、周辺CDRを少数置換する保守的設計に寄っていた。これは、抗体の結合モードではHCDR3が支配的で、データから遠く離れた生成は親和性やdevelopabilityを壊しやすいことを示唆する。抗体創薬でAIを使うなら、「遠くへ生成する」より「実験データ近傍で多目的最適化する」方が現時点では堅い戦略に見える。

## 限界と注意点

著者が述べている限界:

- 初回AIntibody Challengeは単一抗原、SARS-CoV-2 RBDだけを対象にしている。RBDは世界で最も研究された抗原の1つで、公開データや構造情報が豊富なので、任意抗原への一般化を示すものではない。
- challenge 2/3では、通常の発見パイプラインでは後段まで得られないような豊富な親和性情報が与えられており、現在のAI性能に有利な条件だった。
- 主催コンソーシアムが結果論文も報告しており、CASPのような完全に成熟した独立運営・情報遮断設計ではない。
- developability composite scoreは、単一の深刻な物性欠陥を他の良好な項目で相殺できてしまう。challenge 3勝者のHIC問題はこの限界を露呈した。

読んで気づいた限界:

- 各組織の提出数、人的レビュー、社内データ、計算資源、手法の成熟度が異なるため、method class間の比較は厳密なablationではない。
- 一部の勝者実装は公開されたが、全参加者の完全なコードや社内学習データは公開されていない。特に企業モデルでは、性能の源泉が公開データなのか社内データなのかを切り分けにくい。
- 抗原がRBDなので、抗体設計で難しい膜タンパク質、低免疫原性抗原、糖鎖エピトープ、コンフォメーション依存エピトープ、多特異性設計への外挿は慎重にすべき。
- HCDR3やCDR近傍に寄る保守的設計が成功しているため、真のde novo paratope design能力を測るには、より遠い配列距離や未知抗原での評価が必要。
- ライセンス面では、論文本体と図はCC BY 4.0だが、公開GitHubの一部にはLICENSEが見当たらない。商用・再配布・改変利用では個別確認が必要である。

## この論文を読む上での前提知識

- SPR: Surface Plasmon Resonanceの略で、抗体と抗原の結合・解離をリアルタイムで測り、ka、kd、KDを推定する。抗体最適化では単なる結合有無ではなく、off-rateの低下が重要になる。
- KinExA: Solution equilibrium affinityを測る手法で、非常に高親和性の抗体でSPRの限界や表面固定化影響を補う目的で使われる。今回の論文では上位候補の順位確認に使われた。
- Developability: 抗体が医薬品候補として製造・保存・投与可能かに関わる物性群。熱安定性、凝集、疎水性、非特異結合、自己相互作用などが含まれる。
- CDRとHCDR3: CDRは抗体可変領域で抗原認識に強く関わるループ領域。特にHCDR3は再構成で多様性が高く、エピトープ認識と物性の両方に大きく影響する。
- NGS selection output: display libraryなどを抗原で選択した後、濃縮された配列を次世代シーケンスで読むデータ。頻度や濃縮は結合のヒントになるが、親和性そのものではない。
- Protein Language Model: アミノ酸配列を大規模に学習し、残基や配列全体の表現を得るモデル。抗体では一般タンパク質PLMだけでなく、抗体ペア配列にfine-tuneしたモデルが使われる。
- Levenshtein distance: 配列間の編集距離。今回の論文では、設計配列が親抗体や提供データ内配列からどれだけ離れているかを測るために使われた。

## 今日この1報を選んだ理由

前回はCLDN18.2抗体設計のPLOS論文を扱ったため、今回は個別モデル論文ではなく、分野全体の現在地を測るベンチマーク論文を選んだ。2026-08-19公開のNature Biotechnology論文で新着性が高く、AIntibody Challengeという前向き・ブラインド・実験付き評価は、AI抗体設計分野で基準点になりやすい。特に、Aurekaの成功、ProBioGen consensusの強さ、challenge 2でAIがランダムピックに負けた事実、challenge 3勝者のdevelopability問題は、今後の論文を読むときの評価軸として有用である。コードと補足データも複数公開されており、実装・重み・データ可用性まで確認する価値がある。レビュー論文や企業プレスリリースより、一次論文としての学習密度が高い。

## 読む優先度

High。

AI×抗体設計を追うなら、個別生成モデルの精度主張より先に読む価値がある。成功例だけでなく失敗例も実験的に示しており、今後の論文を評価するときに「そのタスクはAIntibodyのどの課題に近いか」「ランダムやconsensusに勝っているか」「developabilityの単一欠陥を見落としていないか」という基準を持てる。

## 自分用メモ

- 後で深掘りしたい点: Supplementary Data 1-3を実際に読み、各submissionのKD、developability score、CDR距離、method classを再解析する。
- 関連して読むべき論文: AIntibody challenge announcement, RESP AI model, Jain et al. developability panel, Protenix/OpenDDE, ft-ESM paired antibody model, AlphaFold3-based antibody-antigen structure scoring。
- 実装を触る場合の入口: まずAurekaBio/Aureka-AIntibody-Challengesでcheckpointを取得し、`predict.sh`の入力形式と出力`predict.pkl`を確認する。次にXencor repoでft-ESM embedding + attention pooling regressorを読む。
- Obsidianでリンクしたいキーワード: [[AIntibody Challenge]], [[抗体developability]], [[抗体親和性成熟]], [[HCDR3]], [[KinExA]], [[SPR]], [[protein language model]], [[CDR design]], [[AuraBind]], [[ft-ESM]]

## 関連キーワード

- antibody design
- affinity maturation
- developability
- prospective benchmark
- blinded benchmark
- SARS-CoV-2 RBD
- HCDR3 cluster
- protein language model
- structure-aware design
- KinExA
- HT-SPR
- AIntibody Challenge

## 検索ログ

- 検索したデータベース/クエリ:
  - Web検索: `2026 antibody artificial intelligence machine learning antibody design bioRxiv August September 2026`
  - Web検索: `site:biorxiv.org antibody design artificial intelligence 2026 protein language model antibody`
  - Web検索: `site:arxiv.org antibody design protein language model 2026`
  - Web検索: `"Dual-Specific Antibody Design Using Artificial Intelligence"`
  - Web検索: `"A blinded, prospective benchmark of in silico antibody discovery anchored to experimental affinity and developability"`
  - Web検索: `"10.1038/s41587-026-03238-6"`
  - Web検索: `"AIntibody Challenge" "Nature Biotechnology" GitHub`
  - Hugging Face検索: `site:huggingface.co AIntibody Challenge antibody`, `site:huggingface.co Aureka AIntibody`, `site:huggingface.co Xencor AIntibody`
- 候補にした論文:
  - Dual-Specific Antibody Design Using Artificial Intelligence, bioRxiv, 2026-08-05
  - The Evolution of Artificial Intelligence in Antibody Design: From Structure-Based Engineering to Generative Models, Antibodies, 2026-09-02
  - IgGM2: An All-Atom Foundation Model for Adaptive Immune Receptor Design, bioRxiv, 2026-07
  - A blinded, prospective benchmark of in silico antibody discovery anchored to experimental affinity and developability, Nature Biotechnology, 2026-08-19
- 最終的にこの1報を採用した理由: 直近公開で、AI抗体設計の前向き・ブラインド・実験付きベンチマークとして分野全体への影響が大きい。個別手法より、今後の論文評価の基準になる。
- 新着論文と基盤論文のバランスをどう考えたか: 前回も新着寄りだったが、今回は単なる新着モデルではなく、ベンチマーク・データセットとして基盤性も持つ新着論文を選んだ。
- PDF/HTML/Supplementaryを確認できたか: Nature HTML、PubMed、Supplementary Information、Supplementary Tables/Dataの存在を確認。Supplementary Data 1-3は機械可読Excelとして提供され、SPR/KinExA/developability/sequence mappingを含むことを確認。
- Figure/Tableを確認したURLやライセンス確認状況:
  - Figure 1: https://www.nature.com/articles/s41587-026-03238-6/figures/1
  - Figure 2: https://www.nature.com/articles/s41587-026-03238-6/figures/2
  - Figure 3: https://www.nature.com/articles/s41587-026-03238-6/figures/3
  - Figure 4: https://www.nature.com/articles/s41587-026-03238-6/figures/4
  - Rights and permissionsでCreative Commons Attribution 4.0 International Licenseを確認。
- GitHub/コード検索で使ったクエリ:
  - `"AIntibody Challenge" "GitHub"`
  - `"Aureka-AIntibody-Challenges"`
  - `"Xencor-AIntibody-Challenges"`
  - `"AIntibody_Competition_WashU_Uploads"`
  - `"aintibody_competition_code"`
- 確認したGitHub URL:
  - https://github.com/AurekaBio/Aureka-AIntibody-Challenges
  - https://github.com/brineylab/ssrmsd
  - https://github.com/Wang-lab-UCSD/aintibody_competition_code
  - https://github.com/Wang-lab-UCSD/AntPack
  - https://github.com/jlparkI/xGPR
  - https://github.com/XenInc/Xencor-AIntibody-Challenges
  - https://github.com/kaszubat/AIntibody_Competition_WashU_Uploads
- 確認したHugging Face URL: 公式AIntibody/Aureka/Xencor/WashU関連のHF model/datasetは見つからず。検索結果にはGinkgo AbDevやSEPIQなど別ベンチマークが出たが、本論文の公式公開物ではない。
- 確認したその他コード/重み/データURL:
  - Aureka checkpoint: https://zenodo.org/records/17784985/files/Covid-design-10.pt?download=1
  - Nature Supplementary: https://www.nature.com/articles/s41587-026-03238-6#Sec25
- リポジトリライセンスの確認元:
  - Aureka: GitHub LICENSE file, MIT
  - brineylab/ssrmsd: GitHub LICENSE/README, MIT
  - UCSD aintibody_competition_code: repository metadata, MIT
  - Xencor: GitHub file list/READMEでLICENSEなし
  - WashU: GitHub file listでLICENSEなし
- モデル重み・チェックポイントの確認先:
  - Aureka READMEのZenodo `Covid-design-10.pt`
  - Xencor GitHub `regressor/*.pth`
  - WashU GitHub `ModelWeights/`
  - UCSD GitHub `models/`
