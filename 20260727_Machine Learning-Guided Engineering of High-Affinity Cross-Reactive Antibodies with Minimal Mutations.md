# Machine Learning-Guided Engineering of High-Affinity Cross-Reactive Antibodies with Minimal Mutations

## まず何の論文か

ヒト標的に対して作られた抗体が、マウスなど前臨床動物のオルソログを十分に認識しない問題を、Deep Mutational Scanning（DMS）と機械学習で解く論文である。対象は、ヒトPD-L1には結合するがマウスPD-L1への結合が弱い完全ヒト抗PD-L1抗体C4で、目的はヒト・マウス両方のPD-L1に高親和性で結合する交差反応性抗体を、できるだけ少ない変異で作ることにある。著者らはまず酵母表面提示FabライブラリとFACS/NGSを使い、ヒトPD-L1への結合を維持しながらマウスPD-L1結合を改善する変異候補をDMSで同定した。通常の組合せライブラリとスクリーニングでは強い交差反応性抗体が得られたが、親配列から13〜15変異も離れていた。そこで、FACSで得た配列を「親株様」「改善」「強く改善」の3クラスに分け、one-hot化した抗体配列を入力するConvNeXt系CNNを3つ学習し、ensembleで配列の親和性クラスを予測した。さらに、DMS由来の高性能クローンから変異を系統的に取り除いた約120,000配列をin silicoで生成し、最高クラスを保つ最少変異の候補を選んだ。結果として、4〜5変異だけを持つDL.1などの候補が、13〜15変異のDMS候補と同程度のBLI結合、CT26細胞上PD-L1結合、PD-1/PD-L1阻害活性を示した。エピトープDMSとAlphaFold3/Protenix-v1/IntelliFoldによる構造モデリングから、親抗体の結合様式を大きく壊さず、ヒト・マウスで保存されたPD-L1面をよりよく使う方向に結合が調整されたと解釈している。抗体AIとしては、大規模な汎用抗体生成モデルではなく、標的ごとの実験DMSデータを小さな深層学習モデルで読み解き、変異負荷を下げる実務的な抗体最適化ワークフローを示した点が重要である。コードや学習データの公開は見つからず、再現性は限定的だが、DMSデータから最小変異の組合せを探索する考え方は、親和性成熟、ヒト化、交差反応性付与、developabilityフィルタとの統合に直接つながる。

## 書誌情報

- URL/DOI: https://www.biorxiv.org/content/10.64898/2026.07.15.738714v1 / DOI: 10.64898/2026.07.15.738714
- 公開日/更新日: 2026-07-16 posted（bioRxiv v1）
- 著者・所属: Hugo Dorison, Anne-Laure Grindel, François Thenier, Chloé Pluchart, Mélanie Munch, Càtia Oliveira, Steven Dubois, Camille Le Drezen, Raphaël Guérois, Bernard Maillère, Charles Truillet, Hervé Nozach。主な所属は Université Paris-Saclay / CEA / INRAE / CNRS / Inserm / BioMaps / I2BC など。
- 掲載誌/プレプリントサーバー: bioRxiv
- リサーチ日: 2026-07-27
- 分類: 新着 / 抗体最適化 / DMS+機械学習 / 創薬応用

## 背景と問題設定

治療抗体は標的への高い選択性が利点だが、その選択性が前臨床評価では問題になる。ヒト標的に対して選抜・最適化された抗体は、マウスや他の動物オルソログと結合しないことがあり、免疫正常マウスモデルで薬効や毒性を評価しにくくなる。代替策としてサロゲート抗体やヒト化動物モデルを使うことはできるが、サロゲート抗体は元の臨床候補と同じ生物学的挙動とは限らず、ヒト化動物も内在性発現や免疫環境を完全には再現しない。

この論文の具体的な問題は、完全ヒト抗PD-L1抗体C4を、ヒトPD-L1への結合を保ったままマウスPD-L1にも強く結合する抗体へ変えることである。PD-L1は免疫チェックポイント阻害で重要な標的で、ヒトPD-L1抗体の多くは似たエピトープを認識するが、ヒトとマウスのPD-L1の局所差により交差反応性は自明ではない。親抗体C4はヒトPD-L1にナノモル級で結合する一方、マウスPD-L1には200〜300 nM程度と弱い結合しか示さない。

