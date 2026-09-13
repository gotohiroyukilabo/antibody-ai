# De novo designed single-domain antibodies protect against lethal cobra venom neurotoxicity in vivo

> HTMLビジュアル版: [同名HTMLを開く](20260914_De%20novo%20designed%20single-domain%20antibodies%20protect%20against%20lethal%20cobra%20venom%20neurotoxicity%20in%20vivo.html)

## まず何の論文か

この論文は、生成AIで設計した単一ドメイン抗体（VHH）が、実験室内で標的に結合するだけでなく、生体内で致死性毒素を中和できるかを検証した研究である。標的はタイコブラ（*Naja kaouthia*）の長鎖型α神経毒α-cobratoxinで、神経筋接合部のニコチン性アセチルコリン受容体（nAChR）を阻害する。著者らはまずGerminal、RFantibody、BoltzGenの3つのVHH設計法を、同じ3種のフレームワーク、同じ標的構造、同じ5つのホットスポット残基、同じAlphaFold 3（AF3）評価で正面比較した。生のipTM中央値は近かったが、設計構造と再予測構造のRMSD < 6 Åを併用すると、Germinalは300件中46件、RFantibodyは900件中6件、BoltzGenは900件中12件が残り、差が明瞭になった。そこでGerminalで約8,000件を生成し、内部フィルターを通過した約3%から48配列を選択した。クローニングに成功した46件中42件が可溶性発現し、FIDAで4件（8.7%）が結合ヒットとなった。強い2候補D2、E2の結合親和性はそれぞれ4.1 nM、10.8 nMで、主な差は解離速度にあった。筋型nAChRのパッチクランプではIC50が4.2 nM、12.9 nMで、結合親和性の差が機能中和にも反映された。マウスに3×LD50の精製α-cobratoxinを投与し、15分後にVHHを静注する救済試験では、両候補が5/5匹を24時間生存させた。全毒液ではD2が5/9匹（56%）、E2が2/9匹（22%）を生存させ、D2のみプラセボとの差が24時間生存率で有意だった。AI設計VHHの実用性を「計算スコア→発現→結合→物性→細胞機能→生体防御」まで一気通貫で示した点が、この論文の最大の価値である。

## 書誌情報

