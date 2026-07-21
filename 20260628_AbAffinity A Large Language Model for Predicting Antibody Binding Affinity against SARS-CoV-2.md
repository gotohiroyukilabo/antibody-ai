# AbAffinity: A Large Language Model for Predicting Antibody Binding Affinity against SARS-CoV-2

## まず何の論文か

この論文は、SARS-CoV-2スパイクタンパク質HR2領域の保存的ペプチドに対するscFv抗体の結合親和性を、抗体配列だけから予測する大規模言語モデル Ab-Affinity を提案した研究である。対象は汎用的な抗体-抗原ペア全般ではなく、複数のSARS-CoV-2変異株に共通するHR2ペプチドに結合する抗体ライブラリで、問題設定はかなりターゲット特異的である。入力は重鎖・軽鎖の可変領域をリンカーでつないだscFv配列で、出力はlog変換した平衡解離定数に基づく結合親和性スコアである。モデルはESM-2/BERT系のエンコーダを土台にし、配列埋め込みから全結合層で親和性を回帰する。著者らは、104,972変異体から前処理後に得た71,834ユニーク抗体を用いて学習し、MSE損失とAdamで100エポック訓練している。比較ではDG-Affinity、ESM-2埋め込み+線形回帰、AbLang埋め込み+線形回帰より高いPearson/Spearman相関を示したと報告している。さらに、Ab-Affinityの埋め込みは親和性クラス分類やseed抗体より親和性が改善したかどうかの分類でもESM-2埋め込みより高いAUCを示す。注意すべき点は、これは「任意の抗原に対する抗体親和性予測器」ではなく、SARS-CoV-2 HR2ペプチドを標的にした変異体系列内の親和性ランドスケープ学習として読むべき論文である。一方で、公開モデル・Hugging Face重み・GitHub実装・テストデータがあり、抗体配列言語モデルを実務的な親和性スクリーニングに接続する教材として価値が高い。抗体設計では、候補変異体の絞り込み、親和性成熟ライブラリの順位付け、埋め込みを使った下流分類、注意マップによるCDR周辺の解釈に使える可能性がある。

## 書誌情報

- URL/DOI: https://arxiv.org/abs/2603.04480 / DOIなし（arXivプレプリント）
- 公開日/更新日: 2026-03-04 v1（arXiv API確認）
- 著者・所属: Faisal Bin Ashraf（Department of Computer Science and Engineering, University of California, Riverside）、Animesh Ray（Riggs School of Applied Life Sciences, Keck Graduate Institute）、Stefano Lonardi（Department of Computer Science and Engineering, University of California, Riverside）
- 掲載誌/プレプリントサーバー: arXiv、q-bio.QM / cs.LG。AAAI 2025 FMs4Bio Workshop発表と記載。
- リサーチ日: 2026-06-28（日本時間）
- 分類: 新着 / モデル起点

## 背景と問題設定

抗体医薬や感染症対策では、候補抗体が標的抗原にどれだけ強く結合するかを早く見積もることが重要になる。実験的にはSPR、ELISA、BLIなどで結合を測るが、候補数が多いライブラリでは精製、測定、再測定のコストが大きい。計算モデルで事前に候補を絞り込めれば、実験資源を有望な配列に集中できる。

ただし抗体-抗原相互作用は通常のタンパク質間相互作用より難しい。抗体側のパラトープはCDR周辺に集中し、抗原側のエピトープも柔軟なループや局所構造に依存することが多い。未結合状態では揺らぎが大きく、構造データベースにも柔軟領域は十分に表れない。そのため、構造を完全に前提にするより、配列から相互作用に効く特徴を学習する方法が有効になる場合がある。

既存研究にはDG-Affinity、CSM-AB、Tag-LLM、FAbConなど、抗体親和性や抗体-抗原相互作用にLLMやグラフモデルを使う流れがある。この論文の問題意識は、汎用のmulti-targetモデルではSARS-CoV-2の特定ペプチドに対する高密度変異体ライブラリの親和性予測に十分合わない、という点にある。そこで、HR2ペプチドという特定標的に対して、抗体変異系列から直接親和性ランドスケープを学習する。

