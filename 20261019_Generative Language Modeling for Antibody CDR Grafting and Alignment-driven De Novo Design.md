# Generative Language Modeling for Antibody CDR Grafting and Alignment-driven De Novo Design

> [HTML版](<20261019_Generative Language Modeling for Antibody CDR Grafting and Alignment-driven De Novo Design.html>)

## まず何の論文か

抗体のCDRはフレームワーク領域の間に挟まれているため、固定した骨格に新しいCDRを設計する仕事は本質的に「穴埋め」である。しかし通常の自己回帰モデルはN末端からC末端へ読むので、CDR1を生成するときに下流のFR2–FR4を参照できない。この論文は、4つのフレームワークを先にすべて並べてプロンプトとし、その後にCDR1–3を可変長の応答としてまとめて生成する **GenCDR** を提案する。モデルは約25Mパラメータのdecoder-only LLaMAで、unpaired抗体用IgGenCDR、paired VH–VL用p-IgGenCDR、VHH用NanoGenCDRの3系列からなる。入力順序を変えることで、全フレームワークを見ながら複数CDRを同時に設計でき、フレームワーク尤度とCDR尤度も分離できる。CDR-disjoint testでは、IgGenCDRは公表済み自己回帰モデルより6ループすべてで高い回復率を示した。NanoGenCDRはhuman OASとcamelid INDIを混ぜることで、VHHらしさを維持しつつhumanised scaffold上のCDR humannessを改善した。さらに、フレームワークを固定prompt、CDRをresponseとみなしてweighted DPOを適用し、PD-L1とのBoltz-2 co-folding指標と配列liabilityを報酬にした。4ラウンドの整列でaccept率は23.8%から99.8–100%へ上がり、interface指標は構造条件付きBoltzGenと競合する水準に達した。ただし結合・発現・安定性の実験検証はなく、成果はすべて配列統計またはin silico構造予測である。コードはApache-2.0で公開済みだが、2026-10-19確認時点でモデル重みはREADME上「今後公開」とされており、論文本文の公開済みという記載とは一致しない。

## 書誌情報