従来のdisplayライブラリとaffinity maturationは有効だが、強く選抜されたクローンはしばしば多変異になる。多変異は親和性を上げる一方、ヒトgermlineからの乖離、免疫原性、発現、安定性、developabilityのリスクを増やしうる。したがって、本研究の問題設定は「高親和性」と「低変異負荷」を同時に満たす変異組合せをどう見つけるかである。これは抗体医薬開発ではかなり実務的な問いで、単に最強のbinderを取るのではなく、前臨床で使える交差反応性と治療候補としての配列らしさを両立する必要がある。

## この論文のコアアイデア

中心は、DMSを単なる変異効果の可視化に使うのではなく、標的抗体・標的抗原に特化した教師データ生成装置として使い、その上に小さめの深層学習分類器を載せる点である。まずC4 FabのVH/VLに対して単変異DMSを行い、ヒトPD-L1結合を落とさずマウスPD-L1結合を上げる変異を拾う。次に候補変異を組合せたVHライブラリを作り、低濃度マウスPD-L1で選抜する。この時点でDMS由来の高親和性クローンは得られるが、13〜15変異と変異数が多い。

そこで著者らは、最終ライブラリをFACSで3つの結合クラスに分け、NGS配列にクラスラベルを付ける。入力は抗体配列のone-hot表現、出力は「-」「+」「++」の3クラスで、モデルはConvNeXt系CNNである。3回のランダムなクラスバランス補正により3つの訓練セットを作り、各セットで1モデルを学習し、多数決ensembleを使う。これは単一モデルの過信を避け、in vitro検証に回す候補を保守的に選ぶための設計と読める。

最も面白いのは、モデルを直接「新規配列生成器」として使うのではなく、DMSで実際に得た高性能配列から変異を1つずつ除く全組合せを作り、変異数を減らしても「++」に分類される候補を探す点である。探索空間は約120,000配列で、3変異以下では最高クラス候補がなく、4変異候補が9配列、5変異候補が138配列見つかった。著者らはensemble全モデルの合意と信頼度を条件に、4変異候補2個、5変異候補6個を選び、IgGとして発現・BLI・細胞結合・阻害活性まで検証した。

## 手法の詳細

- 入力データ: C4抗体VH/VLのDMSおよび組合せライブラリから得た抗体配列。FACSで親株様、改善、強く改善の3ゲートに分け、NGSで配列を読んだもの。構造解釈にはC4/PD-L1複合体の構造予測、エピトープDMSにはヒト・マウスPD-L1 N末端ドメインの単変異ライブラリを使う。
- 出力: 抗体配列の結合クラス（「-」「+」「++」）。実験上の最終出力は、ヒトPD-L1とマウスPD-L1の両方に高親和性で結合する低変異抗体候補。
- モデル/アルゴリズム: 比較対象としてSGD、SVC、KNN、Decision Tree、Random Forest、XGBoost、CNNを評価。最終的には、全配列を直接扱える柔軟性を理由にConvNeXt系CNNを採用。3モデルensembleで最終予測を行う。
- 特徴量・表現学習: 抗体アミノ酸配列のone-hot encoding。XGBoostなどは設計変異位置に絞った表現が必要だったが、CNNは設計外の偶発変異も含めた配列全体を入力できる。
- 学習方法: FACS/NGS後、単一readや複数クラスにまたがる配列を除外。過剰なクラスからランダムに配列を除いて各クラス17,000配列に揃え、このバランス補正を3回繰り返して3データセットを作る。train/testは75/25分割。
- 損失関数・目的関数: 未確認。3クラス分類なので通常はcross entropy系と推測されるが、本文では明記を確認できなかった。
- 推論方法: DMS由来の10個の高性能配列から変異を系統的に削除した約120,000配列を作り、3つのCNNに投入。全モデルの合意と信頼度を見て「++」かつ最少変異の候補を選抜。
- ベースライン: 機械学習面ではRF、XGBなどの一般的分類器。実験面では親抗体C4、通常DMSで得た13〜15変異クローン、手作業で変異を足した候補、Avelumab、非関連抗体Imdevimab。
- 評価指標: モデルはaccuracy, precision, recall, loss。抗体機能はBLIのKD/kon/koff、CT26細胞結合の見かけEC50、PD-1/PD-L1 competition ELISA、エピトープDMS、構造モデルのipTM/DockQベースクラスタリング、in vivo腫瘍モデルでの治療活性。
- 実装上の重要点: CNNの採用理由は最高スコアだけではなく、設計外変異を含む全配列を扱えること。構造モデリングではAlphaFold3、Protenix-v1、IntelliFoldそれぞれで1000モデルを生成し、DockQで冗長性を下げ、抗体-抗原interfaceに限定したipTMで評価している。

