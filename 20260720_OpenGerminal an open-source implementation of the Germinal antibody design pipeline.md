# OpenGerminal: an open-source implementation of the Germinal antibody design pipeline

## まず何の論文か

OpenGerminal は、エピトープ指定型 de novo 抗体設計パイプライン Germinal を、より自由に使えるオープンソース構成へ置き換えた実装・ベンチマーク論文である。Germinal は AlphaFold-Multimer を通した勾配ベースの hallucination と抗体言語モデルを組み合わせ、固定抗体 framework 上の CDR を最適化して標的エピトープに結合する VHH/scFv を設計する。元の Germinal は 4 種類の抗原に対して標的あたり 43〜101 個程度の少数候補を試験し、ナノモル〜低マイクロモルの binder を得た点で重要だった。一方で、元実装は PyRosetta と IgLM に依存しており、商用利用、再配布、コンテナ化の面で制約があった。本論文は PyRosetta の relaxation/scoring を OpenMM、FASPR、FreeSASA、Biopython、sc-rs に置き換え、抗体言語モデルを IgLM から AbLang1-heavy に統一する。PD-L1 と IL-3 の VHH benchmark では、OpenGerminal は初期 cofolding filter に入る割合を PD-L1 で 18.6% から 33.7%、IL-3 で 8.0% から 24.6% に上げた。PD-L1 の最終 accepted seed rate も 1.1% から 1.8% に上がり、accepted design の Chai-1 confidence は同等以上だった。ただし Stage 1 は trajectory あたり約 1.5〜1.6 倍遅く、一部 Rosetta 由来指標は固定値の代替である。実験検証はまだなく、結合成功率を直接示した論文ではない。それでも、GitHub、Zenodo code archive、Apptainer container、Apache-2.0 license が揃っており、Germinal 系抗体設計を実際に試す入口として価値が高い。

## 書誌情報

- URL/DOI: https://doi.org/10.64898/2026.06.25.734527
- 公開日/更新日: bioRxiv posted June 29, 2026。PDF metadata の作成日は 2026-06-26。
- 著者・所属: Bing Han（University of Virginia, Center for Membrane and Cell Physiology / School of Medicine）、Sheng Li（University of Virginia, School of Data Science）
- 掲載誌/プレプリントサーバー: bioRxiv preprint
- リサーチ日: 2026-07-20 JST
- 分類: 新着 / モデル起点 / その他（実装・再現性改善）

## 背景と問題設定

抗体設計では、標的抗原のどのエピトープに結合させるか、抗体らしい発現性や安定性を保てるか、少数候補で実験に進めるかが重要になる。従来の hybridoma、phage display、免疫キャンペーンは有効だが、時間、実験規模、エピトープ制御に制約がある。近年は RFdiffusion や BindCraft のように、構造予測器や生成モデルを設計ループに入れ、少数候補で binder を得る方向が強まっている。Germinal はこの流れを抗体に適用し、AlphaFold-Multimer hallucination と抗体 language model の配列自然性を同時に使うことで、固定 framework 上に機能的 CDR を設計した。

本論文の問題は、Germinal の有望さではなく「使える実装としてどこまで開けるか」である。PyRosetta は強力だが商用利用を含むライセンス確認が必要で、IgLM も非商用条件があり、自由な再配布や企業利用の障壁になる。さらに Germinal は新しいパイプラインで、部品を置き換えた時に性能が保たれるかは十分に検証されていなかった。OpenGerminal は、制約の強い依存をオープンな部品に替え、Germinal architecture の頑健性と実装上の trade-off を評価する研究である。

## この論文のコアアイデア

コアは、Germinal の 4 段階構成を保ちつつ、ライセンス制約の大きい部品を置き換えることにある。Stage 1 は ColabDesign の AlphaFold-Multimer interface を用いた sequence hallucination で、pLDDT、iPTM、PAE などの構造予測信号と、抗体言語モデル log-likelihood を PCGrad で統合して CDR 配列を最適化する。OpenGerminal ではここで IgLM ではなく AbLang1-heavy を使う。Stage 2 は Chai-1 v0.6.1 による independent cofolding と initial filter、Stage 3 は AbMPNN による CDR redesign、Stage 4 は redesign 後の Chai-1 cofolding と final filter である。元 Germinal で PyRosetta FastRelax と score_interface が担っていた部分は、OpenMM + FASPR による relaxation/side-chain repacking、FreeSASA、Biopython、sc-rs による界面評価に置き換えられる。

新しい抗体生成モデルを提案する論文ではなく、Germinal の設計思想を保ったまま、ライセンス、コンテナ配布、multi-chain bug fix、部品置換の影響評価を行う論文として読むのがよい。

