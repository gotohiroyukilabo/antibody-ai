# Preferential CDR masking in paired antibody language models improves binding affinity prediction

## まず何の論文か

この論文は、抗体の可変領域配列をタンパク質言語モデルで表現するとき、全残基を同じ確率で扱うのではなく、抗原結合に直接関わるCDRに学習信号を集中させるべきだと示した研究である。著者らは、ESM2-3BとESM C 600Mを土台に、ペアになったVH/VL配列を入力として、whole-chain masking、CDR-focused masking、hybrid maskingを比較した。提案の中心は、抗体のフレームワーク領域は構造足場として保存性が高く、CDRは結合特異性を担う高多様性領域なので、通常のMLMのように全残基を一様にマスクすると、モデル容量が比較的簡単なフレームワーク復元に使われすぎる、という問題意識である。学習にはOAS由来のヒト抗体ペア配列をクラスタリングした約162万件を用い、評価には6種類の抗体/抗原系、合計9万件超の変異体結合データを用いた。最終モデルはAbCDR-ESM2とAbCDR-ESMCで、Hugging Faceでモデル重みが公開されている。主要結果として、CDR重視の学習はmasked recoveryだけでなく、凍結埋め込みとridge regressionによる結合親和性予測でも改善を示した。とくにanti-Fluoresceinの組合せ変異データでは、AbCDR-ESM2がbase ESM2のR2 0.547を0.693まで上げ、約26.6%の相対改善を示した。単一変異データでもD44、G6、Trastuzumabでbase ESM2を上回り、AbCDR-ESMCは600Mという比較的小さいモデルながら一部の大型抗体PLMを上回った。さらに、ESM2に大規模な非ペア抗体配列で事前学習を追加しても、ペア配列での下流結合予測には明確な上積みがなかった。結論は、抗体PLMでは単純なスケール拡大や非ペア配列の大量投入より、VH/VLペア情報とCDR中心の学習目的を優先する設計が実用的である、というものだ。

## 書誌情報

- URL/DOI: https://doi.org/10.1038/s44488-026-00010-2
- 論文URL: https://www.nature.com/articles/s44488-026-00010-2
- 公開日/更新日: Received 2025-12-06、Accepted 2026-06-16、Published 2026-08-13、Version of record 2026-08-13
- 著者・所属: Mahtab Talaei, Kenji C. Walker, Boran Hao, Eliot Jolley, Yeping Jin, Dima Kozakov, John Misasi, Sandor Vajda, Ioannis Ch. Paschalidis, Diane Joseph-McCarthy。主な所属はBoston University Department of Electrical and Computer Engineering / Biomedical Engineering / Chemistry、The University of Texas at Austin Oden Institute、Boston University National Emerging Infectious Diseases Laboratories。
- 掲載誌/プレプリントサーバー: Communications AI & Computing, volume 1, Article number 7 (2026)
- リサーチ日: 2026-08-24
- 分類: 新着 / モデル起点

## 背景と問題設定

- この研究が解こうとしている問題は、抗体の配列表現をタンパク質言語モデルで作る際に、抗体特有の機能分布をどう学習目的に反映するかである。一般的なPLMはタンパク質配列上の残基をほぼ一様に扱うが、抗体では結合特異性が6つのCDRループに強く集中し、フレームワークは構造足場として比較的保存されている。
- 従来の汎用PLMであるESM2やESM Cは、タンパク質一般の自己教師あり表現を学ぶには強力だが、重鎖・軽鎖ペアの依存関係やCDRの高多様性を明示的には扱わない。抗体特化PLMであるAntiBERTy、AbLang、AntiBERTa、IgBERT、IgT5、AbLang2などは抗体配列を対象にするが、多くは一様マスキングや大量の非ペア配列に依存しており、CDR/FRの機能差をどこまで下流の結合予測に反映できるかが未解決だった。
- 抗体研究に重要な理由は、抗体医薬開発では候補配列の優先順位付け、親和性成熟、抗原特異性の推定、変異導入の設計がボトルネックになるためである。実験で測れる候補数には制限があるので、少ないラベル付きデータでも結合に関係する特徴を抽出できる表現モデルは価値が高い。
- この論文は、抗体PLMの性能改善を「モデルを大きくする」「非ペア配列を大量に足す」方向ではなく、「抗体の生物学的構造に合わせてマスク位置を変える」方向から検証している。ここが学習価値の高い点で、抗体AIモデル設計における目的関数設計の具体例として読める。