抗体研究上の重要性は、抗体生成モデルやコンビナトリアル最適化で作った配列を、実験前に順位付けする「評価器」として使える可能性にある。生成モデル単体では、配列らしさは作れても標的特異的な結合の強弱は別問題である。Ab-Affinityは、その標的特異的評価器を抗体LLMで作る試みとして読める。

## この論文のコアアイデア

コアアイデアは、事前学習済みタンパク質言語モデルの配列表現を、SARS-CoV-2 HR2ペプチドに対するscFv変異体の親和性データでファインチューニングし、抗体配列空間内の親和性ランドスケープを学習することにある。入力は重鎖可変領域と軽鎖可変領域を標準的なGly-Ser系リンカーで接続したscFv配列で、モデルは配列全体をトークン列として処理する。出力はlog K_Dに対応する連続値で、訓練目的は回帰である。

アーキテクチャはBERT/ESM-2型のエンコーダで、各ブロックはmulti-head attention、feed-forward、Add & Normからなる。著者らは層数Nを変えたモデルを試し、最終的にはESM-2からファインチューニングした33層モデルを最良とする。最後のエンコーダ層の出力から配列表現を作り、全結合層で結合親和性を予測する。モデルは親和性だけでなく、残基レベル・配列レベル埋め込み、attentionに基づく残基間マップも出せるとされる。

この論文の主張は、標的特異的な高密度変異体データでファインチューニングすると、汎用タンパク質PLMや抗体PLMの固定埋め込みよりも、親和性に沿った埋め込み空間が得られるというものだ。t-SNEではAb-Affinity埋め込みが親和性の滑らかな勾配を示し、ESM-2埋め込みではその傾向が弱いと説明されている。また、attention mapの差分がCDR-H1、CDR-H2、CDR-L1および近傍に出やすいことを示し、モデルが抗体の結合関連領域に感度を持つ可能性を示している。

## 手法の詳細

- 入力データ: SARS-CoV-2 HR2ペプチドに結合するscFv抗体配列。重鎖・軽鎖の可変領域をリンカーで接続した配列をモデル入力にする。GitHub/Hugging Faceの例では `GGGGSGGGGSGGGGS` リンカーを使う。
- 出力: log変換された結合親和性、論文中ではlog K_Dとして扱われる連続値。
- モデル/アルゴリズム: ESM-2実装に基づくBERT型Transformer encoder。multi-head attention、feed-forward、residual接続、layer normalizationを積み重ね、最後に全結合層で回帰する。
- 特徴量・表現学習: 最終エンコーダ層の出力を配列表現として利用。Hugging Face実装は残基レベル埋め込み（例: L x 1280）と配列レベル埋め込み（1280次元）を返せる。
- 学習方法: 事前学習済みESM-2をファインチューニング。ランダム初期化モデルも比較のため訓練。85%を訓練、15%をvalidationに分け、validation Pearson相関が最良のモデルを保存。
- 損失関数・目的関数: Mean Squared Error（MSE）。最適化はAdam。
- 推論方法: 入力scFv配列または配列リストに対して親和性を回帰。Hugging Face実装では `get_affinity()`、`get_embeddings()`、`get_contact_map()` が提供される。
- ベースライン: DG-Affinity、ESM-2埋め込み+線形回帰、AbLang埋め込み+線形回帰。別データセットではEns-Grad、ESM-F、AntiBERTa2、AbMAP、A2Binder、Ensembles-14H/14Lとも比較。
- 評価指標: Pearson correlation、Spearman correlation、ROC AUC、t-SNE可視化、attention map差分、thermostabilityデータでの埋め込み可視化。
- 実装上の重要点: 訓練は4基のNVIDIA A100 80GB、batch size 128、100 epochs。t-SNEはscikit-learn、perplexity 200。attention map解析はRao et al.の方法に基づくと記載。完全な訓練スクリプトや分割再現手順は公開物だけでは未確認。

## データセットと評価設計

