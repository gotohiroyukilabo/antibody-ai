# Dual-Specific Antibody Design Using Artificial Intelligence

> HTML版: [図解ノートを開く](20260928_Dual-Specific%20Antibody%20Design%20Using%20Artificial%20Intelligence.html)

## まず何の論文か

この研究は、1本の通常型IgGの各抗原結合部位（Fv）が、互いに異なる2つの抗原のどちらにも結合できる「multibody（two-in-one antibody）」を、AIと実験の反復で系統的に設計した報告である。一般的な二重特異性抗体は左右の腕を別々の標的に割り当てたり、scFv/VHHを付加したりするが、本研究の分子は左右対称のIgG構造を保つ。つまり、1つのFvに2つのパラトープを共存させ、各腕が標的AまたはBを競合的に選べるようにする。難所は、一方への親和性を上げた変異が他方への結合や特異性、発現、熱安定性を壊しやすいことである。著者らは、AIによる配列提案、実験スクリーニング、標的ごとの親和性予測モデル、多目的探索、IgGでの最終確認を循環させた。bioRxiv要旨では9件の独立キャンペーン、15種類の非関連標的で治療薬水準の候補を得たと報告する。先行する公開技術報告では7件・11標的が詳細化され、初期候補の4 nM/80 nMを両標的とも一桁nMまたはsub-nMへ改善し、最大約100倍の親和性向上を示す。さらに約200分子をBVP、HIC、DSFで評価し、多くが臨床IgGに近いdevelopability範囲に入った。機能面では、Trop-2/Nectin-4を狙うADC型候補の内在化、同じ2標的を使うT細胞エンゲージャー、IL-13/TSLP二重遮断による免疫抑制という異なる作用機序を実験した。これはモデルの新規アーキテクチャを開示する研究というより、閉ループMLが複雑な抗体フォーマットを前臨床候補まで運べることを示す産業的proof-of-conceptである。ただし、配列、学習データ、モデル構造、予測指標の数値、陰性キャンペーンの情報が公開されておらず、再現性と一般化可能性は独立検証できない。

## 書誌情報