## 手法の詳細

- 入力データ: 標的抗原構造、target chain、binder chain、hotspot residue、VHH framework template、AF-Multimer parameters、Chai-1 weights、AbLang weights。
- 出力: accepted antibody design seeds、AbMPNN redesign 後の配列・PDB、Chai-1 confidence metric、interface geometry metric、pipeline stage ごとの通過ログ。
- モデル/アルゴリズム: Germinal 4-stage architecture。Stage 1 は AF-Multimer hallucination、Stage 2/4 は Chai-1 cofolding と open-source interface scoring、Stage 3 は AbMPNN CDR redesign。
- 特徴量・表現学習: AbLang1-heavy の language-model log-likelihood を抗体配列自然性として利用。構造品質には pLDDT、PTM、iPTM、interface PAE などを使う。
- 学習方法: 新規基盤モデルの学習ではない。入力標的に対して CDR sequence を勾配最適化し、AbMPNN で redesign する。
- 損失関数・目的関数: Stage 1 で structural objective と LM objective を PCGrad で統合。閾値や細かな重みは設定ファイル依存で、本文だけでは一部未確認。
- 推論方法: hallucination trajectory を生成し、Chai-1 cofolding、relaxation/scoring、AbMPNN redesign、final Chai-1 filter を順に通す。
- ベースライン: original Germinal pipeline。GitHub README では Germinal commit 88d7f85 との比較と記載。
- 評価指標: trajectory generation time、cofolding entry/pass rate、accepted seed rate、pLDDT、interface pLDDT、PTM、iPTM、interface PAE、steric clashes、hotspot proximity、CDR3-hotspot contacts、CDR interface fraction、post-hoc PyRosetta correlation。
- 実装上の重要点: PyRosetta utility module の 9 関数 interface を API-compatible に実装。binder_score、interface_dG、interface_hbonds は固定値で、interface_hbonds filter は実質無効化されている。multi-chain target では chain ID、chain renaming、Chai-1 sequence lookup、FreeSASA selection、sc-rs 入力制約を修正。

## データセットと評価設計

主要 benchmark は既報 Germinal の PD-L1 と IL-3 target を使う。両 pipeline で同じ hotspot residues と VHH framework を使い、trajectory timing と yield は job log から抽出した。完全な timing record を持つ trajectory のみを含め、Germinal は PD-L1 n=274、IL-3 n=376、OpenGerminal は PD-L1 n=499、IL-3 n=729 である。

accepted design quality は PD-L1 のみで比較された。Germinal は 7 unique seeds 由来の 16 PDB structures、OpenGerminal は 9 unique seeds 由来の 18 PDB structures である。IL-3 は両 pipeline とも accepted design が 0 だったため、最終構造品質の比較はできない。multi-chain support は insulin の公式 example で検証し、300 hallucination trajectories を走らせた。

評価設計は、同じ target と hotspot を使っているため部品置換の影響を見やすい。一方で標的数が少なく、IL-3 では最終成功がないため一般化評価としては限定的である。実験的 binding validation がなく、計算上の Chai-1 confidence や界面指標が実際の結合能に対応するかは未確認である。

## 主要結果

OpenGerminal の最大の効果は initial cofolding 周辺の yield 改善である。PD-L1 の cofolding entry は Germinal 51/274、18.6% から OpenGerminal 168/499、33.7% へ上がり、1.81 倍となった。IL-3 では 30/376、8.0% から 179/729、24.6% へ上がり、3.08 倍である。cofolding pass rate も PD-L1 で 10.9% から 18.4%、IL-3 で 4.0% から 5.2% に上がった。最終 accepted seed rate は PD-L1 で 1.1% から 1.8% に改善したが、IL-3 は両 pipeline とも 0% だった。

計算時間は悪化している。Stage 1 hallucination の平均時間は PD-L1 で 3.0 分から 4.4 分、IL-3 で 2.6 分から 4.2 分になった。これは trajectory あたり約 1.5〜1.6 倍の増加であり、OpenGerminal は cofolding へ進む候補が多いぶん、batch 全体の wall time はさらに増えうる。著者は AF-Multimer core model code の checksum が同じことを確認し、この差を主に IgLM から AbLang1 への置換に由来すると解釈している。