## この論文のコアアイデア

抗体配列を通常のタンパク質配列として扱うと、モデルは保存性の高いフレームワーク残基をよく復元する方向に学習しやすい。しかし、結合親和性や特異性に効く情報は主にCDR、特にHCDR3を含む高多様性領域にある。そこで著者らは、VH/VLペア配列を「重鎖-軽鎖」のようにセパレータで連結し、CDR注釈を使ってマスク位置を制御した。

比較したマスク方針は3つである。whole-chain maskingは通常のBERT型MLMに近く、全残基の15%をランダムに選ぶ。CDR maskingはCDR内だけを対象にし、CDR残基の50%をマスクする。hybrid maskingは各バッチで80%のサンプルにCDR masking、20%にwhole-chain maskingを使い、CDR重視とフレームワーク保持のバランスを狙う。マスクされた残基はBERT型の80/10/10ルール、つまり80%を[MASK]、10%をランダムアミノ酸、10%をそのままにする。

モデル論文として見ると、入力はペアの重鎖・軽鎖可変領域配列、出力は各残基のMLM予測および下流タスク用の固定長埋め込みである。アーキテクチャは新規に作ったTransformerではなく、既存のESM2-3BとESM C 600Mを抗体ペア配列にfine-tuneする。推論時には最後のencoder layerの残基表現を平均poolingし、ESM2では2560次元、ESM Cでは1152次元のペア抗体埋め込みを作る。下流評価ではモデル本体を凍結し、ridge regressionでlog(KD)を予測する。したがって、この論文の主張は「下流ヘッドの複雑化」ではなく「自己教師あり学習で作る埋め込みの質」が結合予測を押し上げる、という点にある。

## 手法の詳細

- 入力データ: OAS由来のヒト抗体VH/VLペア配列。非ヒト配列と一部の自己免疫疾患レパートリーを除外し、MMseqs2などで95% sequence identity、80% coverageにより近縁配列をクラスタリングした。
- 出力: MLMではマスクされたアミノ酸のtop-1予測。下流評価ではペア抗体配列の固定長埋め込みからlog(KD)を予測。
- モデル/アルゴリズム: ESM2-3BとESM C 600Mを初期化に使い、ペア抗体配列でfine-tune。下流はridge regression。
- 特徴量・表現学習: 重鎖と軽鎖をセパレータ「-」で連結し、最後のencoder layerの全位置表現をmean poolingする。ESM2埋め込みは2560次元、ESM C埋め込みは1152次元。
- 学習方法: ESM2では任意のunpaired pretraining、Stage I whole-chain、Stage II CDRまたはhybridを検討。ESM Cではwhole-chain、直接CDR、whole-chainからCDRへの二段階を検討。
- 損失関数・目的関数: masked language modeling objective。CDR maskingでも、マスクされた残基に対するアミノ酸分類として学習する。
- 推論方法: ペア配列をtokenizeし、凍結モデルから埋め込みを抽出し、ridge regressionでlog(KD)を予測する。Hugging Faceモデルカードには、`HEAVY_CHAIN-LIGHT_CHAIN`形式で入力し、mean poolingして埋め込みを使う例がある。
- ベースライン: Base ESM2、Base ESM C、AntiBERTy、AbLang、AbLang2、ProtBert、IgBert、ProtT5、IgT5、unpaired pretraining variants。
- 評価指標: MLMではtop-1 recovery accuracy。下流結合予測ではR2、MAE、Spearman's rho。
- 実装上の重要点: CDR注釈はOASのIMGT-labeled CDR sequenceを利用。fine-tuningはHugging Face Transformers、Accelerate、DeepSpeed ZeROを用い、16 A100 GPUs、global batch 256、AdamW、learning rate 2e-5、bf16で実施。ESM2のunpaired pretrainingだけは32 A100 GPUsで300 GPU-hours超を要する。

## データセットと評価設計