## データセットと評価設計

使用データセット名として固有の公開データセット名はない。データはC4抗体のDMS/FACS/NGSから著者らが作成した標的特異的データセットである。最初のDMSではVHの各位置に全置換を考え、VHだけでおよそ4×10^3 codon diversityとされる。ライブラリはVHの2領域、VLの2領域に分けて作られ、最終的にはVHの候補変異を組み合わせたサブライブラリと全VHライブラリを選抜している。

機械学習用には、最終候補ライブラリをFACSで3クラスに分けてNGSし、QC後にクラスごとの配列を作る。クラス不均衡は、過剰クラスからランダム除外して各クラス17,000配列に揃えることで補正している。この操作を3回繰り返すことで、わずかに異なる3つのデータセットを作り、3モデルensembleの材料にした。train/testは75/25で、3モデルのテストaccuracyは0.875、0.905、0.92と報告される。

リーク対策としては、同じ配列が複数クラスに出る場合を除外するinter-class QCが明記されている。一方で、配列類似性クラスタ単位のsplitや、完全に独立した別ライブラリ・別抗原への外部検証はない。したがって評価設計は、この標的・この親抗体の局所探索には妥当だが、汎用的な抗体親和性予測モデルとしての汎化性能を測るものではない。

実験検証はかなり厚い。モデル予測候補をIgGにしてBLIでヒト・マウスPD-L1への結合を測り、CT26細胞上のマウスPD-L1認識をFACS saturation assayで見る。さらにPD-1/PD-L1 competition ELISAで機能阻害を確認し、エピトープDMSと構造予測で結合様式を解釈する。最終的にはマウス腫瘍モデルで治療活性も見るため、単なるin silico論文ではなく、抗体工学ワークフローの実証として評価できる。

## 主要結果

まずDMSと通常の組合せ選抜により、親C4に比べてマウスPD-L1結合が大きく改善した10個のDMS候補が得られた。本文ではマウスPD-L1への親和性改善は親抗体に対して約1000倍に近いと説明され、ヒトPD-L1への親和性も改善した。重要なのは、この段階の候補が13〜15変異を持っていたことである。つまり、目的の交差反応性は達成できたが、配列のヒトらしさや不要変異の観点ではまだ粗い解だった。

機械学習部分では、FACSゲート由来の3クラス分類をConvNeXt系CNNで学習し、3つのバランス済みデータセット上のテストaccuracyが0.875、0.905、0.92に到達した。RFやXGBも性能は高かったが、XGBは設計位置に絞った特徴表現に依存するため、著者らは全配列を直接使えるCNNを最終採用した。ここは、単純な表形式モデルの精度だけでなく、抗体設計時の入力柔軟性を重視した判断である。

最少変異探索では、10個のDMS高性能クローンから変異を削る約120,000配列を作り、ensembleで「++」に残る配列を探した。3変異以下では最高クラス候補が見つからず、4変異で9配列、5変異で138配列が候補になった。実験には4変異候補2個と5変異候補6個を選び、すべてがBLIでヒト・マウスPD-L1に対する大幅な親和性改善を示した。DL候補はDMS由来の高変異候補とiso-affinity plot上で近い領域にまとまり、モデル予測クラスと実測BLIも整合した。