PD-L1 accepted designs の Chai-1 confidence は OpenGerminal が同等以上だった。overall pLDDT は median 0.889 から 0.908、PTM は 0.875 から 0.892、interface pLDDT は 0.899 から 0.917 に上がった。iPTM は 0.766 から 0.769、interface PAE は 0.957 から 0.961 で、有意差はない。界面幾何では両者とも steric clashes がなく、hotspot 近傍に位置し、CDR3-hotspot contact count も類似していた。ただし CDR が interface residues に占める割合は OpenGerminal で低く、median 82.8% vs. 100%、p<0.01 だった。

post-hoc PyRosetta validation では、sc-rs と PyRosetta の shape complementarity が Spearman rho=0.882 と高く、interface hydrophobicity は rho=0.763、surface hydrophobicity は rho=0.571 だった。open-source 指標は完全な代替ではないが、一部は PyRosetta と整合的に使えることが示された。multi-chain insulin example は実行完了したが、accepted design は 0 で、4 redesign candidates は binder_near_hotspot filter と interface confidence を満たさなかった。

## 重要なFigure/Table

- Figure/Table番号: Figure 1A
- 何を示しているか: Germinal と OpenGerminal の 4-stage pipeline 比較。IgLM が AbLang1-heavy に、PyRosetta が OpenMM/FASPR/FreeSASA/Biopython/sc-rs に置換される。
- 読み取り方: 変更点は sequence naturalness objective と relaxation/interface scoring に集中しており、Germinal の基本構造は保たれている。
- 主要な数値・傾向: architecture schematic のため数値はない。
- なぜ重要か: この論文の貢献が新規生成モデルではなく、Germinal の実装置換と検証であることが分かる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.64898/2026.06.25.734527v1.full.pdf
- ライセンス確認: 確認済み。PDF上で CC BY 4.0 International license と表示。

- Figure/Table番号: Figure 1B-E
- 何を示しているか: PD-L1 と IL-3 の trajectory generation time と pipeline yield。
- 読み取り方: OpenGerminal は遅いが、cofolding entry/pass が改善する。
- 主要な数値・傾向: PD-L1 cofolding entry 18.6%→33.7%、IL-3 8.0%→24.6%。PD-L1 Stage 1 平均時間 3.0分→4.4分、IL-3 2.6分→4.2分。
- なぜ重要か: OpenGerminal の主要 trade-off を最も直接示す。
- Markdown内での扱い: 要約表
- 出典URL: https://www.biorxiv.org/content/10.64898/2026.06.25.734527v1.full.pdf
- ライセンス確認: 確認済み。CC BY 4.0。

| 指標 | Germinal | OpenGerminal | 解釈 |
|---|---:|---:|---|
| PD-L1 cofolding entry | 18.6% | 33.7% | 1.81倍 |
| IL-3 cofolding entry | 8.0% | 24.6% | 3.08倍 |
| PD-L1 Stage 1 平均時間 | 3.0分 | 4.4分 | 約1.5倍遅い |
| IL-3 Stage 1 平均時間 | 2.6分 | 4.2分 | 約1.6倍遅い |
| PD-L1 accepted seed rate | 1.1% | 1.8% | modest improvement |
| IL-3 accepted seed rate | 0% | 0% | 両者成功なし |

- Figure/Table番号: Figure 1F-N / Supplementary Figure 1
- 何を示しているか: accepted PD-L1 designs の Chai-1 confidence、界面幾何、post-hoc PyRosetta との対応。
- 読み取り方: OpenGerminal は pLDDT/PTM/interface pLDDT で改善し、PyRosetta との shape complementarity も高く対応する。
- 主要な数値・傾向: pLDDT 0.889→0.908、PTM 0.875→0.892、interface pLDDT 0.899→0.917。sc-rs vs PyRosetta shape complementarity は Spearman rho=0.882。
- なぜ重要か: PyRosetta-free 指標が品質評価としてどこまで使えそうかを判断する根拠になる。
- Markdown内での扱い: リンクのみ
- 出典URL: https://www.biorxiv.org/content/10.64898/2026.06.25.734527v1.full.pdf
- ライセンス確認: 確認済み。CC BY 4.0。

## Code / License / Weights