- 使用データセット名: AbCDR-MLM、AbCDR-Binding、OAS、FLAb、AlphaSeq antibody dataset。
- データの規模: MLM用のpaired setは1,617,948 cluster representatives、validation 20,225、test 20,225。unpaired pretraining setは1,220,072,064 cluster representativesからvalidation 12,200,720を選択。下流結合評価はD44 2,048、G6 4,275、Trastuzumab 422、anti-Fluorescein 11,052、anti-H1 Hemagglutinin 1,038、anti-HR2 SARS-CoV-2 71,830。
- 対象の内訳: 単一変異セットはVEGF、hen egg-white lysozyme、HER2を標的とする抗体変異体。組合せ変異セットはSARS-CoV-2 spike HR2 peptide、Fluorescein、H1 Hemagglutininに対する抗体変異体。
- train/validation/test の分け方: MLMではクラスタ代表配列をtrain/validation/testに分ける。masked recoveryのtestは20,225 paired sequencesで、checkpoint selection用validationとは分離。単一変異下流評価は既存研究の10-fold CV splitを再現。組合せ変異は40回のランダム90%/10% train-test split。
- リーク対策やクラスタ分割の有無: MLM用配列では95% identity、80% coverageで近縁配列を除去。下流評価ではnested 5-fold CVでridgeの正則化を訓練fold内だけで選び、評価foldへのリークを避けている。ただし、Table 1の公開baselineをCDR-50%で再評価した部分は、著者自身もbaseline学習データとtest splitの重複可能性を認めており、完全な厳密比較ではない。
- 評価指標: MLM top-1 recovery、R2、MAE、Spearman's rho。
- ベースラインや比較対象: 汎用PLM、抗体PLM、unpaired pretrainingあり/なし、whole-chain/CDR/hybrid masking。
- この評価設計が妥当かどうか: 凍結埋め込み+線形に近いridge regressionで比較しているため、表現品質の差を見やすい設計である。単一変異と組合せ変異、複数抗原、複数モデルスケールを使っており、主張に対して比較的堅い。一方で、抗体-抗原複合体構造やprospective wet-lab validationではなく、既存変異データの回帰性能に限られる点は注意が必要である。

## 主要結果

まずMLMのmasked recoveryでは、base ESM2-3Bはフレームワーク領域では72-92%程度の回復率を示す一方、HCDR3は35.69%、LCDR3は46.06%と低かった。これは、汎用PLMが保存的な領域を比較的うまく扱える一方、抗体の高多様性CDR、特にHCDR3を苦手にすることを示す。whole-chain fine-tuningだけでもフレームワーク平均は97.57%に近づき、CDR平均も85.65%まで改善したが、HCDR3は62.38%でなお最難関だった。

Stage IIでCDRまたはhybrid maskingを入れると、CDR側にさらに小さな上積みが出た。CDR-50% testでは、ESM2 Stage II CDRがHCDR total 74.87%、LCDR total 89.88%を達成し、同じ条件で再評価したIgBERT系baselineより高かった。hybridはフレームワーク性能を保つ狙いがあるが、下流結合予測まで含めると純粋なCDR-focused strategyが強い場面が多い。

下流の単一変異結合予測では、AbCDR-ESM2がbase ESM2に対してD44でR2 0.302から0.359、G6で0.264から0.298、Trastuzumabで0.335から0.350へ改善した。AbCDR-ESMCはD44で0.345、G6で0.313を出し、G6では表中最高だった。TrastuzumabではAbLang2が0.460で最高であり、AbCDRがすべてのケースで勝つわけではない。著者はTrastuzumab setがn=422と小さく分散が大きいことも指摘している。

組合せ変異データでは効果がより明瞭である。anti-Fluorescein setでは、AbCDR-ESM2がMAE 0.594、R2 0.693、Spearman 0.730を示し、base ESM2のR2 0.547から26.6%相対改善した。既存公開モデルで最も良いAbLangのR2 0.531と比べると30.5%相対改善である。anti-HR2 SARS-CoV-2 setでは、AbCDR-ESM2がR2 0.396、MAE 0.841、Spearman 0.599で、IgT5のR2 0.364、MAE 0.866を上回った。anti-H1 Hemagglutinin setでもAbCDR-ESM2はR2 0.411で最高、AbCDR-ESMCはSpearman 0.758でAbCDR-ESM2とほぼ同等だった。

重要なのは、ESM2に巨大なunpaired OAS配列で事前学習を入れても、paired fine-tuning後のmasked recoveryや下流結合予測に明確な改善が出なかった点である。著者らは、非ペア配列での学習はframework regularitiesやgermline/frequency biasへ表現を寄せ、ペア依存の結合物理に対してはrepresentation driftを起こしうると解釈している。これは、抗体PLMでは「データ量」だけでなく「ペア情報」と「学習信号の配置」が重要であることを示している。

