# Design of full-length agonist and antagonist antibodies for hard targets

> HTMLビジュアル版: [20261104_Design of full-length agonist and antagonist antibodies for hard targets.html](20261104_Design%20of%20full-length%20agonist%20and%20antagonist%20antibodies%20for%20hard%20targets.html)

## まず何の論文か

Pythia Labsが、全原子生成モデル **Pythia-1** を中核に、標的構造・狙うエピトープ・ヒト抗体frameworkを指定してfull-length IgG1をde novo設計するパイプラインを報告した論文である。モデルは抗体のbackbone、アミノ酸配列、側鎖配座を同時生成し、別のCTMC配列モデル、自己整合性refolding、信頼度・物理・配列filterを組み合わせて候補を絞る。15の可溶性蛋白／受容体細胞外domainと7つのGPCR、合計22標的に対して1回の計算設計を行い、各標的で最大48本だけをfull-length IgGとして実験評価した。一次hitは可溶性群で8/15標的・31本、GPCR群で7/7標的・18本だったため、「22標的すべてでleadが得られた」という意味ではない。IL11では設計抗体IL11_04がKD 2.5 nM、signal阻害IC50 5.6 nMを示し、同一条件の特許由来対照より強かった。CXCR4設計CXCR4_23はcell-binding EC50 15.5 nMながらβ-arrestin阻害IC50 5.5 nMで、対照REGN7663より機能阻害が強かった。さらにinactive CCR8構造を条件にantagonist、active MRGPRX2構造を条件にagonistを得ており、著者らはde novo設計full-length IgGによるGPCR agonism/antagonismの初の両立例と位置づける。全設計はhuman germline frameworkを使い、CDR長をOAS分布からsamplingし、glycosylationやdeamidationなどのliability motifを抑えることで、発現・熱安定性・疎水性・自己会合も初期設計に組み込んだ。wet-labで機能を示した点は強いが、Pythia-1のコード、学習済み重み、完全なtraining set、設計pipelineは非公開であり、再現できるのは公開されたlead配列の再合成・追試までである。したがって本論文は「大規模かつ前向きな実験デモ」として重要だが、モデル比較benchmarkや独立再現性の論文としては未完成である。

## 書誌情報

