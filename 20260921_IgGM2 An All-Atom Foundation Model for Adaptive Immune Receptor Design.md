# IgGM2: An All-Atom Foundation Model for Adaptive Immune Receptor Design

[HTML版を開く](20260921_IgGM2%20An%20All-Atom%20Foundation%20Model%20for%20Adaptive%20Immune%20Receptor%20Design.html)

## まず何の論文か

IgGM2は、抗体、単一ドメイン抗体（nanobody/VHH）、T細胞受容体（TCR）を別々の専用モデルとして扱わず、同じ全原子生成フレームワークで構造予測と設計を行う研究である。中心となる発想は、まず標的構造の周囲に免疫受容体を正しく配置する構造予測能力を学び、その幾何学的事前分布をCDR設計へ移す「structure-to-design」である。構造予測版IgGM2-Pは、抗原またはpMHCを固定コンテキストとし、受容体側の原子だけを拡散・デノイズする。設計版IgGM2-Dは、CDRのアミノ酸残基と受容体の全原子座標を同じモデル内で生成し、別のinverse foldingやside-chain packingを必須にしない。配列の離散マスキング率を座標拡散のノイズ量に合わせ、推論前半でCDR配列と骨格を決め、後半で配列を固定して側鎖を含む全原子構造を精密化する。FoldBenchでは、エピトープ情報を与えた1回の生成で平均DockQ 0.60、成功率73.8%を報告し、複数候補から選ぶAlphaFold-3等の公式結果を上回った。CDRH3重複除去済み設計テストでは、paired antibodyの平均AARは0.590で最高ではないが、6つのCDRのうち5つで最良の局所Cα RMSDを得た。nanobodyでは平均AARを最良ベースライン0.287から0.372へ引き上げ、Rosetta interface preference（IMP）も0.527に達した。一方、評価は既知複合体の後ろ向き復元と計算スコアが中心で、生成分子の結合、特異性、安定性、developabilityを実験では検証していない。したがって本論文の強みは「実験済み抗体を生み出したこと」ではなく、免疫受容体の予測と設計を全原子・標的条件付きで統一する技術的な足場を示した点にある。

## 書誌情報