## 重要なFigure/Table

- Figure/Table番号: Figure 1
- 何を示しているか: VH/VL配列、CDR/FR注釈、whole-chain/CDR/hybrid masking、fine-tuning pipeline、埋め込み抽出、ridge regressionによる結合親和性予測までの全体像。
- 読み取り方: パネルAで抗体の領域構造を確認し、パネルBでESM2とESM Cの学習レシピの違いを見る。パネルCで、学習済みモデルを凍結してmean pooling embeddingを作り、log(KD)を予測する評価設計を把握する。
- 主要な数値・傾向: ESM2はunpaired pretrainingを含む複数経路を検討し、ESM Cではより単純なdirect CDR trainingが有効だった。hybrid maskingは80% CDR / 20% WC。
- なぜ重要か: この図を見れば、論文の主張が新規アーキテクチャではなく、抗体生物学に合わせたmasking policyとペア配列表現学習にあることが分かる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.nature.com/articles/s44488-026-00010-2/figures/1
- ライセンス確認: 確認済み。論文はCC BY-NC-ND 4.0のため画像保存・改変は避け、要約に留める。

- Figure/Table番号: Table 2
- 何を示しているか: D44、G6、Trastuzumabの単一変異結合データにおける埋め込み回帰性能。
- 読み取り方: 各PLMの凍結埋め込みを同じridge regression pipelineで評価したR2比較として読む。パラメータ数も併記されているため、モデルサイズと性能の関係も見られる。
- 主要な数値・傾向: AbCDR-ESM2はD44 0.359、G6 0.298、Trastuzumab 0.350。AbCDR-ESMCはD44 0.345、G6 0.313、Trastuzumab 0.310。AbLang2はTrastuzumabで0.460と最高。
- なぜ重要か: CDR maskingが単なるMLM指標の改善ではなく、結合親和性予測の埋め込み品質改善につながることを示す中核テーブルである。
- Markdown内での扱い: 要約表
- 出典URL: https://www.nature.com/articles/s44488-026-00010-2/tables/2
- ライセンス確認: 確認済み。CC BY-NC-ND 4.0のため、必要な数値だけを短く再構成。

| モデル | D44 R2 | G6 R2 | Trastuzumab R2 |
| --- | ---: | ---: | ---: |
| Base ESM2 | 0.302 | 0.264 | 0.335 |
| Base ESM C | 0.340 | 0.301 | 0.179 |
| AbLang2 | 0.327 | 0.281 | 0.460 |
| IgT5 | 0.297 | 0.250 | 0.275 |
| AbCDR-ESM2 | 0.359 | 0.298 | 0.350 |
| AbCDR-ESMC | 0.345 | 0.313 | 0.310 |

- Figure/Table番号: Table 3
- 何を示しているか: anti-HR2 SARS-CoV-2、anti-Fluorescein、anti-H1 Hemagglutininの単一・組合せ変異データに対する予測性能。
- 読み取り方: MAE、R2、Spearmanを同時に見て、特に組合せCDR変異に対する一般化性能を確認する。
- 主要な数値・傾向: anti-FluoresceinでAbCDR-ESM2はR2 0.693、Spearman 0.730。anti-HR2 SARS-CoV-2でAbCDR-ESM2はR2 0.396、MAE 0.841。anti-H1 HAでAbCDR-ESM2はR2 0.411、AbCDR-ESM2/ESMCともSpearman 0.758。
- なぜ重要か: 抗体最適化では単一変異だけでなく組合せ変異の予測が重要であり、CDR-focused trainingの実用性を最も強く示す結果である。
- Markdown内での扱い: 要約表
- 出典URL: https://www.nature.com/articles/s44488-026-00010-2/tables/3
- ライセンス確認: 確認済み。CC BY-NC-ND 4.0のため、必要な数値だけを短く再構成。