使用データセットは、SARS-CoV-2 HR2領域の保存的ペプチドに結合する3つの候補抗体をseedにしたscFv変異体ライブラリである。seedとしてAb-14-VH/Ab-14-VL、Ab-91-VH、Ab-95-VH/Ab-95-VLが挙げられている。1、2、3アミノ酸変異を導入して作られた104,972変異体について、選択した標的ペプチドへの結合親和性が3つの独立した生物学的反復で推定された。

ラベルは間接競合結合アッセイから推定された平衡解離定数である。前処理では、3反復のうち外れ値影響を減らすため、互いに近い2値の算術平均を採用し、残り1値を除外する。3反復すべてが欠損した抗体は除外され、最終的に71,834ユニーク抗体がモデル訓練に使われた。データ分割は親和性値の分布を維持しながら85% train、15% validationである。論文中では「同じtest datasetは訓練とモデル開発に使っていない」と記載されているが、train/validation/testの厳密な作り方、seedごとの分割、配列類似性クラスタ分割の有無は明確ではない。

評価指標は、親和性回帰ではPearson/Spearman相関である。Figure 4では全3 seed抗体を含むテストセットでDG-Affinity、ESM-2、AbLang、Ab-Affinityを比較する。Table 1では14H/14LというAb-14由来の重鎖・軽鎖変異データセットに対して既存手法と比較する。分類評価では、親和性クラス（High/Medium/Low）とseed抗体より結合が改善したか（Yes/No）をROC AUCで評価する。thermostabilityについては26個のSARS-CoV-2抗体の実験データでt-SNE可視化しており、これは定量予測というより埋め込みの性質を探索する補助評価である。

評価設計の妥当性は中程度と見る。高密度変異体データに基づく標的特異的モデルとしては、回帰相関・分類・埋め込み可視化・attention解析を組み合わせており学習内容を多面的に確認している。一方で、変異系列内のランダム分割では近縁配列がtrain/testにまたがる可能性が高く、完全な一般化能力の評価にはなりにくい。未知の抗原、未知のseed抗体、CDR構成が大きく異なる抗体への外挿性は、この設計だけでは判断できない。

## 主要結果

Figure 4の相関比較では、Ab-Affinityが最も高いPearson/Spearman相関を示す。画像から読める値は、DG-AffinityがPearson 0.194、Spearman 0.201、ESM-2がPearson 0.566、Spearman 0.549、AbLangがPearson 0.538、Spearman 0.523、Ab-AffinityがPearson 0.655、Spearman 0.608である。DG-Affinityは抗体向け/LLM系の既存手法だが、このSARS-CoV-2 HR2特異的なデータでは低い相関に留まった。著者らは、DG-Affinityの予測ヘッドがSARS-CoV-2特異的データで訓練されていないことを理由の一つとして推測している。

散布図では、DG-Affinityの予測値が2帯に固まり、実測値に対する傾きが0.02程度とほぼ感度を持たない。ESM-2とAbLangは実測に対して正の傾きを示すが、Ab-Affinityのbest-fit lineの傾きが0.49で最も大きく、実測の変化をより反映している。相関値だけでなく、予測レンジと実測レンジの対応を見ると、Ab-Affinityは標的特異的ファインチューニングにより親和性変化を拾いやすくなったと解釈できる。

Table 1では、14H/14L datasetでAb-Affinityが14H Pearson 0.652、Spearman 0.526、14L Pearson 0.712、Spearman 0.713を示す。14HではA2BinderがPearson 0.642、Spearman 0.553で、SpearmanはA2Binderが上回るが、Ab-AffinityはPearsonで最高、Spearmanでも近い。14LではAb-AffinityがPearson/Spearmanとも最高で、A2BinderのPearson 0.683、Spearman 0.688やESM-F、AntiBERTa2、AbMAPを上回る。これはAb-14派生の重鎖・軽鎖変異に対しても、Ab-Affinityが既存の特徴抽出器・予測器より強いことを示す。