- コード公開: あり
- GitHub URL: https://github.com/teaninja/OpenGerminal
- GitHub以外のコードURL: https://doi.org/10.5281/zenodo.20755400
- 実装種別: 公式 / 著者実装
- GitHub確認: 確認済み
- リポジトリのライセンス: Apache License 2.0
- ライセンス確認元: GitHubのLICENSEファイル / README / repository metadata / Zenodo metadata
- Hugging Face URL: 見つからず
- Hugging Face種別: 該当なし
- モデル重み・チェックポイント公開: あり。ただし OpenGerminal 独自重みではなく、既存モデル重み・パラメータを取得して使う形。
- 重み公開URL: Chai-1 weights と AbLang weights は初回実行時 download、AF-Multimer parameters は Google Storage の alphafold_params_2022-12-06.tar を取得する指示。コンテナは https://doi.org/10.5281/zenodo.20756013
- データセット公開: あり / 一部あり
- データセットURL: https://github.com/teaninja/OpenGerminal 、https://doi.org/10.5281/zenodo.20755400 、https://doi.org/10.5281/zenodo.20756013
- 再現性メモ: Zenodo code record は OpenGerminal v1.0.0 zip、license apache2.0。container record は `opengerminal_v1.0.0.sif`、約6.06GB、license apache2.0。README は Linux x86_64、NVIDIA GPU 40GB以上、Apptainer/Singularity、AF-Multimer parameters、Chai-1 weights を要求し、A100推奨。README には Germinal methodology が Stanford/Arc Institute の provisional patent application 対象である可能性への注意書きがあり、ソフトウェアライセンスとは別に商用利用時の特許確認が必要。

## 抗体研究・創薬への意味

OpenGerminal は、抗体設計アルゴリズムの理論よりも実装採用性に効く。Germinal 型設計は、標的エピトープ、抗体 framework、CDR 設計、少数候補での実験検証を結びつけるため、hit generation、エピトープ指定 binder、VHH/scFv ツール抗体、抗体医薬の初期探索に接続しやすい。PyRosetta と IgLM の制約を外したことで、企業内評価、教育、HPCコンテナ配布、派生実装の検討がしやすくなる。

また、AbLang1 が Germinal architecture 内で cofolding entry を大きく増やした点は、抗体言語モデル評価の観点でも重要である。抗体LMは perplexity や infilling 精度だけでなく、構造予測器を含む設計ループ内で productive な配列空間へ誘導できるかで見る必要がある。

## 限界と注意点

著者が述べる限界は明確である。OpenGerminal は Stage 1 で約1.5〜1.6倍遅い。binder_score、interface_dG、interface_hbonds は固定値で、特に interface_hbonds filter は実質無効化されている。binder_score 固定により Stage 2 ensemble selection は最良 relaxed structure を選ぶ挙動から劣化している。multi-chain support は実行可能性を示しただけで、accepted design は得ていない。prospective experimental validation もまだない。

追加の注意点として、benchmark target は PD-L1 と IL-3 に限られ、IL-3 は最終成功がないため、一般化性能は強く主張できない。OpenGerminal の cofolding entry 改善が実験的 hit rate に直結するかは未確認である。accepted design quality の差は AbLang1 と relaxation/scoring replacement の両方が関わるため、完全に分離して解釈できない。さらに Apache-2.0 のコードであっても、Germinal methodology の特許リスクは別問題である。

## この論文を読む上での前提知識

- Germinal: AlphaFold-Multimer hallucination と抗体言語モデルで CDR を設計するエピトープ指定抗体設計パイプライン。
- AlphaFold-Multimer hallucination: 構造予測モデルの信頼度や界面指標を目的関数にして、望ましい複合体を作る配列を逆向きに探索する方法。
- 抗体言語モデル: 抗体配列の自然性や文脈を学習したモデル。設計配列が抗体らしい空間から外れすぎないように使う。
- PyRosetta: タンパク質構造 relaxation、エネルギー評価、界面解析で広く使われる基盤。高機能だがライセンスが採用障壁になりうる。
- Chai-1: 複合体 cofolding と confidence 評価に使われる構造予測モデル。
- AbMPNN: 抗体 backbone/構造を条件に CDR 配列を redesign するモデル。
- VHH/scFv: 抗体断片形式。VHH は単一ドメイン抗体、scFv は重鎖・軽鎖可変領域を linker でつないだ形式。
- pLDDT、iPTM、PAE: 構造予測の confidence metric。binder設計では界面信頼度も重要になる。

## 今日この1報を選んだ理由

今回の検索では、より新しい候補として 2026-07-16 posted の「Machine Learning-Guided Engineering of High-Affinity Cross-Reactive Antibodies with Minimal Mutations」と、2026-07-14 posted の「Frozen Protein Foundation-Model Embeddings Improve Antibody-Antigen ΔΔG Ranking」が見つかった。前者は DMS と深層学習で抗PD-L1抗体のヒト/マウス交差反応性を少数変異で改善する実験色の強い論文、後者は antibody-antigen ΔΔG ranking に protein foundation model embedding を使う直接的な新着論文である。