配列解釈では、A33GとN59Fが全DL候補で共通し、親和性成熟の中心変異と考えられる。一方、S31R、T28D/T28E/G27Yなどは置換可能または二次的、N104A/N104Rは重要だが他変異で補償されうると解釈される。L55VまたはG56Pのどちらかが4番目の変異として必要で、両方同時には不要という関係も示唆された。これは単変異効果の足し算ではなく、変異間の共依存があることを示す。

細胞結合では、CT26マウス大腸がん細胞上のPD-L1に対して、DMS候補の見かけEC50が0.18〜1.02 nM、DL候補が0.23〜0.58 nMと報告され、親C4やAvelumab対照より改善していた。競合ELISAでは、親C4、DL.1、SM.1がPD-1/PD-L1相互作用を阻害することが確認された。エピトープDMSでは、親C4がヒトPD-L1上でマウスと非保存の残基を含むエピトープに依存していたのに対し、成熟抗体は保存残基側に結合の重心を移し、マウスPD-L1上で追加接触も獲得したと解釈される。構造モデリングは、DL.1が全体の結合様式を大きく変えず、局所的な水媒介相互作用や疎水パッキングを改善する可能性を示している。

## 重要なFigure/Table

- Figure/Table番号: Figure 1
- 何を示しているか: DMSにより、C4抗体のどのVH変異がヒトPD-L1結合を維持しつつマウスPD-L1結合を改善するかを見つけ、候補変異を組み合わせて高親和性交差反応性抗体を得る流れ。
- 読み取り方: FACSで親抗体より良い/悪い結合ゲートを取り、NGS enrichmentをヒートマップ化する。マウスPD-L1で正に効き、ヒトPD-L1で悪影響がない変異を候補にし、サブライブラリから全VHライブラリへ進める。
- 主要な数値・傾向: VH単変異は約4×10^3 codon diversity。最終的に10個のIgG候補がBLIで高親和性を示し、マウスPD-L1への改善は親C4比で約1000倍に近い。候補は13〜15変異。
- なぜ重要か: この図は「高性能だが多変異」という出発点を示し、後続のMLによる変異削減の必要性を作っている。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.64898/2026.07.15.738714v1.full#F1
- ライセンス確認: 確認済み（CC BY-NC 4.0）。非商用条件付きのため画像保存・埋め込みはしない。

- Figure/Table番号: Figure 2
- 何を示しているか: 手作業の変異追加と、FACS/NGSデータを用いた機械学習データセット構築・モデル比較・ensemble設計。
- 読み取り方: 3ゲートで配列を分類し、QC後にクラスごとの配列数を確認する。各クラス17,000配列にバランス補正し、SGD/SVC/KNN/決定木/RF/XGB/CNNを比較する。最終的に3つのCNNをensembleにする。
- 主要な数値・傾向: train/testは75/25。選ばれた3モデルのaccuracyは0.875、0.905、0.92。最良epochは9、10、5付近。
- なぜ重要か: この論文のAI部分の核であり、標的特異的DMSデータをどのようにMLタスクへ変換したかが分かる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.64898/2026.07.15.738714v1.full#F2
- ライセンス確認: 確認済み（CC BY-NC 4.0）。画像保存・埋め込みはしない。

- Figure/Table番号: Figure 3
- 何を示しているか: DMS高性能クローンから変異を削ったin silico配列をensembleで分類し、4〜5変異の低変異候補を実験検証する流れ。
- 読み取り方: 約120,000候補を変異数ごとに並べ、「++」に分類される割合を見る。3変異以下では候補がなく、4変異と5変異から保守的に候補を選ぶ。BLIとCT26細胞結合でDL候補を検証する。
- 主要な数値・傾向: 4変異で9候補、5変異で138候補。実験検証した8候補はBLIで高親和性を示し、CT26細胞結合EC50はDL候補で0.23〜0.58 nM。
- なぜ重要か: この論文の主張「DMSで多変異候補を作り、MLで必要最小限の変異へ戻す」が最も直接的に示される。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.64898/2026.07.15.738714v1.full#F3
- ライセンス確認: 確認済み（CC BY-NC 4.0）。画像保存・埋め込みはしない。