分類タスクでは、親和性クラス分類でESM-2埋め込みのAUCがHigh 0.78、Medium 0.70、Low 0.71であるのに対し、Ab-Affinity埋め込みはHigh 0.92、Medium 0.74、Low 0.78を示す。seed抗体より親和性が改善したかどうかの二値分類では、ESM-2埋め込みがNo/YesともAUC 0.74、Ab-Affinity埋め込みがNo/YesともAUC 0.91である。これは、Ab-Affinityが単なる回帰出力だけでなく、下流タスク用の抗体表現としても有用であることを示す。

attention map解析では、強結合抗体と弱結合抗体の残基間attention差分を比較し、差分がCDR-H1、CDR-H2、CDR-L1やその隣接領域に多く現れると報告している。これはモデルがCDR周辺の配列変化に感度を持っている可能性を示すが、attentionが因果的な接触や物理的相互作用を直接表すとは限らない。thermostability解析では、26個のSARS-CoV-2抗体についてAb-Affinity埋め込みのt-SNEが熱安定性の近いクラスターを作ると説明されている。これは興味深いが、サンプル数が少なく、定量的なthermostability予測器として主張するには弱い。

## 重要なFigure/Table

- Figure/Table番号: Figure 3
- 何を示しているか: Ab-Affinityのアーキテクチャ。BERT/ESM-2型Transformer encoderから埋め込みを作り、全結合層でbinding affinityを予測し、別モジュールでattention/contact mapも出す構成。
- 読み取り方: 入力配列にpositional embeddingを足し、N層のencoder blockを通す。出力表現は親和性回帰、埋め込み表現、残基間attention mapの3用途に使われる。
- 主要な数値・傾向: 層数Nを変えて試し、33層かつESM-2ファインチューニングが最良と本文に記載。
- なぜ重要か: この論文は新しい抗体構造モデルというより、ESM-2系PLMを標的特異的な抗体親和性回帰器に変換する研究であることが分かる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://arxiv.org/html/2603.04480v1
- ライセンス確認: 確認済み（arXiv HTML footerにCC BY-NC-SA 4.0表示。ただしObsidianへの画像保存・埋め込みは行わず要約に留める）

- Figure/Table番号: Figure 4
- 何を示しているか: DG-Affinity、ESM-2、AbLang、Ab-Affinityの親和性予測相関と、実測log K_D対予測log K_Dの散布図。
- 読み取り方: 左上の棒グラフでPearson/Spearmanを比較し、各散布図で予測が実測レンジに追従しているかを見る。
- 主要な数値・傾向: PearsonはDG-Affinity 0.194、ESM-2 0.566、AbLang 0.538、Ab-Affinity 0.655。SpearmanはDG-Affinity 0.201、ESM-2 0.549、AbLang 0.523、Ab-Affinity 0.608。Ab-Affinityの散布図のfit slopeは0.49で、DG-Affinityの0.02より大きい。
- なぜ重要か: 提案モデルの中心的な性能主張であり、標的特異的ファインチューニングの効果を最も直接的に示す。
- Markdown内での扱い: 要約表
- 出典URL: https://arxiv.org/html/2603.04480v1
- ライセンス確認: 確認済み（arXiv HTML footerにCC BY-NC-SA 4.0表示。画像保存は行わない）

| Model | Pearson | Spearman |
|---|---:|---:|
| DG-Affinity | 0.194 | 0.201 |
| ESM-2 | 0.566 | 0.549 |
| AbLang | 0.538 | 0.523 |
| Ab-Affinity | 0.655 | 0.608 |

- Figure/Table番号: Table 1 / Figure 6
- 何を示しているか: Table 1は14H/14L datasetでの既存手法比較、Figure 6はAb-Affinity埋め込みを使った下流分類タスクのROC AUC。
- 読み取り方: Table 1ではAb-AffinityがAb-14派生の重鎖・軽鎖変異で既存手法より高い相関を出すかを見る。Figure 6では、ESM-2埋め込みとAb-Affinity埋め込みを同じ分類タスクに使ったときの差を見る。
- 主要な数値・傾向: Table 1でAb-Affinityは14H Pearson/Spearman 0.652/0.526、14L 0.712/0.713。Figure 6でAb-Affinity埋め込みは親和性HighクラスAUC 0.92、改善有無分類AUC 0.91で、ESM-2の0.78および0.74を上回る。
- なぜ重要か: 回帰値だけでなく、埋め込みそのものが抗体最適化の下流タスクに使える可能性を示す。
- Markdown内での扱い: 要約表 / リンクのみ
- 出典URL: https://arxiv.org/html/2603.04480v1
- ライセンス確認: 確認済み（arXiv HTML footerにCC BY-NC-SA 4.0表示。画像保存は行わない）