| データセット | モデル | MAE | R2 | Spearman rho |
| --- | --- | ---: | ---: | ---: |
| anti-HR2 SARS-CoV-2 | Base ESM2 | 0.865 | 0.366 | 0.580 |
| anti-HR2 SARS-CoV-2 | AbCDR-ESM2 | 0.841 | 0.396 | 0.599 |
| anti-Fluorescein | Base ESM2 | 0.791 | 0.547 | 0.697 |
| anti-Fluorescein | AbCDR-ESM2 | 0.594 | 0.693 | 0.730 |
| anti-H1 HA | Base ESM C | 0.160 | 0.376 | 0.756 |
| anti-H1 HA | AbCDR-ESM2 | 0.155 | 0.411 | 0.758 |

## Code / License / Weights

- コード公開: あり
- GitHub URL: https://github.com/noc-lab/AbCDR-ESM
- GitHub以外のコードURL: https://doi.org/10.5281/zenodo.19900188
- 実装種別: 公式
- GitHub確認: 確認済み
- リポジトリのライセンス: MIT
- ライセンス確認元: GitHub README/repository metadata、論文Code availability
- Hugging Face URL: https://huggingface.co/NOC-Lab/AbCDR-ESM2 / https://huggingface.co/NOC-Lab/AbCDR-ESMC
- Hugging Face種別: model
- モデル重み・チェックポイント公開: あり
- 重み公開URL: https://huggingface.co/NOC-Lab/AbCDR-ESM2 / https://huggingface.co/NOC-Lab/AbCDR-ESMC
- データセット公開: あり
- データセットURL: AbCDR-MLM https://doi.org/10.5281/zenodo.18762309 / AbCDR-Binding https://doi.org/10.5281/zenodo.18762978
- 再現性メモ: 論文はコード、データ、source data、モデル重みを公開しており再現性は高い。GitHub READMEにはモデルが「gated access required」と書かれているが、論文Code availabilityでは「without access restrictions」と書かれており、2026-08-24確認時点のHugging Faceページは通常のモデルページとして閲覧できた。実運用ではHugging Face認証やモデルファイルの取得可否を再確認する必要がある。ESM2系は3B paramsで重いため、まずAbCDR-ESMC 600Mの埋め込み抽出から試すのが現実的。

## 抗体研究・創薬への意味

この論文は、抗体配列モデルの設計原理として「抗体らしさ」をどこに入れるべきかを明確に示している。抗体の機能はCDRだけで完全に決まるわけではないが、候補の結合親和性や特異性を予測するには、framework全体の復元精度よりCDR内・CDR間の共変動をうまく表現することが重要である。これは、抗体設計や親和性成熟でCDR変異ライブラリを作る際に、候補順位付け用の埋め込みモデルとして直接使える可能性がある。

抗体-抗原相互作用予測では、抗原側構造やエピトープ情報を明示的に入れたモデルも重要だが、実験変異ライブラリの初期スクリーニングでは抗体配列だけから機能の傾向を拾える表現も価値がある。この論文の評価は、抗原ごとに測定された変異体KDを予測する設定なので、既知抗体の最適化や変異スキャンの補助に近い。新規抗原に対するde novo antibody designを直接解く論文ではないが、生成モデルで作った候補配列をスコアリングする、あるいはCDR設計候補の埋め込み特徴として使う入口になる。

創薬実務上は、600MのAbCDR-ESMCが大型モデルに近い性能を示す点が重要である。社内・研究室環境で大量候補を埋め込み化する場合、3Bモデルより600Mモデルのほうが導入しやすい。加えて、コードとデータが公開されているため、特定抗原や社内アッセイに対して同じridge regressionまたは軽量ヘッドを載せる再現実験をしやすい。

## 限界と注意点

著者が述べている限界:

- 評価した土台モデルはESM familyに限られ、ProtT5、IgBERTなど異なる位置エンコーディングやアーキテクチャで同じCDR maskingがどこまで効くかは未検証。
- 評価対象は主にbinding predictionであり、安定性、発現性、免疫原性、開発可能性など他の抗体医薬特性にCDR-focused representationが有効かは分からない。
- CDRを重視するとframework recoveryとのトレードオフがあり、frameworkが重要な物性では同じ設計が最適とは限らない。

読んで気づいた限界:

- 抗原配列や構造、抗体-抗原複合体構造を直接入力しているわけではないため、未知抗原への特異性予測やエピトープ依存の設計能力を評価したものではない。
- 下流評価は既存データセットでの回帰であり、prospectiveなwet-lab validationはない。実際の開発で上位候補がどれだけヒットするかは別途検証が必要。
- train/test splitは変異体レベルのCVが中心なので、抗体ファミリーや抗原を完全にまたいだ外挿性能は限定的にしか分からない。
- Table 1の一部baseline再評価は、baseline学習データとtest splitの重複可能性があり、著者も上限値として解釈すべきと述べている。
- 論文本体はCC BY-NC-ND 4.0で、図の再利用や改変には制約がある。コード・モデルはMITだが、論文図の扱いとは分けて考える必要がある。
- ESM2-3Bの学習・推論コストは高い。公開重みを使うだけでもGPUメモリ要件が重く、ローカル運用ではESMC版や埋め込みのバッチ処理設計が必要。

## この論文を読む上での前提知識

- CDRとframework: 抗体可変領域は、抗原に接触しやすいCDRと、構造を支えるframeworkに分けられる。CDR、とくにHCDR3は配列多様性が高く、抗原特異性に強く関わる。
- VH/VL pairing: 抗体の結合部位は重鎖可変領域と軽鎖可変領域の組み合わせで形成される。片方の鎖だけの配列からでは、ペア依存の結合特徴を十分に表せないことがある。
- Masked language modeling: 配列中の一部トークンを隠し、元のアミノ酸を予測する自己教師あり学習である。どの残基を隠すかが、モデルがどの情報を重点的に学ぶかに影響する。
- ESM2 / ESM C: タンパク質配列で事前学習されたTransformer系PLMで、配列埋め込みや構造・機能予測に広く使われる。ESM CはESM Cambrian系の比較的新しいモデルで、600Mでも強い表現を持つ。
- Ridge regression: L2正則化を入れた線形回帰で、高次元埋め込みから連続値を予測する軽量な下流ヘッドとして使われる。ここではlog(KD)を予測する。
- KDとlog(KD): KDは解離定数で、低いほど強い結合を意味する。KDは桁で変わるため、回帰ではlog変換して扱うことが多い。
- Spearman's rho: 予測値と実測値の順位相関を測る指標である。抗体候補の優先順位付けでは、絶対値の誤差だけでなく順位が合うかが重要になる。

## 今日この1報を選んだ理由

前回までの候補にはFrozen Protein Foundation-Model Embeddings、DiffAb、MEAN、AbLangなどがあったが、今日は新着性と実装利用価値を優先してこの論文を選んだ。2026-08-13公開の査読済み論文であり、抗体PLMの学習目的そのものを改善する内容なので、特定の抗原や一つの設計キャンペーンに閉じない学習価値がある。さらに、GitHubコード、Zenodoコードアーカイブ、Zenodoデータセット、Hugging Faceモデル重みが揃っており、ノート化後に実際に触れる入口が明確である。

新着候補としては、2026-08-20公開のPLOS Computational Biology論文「CLDN18.2 antibody design with protein language models: A deep learning optimization framework」も直接抗体設計で魅力的だった。一方、今回のAbCDR論文は抗体表現学習の基盤モデルとして使える範囲が広く、CDR maskingという設計原理が他の抗体生成・最適化モデルにも転用しやすい。長期的な知識ベースでは、個別標的の設計例だけでなく、こうしたモデル訓練原理を押さえておく価値が高いと判断した。

## 読む優先度

High。抗体PLMを使う研究では、モデルサイズや学習データ量だけでなく、CDR/FRの機能差をどう目的関数に入れるかが重要になる。この論文はその点を複数モデル、複数データセット、公開コード・重み付きで検証しており、今後の抗体設計・親和性予測・候補順位付けの基盤知識として優先度が高い。

## 自分用メモ

- 後で深掘りしたい点: AbCDR-ESMCの埋め込みを実際に取得し、FLAbや手元の抗体変異データでridge regressionを再現する。CDR-only mean poolingやHCDR3 poolingが全位置mean poolingより強いか試したい。
- 関連して読むべき論文: IgBERT/IgT5の「Large scale paired antibody language models」、AbLang2、Preferential masking of non-templated regions、FLAb benchmark、ESM Cambrian、AntiBERTy。
- 実装を触る場合の入口: GitHubの`AbCDR-ESM/Inference`、Hugging Faceの`NOC-Lab/AbCDR-ESMC`、ZenodoのAbCDR-Binding。
- Obsidianでリンクしたいキーワード: [[CDR masking]], [[抗体言語モデル]], [[VH VL pairing]], [[OAS]], [[FLAb]], [[ESM2]], [[ESM C]], [[抗体親和性予測]]