- URL/DOI: [bioRxiv](https://www.biorxiv.org/content/10.64898/2026.10.21.760573v1) / [10.64898/2026.10.21.760573](https://doi.org/10.64898/2026.10.21.760573)
- 公開日/更新日: 2026-10-26（v1。本文日付は2026-10-21）
- 著者・所属: Pythia Labs Team; Robin A. Aglietti, Ramin Ansari, Peter M. Bowers, Tim-Henrik Buelles, Hadrien Chauvin, Thibault M. Duplay, Mohamed El Hibouri, Julien Jorda, Joshua Laniado, Matthias M. A. Malago, Kyle Meador, Anya Polovina, Katty Polyak, Justyna Sosna, Robert Tacke, Todd O. Yeates, Lucas Zanini; Pythia Labs
- 掲載誌/プレプリントサーバー: bioRxiv（査読前）
- リサーチ日: 2026-11-04（日本時間）
- 分類: 新着 / モデル起点 / 抗体設計 / 前向き実験検証
- 論文ライセンス: CC BY-NC 4.0

## 背景と問題設定

従来の抗体探索は動物免疫またはphage/display libraryに依存し、狙ったepitope・標的conformation・作用機序を最初から制御しにくい。結合hitが得られても、多特異性、自己会合、低発現、疎水性などを後から直す必要があり、lead化までの反復が長い。特にGPCRは細胞外表面が狭く、活性状態が動的で、天然ligandが深いorthosteric pocketへ入るため、通常のIgGで機能性epitopeを狙うのが難しい。

近年の生成モデルはminiproteinやVHHの設計成功率を上げたが、paired heavy/light chainを持つfull-length IgGは6本の柔軟なCDR loopを同時に扱う必要がある。VHHは3本のCDRだけで済み、framework接触も利用しやすい一方、通常IgGとは分子format、Fc機能、薬物動態、developabilityが異なる。本研究が埋めようとする穴は、**狙うepitopeとconformationを条件に、最初からヒトIgG format・機能・製造適性を同時設計し、少数の実験で検証すること**である。

## この論文のコアアイデア

設計入力は、標的蛋白の3D構造、狙う表面残基、human germline由来のheavy/light framework、CDR境界である。Pythia-1はconditional all-atom flow-matching modelで、抗体backbone、配列、側鎖を標的との複合体として同時生成する。標的のactive/inactive conformationとepitopeを変えることで、同じ「結合」ではなく、receptor競合、signaling阻害、活性状態安定化などの機能を幾何学的に誘導する。

配列は必要に応じてbackbone-conditioned CTMC modelでも再設計し、N-linked glycosylation、deamidation、isomerization、望ましくないdisulfideなどを制約する。候補配列をPythia-1で再foldし、意図した結合poseが回復するかをself-consistencyで確認する。pLDDT、PAE、PDEを予測する独立confidence modelと、steric clash、静電・非共有結合、配列liabilityのfilterを組み合わせ、最大数千配列から標的ごとに最大48本へ落とす。

## 手法の詳細

- 入力データ: 標的の実験構造または構造model、狙うepitope残基、human germline heavy/light framework、CDR境界、必要に応じて標的のactive/inactive conformation。
- 出力: 標的に結合するfull-length human IgG1のheavy/light配列とall-atom複合体model。
- モデル/アルゴリズム: Pythia-1 conditional all-atom flow matching、独立confidence model、任意のCTMC sequence model、beam-search型の反復refinement。
- 特徴量・表現学習: backbone座標、アミノ酸identity、側鎖配座を同一生成過程で扱う。詳細なnetwork depth、parameter数、atom/token表現は未確認（本文に非開示）。
- 学習方法: protein monomer、multimer、抗体–抗原複合体を含むmultitask generative training。
- 損失関数・目的関数: stochastic interpolantsに基づくflow-matching objective。各taskの重み、training schedule、optimizerは未確認。
- 推論方法: learned velocity fieldから構造・配列をsamplingし、CTMC変異、自己整合性refolding、confidence/physics/sequence filtering、構造多様性選抜を順に行う。
- ベースライン: 直接の生成モデル比較や同一標的でのDiffAb、RFantibody、BoltzGen等とのhead-to-headはない。実験では特許・既存抗体や天然ligandをtarget別positive controlとして使用。
- 評価指標: BLI KD、cell-binding EC50、β-arrestin/STAT3/kinase/cell proliferation assayのIC50またはEC50、CHO発現titer、DSF Tm、HIC retention、DLS、AC-SINS、非特異cell binding、OASis humanness、配列/構造novelty。
- 実装上の重要点: soluble/ectodomain標的は360構造×各3 sequence variantで最大1,440配列、GPCRは720×各3 variantで最大2,880配列を作り、最終screenは最大48本。計算生成は約1日、DNA合成から一次screenまでは3週間未満。

## データセットと評価設計

### 学習データ

PDBの2025年6月までの構造と、AlphaFold2の高信頼single-chain monomer予測を合わせ、約9,000万entryを使用したと記載される。sequence・structure similarityでcluster化し、SAbDab entryにはCDR geometry clusteringも追加した。可溶性標的panelでは、標的構造と同一clusterのhomologをtrainingから除外した。GPCR panelではantibody-bound PDBが少ないため同じ除外を行わず、後解析で入力GPCR構造がtrainingに含まれたかを確認した結果、CCR8 antagonistとCXCR7は例外だった。これはnovelty解析で補っているが、厳密なtarget-holdoutではない。

### 実験panel

| 群 | 標的数 | 実際にscreenした配列 | 一次hit | hitが出た標的 | 確認方法 |
|---|---:|---:|---:|---:|---|
| 可溶性蛋白／細胞外domain | 15 | 672 | 31 | 8/15 | 非avid soluble antigen 1 µMでHTP-BLI、後続5濃度BLI |
| GPCR | 7 | 256 | 18 | 7/7 | 過剰発現cell flow cytometry、off-target GPCR cell、機能assay |
| 合計 | 22 | 928 | 49 | 15/22 | full-length IgG1として評価 |

一次hit基準は可溶性群で150秒時のBLI response >0.06 nm、GPCRでrobust z-score >10。confirmed binderは可溶性群でKD <5 µM、GPCRでcell-binding EC50 <1 µMである。screen本数は少なく前向き評価として魅力的だが、target選定、epitope手動調整、reagent/assay availabilityの影響を受ける。設計計算の全候補に対するblindな順位精度、失敗例の構造的原因、既存生成法との同条件比較は示されない。

## 主要結果

1. **可溶性群は8/15標的でhit**: 31一次hit中12本がsingle〜double-digit nM KD。negative-control antigenへの測定可能な結合はなかった。IL11は48本中5 hitで、IL11_04はKD 2.53 nM、STAT3 reporter阻害IC50 5.6 nM。対照3C6は同条件でKD 9.4 nM、IC50 24.6 nMだった。
2. **β-NGFは結合から機能へ接続**: 48本中8 hit、うち6本がsingle/double-digit nM。βNGF_18はapparent KD 7.6 nM、TrkA reporter阻害IC50 6.7 nM、TF-1 proliferation阻害IC50 28 nM。ただし高親和性positive controlより阻害は弱く、二量体antigenを用いたBLI値はavidityの影響を受けうる。TrkA直接設計は6一次hitを得たが、5本（KD 128 nM〜3.1 µM）はproliferation assayで阻害を示さなかった。
3. **GPCRは7/7で一次hit、4/7で機能hit**: CCR8では48本中9一次hit、6本中5本が機能的antagonist。CCR8_13/14はIC50 7.6/7.1 nM。CXCR4はCXCR4_23がcell-binding EC50 15.5 nM、β-arrestin阻害IC50 5.5 nMで、対照REGN7663のIC50 33 nMより約6倍低かった。CXCR7にもantagonist活性、MRGPRX2にはagonist活性を確認したが、AMY1、CB1、MC4Rはbinding後の機能活性を示さなかった。
4. **active-state設計でagonism**: MRGPRX2_15はcell-binding EC50 84.5 nM、β-arrestin activation EC50 213.5 nM。天然ligand cortistatin-14は同assayで109 nM。設計modelではHCDR3が天然ligandと共通のreceptor cavityへ入り、active conformationを安定化するという解釈である。
5. **結合強度と機能は一致しない**: CCR8でpaired IC50/EC50比は約0.8〜6.3にばらついた。CXCR4_23も対照よりcell bindingは弱いのに機能阻害が強い。epitope、侵入深度、pose、receptor stateが作用機序を左右し、「最も強く結合する抗体」を選ぶだけでは不十分と分かる。
6. **developabilityを初期設計へ統合**: OASis identity ≥0.8、CHO titer ≥250 mg/L、Fab Tm ≥65°C、HIC retention ≤1.8 minをpass目安とし、多くのconfirmed hitが臨床段階抗体と同等範囲だった。DLS、AC-SINS、pI、net charge、非特異cell bindingも確認した。ただし長期保存、high-concentration viscosity、chemical stability、PK、免疫原性、in vivo safetyは未評価である。

## 重要なFigure/Table

- Figure/Table番号: Figure 1 / Supplementary Figure S1 / S3
- 何を示しているか: 入力仕様から生成・filter・full-length IgG screenまでのpipelineと、22標的のprimary/confirmed/function hit、developability。
- 読み取り方: 「全標的でbinding」「全標的で機能」「すべてが高親和性」を区別する。可溶性群は8/15、GPCR一次hitは7/7、機能GPCRは4/7。
- 主要な数値・傾向: 928本screen、49一次hit。可溶性31、GPCR18。
- なぜ重要か: 本研究のclaimの分母と成功段階を最も正確に読める。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: 独自のfunnelと標的別進捗図
- 出典URL: [bioRxiv PDF](https://www.biorxiv.org/content/10.64898/2026.10.21.760573v1.full.pdf)
- ライセンス確認: 確認済み（CC BY-NC 4.0）。原図は保存せず独自再構成。

- Figure/Table番号: Figure 2 / Figure 5 / Figure 6
- 何を示しているか: IL11 receptor-blocking antagonist、CXCR4 orthosteric-pocket antagonist、MRGPRX2 active-state agonistの予測poseと実測dose response。
- 読み取り方: target conformationとepitope指定が、単なるbindingから機能へつながるかを見る。
- 主要な数値・傾向: IL11_04 KD 2.53 nM / IC50 5.6 nM、CXCR4_23 EC50 15.5 nM / IC50 5.5 nM、MRGPRX2_15 binding EC50 84.5 nM / agonism EC50 213.5 nM。
- なぜ重要か: affinity最大化だけでなく、poseと状態選択を設計変数にできる可能性を示す。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: 独自の機能比較chartとstate-to-function模式図
- 出典URL: [bioRxiv PDF](https://www.biorxiv.org/content/10.64898/2026.10.21.760573v1.full.pdf)
- ライセンス確認: 確認済み（CC BY-NC 4.0）。原図は保存せず独自再構成。

- Figure/Table番号: Supplementary Figure S4 / S7 / S11
- 何を示しているか: developability、既知抗体からの配列・構造novelty、GPCR hitのscreeningからfunctional potencyまで。
- 読み取り方: 良いbinding値だけでなく、製造可能性、off-target cell、novelty、機能を同じ候補で追う。
- 主要な数値・傾向: CCR8_38はbinding EC50 2.987 nMだがIC50 18.7 nM、CCR8_14は5.732/7.1 nM、CXCR4_23は15.53/5.5 nM。
- なぜ重要か: affinityとfunctionの非単調性、lead selectionの多目的性が見える。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: binding/functionのpaired table
- 出典URL: [bioRxiv PDF](https://www.biorxiv.org/content/10.64898/2026.10.21.760573v1.full.pdf)
- ライセンス確認: 確認済み（CC BY-NC 4.0）。原図は保存せず独自再構成。

## Code / License / Weights

- コード公開: 見つからず
- GitHub URL: 該当なし（Pythia-1抗体モデルの公式repositoryは見つからず）
- GitHub以外のコードURL: 該当なし
- 実装種別: 公式だが非公開
- GitHub確認: 該当なし
- リポジトリのライセンス: 該当なし
- ライセンス確認元: 該当なし
- Hugging Face URL: 該当なし（Pythia Labs/Pythia-1抗体モデルの公式model/datasetは見つからず。EleutherAIの同名LLMは無関係）
- Hugging Face種別: 該当なし
- モデル重み・チェックポイント公開: 見つからず
- 重み公開URL: 該当なし
- データセット公開: 一部あり（lead抗体配列）
- データセットURL: [Pythia-1m antibody sequences v1](https://pythialabs.com/antibody-sequences/pythia-1m/v1)
- 配列ライセンス: Pythia Labs独自ライセンス。研究・改変・発現・共有は無償、帰属必須、商用利用とderivativeの商用利用は別途書面licenseが必要。
- 再現性メモ: 公式release pageは「80 sequences / 40 heavy-light pairs / FASTA」と表示するが、2026-11-04に利用条件へ同意して取得した`Pythia1mAbs.fasta`は78 record（39 HC、39 LC、39 pair）だった。本文Supplementary Table S2はreference antibody配列を掲載するが、Pythia-1のnetwork詳細、学習code/weight、約9,000万entryのcurated manifest、標的ごとのepitope specification、全生成候補・ranking scoreは公開されていない。独立研究者はlead配列の再合成はできるが、設計pipelineの再実行やrank再現はできない。

## 抗体研究・創薬への意味

最大の意味は、抗体設計の目的関数を「結合するか」から「どの状態・どの面へ結合し、何を起こすか」へ移した点にある。IL11/β-NGFでは天然receptor interfaceを塞ぎ、CCR8/CXCR4ではrecessed pocketへCDRを入れ、MRGPRX2ではactive stateを安定化する。将来、同じ考え方をbiased agonism、allosteric modulation、pH-dependent binding、species selectivityへ広げられる可能性がある。

また、human germline、OAS由来CDR長、sequence liability filter、IgG format assayを最初から使う設計は、binding hit後にdevelopabilityを修復する直列工程から、多目的同時最適化への転換を示す。一方、modelが非公開なので現時点ではコミュニティが使えるplatformというより、企業内技術のcapability demonstrationである。学術的な次の一歩は、同一標的・同一screen予算で公開法と比較し、epitope success、function success、developability、計算コストをblindに評価することである。

## 限界と注意点

**著者が述べる限界**

- in vivo PK、efficacy、safetyは未評価であり、therapeutic candidateとしての適性は未確立。
- β-NGF BLIは二量体antigenによるmultivalent interactionのため、報告KDはapparent valueとして読む必要がある。
- TrkAのようにbinding hitが機能阻害へ進まない標的がある。

**精読して気づく限界**

- 全著者がPythia Labs社員で、査読前のcompany preprint。独立実験validationはない。
- 22標的中、一次hitは15標的、機能実証はさらに一部である。abstractの「across 22 targets」は全標的で成功したという意味ではない。
- 直接baselineがなく、既存生成法、免疫/display、random CDR libraryと同じ標的・本数で比較していない。48本という小screenの価値は高いが、相対優位を定量できない。
- epitope定義を文献、PDB、surface解析後に手動調整しており、人間の構造知識と試行の寄与が分離されていない。
- 可溶性panelはcluster除外したが、GPCRは厳密holdoutでなく、CCR8とCXCR7入力構造はtrainingに含まれる。配列noveltyはmemorization懸念を軽減するが、target/pose priorの漏洩を排除しない。
- 配列・structureのpredicted noveltyは新規機能の証明ではない。設計poseの実験構造（cryo-EM/X-ray）もない。
- developabilityは初期評価に限られ、high-concentration viscosity、長期安定性、化学分解、免疫原性、Fc function、PKは未確認。
- 論文はCC BY-NC 4.0、公開配列は別の独自non-commercial license。コードや重みのlicenseは存在しないため、商用再利用可能性を混同しない。
- 公式release表示40 pairと取得FASTA 39 pairの不一致は、data releaseのversion管理上の注意点である。

## この論文を読む上での前提知識

- **Full-length IgG / paired VH–VL**: heavy/light可変domainに加えてFcを持つ通常抗体format。VHHより構造自由度は高いが、薬物動態・effector functionを実薬に近い形で評価できる。
- **CDRとgermline framework**: 抗体の6本のCDR loopが主に標的へ接触し、frameworkがloopを支える。human germlineを固定するとhumannessと製造適性を保ちやすい。
- **Flow matching**: 単純分布からdata分布へ連続変換するvelocity fieldを学習する生成法。本研究では配列と全原子構造を条件付きで同時生成する。
- **CTMC sequence model**: 離散アミノ酸状態を連続時間Markov過程でdenoise/samplingする方法。backbone条件付き配列variant生成とmotif制約に使う。
- **BLI KDとcell-binding EC50**: KDは精製系の平衡親和性、EC50は細胞表面で半最大bindingを与える濃度。receptor発現量やavidityが異なり、直接同一視できない。
- **Agonist / antagonist**: agonistは受容体活性状態を誘導・安定化し、antagonistはligand結合やsignalを阻害する。同じbindingでもposeとconformationで作用が変わる。
- **Developability**: 発現、熱安定性、疎水性、自己会合、非特異結合など、薬として製造・高濃度投与できる可能性を決める性質群。

## 今回この1報を選んだ理由

前回リサーチ後の2026-10-26公開で、配列条件付きmodelだったLangAAIから一歩進み、全原子full-length IgG設計を22標的で前向き実験検証している。特にGPCRのactive/inactive conformationを設計入力にし、agonistとantagonistを作り分けた点は、新規性と創薬上の重要性が高い。公式コードや重みがない点は弱いが、40 pairと告知されたlead配列releaseを実際に取得・監査でき、再現性とlicenseの問題を学ぶ材料にもなる。同時期のSANDAN論文もAI設計CDR-H3とphage displayを結ぶ重要な新着だが、1標的・6 bindersに対し、本論文は標的範囲、作用機序、full-length format、developabilityの広さで上回るため採用した。

## 読む優先度

**High**。2026年時点の抗体生成AIが、bind/not-bind benchmarkから「少数screenでfull-length IgGの機能を作る」段階へ進んだことを示す代表的なindustry preprintである。ただし非公開modelとbaseline不在のため、手法論の確立ではなく、能力上限を示す実験報告として読むべきである。

## 自分用メモ

- 後で深掘りしたい点: 公開39 pairをANARCI numberingし、CDR長、liability motif、OASis、Thera-SAbDab距離を独立再解析する。
- 関連して読むべき論文: Chai-2、RFAntibody、BoltzGen、IgGM2、Germinal/OpenGerminal、SANDAN、AIntibody Challenge。
- 実装を触る場合の入口: Pythia-1は非公開。公開配列の再合成前に独自licenseを確認し、まずin silicoでaggregation、immunogenicity、off-target motifを再評価する。
- Obsidianでリンクしたいキーワード: [[full-length IgG de novo design]]、[[GPCR antibody]]、[[agonist antibody]]、[[flow matching]]、[[developability]]、[[epitope-conditioned design]]。

## 関連キーワード

- Pythia-1
- all-atom generative model
- full-length IgG design
- GPCR agonist antibody
- GPCR antagonist antibody
- epitope-conditioned design
- structure-conditioned flow matching
- antibody developability

## 検索ログ

- 検索したデータベース/クエリ: bioRxiv / `antibody artificial intelligence October 2026`、arXiv / `antibody design October 2026 protein language model`、PubMed、Crossref相当検索、論文title完全一致、`Pythia-1 antibody model GitHub`、`Pythia Labs Hugging Face`。
- 候補にした論文: 本論文、*Integrated AI-Guided de novo CDR-H3 Design and Combinatorial Phage Display for Antibody Discovery*（SANDAN、Antibody Therapeutics、2026-10-27）、*A foundation model learns the sequence and functional grammar of fully human heavy-chain-only antibodies*（HCAbLM）。
- 最終的にこの1報を採用した理由: 22標的、full-length IgG、GPCR、agonist/antagonist、developability、少数screenという学習価値を優先。
- 新着論文と基盤論文のバランス: 今回は新着枠。flow matching、OAS、SAbDab、抗体developabilityを基盤概念として接続した。
- PDF/HTML/Supplementaryを確認できたか: bioRxiv 47ページPDF（本文24ページ＋Supplementary 23ページ）を全ページtext extractionおよびPNG renderで確認。abstract HTMLも確認。
- Figure/Tableを確認したURLやライセンス確認状況: Figure 1–6、Supplementary Figure S1–S12、Supplementary Table S1–S2を確認。PDF各ページ上部でCC BY-NC 4.0を確認。
- GitHub/コード検索で使ったクエリ: `site:github.com "Pythia-1" antibody`、`site:github.com "Pythia Labs" antibody`。同名のEleutherAI LLM、VQA Pythia、Wublab Pythiaは無関係として除外。
- 確認したGitHub URL: Pythia-1抗体モデルの公式URLは見つからず。
- 確認したHugging Face URL: Pythia Labs/Pythia-1抗体モデルの公式model、dataset、organizationは見つからず。
- 確認したその他コード/重み/データURL: https://pythialabs.com/news、https://pythialabs.com/antibody-sequences/pythia-1m/v1、公式APIから取得した`Pythia1mAbs.fasta`。
- リポジトリライセンスの確認元: repository非公開のため該当なし。論文はbioRxiv PDFでCC BY-NC 4.0、配列は公式release pageの独自Antibody Sequence License（2026-10-21版）を確認。
- モデル重み・チェックポイントの確認先: 論文Data availability、公式site、GitHub検索、Hugging Face検索。見つからず。