## Code / License / Weights

- コード公開: あり
- GitHub URL: https://github.com/ucrbioinfo/AbAffinity
- GitHub以外のコードURL: https://huggingface.co/faisalashraf/abaffinity / https://pypi.org/project/AbAffinity/
- 実装種別: 公式 / 著者実装
- GitHub確認: 確認済み
- リポジトリのライセンス: GitHub repository metadataではlicense null、GitHub LICENSE APIは404。READMEにはMIT Licenseと記載されているが、リンク先が `yourusername/AbAffinity` になっており実LICENSEファイルは確認できなかった。Hugging Face model card metadataはlicense: mit。
- ライセンス確認元: README / repository metadata / Hugging Face metadata。GitHubのLICENSEファイルは見つからず。
- Hugging Face URL: https://huggingface.co/faisalashraf/abaffinity
- Hugging Face種別: model
- モデル重み・チェックポイント公開: あり
- 重み公開URL: https://huggingface.co/faisalashraf/abaffinity/blob/main/abaffinity/abaffinity_weights.pth
- データセット公開: あり
- データセットURL: https://github.com/ucrbioinfo/AbAffinity/tree/main/data
- 再現性メモ: Hugging Face APIで `abaffinity/abaffinity_weights.pth`、実装ファイル、setup.pyを確認。GitHub dataディレクトリには `test_data_with_labels.csv`、`cdr_similarity_sabdab.csv`、`thermostability_dataset.csv` がある。モデル利用例はREADMEにあり、`pip install git+https://huggingface.co/faisalashraf/abaffinity` で導入する形。完全な訓練データ全体、分割、訓練スクリプトの再現性は公開物だけでは未確認。

## 抗体研究・創薬への意味

この論文の直接的な価値は、抗体変異体ライブラリを実験的に大量測定したあと、その標的に対する親和性ランドスケープを学習し、未測定候補を順位付けする流れを示している点にある。抗体医薬開発では、親和性成熟や配列最適化で候補数が爆発するため、こうしたモデルはwet実験前のフィルタとして使える。特にAb-AffinityはscFv配列を直接入力にするため、構造がない候補配列にも適用しやすい。

生成モデルとの接続も重要である。抗体配列生成モデルは多様なCDR変異やフレームワーク保持配列を作れるが、標的抗原への結合改善を保証しない。Ab-Affinityのような標的特異的評価器を併用すれば、生成候補を親和性でリランキングし、探索を効率化できる可能性がある。論文中で示された改善有無分類のAUC 0.91は、seed抗体からの改善候補を拾う実務タスクに近い。

また、埋め込みが親和性クラスやthermostabilityに関連する傾向を持つなら、多目的最適化の特徴量としても使える。ただしthermostabilityは26抗体の可視化に留まるため、安定性予測に使うには追加検証が必要である。attention mapがCDR周辺に差を示す点は、変異候補の解釈や設計仮説の生成に役立つ可能性があるが、attentionをそのまま物理的接触や因果部位と同一視するのは危険である。

## 限界と注意点

著者が述べている、または本文から読み取れる限界として、モデルはSARS-CoV-2 HR2ペプチドを標的にした特定データで訓練されている。したがって、任意抗原に対する汎用抗体親和性予測モデルとは言えない。抗原配列や抗原構造を明示的に入力していないため、標的が変わると学習済みの親和性意味がそのまま移る保証はない。論文中でも、既存multi-target affinity prediction modelがSARS-CoV-2に合わないという動機で、逆に標的特異的モデルを作っている。