それでも OpenGerminal を採用したのは、Germinal という重要な抗体設計パイプラインの再現性・ライセンス・実装可能性に直接効き、GitHub、Zenodo code archive、Zenodo container、Apache-2.0 license まで確認できたからである。前回までに AbAffinity、CHIMERA-Bench、AgForce を採用しており、今回は設計パイプラインを実際に動かす側の知識を補完できる。2026-06-29 posted で新着性も十分あり、基盤性とのバランスがよい。

## 読む優先度

High。Germinal はエピトープ指定 de novo 抗体設計の代表的な流れになりうるため、その open-source 実装は実務的に重要である。特に PyRosetta/IgLM のライセンス制約、AbLang1 置換、Chai-1 filter、AbMPNN redesign、Zenodo container がまとまって確認できる。ただし実験検証は未実施なので、設計成功率の論文ではなく「Germinal をどう使い、どこに注意するか」を学ぶ論文として読む。

## 自分用メモ

- 後で深掘りしたい点: original Germinal の experimental hit rate、filter threshold、CDR設計範囲、patent disclaimer。
- 関連して読むべき論文: Efficient generation of epitope-targeted de novo antibodies with Germinal、IgLM、AbLang、BindCraft、FreeBindCraft、RFdiffusion antibody design、Chai-1、AbMPNN。
- 実装を触る場合の入口: Zenodo の Apptainer container を取得し、README の PD-L1 config で small run。ただし A100級 GPU が必要。
- Obsidianでリンクしたいキーワード: [[Germinal]], [[OpenGerminal]], [[AbLang]], [[IgLM]], [[PyRosetta]], [[OpenMM]], [[Chai-1]], [[AbMPNN]], [[抗体設計]], [[エピトープ指定設計]], [[VHH]]

## 関連キーワード

- antibody design
- epitope-targeted antibody design
- Germinal
- OpenGerminal
- AlphaFold-Multimer hallucination
- AbLang1
- IgLM
- PyRosetta-free
- OpenMM
- FreeSASA
- sc-rs
- AbMPNN
- Chai-1
- VHH
- scFv
- CDR redesign

## 検索ログ

- 検索したデータベース/クエリ: Web検索 `2026 antibody design machine learning arXiv IgCraft AbMEGD RFAntibody AbBiBench`、`site:arxiv.org antibody design deep learning 2026 antibody antigen generation`、`site:biorxiv.org antibody design machine learning 2026 deep learning`、`AI antibody bioRxiv 2026 July`、`antibody machine learning bioRxiv July 2026`、`OpenGerminal bioRxiv`。
- 候補にした論文: OpenGerminal、Machine Learning-Guided Engineering of High-Affinity Cross-Reactive Antibodies with Minimal Mutations、Frozen Protein Foundation-Model Embeddings Improve Antibody-Antigen ΔΔG Ranking、CD98hc-targeted antibody shuttles for central nervous system delivery with broad cross-species reactivity。
- 最終的にこの1報を採用した理由: Germinal 系抗体設計パイプラインのライセンス・再現性・実装可能性に直接効き、コード、LICENSE、Zenodo code archive、Zenodo container が一次情報で確認できたため。
- 新着論文と基盤論文のバランス: 2026年6月末公開の新着寄り論文として扱った。より新しい7月中旬候補もあったが、基盤パイプラインへの接続と再現性確認を優先した。
- PDF/HTML/Supplementary確認: bioRxiv PDF を確認。Supplementary Figure 1 の存在は本文で確認したが、個別 supplementary file は未確認。
- Figure/Table確認URLとライセンス: https://www.biorxiv.org/content/10.64898/2026.06.25.734527v1.full.pdf 。PDF上で CC BY 4.0 International license 表示を確認。画像保存はせず要約・リンク扱い。
- GitHub/コード検索クエリ: `OpenGerminal GitHub`, `teaninja OpenGerminal`, `OpenGerminal LICENSE`, `OpenGerminal Zenodo`, `site:huggingface.co OpenGerminal opengerminal`。
- 確認したGitHub URL: https://github.com/teaninja/OpenGerminal
- 確認したHugging Face URL: 見つからず。
- 確認したその他コード/重み/データURL: https://doi.org/10.5281/zenodo.20755400 、https://doi.org/10.5281/zenodo.20756013
- リポジトリライセンス確認元: GitHub LICENSE、GitHub repository metadata、README、Zenodo metadata。Apache-2.0 を確認。
- モデル重み・チェックポイント確認先: GitHub README と Zenodo container metadata。Chai-1 weights と AbLang weights は初回実行時 download、AF-Multimer parameters は Google Storage から取得する指示。OpenGerminal 独自の学習済み重みは該当なし。