- URL/DOI: [bioRxiv本文](https://www.biorxiv.org/content/10.64898/2026.09.01.748349v1) / [10.64898/2026.09.01.748349](https://doi.org/10.64898/2026.09.01.748349)
- 公開日/更新日: 原稿日 2026-09-01、bioRxiv掲載 2026-09-03（v1）
- 著者・所属: Max D. Overath、Emil V. S. Lundquist、Kasper H. Björnssonほか。Technical University of Denmark、Center for Translational Protein Design、AffinityAI、University of Northern Colorado、Sophion Bioscience、University of Copenhagen/NIVI
- 掲載誌/プレプリントサーバー: bioRxiv（査読前プレプリント）
- リサーチ日: 2026-09-14（JST）
- 分類: 新着 / 抗体設計 / 実験検証 / ベンチマーク的比較
- 論文ライセンス: CC BY-NC-ND 4.0

## 背景と問題設定

RFdiffusionやBindCraftなどにより、新規タンパク質バインダーは標的構造から直接設計できるようになった。しかし成功例の多くは自然界にない小型スキャフォールドであり、免疫原性、薬物動態、製造、規制上の経験が限られる。抗体は臨床実績が豊富だが、可動性の高いCDRを所望のエピトープへ向けつつ、配列・構造・物性を同時に成立させる必要があり、de novo設計は難しい。

近年、Germinal、RFantibody、BoltzGenはいずれもVHH設計を扱えるようになったが、アーキテクチャ、サンプリング速度、配列設計器、フィルターが異なる。共通標的・共通入力・共通評価器による比較が乏しく、どの計算スコアを実験候補選抜に使うべきかも定まっていない。さらに既報のde novo VHHは主としてin vitro検証までで、治療に近い「毒素曝露後の生体内救済」まで到達していなかった。

α-cobratoxinはnAChR結合面が構造的に明確で、標的エピトープを指定しやすい。一方、従来型抗毒素では十分中和されにくい毒素でもある。このため、設計手法の比較と機能実証を同時に行うモデル標的として適している。

## この論文のコアアイデア

コアは、生成法の出力をそのモデル固有の内部スコアだけで比べず、外部の共通構造予測器で再予測し、「高い界面信頼度」と「意図した結合様式の再現性」を同時に要求することにある。単にAF3 ipTMが高いだけでは、設計時とは異なるドッキング姿勢でも高得点になりうる。そこで標的鎖を重ねた後のVHH Cα RMSDをself-consistency指標とし、RMSD < 6 Åを追加した。

比較後は最も通過率の高いGerminalに集中した。GerminalはColabDesign由来のhallucinationで標的ホットスポットに合うCDR/骨格配置を探索し、抗体言語モデルで抗体らしい配列へ誘導し、AbMPNNで配列を再設計し、構造再予測と多段フィルターで候補を絞る。設計の最終出力は、指定エピトープを狙うVHH配列と予測複合体構造である。

## 手法の詳細

- 入力データ: α-cobratoxin構造、ホットスポットD27/R33/K35/R36/V37、VHHフレームワーク3種（PDB 9GCN、3EAK、7XL0）、各親フレームワークと同じCDR長。
- 出力: VHH配列、VHH–毒素予測複合体、界面・構造・developabilityスコア。
- モデル/アルゴリズム: Germinal（hallucination + 抗体言語モデル + AbMPNN + 構造再予測）、RFantibody（抗体fine-tuned RFdiffusion + ProteinMPNN + RF2）、BoltzGen（拡散生成 + BoltzGen-IF）。共通評価はAF3 v3.0.1、外部検証はESMFold2 v3.4.0。
- 特徴量・表現学習: AF3/ESMFold2のipTM、ipSAE_min、予測複合体座標、CDR3編集距離、SAbDab/OAS/INDIへの配列同一性、CDR3幾何分類、TNP物性指標。
- 学習方法: 本研究では新規学習なし。公開済み学習済み設計モデルを推論に使用。
- 損失関数・目的関数: 各基盤モデル固有。Germinalは構造信頼度・界面・抗体言語モデルなどを組み合わせたhallucination目的を使うが、詳細な重みは論文本文では未確認。
- 推論方法: 共通比較ではGerminal 100件/フレームワーク、RFantibodyとBoltzGen 300件/フレームワーク。AF3は1 seed × 5 diffusion models、標的のみMSA、上位モデルを採用。計算時間はNVIDIA L40Sで測定。
- ベースライン: 3手法の相互比較。実験段階では毒素のみ、PBSプラセボ、VHH単独群。
- 評価指標: ipTM、標的整列後VHH Cα RMSD、ipSAE_min、通過候補1件あたり計算時間、CDR3同一性、可溶性発現率、FIDA結合、BLI KD/kon/koff、SEC単量体率、CD、polyreactivity、パッチクランプIC50、24時間生存率、臨床重症度、Fisher正確確率検定、log-rank検定。
- 実装上の重要点: 比較段階ではGerminalの内部フィルターによる有利さを避けるため、成功/失敗trajectoryを問わずAbMPNN前の全出力を評価した。大規模設計ではGerminal標準フィルターを使いつつ、CDRが狙うホットスポット最小数を3から2へ緩和した。

## データセットと評価設計

固定データセットをtrain/validation/testへ分割する研究ではなく、前向きの設計キャンペーンである。設計対象は1毒素、フレームワークは3種。共通比較の生成数はGerminal 300件、RFantibody 900件、BoltzGen 900件で、後二者を多くしたのは生成スループットが高いためである。構造リークの観点では、3EAKは各設計法・評価器の学習データに含まれ、9GCNは全てで未収録、7XL0はRFantibody/BoltzGenには含まれるがGerminal/AF3/ESMFold2には含まれないとTable S1で整理されている。

大規模Germinalキャンペーンでは約8,000設計の約3%が内部フィルターを通過し、48件（3EAK 8、9GCN 20、7XL0 20）を合成対象とした。2件がクローニングで失敗し、46件を発現・結合評価した。ランキング評価は真の結合ヒット4件が46件中どこに来るかを事後評価し、AF3 ipSAE_minの平均順位5.75はAF3 ipTMの12.0より良かった。

評価設計は、同一入力・同一外部評価器・同一GPUでモデル差を切り分けた点、さらに計算指標をwet-labとin vivoへ接続した点が強い。一方、モデル間比較は1標的のみで、RFantibody/BoltzGen候補を実験しなかったため、最終的な「どのモデルが実験成功率で優れるか」は証明していない。

## 主要結果

| 段階 | D2 / Germinal | E2 / Germinal | 比較・解釈 |
|---|---:|---:|---|
| 共通比較でRMSD < 6 Å通過 | Germinal 46/300 | — | RFantibody 6/900、BoltzGen 12/900 |
| 可溶性発現 | 合計42/46（91%） | — | 3EAK 8/8、9GCN 19/20、7XL0 15/18 |
| FIDA結合ヒット | 合計4/46（8.7%） | — | 9GCN由来2、7XL0由来2、3EAK由来0 |
| SEC単量体率 | 80.7% | 91.8% | E2が良好、D2には改善余地 |
| BLI KD | 4.09 ± 0.08 nM | 10.79 ± 0.11 nM | D2が約2.6倍強い |
| koff | 1.75×10^-4 s^-1 | 4.33×10^-4 s^-1 | E2が約2.5倍速く解離 |
| パッチクランプIC50 | 4.2 nM | 12.9 nM | D2は90%中和に約11倍量、E2は約188倍量 |
| 精製毒素後の生存 | 5/5（100%） | 5/5（100%） | placebo 0/5、各p=0.0079（Fisher） |
| 全毒液後の生存 | 5/9（56%） | 2/9（22%） | D2 vs placebo p=0.029、E2はp=0.47 |

3手法の全設計におけるipTM中央値はGerminal 0.26、BoltzGen 0.24、RFantibody 0.23と近い。しかしself-consistency通過後はGerminal 46件・中央値0.61、RFantibody 6件・0.26、BoltzGen 12件・0.31となった。したがって「見かけ上の界面スコア」ではなく、再予測時に結合姿勢を維持するかがモデル比較を分けた。

配列面ではSAbDab最近傍に対するCDR3同一性中央値がGerminal 0.33、RFantibody 0.35、BoltzGen 0.44で、Germinalが最も新規性の高い領域を探索した。D2/E2は9GCNフレームワーク由来だが、D2のCDR2で同一なのは1残基、E2は3残基で、その他のCDR残基は異なる。両者は同じGerminal trajectoryから得たAbMPNN variantで、予測パラトープ19残基は同一、うちCDR3が10、フレームワークが6残基を担う。

## 重要なFigure/Table

- Figure番号: Figure 1
- 何を示しているか: 3手法×3フレームワークの比較、全体ワークフロー、ipTMとRMSD self-consistency、成功候補あたり計算時間、配列新規性、CDR3構造分布。
- 読み取り方: ipTM単独よりRMSD条件併用の列を見る。Germinalは生の速度は遅いが、通過候補1件あたりでは差が縮む。
- 主要な数値・傾向: RMSD < 6 Åで46/300対6/900対12/900。CDR3同一性中央値0.33対0.35対0.44。
- なぜ重要か: 生成速度ではなく「実験へ送れる設計1件あたりのコスト」で手法を比較すべきことを示す。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: 独自のフロー図・比較チャート
- 出典URL: [bioRxiv Figure 1を含む本文](https://www.biorxiv.org/content/10.64898/2026.09.01.748349v1.full)
- ライセンス確認: CC BY-NC-ND 4.0確認済み。NDのため原図は転載しない。

- Figure番号: Figure 2 / Table S3
- 何を示しているか: 46設計の発現・結合スクリーニング、予測スコアのヒット順位、CDR3形状、developability、D2/E2の速度論。
- 読み取り方: 設計ファネルと、ipTMよりipSAE_minが真のヒットを上位化する点を見る。
- 主要な数値・傾向: 42/46発現、4/46結合、D2/E2はAF3/ESMFold2 ipSAE_minで上位4位以内。KD 4.09/10.79 nM。
- なぜ重要か: 計算スコアから実験成功へ移る際のボトルネックが定量化されている。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: 独自のファネル図・速度論カード
- 出典URL: [bioRxiv本文・Supplementary](https://www.biorxiv.org/content/10.64898/2026.09.01.748349v1.supplementary-material)
- ライセンス確認: CC BY-NC-ND 4.0確認済み。原図は転載しない。

- Figure番号: Figure 4
- 何を示しているか: D2/E2のnAChR機能中和と、精製毒素・全毒液を用いたマウス救済試験。
- 読み取り方: 15分遅延投与という条件、精製毒素と全毒液の難易度差、群サイズを併せて読む。
- 主要な数値・傾向: 精製毒素は両群5/5生存、全毒液はD2 5/9、E2 2/9。D2とE2の直接差は非有意（log-rank p=0.11）。
- なぜ重要か: 「結合するAI設計抗体」から「曝露後に生体を救う候補」への橋を示す中心結果。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: 独自の生存率比較チャート
- 出典URL: [bioRxiv Figure 4を含む本文](https://www.biorxiv.org/content/10.64898/2026.09.01.748349v1.full)
- ライセンス確認: CC BY-NC-ND 4.0確認済み。原図は転載しない。

## Code / License / Weights

- コード公開: あり（使用した3設計法）。本研究専用解析スクリプトは見つからず。
- GitHub URL: [Germinal](https://github.com/SantiagoMille/germinal) / [RFantibody](https://github.com/RosettaCommons/RFantibody) / [BoltzGen](https://github.com/HannesStark/boltzgen)
- GitHub以外のコードURL: 該当なし
- 実装種別: 公式・著者実装（各基盤手法）
- GitHub確認: 確認済み。論文指定version/commitはGerminal `88d7f85`、RFantibody `8d9d402`、BoltzGen v0.2.0。
- リポジトリのライセンス: Germinal Apache-2.0、RFantibody MIT、BoltzGen MIT。ただしGerminalのIgLMとPyRosettaは別ライセンスで非商用/学術利用制約があり、AlphaFold 3 weightsにも別条件がある。
- ライセンス確認元: 各GitHub LICENSE、README、repository metadata。
- Hugging Face URL: [BoltzGen model](https://huggingface.co/boltzgen/boltzgen-1) / [BoltzGen training dataset](https://huggingface.co/datasets/boltzgen/boltzgen1_train)
- Hugging Face種別: model / dataset
- モデル重み・チェックポイント公開: あり（RFantibody、BoltzGen）。Germinal独自の単一checkpointは該当なしで、ColabDesign/AF、抗体LM、AbMPNN等の外部パラメータを組み合わせる。
- 重み公開URL: [RFantibody download instructions](https://github.com/RosettaCommons/RFantibody#3-download-model-weights) / [BoltzGen model](https://huggingface.co/boltzgen/boltzgen-1)
- データセット公開: あり。本研究の48設計の配列、発現、結合、developability、物性、全AF3予測はbioRxiv Supplementary Materialに収録とData Availabilityに明記。
- データセットURL: [bioRxiv Supplementary Material](https://www.biorxiv.org/content/10.64898/2026.09.01.748349v1.supplementary-material)
- 再現性メモ: 入力構造、ホットスポット、モデルversion、GPU、AF3 seed/model数、フィルター閾値、実験条件まで比較的詳細。ただし約8,000件を生成した完全なラン設定、本研究専用解析コード、全環境lockfileは論文側公開を確認できず、AF3・PyRosetta等の取得条件と40 GB級GPUが再現の障壁になる。

## 抗体研究・創薬への意味

第一に、構造生成モデルの評価をAARやipTMだけで終わらせず、発現・結合・機能・生体防御までつなぐ評価系を提示した。抗体創薬では高スコア候補数より、候補1件を実験へ送るまでの実効コストと、下流で脱落しない多目的品質が重要である。

第二に、VHHフレームワーク選択が生成モデルと同程度に重要であることを示す。3EAK由来8件は結合せず、extended CDR3を持つ9GCN/7XL0からのみヒットした。固定フレームワーク1種で大量生成するより、複数のCDR3幾何を持つフレームワークを並行評価する方が合理的である。

第三に、D2/E2のわずかなAbMPNN配列差がkoffと中和能を変えたことは、初回de novo hitを終点ではなく、親和性成熟の出発点として扱うべきことを示す。抗体AIの実務では、生成→外部再予測→低コスト結合screen→速度論→機能assayという閉ループに接続しやすい。

## 限界と注意点

著者が述べる限界:

- 標的はα-cobratoxin 1種、フレームワークは3種だけで、モデル優劣を一般化できない。
- RFantibody/BoltzGenは共通計算フィルターを通過しにくく、実験比較はGerminalだけである。
- 9GCNはヒト化・developability最適化済みではなく、免疫原性評価が必要。
- D2/E2の予測パラトープ19残基中6残基がframeworkにあり、ヒト化で親和性を損なう可能性がある。
- 全毒液中のα-cobratoxin量は32.3%という推定値で投与量を決めており、実際のモル過剰量には不確実性がある。

追加で注意すべき点:

- preprintで未査読。in vivo群は精製毒素n=5、全毒液n=9と小さい。
- D2対E2の全毒液生存差は統計的に有意ではなく、56%対22%を確定的な優劣と解釈できない。
- AF3とESMFold2の全設計ipTM相関はr=0.10と低く、self-consistent subsetでのみr=0.77（n=21）。予測器の合意自体が候補選抜を強く左右する。
- D2の単量体率80.7%と軽度polyreactivity signalは開発上の改善点。急性忍容性は長期毒性・免疫原性を保証しない。
- 論文ライセンスはCC BY-NC-ND 4.0、Germinalは外部依存に非商用ライセンスを含む。商用利用ではコード本体のOSS表記だけで判断できない。

## この論文を読む上での前提知識

- **VHH / nanobody**: ラクダ科重鎖抗体の可変ドメインに由来する単一ドメイン抗体。小型で発現しやすいが、framework残基が抗原接触に関与しやすい。
- **CDRとframework**: CDRは主に抗原認識を担う可変ループ、frameworkはVHH骨格を支える領域。VHHではCDR3が長く、側面結合やframework接触も多い。
- **ipTM / ipSAE**: 複合体界面の予測信頼度指標。高値・良順位でも実結合を保証せず、異なる指標やself-consistencyを組み合わせる必要がある。
- **RMSD self-consistency**: 生成時の結合姿勢が、配列からの再予測でも再現されるかを測る。標的を重ねてbinderのずれを測ることで、姿勢崩壊を検出する。
- **BLIのKD、kon、koff**: 結合平衡と会合・解離速度を測る。D2/E2ではkonは近く、koffの差がKD差をほぼ説明した。
- **パッチクランプ**: 細胞膜電流を測り、VHHが毒素によるnAChR阻害をどこまで回復するかを直接評価する。
- **LD50とpost-exposure rescue**: LD50は半数致死量。今回は3×LD50投与後15分でVHHを与えるため、予防投与より厳しい治療模擬条件である。

## 今回この1報を選んだ理由

2026-09-03公開の新着であり、3つの主要な抗体生成パイプラインを共通条件で比較している。さらに、46設計の実験screen、低nM親和性、developability、細胞機能、マウス救済まで一気通貫で示し、Figure 1〜4とSupplementary Tablesから設計実務を学べる。他候補のDual-Specific Antibody Design Using Artificial Intelligenceは臨床展開が興味深い一方、設計アルゴリズムの詳細と公開実装が限定的である。Frozen Protein Foundation-Model Embeddingsは明快だが内部モデル・独自データに依存する。本論文は新着性、公開実装への接続、評価設計、実験的説得力のバランスが最も高い。

## 読む優先度

**High**。抗体生成モデルを「どのスコアで選ぶか」「フレームワークをどう扱うか」「実験ファネルでどこまで残るか」を具体的な数値で学べる。in vivo成功まで示す一方、単一標的・小規模群・モデル間実験比較なしという限界も明瞭で、現在地を過大評価せず理解できる。

## 自分用メモ

- 後で深掘りしたい点: D2/E2の実験構造、framework接触L52Rの寄与、ヒト化後のKD/単量体率、全毒液cocktail化。
- 関連して読むべき論文: Germinal原著、RFantibody原著、BoltzGen、2025 Nature「De novo designed proteins neutralize lethal snake venom toxins」、2025 Natureのnanobody cocktail antivenom。
- 実装を触る場合の入口: まずGerminalで公開例を再現し、Table S2の最終フィルターを固定してAF3/ipSAE_minとself-consistencyのランキングを比較する。
- Obsidianでリンクしたいキーワード: [[Germinal]]、[[RFantibody]]、[[BoltzGen]]、[[VHH]]、[[AlphaFold3]]、[[ipSAE]]、[[抗毒素]]、[[CDR3 conformation]]

## 関連キーワード

- de novo antibody design
- VHH / nanobody
- α-cobratoxin / long-chain α-neurotoxin
- Germinal / RFantibody / BoltzGen
- AlphaFold 3 / ESMFold2
- self-consistency / ipTM / ipSAE_min
- recombinant antivenom
- in vivo neutralization

## 検索ログ

- 検索したデータベース/クエリ: bioRxiv、arXiv、PubMed相当、Google検索。`antibody AI machine learning September 2026`、`site:biorxiv.org antibody design September 2026`、`IgGM2 antibody 2026`、`Dual-Specific Antibody Design Using Artificial Intelligence`、`Frozen Protein Foundation-Model Embeddings antibody antigen`。
- 候補にした論文: 本論文、Dual-Specific Antibody Design Using Artificial Intelligence、IgGM2、Frozen Protein Foundation-Model Embeddings Improve Antibody–Antigen Binding Affinity Prediction。
- 最終的にこの1報を採用した理由: 直近新着で、共通条件のモデル比較と強い前向き実験検証、さらに曝露後in vivo救済を両立したため。
- 新着論文と基盤論文のバランス: 今回は新着を優先。Germinal/RFantibody/BoltzGenという基盤手法を比較するため、基盤知識も同時に補える。
- PDF/HTML/Supplementaryを確認できたか: 27ページPDF本文・本文内Supplementary Tables/Figuresを確認。bioRxiv HTMLは検索取得、Supplementary Materialページはリンク確認。
- Figure/Tableを確認したURLやライセンス確認状況: bioRxiv本文のFigure 1〜4、Table 1、Table S1〜S3、Fig. S1〜S12を確認。各PDFページにCC BY-NC-ND 4.0表示あり。NDのため原図は保存・転載せず独自要約図を作成。
- GitHub/コード検索で使ったクエリ: 論文PDF埋め込みURL、各手法名 + GitHub + LICENSE + weights + Hugging Face。
- 確認したGitHub URL: [Germinal](https://github.com/SantiagoMille/germinal)、[RFantibody](https://github.com/RosettaCommons/RFantibody)、[BoltzGen](https://github.com/HannesStark/boltzgen)、[AF3 v3.0.1](https://github.com/google-deepmind/alphafold3/releases/tag/v3.0.1)、[ESMFold2 v3.4.0](https://github.com/Biohub/esm/releases/tag/v3.4.0)。
- 確認したHugging Face URL: [boltzgen/boltzgen-1](https://huggingface.co/boltzgen/boltzgen-1)、[boltzgen/boltzgen1_train](https://huggingface.co/datasets/boltzgen/boltzgen1_train)。Germinal/RFantibodyの公式HF公開は見つからず。
- 確認したその他コード/重み/データURL: bioRxiv Supplementary Material、RFantibody `include/download_weights.sh`、BoltzGen READMEのdownload手順。
- リポジトリライセンスの確認元: GitHub LICENSE/README/repository metadata。Germinal Apache-2.0（外部IgLM/PyRosettaは別条件）、RFantibody MIT、BoltzGen MIT。
- モデル重み・チェックポイントの確認先: RFantibody README、BoltzGen Hugging Face、Germinal READMEの外部パラメータ要件。研究固有の新規checkpointは該当なし。