私が読んで気づいた限界は、まずデータ分割である。高密度な1、2、3点変異ライブラリでは、trainとvalidation/testに非常に近い配列がまたがりやすい。クラスタ分割、seed抗体単位のholdout、CDR類似度に基づく厳しい外挿評価がなければ、未知抗体ファミリーへの一般化は過大評価される可能性がある。GitHubには `cdr_similarity_sabdab.csv` があるが、論文本文からはリーク対策の詳細は十分読み取れない。

第二に、ラベルは間接競合結合アッセイから推定されたK_Dで、3反復から近い2つを平均する前処理を使う。外れ値対策として合理的だが、測定ノイズや系統誤差、アッセイ条件依存性は残る。第三に、比較ベースラインの一部は固定埋め込み+線形回帰であり、各モデルに最適なファインチューニングを行った比較とは限らない。Ab-Affinityは標的特異的にファインチューニングされているため、有利な設定である。

第四に、attention mapの解釈には注意が必要である。CDR周辺に差が出ることは妥当だが、attention強度は実際の接触、結合エネルギー寄与、変異効果の因果性を直接意味しない。第五に、ライセンス面ではHugging FaceはMITタグ、READMEもMITと述べる一方、GitHubにはLICENSEファイルが見つからず、repository metadataもlicense nullである。商用利用や再配布を厳密に判断する場合は、著者に確認した方がよい。

## この論文を読む上での前提知識

- scFv: single-chain fragment variableの略で、抗体の重鎖可変領域と軽鎖可変領域をペプチドリンカーでつないだ形式。抗体の結合部位をコンパクトに扱えるため、ライブラリスクリーニングや配列モデル入力に向く。
- K_D / 結合親和性: 平衡解離定数で、一般に値が小さいほど強く結合する。論文ではlog変換したK_Dを回帰対象にしているため、数値の解釈には変換方向を意識する必要がある。
- CDR / パラトープ: CDRは抗体可変領域の相補性決定領域で、抗原との接触に大きく関わる。パラトープは抗体側の抗原認識面で、CDR周辺に形成されることが多い。
- ESM-2: 大規模タンパク質配列で事前学習されたTransformer言語モデル。配列だけから構造・機能に関係する表現を得られるため、抗体やタンパク質設計の特徴抽出器としてよく使われる。
- AbLang: 抗体配列に特化した言語モデル。重鎖・軽鎖の抗体らしい配列表現を学習しており、抗体タスクのベースラインとして使われる。
- Pearson / Spearman相関: Pearsonは線形な対応、Spearmanは順位の対応を見る指標。親和性予測では絶対値精度だけでなく、候補を正しく順位付けできるかが重要なので両方を見る。
- ROC AUC: 二値またはone-vs-rest分類で、閾値を動かしたときの識別性能を測る指標。抗体候補のスクリーニングでは、上位候補を拾う能力の目安になる。
- t-SNE: 高次元埋め込みを2次元に可視化する手法。局所的な近さの把握には有用だが、距離やクラスタ形状を過度に定量解釈してはいけない。

## 今日この1報を選んだ理由

今日は新規性と実装確認価値を優先し、2026年3月公開のAbAffinityを採用した。AI×抗体の中でも、抗体配列から親和性を予測する直接的なテーマであり、SARS-CoV-2抗体の高密度変異体データ、ESM-2ファインチューニング、下流分類、attention解釈、thermostability可視化まで含むため、学習価値が高い。さらにGitHub、Hugging Face、モデル重み、データファイルが公開されており、単に読むだけでなく後で実装を触れる。

他候補としては抗体設計・構造予測・抗体生成系の新着もあり得るが、AbAffinityは「生成した抗体候補をどう評価するか」という抗体設計パイプラインのボトルネックに直結している。特に、標的特異的に測定した変異体データからPLMをファインチューニングする設計は、今後の抗体最適化プロジェクトに転用しやすい。一方で汎用モデルではないため、読む際には適用範囲の狭さを同時に理解する必要がある。その意味で、性能値だけでなく限界も学べる論文として今日の1報に適している。

## 読む優先度