## 関連キーワード

- antibody language model
- paired antibody sequences
- CDR-focused masking
- masked language modeling
- binding affinity prediction
- AbCDR-ESM2
- AbCDR-ESMC
- OAS
- FLAb
- ridge regression

## 検索ログ

- 検索したデータベース/クエリ:
  - Web検索: `site:biorxiv.org antibody antigen binding affinity machine learning August 2026 antibody AI`
  - Web検索: `site:arxiv.org antibody antigen machine learning antibody design 2026 "Hugging Face"`
  - Web検索: `"Frozen Protein Foundation-Model Embeddings Improve Antibody-Antigen"`
  - Web検索: `"AI" antibody design bioRxiv August 2026`
  - Web検索: `"Teaching AI the biology of antibodies" "Boston University" paper antibody-specific language model`
  - Web検索: `"Preferential CDR masking in paired antibody language models improves binding affinity prediction" GitHub`
  - Web検索: `"Preferential CDR masking" Hugging Face`
- 候補にした論文:
  - `Preferential CDR masking in paired antibody language models improves binding affinity prediction`（採用）
  - `CLDN18.2 antibody design with protein language models: A deep learning optimization framework`（PLOS Computational Biology, 2026-08-20）
  - `Dual-Specific Antibody Design Using Artificial Intelligence`（bioRxiv, 2026-08-05）
  - `Frozen Protein Foundation-Model Embeddings Improve Antibody-Antigen ΔΔG Ranking`（bioRxiv, 2026-07-14）
  - `Breaking the Synthesis Barrier for AI-Designed DNA Libraries`（bioRxiv, 2026-07-07）
- 最終的にこの1報を採用した理由: 査読済み新着で、抗体PLMの基盤的設計原理、複数ベンチマーク、公開コード、公開データ、公開モデル重みが揃っているため。CDR maskingは抗体生成・親和性成熟・候補スコアリングへ広く転用できる。
- 新着論文と基盤論文のバランスをどう考えたか: 直近の新着から選びつつ、内容は抗体PLMの基盤技術に近い。前回はSEPIAの実験検証寄りだったため、今回は表現学習・モデル起点の論文を選んでバランスを取った。
- PDF/HTML/Supplementaryを確認できたか: Nature HTML、PDF、Supplementary information PDF、Source data ZIPリンクの存在を確認。Source data ZIPはブラウザツールで中身までは展開できなかったが、論文のData availabilityでCSV提供が明記されている。
- Figure/Tableを確認したURLやライセンス確認状況:
  - 論文URL: https://www.nature.com/articles/s44488-026-00010-2
  - PDF: https://www.nature.com/articles/s44488-026-00010-2.pdf
  - Figure 1/2/3、Table 1/2/3をHTML/PDFで確認。
  - ライセンス: 論文本体はCC BY-NC-ND 4.0。画像保存はせず、要約と短い再構成表に留めた。
- GitHub/コード検索で使ったクエリ:
  - `"Preferential CDR masking in paired antibody language models improves binding affinity prediction" GitHub`
  - `"Preferential CDR masking" Hugging Face`
  - `AbCDR-ESM GitHub`
- 確認したGitHub URL: https://github.com/noc-lab/AbCDR-ESM
- 確認したHugging Face URL:
  - https://huggingface.co/NOC-Lab/AbCDR-ESM2
  - https://huggingface.co/NOC-Lab/AbCDR-ESMC
- 確認したその他コード/重み/データURL:
  - Code archive: https://doi.org/10.5281/zenodo.19900188
  - AbCDR-MLM dataset: https://doi.org/10.5281/zenodo.18762309
  - AbCDR-Binding dataset: https://doi.org/10.5281/zenodo.18762978
  - FLAb: https://github.com/Graylab/FLAb
  - AlphaSeq antibody dataset: https://doi.org/10.5281/zenodo.7783546
- リポジトリライセンスの確認元: GitHub README/repository metadataでMIT license、論文Code availabilityでコードMIT、Hugging FaceモデルカードでMIT licenseを確認。
- モデル重み・チェックポイントの確認先: Hugging Face `NOC-Lab/AbCDR-ESM2` と `NOC-Lab/AbCDR-ESMC`。ESM2は3B params、ESMCは600M、どちらもsafetensorsとMIT表示を確認。