## Code / License / Weights

- コード公開: 見つからず
- GitHub URL:
- GitHub以外のコードURL:
- 実装種別: 該当なし
- GitHub確認: 確認済み（論文タイトル、DOI、著者名、PD-L1関連語で検索。Methods中のMutation MakerとDockQは外部ツールであり、本研究の著者実装ではない）
- リポジトリのライセンス: 該当なし
- ライセンス確認元: 該当なし
- Hugging Face URL:
- Hugging Face種別: 該当なし
- モデル重み・チェックポイント公開: 見つからず
- 重み公開URL:
- データセット公開: 見つからず
- データセットURL:
- 再現性メモ: bioRxiv本文とSupplementary figures PDFは公開されているが、FACS/NGS配列、学習済みCNN、学習コード、候補生成スクリプト、構造モデル一式の公開リンクは確認できなかった。MethodsにはMutation Maker（https://github.com/Merck/Mutation_Maker）とDockQ（https://github.com/wallnerlab/DockQ）が記載されているが、これらはオリゴ設計・構造比較に使った外部ツールで、本論文のML実装公開ではない。bioRxivページ上のSupplementary materialは supplementary figures PDF（https://www.biorxiv.org/content/biorxiv/early/2026/07/16/2026.07.15.738714/DC1/embed/media-1.pdf?download=true）のみ確認。

## 抗体研究・創薬への意味

この論文の実務的価値は、抗体を「高親和性にする」だけでなく、「少ない変異で高親和性にする」ことを明確な最適化目標にしている点にある。抗体医薬では、親和性が高ければよいわけではなく、発現、安定性、粘性、免疫原性、ヒトgermlineとの距離、種差評価、標的発現量に応じた適切な親和性など、複数の制約がある。DMSとMLの組合せは、このような制約下で局所的な配列空間を効率よく探索する方法として有用である。

交差反応性抗体の設計では、ヒト標的への結合を維持しながらマウス標的へ結合する必要があるため、単純な親和性最大化より難しい。本研究は、DMSでヒト・マウス双方の結合情報を取り、選抜条件を重ねて「ヒトに悪くなく、マウスに良い」変異を選ぶ。さらにMLで変異間の依存関係を読むことで、13〜15変異から4〜5変異へ削る。これはヒト化、germline復帰、developability rescueにも近い考え方である。

また、巨大な基盤モデルに依存しない点も重要である。抗体AIでは汎用言語モデルや構造生成モデルが注目されるが、創薬現場では特定親抗体・特定抗原・特定アッセイのデータが最も信頼できることが多い。この論文は、標的ごとのDMSデータから小規模モデルを学習し、実験で閉じたループを作る設計を示す。将来的には、ここにTAPなどのdevelopability予測、免疫原性予測、構造ベースのparatope/epitope制約、ライブラリ設計アルゴリズムを組み込める。

## 限界と注意点

著者が述べている限界として、最小変異の定義は目標性能に依存する。つまり「4変異が絶対的に最小」というより、この研究で設定した最高親和性クラスと保守的なconfidence条件のもとでの最小である。著者らも、より攻めた選抜をすれば他の候補も使える可能性があると示唆している。また、DMSとMLは親抗体C4とPD-L1という特定の系に強く依存するため、別抗体・別抗原へそのまま汎化するモデルではない。

読んで気づいた限界として、コード・データ・モデル重みの公開が見つからないため、ML部分の完全再現は難しい。CNN architecture、optimizer、loss、学習ハイパーパラメータの詳細も本文だけでは十分ではない。train/test splitは配列単位であり、変異組合せ空間の近傍配列がtrain/testにまたがる可能性があるため、テストaccuracyは汎化性能というより同一ライブラリ内補間性能と見るべきである。