- URL/DOI: [bioRxiv](https://www.biorxiv.org/content/10.64898/2026.07.09.737510v1) / [10.64898/2026.07.09.737510](https://doi.org/10.64898/2026.07.09.737510)
- 公開日/更新日: 2026-07-09（v1）
- 著者・所属: Jian Ma, Fandi Wu, Lin Yao, Jing Gao, Rubo Wang, Qifeng Li, Nianzu Yang, Songlin Jiang, Dawei Huang, Xiaoyong Pan, Yiheng Zhu, Tingjun Hou, Jianhua Yao, Junchi Yan。上海交通大学、Tencent AI for Life Sciences Lab、Zhongguancun Institute of Artificial Intelligence、浙江大学ほか
- 掲載誌/プレプリントサーバー: bioRxiv（査読前プレプリント）
- リサーチ日: 2026-09-21（JST）
- 分類: 新着 / モデル起点
- 論文ライセンス: CC BY-NC-ND 4.0

## 背景と問題設定

抗体設計では、CDR配列、CDRループの立体配座、側鎖配置、抗原に対する結合姿勢が相互依存する。しかし従来の代表的パイプラインは、RFdiffusionで骨格を生成しProteinMPNNで配列を回収するように、構造生成と配列設計を分離することが多い。抗体専用法でも、固定骨格上のinverse folding、既にdockingされた複合体、テンプレート、外部side-chain packingのいずれかに依存しやすい。また抗体/VHHとTCRはCDRを介して標的を認識するという共通性があるのに、別々のモデルとして研究されてきた。

この分離は、配列変異に伴ってframeworkや側鎖が再配置される現実と整合しにくい。IgGM2は「標的を固定し、受容体全体を生成する」条件付き構造予測を共通基盤にし、同じ表現をCDR共設計へ転用する。抗体創薬上は、予測・docking・配列設計・側鎖再構築の間で誤差が累積する問題を減らせる可能性がある。

## この論文のコアアイデア

IgGM2には構造予測用のIgGM2-Pと、CDR配列・構造共設計用のIgGM2-Dがある。IgGM2-PはAF3型の拡散アーキテクチャをProtenixの事前学習パラメータから初期化し、MSAとtemplate入力を外したsingle-sequenceモデルとして免疫受容体データにfine-tuningする。複合体予測では抗原/pMHCを固定し、抗体・VHH・TCR側だけを生成する。固定原子マスク、標的の幾何、任意のhotspot/epitope情報はConstraintEmbedを通じてPairformerへ入る。

IgGM2-Dはこの構造事前分布をwarm startとして使い、CDR残基をUNKへ置換した「sequence-design mode」と、正解配列を与えて全原子を復元する「structure-prediction mode」を30:70で混合学習する。配列マスク率はEDM座標拡散のskip coefficientに合わせ、座標の信頼度が低い時ほど多くの配列を隠す。推論は二段階で、高ノイズ段階ではCDR配列と骨格を同時予測し、切替点で配列を確定、その後に側鎖を含む全原子座標を精密化する。atom14は固定長の原子スロットとして使うだけで、残基種を表す非物理的なvirtual atomにはしない。

## 手法の詳細

- 入力データ: 受容体配列（抗体H/L、VHH、TCR α/β等）、固定された抗原またはpMHC構造、任意のepitope/hotspot残基。設計時は非設計framework配列と、マスクされたCDR配列を使う。
- 出力: IgGM2-Pは受容体の全原子構造または受容体–標的複合体。IgGM2-DはCDR配列と、それに整合したframeworkを含む受容体全原子構造・標的結合姿勢。
- モデル/アルゴリズム: AF3-like diffusion、Pairformer、ConstraintEmbed、atom encoder/decoder、IgGM2-Dのみsequence denoiser。座標拡散はEDM preconditioning。
- 特徴量・表現学習: 配列トークン、atom14全原子表現、固定コンテキストの原子マスク・幾何、hotspot特徴。MSAとtemplateは使わない。
- 学習方法: IgGM2-PをProtenixから初期化して免疫受容体データで学習し、IgGM2-DをIgGM2-Pからwarm start。設計:構造の混合比は30:70。16基のA100 GPU、最大crop 512 token、Adam、学習率1.8×10^-3、3,000 warmup steps、1,000 stepsごとに0.95倍減衰。設計サンプルのCDR骨格へ0.1 Å Gaussian noiseを加える。
- 損失関数・目的関数: 生成原子だけを対象にする座標損失、distogram/距離系の構造損失、設計モードでのみCDR配列損失。論文本文の係数詳細は一部未確認。
- 推論方法: 高ノイズ相で配列＋骨格を生成し、T_switch=75でCDR配列を固定して低ノイズ相で全原子精密化。IgGM2-Pはconfidence headを持たず、主要比較では1 seed×1 sampleで候補選別なし。
- ベースライン: AlphaFold-3/-Multimer、Protenix、Chai-1、Boltz-1、HelixFold-3、tFold系、TCRModel2、IgFold、ImmuneBuilder等。設計はDiffAb、AbX、dyMEAN、RFantibody、IgGM。
- 評価指標: DockQ、DockQ≥0.23のsuccess rate、LRMS/iRMS、TM-score、GDT、CDR amino-acid recovery（AAR）、framework-aligned local Cα RMSD、Rosetta IMP、dG_separated。
- 実装上の重要点: 受容体全残基を常に保持し、抗原は受容体重心に近い順でcropする。epitopeはCα距離6 Å未満で定義し、学習時に部分的に隠して完全な注釈への過依存を弱める。

## データセットと評価設計

学習データはSAbDabとSTCRDabからcurateされるが、論文は最終的なtrain/validationの件数を明示していない。構造予測用は2021-09-30以前の構造を学習に使い、FoldBench等のtestとfull-sequence identity 95%超の学習例を除外する。配列設計用は2024-01-01を時間cutoffとし、それ以前をtrain、以後をvalidation/testに割り当てる。両設定で配列重複除去後、MMseqs2（min_seq_id=0.8、cov_mode=1）によりCDRH3 clusterを作り、設計のtrain-test間CDRH3 similarityを0.8未満にする。

主要構造評価はFoldBench、SAbDab-22H2-AbAg、STCRDab-22-TCR_pMHC。補足でpaired antibody、nanobody単体、unliganded TCRを評価する。FoldBenchは113 PDB biological assemblies由来172 interfaces（Ab–Ag 122、Nano–Ag 48、scFv–Ag 2）。TCR–pMHCは18 complexesである。設計のclean testは269 complexes（paired antibody 195、nanobody 74）だが、test内部にも類似CDRH3が残るため、主要表ではさらに0.8未満へ重複除去した203 complexes（148/55）を使う。

時間分割、train-test cluster分割、test内部重複除去を併用した点は妥当である。一方、FoldBench比較はIgGM2だけが固定抗原とepitope条件を明示的に受けるため、無条件の全複合体予測と同じ難易度ではない。各benchmarkで公開済み結果を流用し、全手法を統一環境で再評価していない点にも注意が必要である。

## 主要結果

FoldBenchでIgGM2-P(epitope)は平均DockQ 0.60、success rate 73.8%（127/172）だった。AlphaFold-3は0.36/47.9%、Protenix-1.0は0.36/48.8%である。ただしbaselineは5 seeds×5 samplesから選ぶ公式設定、IgGM2は1×1かつrankingなしであり、効率面は有利だがepitope情報という強い条件を使う。内訳ではAb–AgのSR 76.23%、Nano–Ag 66.67%、scFv–Ag 100%（2/2）。全体の45.3%はhigh-quality DockQ≥0.80だった一方、26.2%はDockQ<0.23で失敗しており、万能ではない。

TCR–pMHC 18例ではDockQ 0.746、LRMS 2.966 Å、iRMS 1.262 Å、GDT 0.887、acceptable/medium/high SRは100.0/88.9/38.9%。AlphaFold-3のDockQ 0.632、LRMS 5.316 Å、iRMS 2.017 Åを上回る。SAbDab-22H2-AbAgではepitope条件付きIgGM2-PはDockQ 0.602、SR 75.8%だが、contact restraintを使うtFold-AgはDockQ 0.703、SR 97.0%であり、「全ての制約付き法に勝った」わけではない。

paired antibody設計では平均AAR 0.590で、AbX 0.610、dyMEAN 0.596に僅かに及ばない。しかしlocal Cα RMSDはH1 1.011、H2 0.940、H3 3.492、L1 0.953、L2 0.674、L3 1.231 Åで、H3を除く5 CDRで最良だった。Rosetta評価ではIMP 0.615、mean dG_separated -23.27で最良。nanobodyでは平均AAR 0.372（DiffAb 0.287、IgGM 0.267）、HCDR3 AAR 0.238（IgGM 0.154、相対54.55%増）で、3 CDRすべてのlocal RMSDも最良だった。全clean testのco-generated complexでは、paired antibodyのDockQ 0.292/SR 0.503、nanobodyのDockQ 0.216/SR 0.284であり、最良でも絶対値はまだ低い。

ablationではT_switch=75、設計:構造=30:70、noise=0.1をバランス設定として選ぶ。最高AAR自体は50:50/noise 0.5の0.534だがIMPが下がり、70:30/noise 0.1/T_switch=75はIMP 0.677でもAARが0.515へ落ちる。単一指標を最大化せず、配列回復とinterface qualityのtrade-offを示した点は重要である。

## 重要なFigure/Table

- Figure/Table番号: Figure 1・Figure 2
- 何を示しているか: 構造予測から設計へ事前分布を移す全体像と、固定標的→Pairformer→座標/配列denoiser→二段階推論の内部構成。
- 読み取り方: 「標的は固定、受容体だけ生成」「前半は配列＋骨格、後半は確定配列で側鎖精密化」という二つの分離を見る。
- 主要な数値・傾向: 30:70 mixed training、T_switch=75、512-token crop、16×A100。
- なぜ重要か: IgGM2が単なる抗体構造予測器ではなく、予測能力をCDR共設計へ移す統一モデルであることを最短で理解できる。
- Markdown内での扱い: 独自の文章フロー
- HTML内での扱い: 原論文Figure 1/2を参考にした独自の要約フロー図（原図の転載ではない）
- 出典URL: [bioRxiv full text](https://www.biorxiv.org/content/10.64898/2026.07.09.737510v1.full)
- ライセンス確認: 確認済み（CC BY-NC-ND 4.0、NDのため原図を保存・改変転載しない）

- Figure/Table番号: Table 1・Table S1
- 何を示しているか: FoldBenchの複合体予測性能と受容体種別内訳。
- 読み取り方: DockQ平均だけでなく、DockQ≥0.23の成功率とincorrect 26.2%を合わせて見る。
- 主要な数値・傾向: IgGM2 0.604/73.84%、AF3 0.36/47.9%。172 interfaces中high 78、incorrect 45。
- なぜ重要か: 強い平均値と残る失敗率、epitope条件付きという前提を同時に把握できる。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: 独自の横棒チャートと品質分布
- 出典URL: 上記bioRxiv
- ライセンス確認: 確認済み（数値のみ再構成）

- Figure/Table番号: Table 3・Table 4・Table 5
- 何を示しているか: paired antibody/VHHのCDR回復、局所構造、Rosetta interface score。
- 読み取り方: AAR単独で順位を決めず、local RMSDとIMP/dGを並べて読む。
- 主要な数値・傾向: paired AAR 0.590、5/6 CDRで最良RMSD、IMP 0.615。nanobody AAR 0.372、IMP 0.527。
- なぜ重要か: 「native配列を最も再現するモデル」ではなく「配列と構造の整合性を重視するモデル」という主張を支える。
- Markdown内での扱い: 主要数値の短い再構成
- HTML内での扱い: AAR、local geometry、interface scoreを分離した独自カード/チャート
- 出典URL: 上記bioRxiv
- ライセンス確認: 確認済み（数値のみ再構成）

## Code / License / Weights

- コード公開: 見つからず（IgGM2固有）
- GitHub URL: IgGM2固有は見つからず。前世代IgGMの公式実装は [TencentAI4S/IgGM](https://github.com/TencentAI4S/IgGM)
- GitHub以外のコードURL: 見つからず
- 実装種別: IgGMは公式だがIgGM2とは別モデル。IgGM2は該当公開なし
- GitHub確認: 確認済み（GitHub repository search/API、TencentAI4S organization、論文PDF）
- リポジトリのライセンス: IgGMはMIT。IgGM2固有は該当なし
- ライセンス確認元: IgGM GitHubのLICENSEファイルとrepository metadata
- Hugging Face URL: 見つからず
- Hugging Face種別: 該当なし
- モデル重み・チェックポイント公開: 見つからず（IgGM2固有）
- 重み公開URL: 見つからず
- データセット公開: 原データベースあり、論文固有splitファイルは見つからず
- データセットURL: [SAbDab](https://opig.stats.ox.ac.uk/webapps/sabdab-sabpred/sabdab)、[STCRDab](https://opig.stats.ox.ac.uk/webapps/stcrdab/)
- 再現性メモ: モデル構造、擬似コード、optimizer、crop、主要hyperparameterは記載されるが、学習splitの最終件数、IgGM2コード、設定ファイル、checkpoint、再現用indicesが公開確認できず、完全再現は困難。Protenix初期値とRosetta/HDOCKを含む比較環境も揃える必要がある。

## 抗体研究・創薬への意味

抗体設計では、CDR配列だけを変えた後に構造が崩れる、あるいは骨格生成後のinverse foldingで標的結合姿勢が変わるという「工程間の不整合」が大きい。IgGM2-Dの全原子共設計は、この不整合を一つの生成過程で扱う方向性を示す。H鎖/L鎖抗体とVHHを同じ枠組みで比較できること、長いHCDR3を持つVHHでも改善したこと、epitope hotspotを条件にできることは、エピトープ指定抗体設計やaffinity maturationの初期候補生成に有用である。

さらにTCR–pMHCも同じ表現で扱うため、抗体とTCRに共通する「適応免疫受容体の標的条件付き設計」という研究軸を作れる。ただし現状の出力は候補仮説であり、SPR/BLI、特異性、発現量、凝集、熱安定性、polyreactivity等を同時最適化する創薬モデルではない。実運用ではIgGM2スコアを選抜の一段に置き、developability予測、物理スコア、実験スクリーニングで閉ループ化する必要がある。

## 限界と注意点

**著者が述べている限界**

- CDR3、特に長いnanobody HCDR3のAARには改善余地があり、より強い配列事前分布が必要。
- 二段階推論では前半の配列/骨格誤差が後半の側鎖packingへ伝播しうる。
- 正確な標的構造と、場合によってはepitope情報を仮定する。予測標的、柔軟標的、不確かなepitopeでは性能が落ちうる。
- AAR、RMSD、DockQ、Rosetta energyは結合親和性、特異性、安定性、developabilityを保証せず、実験検証が必要。

**読者として追加で注意すべき限界**

- FoldBenchのheadline比較はepitope-conditioned IgGM2と一般的複合体予測器の比較で、入力情報量が同一ではない。Table S2ではcontact-restraint tFold-AgがIgGM2を上回る。
- 主要な設計評価は既知native complexへの回復であり、真に新規なepitope/antigenへのde novo generalizationを直接示さない。
- paired antibodyの平均AARはAbX/dyMEANに負け、co-generated DockQも0.292に留まる。Rosetta scoreの改善を実結合改善と解釈してはいけない。
- baselineはbenchmarkごとの既報値、公開checkpoint、再学習版が混在し、計算予算も統一されていない。
- train setの最終規模、IgGM2固有コード・重み・splitが未公開で、再現性と商用利用条件を判断できない。
- 16×A100での学習は高コストであり、実験室単位での再学習は容易でない。

## この論文を読む上での前提知識

- **CDRとframework**: CDRは抗原認識の中心となる可変ループで、frameworkはそれを支える。配列変異はCDRだけでなくframeworkの配置にも影響する。
- **全原子拡散モデル**: 原子座標へノイズを加え、段階的に除去して3D構造を生成する。backboneだけでなく側鎖原子まで扱うと化学的整合性の課題が増える。
- **Inverse folding**: 与えられた骨格に合うアミノ酸配列を予測する問題。IgGM2はこれを独立工程にせず、配列と構造を同じ生成モデルで結ぶ。
- **DockQ**: 複合体interfaceの正しさを0〜1で評価する統合指標。0.23以上がacceptableの目安だが、親和性そのものではない。
- **AAR**: 生成配列がnative残基をどれだけ回復したか。de novo設計では高いほど常に良いわけではないが、既知motif保存の目安になる。
- **RMSDとframework alignment**: frameworkを重ねてCDR局所RMSDを見ることで、ループ幾何の再現を測る。
- **pMHC/TCR**: MHCに提示されたpeptideをTCRが認識する複合体。抗体–抗原とは構成が異なるが、CDRによる認識という共通性がある。

## 今回この1報を選んだ理由

2026-09-14以降の検索では、AI×抗体の一次研究として本論文を明確に上回る新着を確認できなかった。レビュー論文は新しかったが、IgGM2は抗体/VHH/TCR、構造予測/CDR設計、配列/全原子構造を一つに統合するため、長期的なモデル起点として学習価値が高い。主要表だけでなく、deduplication、ablation、失敗率、補足benchmarkまで情報量が多く、強みと限界を数値で検討できる。一方でコード未公開・実験未検証という弱点も明確で、現在のAI抗体設計を過大評価せず理解する題材として適している。

## 読む優先度

**High**。抗体構造予測、epitope-conditioned design、全原子生成をつなぐ設計思想が明確で、次世代の統合型抗体モデルを理解する土台になる。ただし実験的成功率を示す論文ではないため、Table 1の高いDockQだけでなくTable 3–5、Table S6–S7、limitationsをセットで読むべきである。

## 自分用メモ

- 後で深掘り: IgGM2の配列マスクとEDM noise alignmentをDiffAb/flow matchingと比較する。
- 関連して読む: IgGM、Protenix、ODesign、DiffAb、dyMEAN、RFantibody、FoldBench。
- 実装入口: IgGM2公開待ち。先にMITのIgGMとProtenix、SAbDab split生成を確認する。
- Obsidianリンク候補: [[全原子拡散]]、[[CDR共設計]]、[[エピトープ条件付き抗体設計]]、[[DockQ]]、[[SAbDab]]、[[TCR-pMHC]]。

## 関連キーワード

- antibody design, nanobody design, TCR design, all-atom diffusion, immune receptor foundation model, CDR co-design, epitope conditioning, structure-to-design, SAbDab, STCRDab, DockQ

## 検索ログ

- 検索した情報源/クエリ: bioRxiv、PubMed、arXiv、Nature、GitHub Search/API、Hugging Face API。`antibody artificial intelligence machine learning September 2026`、`antibody design protein language model September 2026`、`IgGM2 GitHub`、`IgGM2 Hugging Face`、`IgGM2 code model weights` 等。
- 候補: *Dual-Specific Antibody Design Using Artificial Intelligence*、*Frozen Protein Foundation-Model Embeddings Improve Antibody-Antigen ΔΔG Ranking*、AbAgKer、IgGM2、AI抗体設計レビュー。
- 採用理由: IgGM2は統合モデルとしての新規性、Figure/Tableの学習価値、抗体/VHH/TCRへの直接性が最も高かった。
- 新着/基盤バランス: 直近1週間の強い一次研究が見つからなかったため、2026年7月公開の新しめのモデル起点論文を採用。基盤論文だけに偏らない。
- PDF/HTML/Supplementary: bioRxiv HTMLと24ページPDF（本文＋appendix）を確認。
- Figure/Table/ライセンス: Figure 1–3、Table 1–7、Table S1–S12を確認。PDF各ページにCC BY-NC-ND 4.0表示。原図は保存・転載せず、HTMLで独自再構成。
- GitHub/コード検索: GitHub repository search/API、TencentAI4S、論文PDF中のURLを確認。IgGM2固有repoは見つからず。関連の前世代 [IgGM](https://github.com/TencentAI4S/IgGM) は確認。
- Hugging Face: model/dataset APIで`IgGM2`を検索し0件。
- その他: bioRxiv DOI、SAbDab、STCRDab、NeurIPS 2026 downloads listingを確認。論文固有重み・データsplitは見つからず。
- ライセンス確認元: 論文PDF（CC BY-NC-ND 4.0）、IgGM GitHub LICENSE/repository metadata（MIT）。
- モデル重み確認先: GitHub repositories/releases相当、Hugging Face models/datasets、論文本文・appendix。IgGM2は見つからず。