High。抗体親和性予測は抗体設計・親和性成熟・生成モデル評価に直結するうえ、この論文はコード、Hugging Faceモデル、重み、データの一部が公開されている。汎用性には制約があるが、標的特異的な抗体LLM評価器を作る実例として優先的に読む価値がある。

## 自分用メモ

- 後で深掘りしたい点: train/validation/test分割の厳密な再現、seed抗体単位holdoutでの性能、近縁配列リークの影響、実際のラベル分布。
- 関連して読むべき論文: DG-Affinity、A2Binder、AbMAP、AntiBERTa2、AbLang、ESM-2、FAbCon、Tag-LLM、Ashraf et al. 2024の親和性成熟bioRxiv論文。
- 実装を触る場合の入口: Hugging Face `faisalashraf/abaffinity` をcloneし、`AbAffinity().get_affinity()` と `get_embeddings()` を動かす。GitHubの `Example_Ab-Affinity.ipynb` と `data/test_data_with_labels.csv` を確認する。
- Obsidianでリンクしたいキーワード: [[抗体親和性予測]], [[SARS-CoV-2抗体]], [[scFv]], [[ESM-2]], [[抗体言語モデル]], [[親和性成熟]], [[CDR]], [[attention map]], [[Hugging Face models]]

## 関連キーワード

- antibody binding affinity prediction
- antibody language model
- SARS-CoV-2 HR2 peptide
- scFv
- ESM-2 fine-tuning
- AbLang
- DG-Affinity
- antibody affinity maturation
- protein language model embeddings
- CDR attention map

## 検索ログ

- 検索したデータベース/クエリ: web検索で `2026 AI antibody design deep learning antibody paper GitHub Hugging Face`、`2025 AI antibody design diffusion model antibody antigen binding prediction GitHub`、`site:biorxiv.org antibody design machine learning 2026` を確認。arXiv APIで `2603.04480` を確認。
- 候補にした論文: AbAffinity: A Large Language Model for Predicting Antibody Binding Affinity against SARS-CoV-2。他に抗体設計・抗体生成系の新着候補を探索したが、今日の1報としては実装・重み公開と抗体親和性予測への直接性を優先した。
- 最終的にこの1報を採用した理由: 新着性があり、抗体研究に直接関係し、GitHub/Hugging Face/重み/データが確認でき、Figure/Tableから手法・性能・限界を学べるため。
- 新着論文と基盤論文のバランスをどう考えたか: 初回実行で蓄積がまだないため、新着かつ実装確認可能な論文を採用。今後は基盤論文としてESM-2、AbLang、AntiBERTa系、抗体構造/デザイン系も混ぜる。
- PDF/HTML/Supplementaryを確認できたか: arXiv HTMLを確認。PDFは取得したがローカルに `pdftotext` とPython PDFライブラリがなく全文抽出は未実施。Supplementaryは未確認。
- Figure/Tableを確認したURLやライセンス確認状況: arXiv HTML https://arxiv.org/html/2603.04480v1 を確認。HTML footerにCC BY-NC-SA 4.0表示あり。Figure 3/4/5/6を画像として目視確認したが、ノートには画像保存せず要約とリンクのみ。
- GitHub/コード検索で使ったクエリ: 論文中のCode URL、GitHub API、raw README、contents APIを確認。
- 確認したGitHub URL: https://github.com/ucrbioinfo/AbAffinity
- 確認したHugging Face URL: https://huggingface.co/faisalashraf/abaffinity
- 確認したその他コード/重み/データURL: https://pypi.org/project/AbAffinity/、https://github.com/ucrbioinfo/AbAffinity/tree/main/data、https://huggingface.co/faisalashraf/abaffinity/blob/main/abaffinity/abaffinity_weights.pth
- リポジトリライセンスの確認元: GitHub README、GitHub repository metadata、GitHub license API、Hugging Face model card metadata。GitHub LICENSEファイルは見つからず、Hugging Face metadataはMIT。
- モデル重み・チェックポイントの確認先: Hugging Face API siblingsに `abaffinity/abaffinity_weights.pth` があることを確認。