また、FACSゲート由来の3クラスは実用的だが、絶対的なKDラベルではない。BLIで検証した候補は8個と限られ、モデルの失敗例や境界候補の挙動は十分には見えない。in vivo評価は重要だが、腫瘍モデル・投与条件・比較抗体が限定されるため、臨床候補としての有効性を広く保証するものではない。ライセンス面ではbioRxiv本文と図はCC BY-NC 4.0で非商用条件付きであり、商用利用を前提にした社内資料・データ再利用では注意が必要である。さらに、複数著者が関連特許出願の発明者であることが明記されており、配列や応用の自由実施性は別途確認が必要である。

## この論文を読む上での前提知識

- Deep Mutational Scanning（DMS）: タンパク質や抗体の多数の変異体を一括で作り、選抜前後の頻度変化から変異効果を推定する方法。単変異効果だけでなく、選抜条件を工夫すると抗原別の有利/不利も読める。
- Yeast surface display / FACS: 抗体Fabなどを酵母表面に提示し、蛍光標識した抗原結合量と発現量で細胞をソートする手法。配列と表現型を結び付けられるため、NGSと組み合わせてML用データを作りやすい。
- Bio-layer interferometry（BLI）: 抗体と抗原の結合・解離をリアルタイムで測り、kon、koff、KDを推定する実験法。FACSの蛍光強度より定量的な親和性評価に使われる。
- 抗体humanness / germline乖離: 治療抗体配列がヒトgermline配列にどれだけ近いかという観点。変異が多いほど免疫原性やdevelopabilityの懸念が増える場合がある。
- PD-1/PD-L1 checkpoint: PD-L1がPD-1に結合するとT細胞応答が抑制されるため、がん免疫療法ではこの相互作用を阻害する抗体が使われる。
- ConvNeXt/CNNによる配列分類: アミノ酸配列をone-hot行列として扱い、局所パターンを畳み込みで読む。ここでは汎用PLMではなく、標的特異的なDMSラベルを直接学習する。
- Epitope / paratope: エピトープは抗原側の認識部位、パラトープは抗体側の結合部位。交差反応性設計では、種間で保存されたエピトープに結合を寄せることが重要になる。
- DockQ / ipTM: タンパク質複合体構造モデルの品質やinterface信頼度を見る指標。著者らは抗体-抗原interfaceに限定したipTMを使い、heavy-light interfaceによる過信を避けている。

## 今日この1報を選んだ理由

前回メモの次回候補に入っていた2026年7月中旬の新着で、抗体そのものを対象にし、AI/MLが実験ワークフローの中心に入っているため採用した。新規性は、DMSで高親和性候補を得た後に、MLで「必要な変異だけを残す」方向へ戻す点にある。これは単なる親和性予測よりも、実際の抗体最適化で問題になる変異負荷・humanness・交差反応性を扱っているため学習価値が高い。

他候補としては、Frozen Protein Foundation-Model Embeddings Improve Antibody-Antigen ΔΔG Ranking や、より基盤寄りのDiffAb/MEAN/AbLangなども考えられた。ただし今日は、直近公開で、抗体-抗原結合、DMS、ML、BLI、細胞結合、エピトープ、構造、in vivoまでつながる実験密度の高い論文を優先した。コード公開がない点は弱いが、Figureから学べるワークフロー設計が多く、Obsidianの知識ベースには価値が高い。

## 読む優先度

High。抗体AIの実務応用として、DMSデータを使った標的特異的ML、低変異化、交差反応性付与、実験検証が一つの流れで示されている。汎用モデル論文ではないが、創薬・抗体最適化の現場でAIをどう組み込むかを学ぶには優先度が高い。コード公開がないため再現実装には限界があるが、ワークフローの設計思想は他標的にも転用しやすい。

## 自分用メモ