- URL/DOI: [bioRxiv](https://www.biorxiv.org/content/10.64898/2026.08.03.742397v1) / [10.64898/2026.08.03.742397](https://doi.org/10.64898/2026.08.03.742397)
- 公開日/更新日: 2026-08-05公開（v1）
- 著者・所属: Michael Peer, Inbar Amit, Yael Diesendruck, Ziv Erlich, Yonit Ben David, Meital Gadrich, Nino Oren, Tzvika Hartman, Sharon Fischman, Guy Nimrod, Marek Strajbl, Avi Haleva, Reshef Shilon, Yehezkel Sasson, Reut Barak-Fuchs, Itzhak Meir, Liron Danielpur, Yuval Mor-Scheerer, Nitzan Dubovski, Tal Vana, Dagan Hadar, Anna Voropaev, Yair Fastman, Yanay Ofran / Biolojic Design
- 掲載誌/プレプリントサーバー: bioRxiv（未査読プレプリント）
- リサーチ日: 2026-09-28（JST）
- 分類: 新着 / 抗体設計 / 実験検証付きAIプラットフォーム

## 背景と問題設定

通常の単一特異性IgGは、左右対称な2本の腕、標準的なFc、確立した精製・製造工程を持ち、半減期や安定性も理解されている。一方、2つの標的を同時に扱う二重特異性抗体は、腫瘍の抗原不均一性、冗長なサイトカイン経路、免疫細胞リクルートなどに有利だが、非対称鎖、scFv/VHH付加、鎖誤対合対策などがCMC・凝集・免疫原性・PKの負担になる場合がある。

multibodyは構造の複雑化を避け、同じ6本のCDRが作る1つのFv表面に2つの結合様式を持たせる。各腕が局所濃度に応じてA/A、B/B、A/Bの結合状態を取り得るため、単一標的しかない細胞でも二価結合を保ちやすい。反面、2つの結合面は同じ残基集合を共有または近接して使う可能性があり、多目的最適化は強いトレードオフを持つ。歴史的にはHER2/VEGF two-in-one抗体などの成功例があったが、偶発的・個別最適化に依存し、任意の標的対へ展開できる設計法は確立していなかった。

本研究の問題設定は、(1) 二重結合ヒットを作る、(2) AとBの親和性を同時に改善する、(3) polyreactivity、疎水性、熱安定性、発現を薬剤候補範囲に保つ、(4) 親和性だけでなく目的とする作用機序を成立させる、という4条件を現実的な期間で満たすことである。

## この論文のコアアイデア

核心は、万能な1モデルで一発生成することではなく、キャンペーン固有の実験データを逐次集め、標的Aと標的Bの応答面を別々に学習し、その交差領域から両方に強く、かつdevelopability制約を満たす配列を選ぶ閉ループにある。

1. 標的、望ましいエピトープ、作用機序、抗体特性を専門家が入力し、独自AIが多様な配列ライブラリを提案する。
2. 提案配列を実験し、両標的に結合する弱〜中親和性の初期ヒットを数十件得る。
3. 有望ヒット周辺のライブラリを作り、A用とB用に別々にスクリーニングする。各標的の親和性予測器に加え、必要に応じて発現・安定性などの予測器も学習する。
4. 2つの親和性とdevelopabilityが両立すると予測された配列を探索し、IgGとして発現・精製・BLI/SPR/ELISA/flow cytometry、BVP/HIC/DSF、機能アッセイで確認する。

この分解により「両標的へ強い完成品が最適化ライブラリ内に最初から存在する必要」はない。各標的に有益な変異の情報を別々に学び、後段で多目的に組み合わせる点が実務的である。ただし、ニューラルネットワークか木モデルか、入力表現、獲得関数、配列探索法、ライブラリ規模は非公開である。

## 手法の詳細

- 入力データ: 初期抗体配列、標的A/Bへの高スループット結合測定、専門家が指定する標的・エピトープ・作用機序・望ましい物性。構造情報を明示的に入力したかは未確認。
- 出力: 両標的への親和性、発現、安定性、特異性を満たすと予測された抗体変異配列と、実験選抜された標準IgG候補。
- モデル/アルゴリズム: 独自AIモデル。初期提案器、標的別親和性予測器、任意の物性予測器、多目的配列探索から成ることは示されるが、アーキテクチャは未確認。
- 特徴量・表現学習: 未確認。抗体言語モデル、one-hot、構造特徴などの別は開示されていない。
- 学習方法: キャンペーン内の実験スクリーニング値を用い、標的ごとにモデルを学習。unseen dataで予測値と実測値が相関すると図示されるが、分割法と相関係数は未確認。
- 損失関数・目的関数: 未確認。実務上はA親和性、B親和性、developabilityの多目的最適化だが、重み付けやPareto選択は非開示。
- 推論方法: 変異候補のスコアリングと配列空間探索。探索アルゴリズム、変異数制約、ヒト生殖系列制約は未確認。
- ベースライン: 臨床段階または承認済みの単一特異性抗体。計算モデル同士の比較やランダム/頻度ベース設計との比較はない。
- 評価指標: BLI/SPRのKd、ELISA/flow cytometryのEC50、BVP結合、HIC保持時間、DSF Tm1、内在化蛍光、LDHによるT細胞依存性細胞傷害、TARC（CCL17）分泌。
- 実装上の重要点: 親和性はELISA-EC50、SPR、flow-EC50が混在し、#5–6はyeast display、それ以外は原則IgGで測定される。値の単純な横比較はできない。最終候補はIgGフォーマットで確認する。

## データセットと評価設計

公開データセットではなく、各キャンペーンで生成した独自の配列–実験値データを使う。bioRxiv要旨は9キャンペーン・15非関連標的、公開技術報告本文は先行版として7キャンペーン・11標的を記載する。標的にはGPCR、膜タンパク質、小型サイトカインが含まれる。詳細公開版ではTrop-2/Nectin-4を異なるエピトープ・作用機序で狙う2キャンペーン、IL-13/TSLPを遮断する候補が説明される。

train/validation/testの件数、配列同一性クラスタ分割、同一クローン系列の分離、時系列分割は未確認である。Figure 3Bはunseen dataで予測と実測が相関するとするが、サンプル数、R²/Spearman、反復、誤差棒は本文から確認できない。従って「キャンペーン内で候補順位付けに使える」ことは支持される一方、「未知標的へそのままゼロショット一般化するモデル」とは解釈できない。

developabilityは4キャンペーン由来のAI提案約200分子をIgGでBVP、HIC、DSF評価した。BVPは1,000 nMで非特異結合、HICは疎水性、nanoDSFは0.5 mg/mL、25–95°C、1°C/minでTm1を見る。機能評価は、24時間のpH感受性色素内在化、PBMCと標的発現HEK293の48時間LDH killing、IL-13/TSLP刺激PBMCの48時間TARC ELISAである。

評価は「実際に作って測る」点で強いが、盲検、事前登録、独立ラボ、統計検定、ランダム設計対照、失敗キャンペーンの記載が乏しい。臨床ベンチマークとの比較もアッセイ内比較として有用だが、PK、毒性、免疫原性、製造スケールの直接比較ではない。

## 主要結果

### 設計成功と親和性改善

- bioRxiv要旨では9件の独立キャンペーン、15の非関連標的対で目的のdual binderを得たと報告する。公開技術報告の先行版では7/7キャンペーン成功とされる。
- 代表例では初期dual binderが標的Aに4 nM、標的Bに80 nMだった。最適化後は両標的とも一桁nMまたはsub-nMの複数候補となり、標的によって最大約100倍改善した。
- 代表multibodyの結合曲線は、各標的に対する臨床単一特異性抗体と同等、場合によっては上回った。ただし7/9全件の完全な生データ、統計量、配列は非公開である。

### developability

- 4キャンペーン、約200のAI提案IgGについてBVP、HIC、DSFを測り、「多く」が良好な範囲に入った。
- Multibody #1は非特異結合、疎水性、熱安定性が臨床ベンチマークの許容域で、いくつかの市販抗体より良好と報告された。
- ただし合格率、閾値、分布の数表、発現収量、濃縮安定性、粘度、長期保存、in vivo PKは十分に開示されていない。「天然型IgGだから必ず良い物性」と一般化するのは早い。

### 3種類の作用機序

- Multibody #1（Trop-2/Nectin-4、ADC用途）は両標的共発現細胞でsacituzumab/enfortumab系の単一標的ベンチマークより高い内在化を示した。両腕がどちらの標的にも結合できるため、一方の発現が低下しても残る標的へ二価架橋できる、という機序仮説が提示される。
- Multibody #2（同じTrop-2/Nectin-4だが別エピトープ）はanti-CD3とDuobody化したT細胞エンゲージャーの腫瘍側アームとして、Trop-2発現細胞とNectin-4発現細胞の両方で用量依存的な細胞傷害を誘導した。
- Multibody #7（IL-13/TSLP）は各サイトカイン単独刺激ではtezepelumab、tralokinumab/lebrikizumab相当、両サイトカイン同時刺激では完全なTARC抑制を維持し、単一特異性対照の部分抑制を上回った。これは局所のドライバーが変化する炎症疾患に対する多経路遮断の価値を示す。
- 2候補がIND-enabling study段階、構想からIND-enabling候補まで約9か月と報告される。ただし臨床効果を示す結果ではなく、first-in-human予定は将来計画である。

## 重要なFigure/Table

### Figure 3: AI–実験閉ループと代表親和性改善

- 何を示しているか: AI提案→実験ヒット→標的別モデル→多目的最適化→IgG実験確認の4段階、および4 nM/80 nMの初期分子から一桁nM/sub-nMへの改善。
- 読み取り方: 横軸を工程、二次元の親和性空間をAとBへの性能として見る。片方だけ良い点ではなく、両方が良い領域へ候補を移すことが目的。
- 主要な数値・傾向: 最大約100倍改善、最終候補はsingle-digit nMまたはsub-nM。
- なぜ重要か: モデル名よりも、この閉ループのデータ獲得設計が成果の中核だから。
- Markdown内での扱い: 独自の再構成フロー（文章）
- HTML内での扱い: 独自の4段階フローと親和性改善チャート
- 出典URL: [技術報告PDF](https://biolojic.com/wp-content/uploads/2026/05/Biolojic-Design-Multibodies-technical-report-1.pdf)
- ライセンス確認: 不明。原図は保存・転載しない。

### Figure 5: 約200 IgGのdevelopability

- 何を示しているか: 4キャンペーン由来候補のBVP、HIC、DSF分布と、Multibody #1の臨床抗体比較。
- 読み取り方: BVPは低いほど非特異結合が少なく、HIC保持は過度な疎水性を避け、Tm1は高いほど熱安定性の目安になる。ただし単一指標で開発適性は決まらない。
- 主要な数値・傾向: n≈200。多くが良好とされるが、合格率と閾値の数表は未開示。
- なぜ重要か: dual specificityの獲得がpolyreactivity増加と物性悪化を招いていないかを直接問う図だから。
- Markdown内での扱い: 要約表
- HTML内での扱い: 三指標カードと「確認済み/未開示」の区別
- 出典URL: [技術報告PDF](https://biolojic.com/wp-content/uploads/2026/05/Biolojic-Design-Multibodies-technical-report-1.pdf)
- ライセンス確認: 不明。原図は保存・転載しない。

### Figure 6: 3作用機序の機能検証

- 何を示しているか: ADC内在化、二標的T細胞傷害、IL-13/TSLP同時刺激時のTARC抑制。
- 読み取り方: 単にA/Bへ結合したかではなく、設計時に指定した機能が細胞系で発現したかを見る。
- 主要な数値・傾向: #1は単一標的対照より高い内在化、#2は標的A/B各発現細胞で用量依存的傷害、#7は二重刺激で完全抑制を維持。
- なぜ重要か: 二重特異性の薬理学的意味を、異なる治療モダリティで示すため。
- Markdown内での扱い: 独自の要約表
- HTML内での扱い: 3列の作用機序カードと比較図
- 出典URL: [技術報告PDF](https://biolojic.com/wp-content/uploads/2026/05/Biolojic-Design-Multibodies-technical-report-1.pdf)
- ライセンス確認: 不明。原図は保存・転載しない。

| 検証軸 | multibody | 比較対象 | 読み取れること |
|---|---|---|---|
| 親和性 | 2標的とも一桁nM/sub-nM例 | 臨床単一特異性抗体 | dual化が必ずしも各標的の結合を犠牲にしない |
| 物性 | BVP/HIC/DSFで多くが良好 | 市販・臨床IgG | 標準IgGフォーマットの利点を保持する可能性 |
| ADC内在化 | #1で増強 | Trop-2またはNectin-4対照 | 二価結合と標的不均一性への適応が有効かもしれない |
| TCE | #2がA/B各細胞を傷害 | 単一TAA認識 | どちらかの腫瘍抗原があれば機能可能 |
| サイトカイン遮断 | #7が二重刺激で完全抑制 | 各単一特異性抗体 | 冗長な炎症経路を1分子で覆える |

## Code / License / Weights

- コード公開: 見つからず
- GitHub URL: 該当なし
- GitHub以外のコードURL: 該当なし
- 実装種別: 独自・非公開
- GitHub確認: GitHub検索および一般Web検索を確認したが論文公式リポジトリは見つからず
- リポジトリのライセンス: 該当なし
- ライセンス確認元: 該当なし
- Hugging Face URL: 該当なし
- Hugging Face種別: 該当なし
- モデル重み・チェックポイント公開: 見つからず
- 重み公開URL: 該当なし
- データセット公開: 見つからず
- データセットURL: 該当なし
- 再現性メモ: 論文ページ、Biolojic Designサイト、GitHub、Hugging Face、Zenodoをタイトル、DOI、著者名、企業名、multibodyで検索した。公開されたのはbioRxivページと12ページの技術報告で、配列、学習用測定値、モデル、重み、split、最適化コードは見つからなかった。論文本体の再利用ライセンスも確認できなかったため、原Figureは保存せず、HTMLでは事実と報告値のみを独自構成で再可視化した。

## 抗体研究・創薬への意味

第一に、AI抗体設計の価値が「新規配列を大量生成すること」だけでなく、ウェット実験をどの順番で行い、次のライブラリへどう情報を戻すかにあると示す。標的対ごとにデータを集めるキャンペーン内学習は、汎用基盤モデルのゼロショット性能が不十分でも産業価値を作れる。

第二に、Fv単位で多重特異性を持たせると、標準IgGの製造性と二価性を残しながら、腫瘍抗原の不均一性や冗長なサイトカイン経路に適応できる可能性がある。左右異なる二重特異性抗体では一方の標的しかない細胞に対して実質一価になり得るが、multibodyは同一標的へ両腕を使える。

第三に、親和性だけを目的にせず、エピトープ、内在化、免疫シナプス、サイトカイン遮断、polyreactivity、疎水性、熱安定性を同時に扱っている。今後の抗体AI評価は配列回収率やDockQだけでなく、作用機序に直結する細胞機能とdevelopabilityまで含むべきだと分かる。

## 限界と注意点

### 著者が認識している課題

- 2つのパラトープを同じFvへ収めつつ、一方の最適化で他方を壊さず、polyreactivityを増やさないことが本質的に難しい。
- キャンペーンによりパイプラインの旧版と現行版が混在する。
- 一部親和性はIgGではなくyeast displayで測られ、ELISA、SPR、flow cytometryの異種指標が混在する。

### この精読での追加評価

- AIの入力表現、アーキテクチャ、学習件数、損失、探索アルゴリズムが非公開で、計算手法を再現できない。
- 配列、生データ、陰性例、候補の脱落数、選抜閾値がなく、9/9成功という主張に選択バイアスがないか評価できない。
- unseen dataの作り方が不明で、近縁変異がtrain/testに跨るリークを排除できない。
- 「臨床水準」「excellent developability」は限定されたin vitro試験に基づく。高濃度粘度、凝集、化学安定性、免疫原性、PK、毒性、製造スケールは公開結果から十分確認できない。
- 内在化、T細胞傷害、TARCは機能的に重要だが、詳細な反復数、統計、有害作用、in vivo disease modelは不足する。
- bioRxiv要旨の9キャンペーン・15標的と、入手できた先行技術報告本文の7キャンペーン・11標的に版差がある。数値を混同せず、最新集計と詳細公開版を分けて読む必要がある。
- 企業著者のみの報告で、特許・営業秘密の制約が強い。独立ラボによる再現と査読を待つべきである。
- 「標準IgGだから免疫原性や半減期が必ず優れる」という因果は未証明であり、配列改変そのもののリスクは残る。

## この論文を読む上での前提知識

- **FvとCDR**: VHとVLが作る可変領域がFvで、6本のCDRが主に抗原表面を認識する。multibodyでは同じFv表面が2種類の抗原を認識する。
- **bispecificとtwo-in-one**: 通常のbispecificは異なる結合部位を分子内に並べる。two-in-oneは1つの結合部位が2標的を認識するため、標的は同時ではなく競合的に同じFvへ結合する。
- **親和性とアビディティ**: Kdは1つの相互作用の強さ、アビディティは多価結合全体の見かけの強さ。各腕がA/B双方に使えることがmultibodyのアビディティ上の特徴である。
- **BVP assay**: baculovirus particleへの非特異結合を見てpolyreactivityリスクを推定する。低い方が一般に望ましい。
- **HICとDSF**: HIC保持は表面疎水性、DSF/nanoDSFのTmは熱安定性の指標。いずれもdevelopabilityの一部であり、単独で臨床適性を保証しない。
- **閉ループactive learning**: モデルが候補を出し、実験結果を再学習へ戻す反復。探索空間が巨大でラベルが高価な抗体工学と相性が良い。
- **TARC/CCL17**: Th2炎症に関係するケモカイン。IL-13とTSLP刺激の下流 readout として用いられる。

## 今回この1報を選んだ理由

前回のIgGM2は構造・配列生成の基盤モデルだったのに対し、本稿はAIを実験閉ループに組み込み、ADC、TCE、サイトカイン遮断という3つの薬理作用へ接続している。新規性は「多標的抗体」そのものではなく、過去には個別例だったtwo-in-one IgGを複数キャンペーンへ体系展開した点にある。コード公開の価値はない反面、公開モデルだけを追うと見落としやすい、産業現場の多目的最適化・CMC・機能検証を学べる。最新要旨で9キャンペーン・15標的、2候補がIND-enablingとされ、翻訳研究上のインパクトが高いため採用した。

## 読む優先度

**High**。AI抗体設計を、配列生成ではなく実験データ獲得、標的別予測、多目的選抜、作用機序、developabilityまで一気通貫で見る教材として価値が高い。一方、モデルとデータが非公開なので、アルゴリズムを再実装したい読者は方法論論文ではなくケーススタディとして読むべきである。

## 自分用メモ

- 後で深掘りしたい点: bioRxiv本文の9キャンペーン版で追加された2キャンペーン、各モデルのsplit、Figure 3Bの相関係数、200候補の合格率。
- 関連して読むべき論文: Bostrom et al., Science 2009（HER2/VEGF two-in-one）、Schaefer et al., Cancer Cell 2011（HER3/EGFR）、Beckmann et al., Nat Commun 2021（DutaFab）、A blinded prospective AIntibody benchmark 2026。
- 実装を触る場合の入口: 公開実装はない。代替として、標的別回帰器2本＋BVP/HIC/Tm予測器を作り、Pareto frontierまたは制約付きBayesian optimizationで再現する。
- Obsidianでリンクしたいキーワード: [[bispecific antibody]], [[two-in-one antibody]], [[multibody]], [[active learning]], [[multi-objective optimization]], [[antibody developability]], [[polyreactivity]], [[ADC]], [[T-cell engager]]

## 関連キーワード

- dual-specific antibody, two-in-one antibody, multibody, symmetric IgG, Fv, paratope overlap
- active learning, closed-loop optimization, affinity maturation, multi-objective optimization
- Trop-2, Nectin-4, IL-13, TSLP, ADC, T-cell engager, TARC
- BVP, HIC, DSF, developability, polyreactivity

## 検索ログ

- 検索したデータベース/クエリ: bioRxiv、PubMed、arXiv、Crossref相当のWeb検索。`antibody machine learning September 2026`、`antibody design AI 2026`、`Dual-Specific Antibody Design Using Artificial Intelligence`。
- 候補にした論文: 本稿、SimBinder-IF、AbTune、Application of protein language models for antibody developability prediction、Frozen Protein Foundation-Model Embeddings関連、新着タンパク質binder設計。
- 最終的にこの1報を採用した理由: two-in-one IgGを複数標的対へ展開し、親和性、物性、3作用機序を実験確認した翻訳価値が高い。前回の基盤モデルと補完的。
- 新着論文と基盤論文のバランス: 今回は2026-08公開の新着を選択。歴史的位置づけのため2009/2011年のtwo-in-one基盤例を併読した。
- PDF/HTML/Supplementaryを確認できたか: bioRxiv要旨を確認。Biolojic Design公開の12ページ技術報告PDFを全文確認。Supplementaryは見つからず。
- Figure/Tableを確認したURLやライセンス確認状況: 技術報告PDFのFigure 1–6、Table 1を確認。再利用ライセンスは確認できず、原図は保存しなかった。
- GitHub/コード検索で使ったクエリ: 論文タイトル、DOI、`Biolojic Design multibody GitHub`、著者名、`two-in-one antibody AI code`。
- 確認したGitHub URL: 公式該当リポジトリは見つからず。
- 確認したHugging Face URL: 公式model/datasetは見つからず。
- 確認したその他コード/重み/データURL: [bioRxiv](https://www.biorxiv.org/content/10.64898/2026.08.03.742397v1)、[技術報告PDF](https://biolojic.com/wp-content/uploads/2026/05/Biolojic-Design-Multibodies-technical-report-1.pdf)、[Biolojic capabilities](https://biolojic.com/capabilities/)、[BD9/TEV-325情報](https://biolojic.com/teva-and-biolojic-design-initiate-ind-enabling-studies-of-bd9-a-multibody-designed-to-explore-the-treatment-of-atopic-dermatitis-and-asthma/)。
- リポジトリライセンスの確認元: 該当なし。
- モデル重み・チェックポイントの確認先: GitHub、Hugging Face、Zenodo、企業サイトを検索したが見つからず。

---

作成日: 2026-09-28（JST）
注: 本ノートは未査読プレプリントと企業技術報告に基づく。bioRxiv要旨と先行技術報告の集計差は版差として明示し、非公開事項を推測で補っていない。