- URL/DOI: [bioRxiv](https://www.biorxiv.org/content/10.64898/2026.09.06.749721v1) / [10.64898/2026.09.06.749721](https://doi.org/10.64898/2026.09.06.749721)
- 公開日/更新日: 2026-09-10（v1）
- 著者・所属: Ferran Gonzalez Hernandez, Oliver M. Turnbull, Maliha Sultana, Lorena Roldán-Martín, Rohan J. Kumar, Tom Diethe, Rebecca Croasdale-Wood, Charlotte Deane, Dino Oglic。AstraZeneca AI for Discovery Science、University of Oxford Department of Statistics、AstraZeneca Biologics Engineering。
- 掲載誌/プレプリントサーバー: bioRxiv（査読前）
- リサーチ日: 2026-10-19（JST）
- 分類: 新着 / モデル起点

## 背景と問題設定

抗体の抗原認識を担うCDRは、保存性の高いFR1–FR4の間に分散している。既存のleft-to-right自己回帰モデルでは、上流CDRを出す時点で下流フレームワークを使えず、CDRとフレームワークの尤度も混ざる。IgLMは任意の1 spanをマスクして周囲から補完できるが、3 CDRを同時に未知とした共同設計には自然に拡張できない。RFantibody、BoltzGen、Germinalのような構造ベース法は抗原形状を直接使える一方、学習・推論で構造を要求し、設計長を事前指定しやすい。著者らの問いは、配列だけで大規模事前学習できる自己回帰モデルに、全フレームワーク文脈、可変長multi-CDR生成、CDRだけへのreward attributionを同時に与えられるか、である。

## この論文のコアアイデア

自然な並び `FR1-CDR1-FR2-CDR2-FR3-CDR3-FR4` を、そのまま自己回帰で読むのをやめる。代わりに `FR1|FR2|FR3|FR4 <HPRED> CDR1|CDR2|CDR3` と並べ、前半を固定prompt、後半を生成responseとする。pairedモデルではVH/VL両方のフレームワークを先に提示し、その後に両鎖のCDR blockを生成する。この因数分解により、各CDRは全フレームワークを参照でき、response tokenだけの平均log-likelihood ratioをweighted DPOに使える。

PD-L1設計では、4種のhumanised VHH scaffoldごとにCDRを生成し、長さと6種類のliability motifでscreenし、合格配列だけをBoltz-2で抗原とco-foldする。報酬は `0.3×ipSAE + 0.2×pTMchain + 0.2×fCDR + 0.15×(1−phelix) + 0.15×(1−pstrand)`。同じscaffold内で応答を比較するweighted DPOを4ラウンド繰り返し、抗原を直接入力しない配列生成器に構造評価器からのフィードバックを学ばせる。

## 手法の詳細

- 入力データ: ANARCIでIMGT/Kabat/Chothia境界を付けた抗体・VHH配列。番号付け方式は例ごとに均等サンプリング。
- 出力: 固定フレームワークに適合する可変長CDR1–3。paired版はVH/VLの6 CDRを共同生成。
- モデル/アルゴリズム: decoder-only LLaMA、4層、hidden 768、FFN 2,048、12 attention heads / 4 KV heads、RoPE、RMSNorm、SwiGLU、約25M parameters、context 1,024、residue-level vocabulary 48。
- 特徴量・表現学習: 配列のみ。抗原配列・抗原構造は事前学習時も通常生成時も入力しない。抗原情報はalignment時のBoltz-2報酬を通じて間接的に入る。
- 学習方法: IgGenCDRは10 epochs。80% single chain、20% random H–L pair。p-IgGenCDRとNanoGenCDRはbase checkpointから5 epochs fine-tuning。
- 損失関数・目的関数: 事前学習はframeworkを含む全tokenのcross entropy。整列はscaffold-grouped weighted DPO、β=0.15、CDR completion tokenだけでpolicy/referenceのlog-ratioを計算。
- 推論方法: top-p=0.95、温度を調整して同一frameworkから複数CDRをsample。論文はT=0.90–1.15を自然さと新規性の実用的範囲とする。
- ベースライン: 同サイズLTR ablation、p-IgGen、ProGen2系列、IgLM/IgLM-S。nanobody alignmentではBoltzGen + BoltzIF + Boltz-2。
- 評価指標: CDR next-token recovery、OASis 9-mer humanness、ESM-2/AbLang PLL、training setへのHamming distance、FLAb/NbBench Spearman ρ、NanoBodyBuilder2 confidence、Therapeutic Nanobody Profiler、ipSAE、pTMchain、paratopeのCDR fraction、CDR secondary-structure penalty、accept率。
- 実装上の重要点: 事前学習は8×A100 40GB、BF16、effective batch 1,024、AdamW、peak LR 2×10^-4。alignmentは各round 5,000 ACCEPT、10 epochs、single A100、LR 5×10^-5、batch 512。co-foldはBoltz-2、5 diffusion samples、200 steps、3 recycles。

## データセットと評価設計

| 用途 | データ | 規模・分割 | 評価上の意味 |
|---|---|---|---|
| unpaired事前学習 | human OAS由来p-IgGen data | 約254M（heavy約118M、light約136M） | 大規模なヒト抗体文法 |
| unpaired test | 再構成したCDR-disjoint set | heavy 202,138、light 31,014 | train CDR3と長さ一致かつidentity ≥95%のtest配列を除去 |
| paired fine-tuning | paired OAS | 約1.8M、test約69.8K pair | VH–VL相互依存を評価 |
| VHH fine-tuning | INDI/INDI2 + human OAS heavy | 非冗長camelid約6.5Mとhumanをbatch内1:1 | humanised VHHでのhumanness保持 |
| fitness | FLAb | Marks n=217、Koenig n=4,276 | immunogenicity/expressionのzero-shot順位相関 |
| nanobody fitness | NbBench | thermo-seq n=764、thermo-tm n=567 | 熱安定性のzero-shot順位相関 |
| antigen alignment | PD-L1、4 VHH frameworks | GenCDR 4×5,000、BoltzGen 20,000 co-fold | 同じco-fold budgetと共通filterで比較 |

通常のwhole-domain 95% cluster splitではCDRが跨いで重複しうるため、著者らはtrain CDR3とのlength-matched identity ≥95%を除去した。これによりraw splitよりH-CDR3回復率が約13 points下がり、元評価のleak影響を定量化しているのは強い点である。ただし全CDRのcluster-disjointではなく、主にCDR3を基準にした除外である。alignment比較はscaffold、CDR長制約、liability rule、Boltz-2設定、co-fold数を揃えているが、評価報酬と報告指標が同じであるため、独立評価ではない。

## 主要結果

IgGenCDRのIMGT recoveryはH-CDR1/2/3が91.47/89.81/49.75%、L-CDR1/2/3が85.63/84.25/58.54%。同サイズLTRに対して5/6 loopで上回り、とくにH-CDR1 +2.91 points、L-CDR1 +3.44 pointsだった。IgLM CDR infillに対してH-CDR3は+10.66 points、L-CDR3は+5.37 pointsである。paired fine-tuningはbaseからH-CDR3を+1.80 points、L-CDR3を+3.20 points改善し、p-IgGenで見られる生成方向依存を小さくした。

NanoGenCDRのscheme平均recoveryはCDR1/2/3で79.03/78.90/56.96%で、human OASだけのbase（39.68/43.03/13.34%）を大きく上回る。humanised VHH 5 scaffold上のCDR OASis平均はOAS混合モデル0.471、INDI-only 0.410（p<0.001）。この混合はcamelid recoveryを1.35 points失う一方、human OAS recoveryを44.19%から80.31%へ36.12 points保持する。NbBenchのframework scoreはthermo-seq ρ=0.331、thermo-tm ρ=0.325で比較モデル中最高だった。

PD-L1 alignmentではaccept率がround 0の23.8%からround 1の99.8%、round 2–3の100%へ上昇した。同じ20,000 co-foldを埋めるraw generationはGenCDR約36,000、BoltzGen約63,000で約43%少ない。round 0→2でreward中央値は0.715→0.779、ipSAE 0.404→0.530、fCDR 0.828→0.943。BoltzGen中央値はipSAE 0.474、fCDR 0.791、reward 0.719で、GenCDR round 2はこれらを上回る一方、pTMchainは0.975対0.978、helix suppressionは0.778対0.811でBoltzGenがわずかに高い。top 5%ではround 2のipSAE 0.745、reward 0.859で、BoltzGenの0.741、0.845に並ぶか上回った。

## 重要なFigure/Table

### Figure 1: framework-first生成とalignment loop

- Figure/Table番号: Figure 1
- 何を示しているか: 自然配列順とGenCDR順序の差、3モデル系列、generate→filter→co-fold→reward→wDPOの反復。
- 読み取り方: 抗原条件付けは生成器への直接入力ではなく、co-fold rewardを介した反復更新で入る点に注目する。
- 主要な数値・傾向: 254M OAS、1.8M paired OAS、6.5M INDI + human OAS。
- なぜ重要か: 論文の新規性を1枚で表す。
- Markdown内での扱い: リンクと本文要約。
- HTML内での扱い: 原図をトレースしない独自フロー図。
- 出典URL: [bioRxiv full text](https://www.biorxiv.org/content/10.64898/2026.09.06.749721v1.full)
- ライセンス確認: CC BY 4.0確認済み。

### Table 1: CDR回復率

- Figure/Table番号: Table 1
- 何を示しているか: CDR-disjoint test上の6 loopのnext-token recovery。
- 読み取り方: H/LそれぞれCDR1–3を列方向に見て、framework-firstと同サイズLTRおよび既存ARを比較する。
- 主要な数値・傾向: IgGenCDRは91.47/89.81/49.75%（H）、85.63/84.25/58.54%（L）。6 loopすべてで公表済みAR baselineを上回る。
- なぜ重要か: 新しい入力順序が単なる実装上の工夫ではなく、下流文脈を必要とするloop回復に効くことを示す。
- Markdown内での扱い: 必要列を再構成。
- HTML内での扱い: 主要モデルのみの比較表。
- 出典URL: [bioRxiv PDF](https://www.biorxiv.org/content/10.64898/2026.09.06.749721v1.full.pdf)
- ライセンス確認: CC BY 4.0確認済み。

### Table 3 / Figure 4: PD-L1 alignment

- Figure/Table番号: Table 3、Figure 4
- 何を示しているか: 各roundのaccept率・必要生成数と、構造報酬5成分・複合報酬の分布。
- 読み取り方: round 0→2でinterface関連2成分が伸び、foldability/geometryはほぼ維持されることを見る。
- 主要な数値・傾向: accept 23.8→99.8→100%、ipSAE 0.404→0.530、fCDR 0.828→0.943、reward 0.715→0.779。生成予算は約36k対63k。
- なぜ重要か: prompt–response分離が、CDR限定のpost-trainingとfilterの内在化につながることを示す。
- Markdown内での扱い: 数値表と詳細解説。
- HTML内での扱い: 独自bar chartとfunnel比較。
- 出典URL: [bioRxiv full text](https://www.biorxiv.org/content/10.64898/2026.09.06.749721v1.full)
- ライセンス確認: CC BY 4.0確認済み。

## Code / License / Weights

- コード公開: あり
- GitHub URL: https://github.com/AstraZeneca/GenCDR
- GitHub以外のコードURL: なし
- 実装種別: 公式
- GitHub確認: 確認済み（2026-10-19）
- リポジトリのライセンス: Apache-2.0
- ライセンス確認元: GitHubのLICENSEファイル / README / repository metadata
- Hugging Face URL: https://huggingface.co/AstraZeneca （organizationのみ。公開model 0、dataset 0）
- Hugging Face種別: organization
- モデル重み・チェックポイント公開: 見つからず
- 重み公開URL: なし。READMEは「Model weights and download instructions will be added here」と記載。
- データセット公開: あり
- データセットURL: [p-IgGen/OAS-derived data, Zenodo 10.5281/zenodo.13880874](https://zenodo.org/records/13880874)、[INDI2](https://naturalantibody.com/indi2/)
- 再現性メモ: inference、scoring、weighted DPOのコードとhyperparameterは公開されているが、現時点で初期checkpointがないためREADMEの`from_pretrained("iggencdr")`をそのまま再現できない。論文Data availabilityは「trained models are available」と書くが、現行READMEとAstraZeneca Hugging Faceには重みがない。INDI/INDI2は非商用研究向け条件があり、企業利用は問い合わせが必要。

## 抗体研究・創薬への意味

最も重要なのは、CDR graftingをLLMのprompt–response問題として明示的に定式化した点である。これにより、結合、発現、安定性、免疫原性、liabilityなど異種のscalar assayをresponse側だけへ割り当てられる。paired版ならVH/VL両鎖を固定または共同設計でき、VHH版では人化とcamelid特性のtrade-offをtraining mixtureで調整できる。実験ループに接続する場合、roundごとのrewardをBoltz-2だけでなくSPR/BLI、発現量、SEC、HIC、Tmに置き換え、同一scaffold内で比較するclosed-loop affinity maturationへ拡張できる。一方で抗原を直接encodeしないため、標的が変わるたびに多数のco-foldまたは実験feedbackが必要で、zero-shot epitope-specific designモデルではない。

## 限界と注意点

**著者が述べる限界:** alignment手法はweighted DPOのみで、online RLや別offline objectiveとの比較がない。抗原はPD-L1の1種類、frameworkは4種類だけである。最終designはすべてin silicoで、発現・安定性・結合の実験確認がない。reward上位ではbounded metricが飽和し、softmax targetが平坦になるためtail改善が小さい。

**精読して気づいた限界:** Boltz-2由来のipSAE等を学習報酬と評価に共用しており、同一oracleへの過適合を独立に否定できない。BoltzGen比較はco-fold数を揃えるが、事前学習・構造生成・inverse foldingの総計算量を同じ尺度で比較していない。「約半分のsampling budget」はraw generation数の比較でありwall-clockやGPU-hourではない。CDR recoveryは自然配列の再構成能力であり、抗原結合能そのものではない。CDR3 leakage対策は評価を改善するが、他CDRやgermline/framework近縁性を完全排除したsplitではない。25Mモデルで扱いやすい一方、OAS 254M配列を10 epochs学習する計算資源は大きい。重み未公開のため完全再現性はまだ不足し、INDIデータの非商用条件は創薬企業での再利用に注意が必要である。

## この論文を読む上での前提知識

- **CDRとframework:** VH/VLまたはVHHの可変領域は、比較的保存されたFRと抗原接触が集中するCDRからなる。IMGT/Kabat/Chothiaで境界が異なる。
- **自己回帰言語モデル:** 既出tokenから次tokenを予測する。順序により利用できる文脈が変わるため、抗体配列の並べ替えが性能と用途を変える。
- **Direct Preference Optimization:** rewardの高い応答を参照policyより相対的に高尤度にするpost-training。本研究は連続rewardをsoftmax化するweighted DPOを使う。
- **ipSAE / pTM:** 構造予測のPAEやfold confidenceから作るin silico指標で、実測affinityではない。
- **OASis:** 抗体repertoire由来9-merとの一致を使うhumanness指標。高いほどヒト抗体に典型的な局所配列が多い。
- **developability liability:** deamidation、isomerization、unpaired cysteine、CDR N-glycosylationなど、製造・安定性・不均一性のリスクとなるmotif。

## 今回この1報を選んだ理由

前回のCA-MAPが物性予測だったため、今回は生成・post-training側を補完する新着論文を選んだ。GenCDRは、入力順序という小さく見える変更を、unpaired、paired、VHH、antigen alignmentまで一貫して検証している。コードは公式かつApache-2.0で、CDR-disjoint split、同サイズablation、共通co-fold budgetなど評価設計にも学習価値がある。とくに、実験値を将来rewardに差し替えられるprompt–response境界は、抗体のclosed-loop最適化を考えるうえで重要である。重み未公開という再現性上の欠点も含め、追跡価値が高い。

## 読む優先度

**High。** 抗体言語モデルの生成順序、CDR-specific scoring、paired生成、DPO整列を1つの設計原理でつなぐ。実験検証前なので性能主張は割り引く必要があるが、実装と評価設計から学べる内容が多い。

## 自分用メモ

- 後で深掘りしたい点: checkpoint公開後、同一frameworkでIgGenCDR likelihoodとAbLang/OASisの相関を検証する。
- 関連して読むべき論文: IgLM、p-IgGen、BoltzGen、Germinal、Aligning protein generative models with experimental fitness via DPO。
- 実装を触る場合の入口: GitHubの`GenCDRGenerator.from_pretrained`、`gencdr score`、`gencdr align`。ただし重み公開待ち。
- Obsidianでリンクしたいキーワード: [[CDR grafting]]、[[antibody language model]]、[[weighted DPO]]、[[Boltz-2]]、[[nanobody humanization]]、[[developability]]。

## 関連キーワード

- GenCDR, IgGenCDR, p-IgGenCDR, NanoGenCDR, CDR infilling, framework-first, autoregressive model, OAS, INDI, weighted DPO, PD-L1, BoltzGen, Boltz-2, OASis

## 検索ログ

- 検索したデータベース/クエリ: bioRxiv、PubMed、arXiv、Nature、Google相当Web検索。`antibody artificial intelligence machine learning October 2026`、`antibody design machine learning October 2026`、`GenCDR antibody`、`GenCDR model weights`、Hugging Face上の`IgGenCDR/NanoGenCDR/p-IgGenCDR`。
- 候補にした論文: 本論文、`A foundation model learns the sequence and functional grammar of fully human heavy-chain-only antibodies`、`Automation and Active Learning for the Multi-Objective Optimization of Antibody Formulations`。前者はspecialised HCAb foundation model、後者は製剤active learningとして有用だが、GenCDRは抗体設計への直接性、コード、paired/VHH/alignmentの広さで優先した。
- 最終的にこの1報を採用した理由: 新着、直接的な抗体生成、明確なablation、CDR-disjoint評価、公式コード、closed-loop設計への接続性。
- 新着論文と基盤論文のバランス: 今回は2026-09-10公開の新着を採用。比較対象としてIgLM、p-IgGen等の基盤モデルをノート内で整理した。
- PDF/HTML/Supplementaryを確認できたか: bioRxiv 27ページPDF（main + supplementary）を全文抽出し、Figure 1–4、Table 1–3、Figure/Table S1–S16を確認。Figure 1、3、Table 3、Figure 4のページをレンダリングして視認した。
- Figure/Tableを確認したURLやライセンス確認状況: bioRxiv HTML/PDF。各ページ上部とbioRxiv metadataでCC BY 4.0を確認。原図は保存せず、HTMLに独自のフロー、比較表、bar chartを作成した。
- GitHub/コード検索で使ったクエリ: `AstraZeneca GenCDR GitHub`、README、LICENSE、source tree、weights section。
- 確認したGitHub URL: https://github.com/AstraZeneca/GenCDR
- 確認したHugging Face URL: https://huggingface.co/AstraZeneca （公開model/datasetなし）
- 確認したその他コード/重み/データURL: https://zenodo.org/records/13880874 、https://naturalantibody.com/indi2/
- リポジトリライセンスの確認元: GitHub LICENSE、README、repository metadata（Apache-2.0）。
- モデル重み・チェックポイントの確認先: GitHub READMEのObtaining model weights、GitHub Releases、AstraZeneca Hugging Face organization、Web検索。2026-10-19時点では見つからず。