- 後で深掘りしたい点: ConvNeXt入力長、具体的なlayer構成、optimizer/loss、confidence metricの定義、FACSゲートの境界設定。
- 関連して読むべき論文: Li et al. 2023 Nature Communications「Machine learning optimization of candidate antibody yields highly diverse sub-nanomolar affinity antibody libraries」、Mason et al. 2021 Nature Biomedical Engineering、Frei et al. 2025 Nature Biomedical Engineering、DiffAb、MEAN/dyMEAN、AbLang、TAP/developability関連。
- 実装を触る場合の入口: 公開コードはないため、独自にone-hot配列分類器をPyTorchで再実装し、DMSライブラリデータがある抗体系で試す。ライブラリ設計にはMutation MakerやSwiftLib系、構造クラスタにはDockQが参考になる。
- Obsidianでリンクしたいキーワード: [[Deep Mutational Scanning]], [[Yeast surface display]], [[antibody affinity maturation]], [[PD-L1]], [[cross-reactive antibody]], [[ConvNeXt]], [[antibody humanness]], [[BLI]], [[epitope mapping]]

## 関連キーワード

- DMS
- antibody affinity maturation
- cross-reactive antibody
- PD-L1
- yeast surface display
- FACS
- NGS
- ConvNeXt
- ensemble model
- humanness
- BLI
- epitope mapping
- AlphaFold3
- DockQ

## 検索ログ

- 検索したデータベース/クエリ: Web検索で `site:biorxiv.org antibody machine learning antibody antigen ddG ranking July 2026`、`"Machine Learning-Guided Engineering of High-Affinity Cross-Reactive Antibodies with Minimal Mutations"`、`"Frozen Protein Foundation-Model Embeddings Improve Antibody-Antigen"` を検索。bioRxiv本文、PDF、Supplementary materialページを確認。
- 候補にした論文: Machine Learning-Guided Engineering of High-Affinity Cross-Reactive Antibodies with Minimal Mutations / Frozen Protein Foundation-Model Embeddings Improve Antibody-Antigen ΔΔG Ranking / Machine Learning-Guided Engineering of High-Affinity Cross-Reactive Antibodies with Minimal Mutations のbioRxiv full text。
- 最終的にこの1報を採用した理由: 新着で、抗体そのものを対象にし、DMS+ML+実験検証が揃い、抗体医薬の前臨床評価に直結する交差反応性問題を扱っているため。
- 新着論文と基盤論文のバランスをどう考えたか: 前回も新着OpenGerminalだったが、今回も7月中旬の直接抗体論文が強かったため新着を優先。次回はDiffAb、MEAN、AbLangなど基盤寄りを挟む候補。
- PDF/HTML/Supplementaryを確認できたか: bioRxiv HTML full text、PDF、Supplementary materialページを確認。Supplementaryは supplementary figures PDF のみ確認。
- Figure/Tableを確認したURLやライセンス確認状況: Figure 1〜3は https://www.biorxiv.org/content/10.64898/2026.07.15.738714v1.full の各Figure anchorで確認。bioRxiv metadataとPDFリンクからCC BY-NC 4.0を確認。非商用条件付きのため画像は保存しない。
- GitHub/コード検索で使ったクエリ: `"Machine Learning-Guided Engineering of High-Affinity Cross-Reactive Antibodies" "github.com"`、`"2026.07.15.738714" GitHub`、`"Hugo Dorison" "GitHub" antibody`、`"Herve Nozach" "GitHub" antibody`、`"ML-guided" "cross-reactive antibodies" "PD-L1" code`
- 確認したGitHub URL: 著者実装は見つからず。Methods記載の外部ツールとして https://github.com/Merck/Mutation_Maker と https://github.com/wallnerlab/DockQ を確認。
- 確認したHugging Face URL: 見つからず。`site:huggingface.co` で論文タイトル、DOIを検索。
- 確認したその他コード/重み/データURL: bioRxiv supplementary figures PDF: https://www.biorxiv.org/content/biorxiv/early/2026/07/16/2026.07.15.738714/DC1/embed/media-1.pdf?download=true
- リポジトリライセンスの確認元: 著者実装リポジトリがないため該当なし。bioRxiv本文・図はCC BY-NC 4.0。
- モデル重み・チェックポイントの確認先: GitHub、Hugging Face、Zenodo検索、bioRxiv本文/Supplementary。見つからず。
