# SaProt: Protein Language Modeling with Structure-aware Vocabulary

## まず何の論文か

SaProt は、アミノ酸配列だけで学習する従来の protein language model（PLM）に、三次元構造を「語彙」として取り込んだ汎用モデルである。各残基について、アミノ酸20種と Foldseek が割り当てる20種の 3Di 構造状態を組み合わせ、配列と局所構造を一つの structure-aware（SA）token で表す。これにより、座標を直接処理する大きな GNN を追加せず、ESM-2 とほぼ同じ Transformer を利用できる。650M parameter の主モデルは、AlphaFold2（AF2）で予測された約4,000万の配列–構造ペアで masked language modeling（MLM）事前学習された。

学習時には、maskした残基の構造tokenを見せたまま、主にアミノ酸を当てさせる。誤りを含みうる予測構造tokenを正解として強制せず、構造は残基予測を助ける文脈として使う。ClinVar、ProteinGym、熱安定性、HumanPPI、metal-ion binding、EC、GO、DeepLocを含む10種の評価で、同規模の ESM-2 や既存構造モデルを概ね上回った。特に HumanPPI は accuracy 76.67%から86.41%へ改善したが、ProteinGym の zero-shot Spearman ρ は0.475から0.478であり、全課題で改善幅が大きいわけではない。

抗体研究に直接特化したモデルではないが、抗体・VHHの予測構造を入力し、変異効果、安定性、発現性などの downstream model の encoder にできる可能性がある。ただし、本論文には抗体–抗原 affinity、CDR設計、humanization、developability の専用評価も wet-lab validation もない。SaProt は抗体生成器ではなく、「配列と構造を同じtoken列として扱う基盤表現モデル」と位置付けるのが正確である。

## 書誌情報

- URL/DOI: https://doi.org/10.1101/2023.10.01.560349
- 指定版: https://www.biorxiv.org/content/10.1101/2023.10.01.560349v5.full
- ICLR proceedings: https://proceedings.iclr.cc/paper_files/paper/2024/hash/1c42513b8895ab11fbbb5b7e8e6b6b02-Abstract-Conference.html
- 公開日/更新日: v1 2023-10-02、v5 2024-04-19
- 著者・所属: Jin Su、Chenchen Han、Yuyang Zhou、Junjie Shan、Xibin Zhou、Fajie Yuan。Zhejiang University、Westlake University
- 掲載誌/プレプリントサーバー: ICLR 2024 conference paper（公式GitHubでは Spotlight と記載）/ bioRxiv
- 論文ライセンス: bioRxiv v5 は `All rights reserved. No reuse allowed without permission.`
- リサーチ日: 2026-07-30 JST
- 分類: 基盤 / モデル起点 / protein language model

## 背景と問題設定

ESM 系PLMは、大量の一次配列から進化的制約や長距離依存を学ぶが、機能を直接規定する三次元配置は暗黙にしか扱わない。一方、実験構造で学ぶモデルはデータ数が限られ、GNN や pair representation は大規模化しにくかった。AlphaFoldDB により予測構造が大量に得られるようになったが、予測構造には model 固有の誤差もある。

著者らは予備実験で、AF2構造を graph として扱う MIF と、距離・角度を attention bias に入れる Evoformer-inspired PLM を MLM 学習した。AF2 validation loss は下がっても実験PDB構造の loss は連動せず、下流性能も弱かった。モデルが生物学的構造ではなく AF2 固有の予測痕跡へ適合した可能性がある。そこで本研究は、構造を離散的な「文字」に圧縮し、既存の sequence Transformer で約4,000万構造を学ぶ。

## この論文のコアアイデア

Foldseek は、各残基と三次元的な最近傍残基との幾何関係を20種の 3Di state に離散化する。SaProt は位置 \(i\) のアミノ酸 \(s_i\) と構造状態 \(f_i\) を連結し、`Md`、`Kc` のような一つの SA token \(s_if_i\) にする。両alphabetに未知記号 `#` を加えるため、語彙は \(21 \times 21 = 441\) token になる。

単純に SA token 全体を隠して残基と構造状態を両方当てるのではなく、入力を `#f_i` として構造を残し、残基を予測させる。Foldseek token は context であり、誤差を含む予測構造を強い教師ラベルにしない。AF2の pLDDT < 70 の領域では構造を `#` に置き換え、残基情報だけを使う。

## 手法の詳細

- 入力データ: アミノ酸配列と、AF2/PDB/ESMFold構造から Foldseek で得た残基単位の 3Di token。
- 出力: MLMのtoken probability、残基/タンパク質embedding。task headで変異効果、回帰、分類、多ラベル注釈、contactを予測。
- モデル: ESM-2 650M と同じ Transformer・parameter規模。主な変更は441 SA tokensへのembedding拡張。
- 事前学習data: ESM-2に準じてfilterした配列のUniProt IDから取得した約4,000万AF2構造。公開 `AF2_UniRef50` は現在約4,163万行。
- 学習: batchの15%を選び、80%は残基をmaskして構造を残す。10%はrandom token、10%は変更しない。pLDDT < 70 は構造を未知扱い。
- training: AdamW、max 1,024 token、batch size 512、約300万step、mixed precision。64基のA100 80 GBで約3か月。
- zero-shot変異効果: mutation位置の mutant と wild type の log probability差。各アミノ酸の確率は、そのアミノ酸を含む全構造状態について和を取る。
- 変異体構造: assay内の全variantに wild-type 構造を共用する。
- fine-tuning: 全parameterを学習。通常learning rate 2e-5、batch size 64。validation best checkpointを採用。
- ベースライン: ESM-1b/1v/2、Tranception、ESM-IF、MIF-ST、EVE、MSA Transformer、GearNet、ProstT5。
- 評価指標: AUC、Spearman ρ、accuracy、Fmax、contactのP@L。
- 実装上の重要点: AF2入力では pLDDT mask を使う。実験PDB中心なら PDB追加学習版を比較し、Foldseek binaryとmodel checkpointのversionを固定する。

## データセットと評価設計

| 区分 | データセット | train / valid / test | 指標・分割 |
|---|---|---:|---|
| zero-shot | ProteinGym substitution | filter後件数は未記載 | DMSごとのSpearman ρ。長さ>1,024、AFDB構造なしを除外 |
| zero-shot | ClinVar（EVE由来） | filter後件数は未記載 | AUC。1 Gold Star以上 |
| function | FLIP Thermostability | 5,056 / 639 / 1,336 | Spearman ρ |
| function | Metal Ion Binding | 5,067 / 662 / 665 | accuracy、30% identity cluster split |
| localization | DeepLoc subcellular / binary | 8,747 / 2,191 / 2,747、5,477 / 1,336 / 1,731 | accuracy |
| annotation | EC / GO | 13,089 / 1,465 / 1,604、26,224 / 2,904 / 3,350 | Fmax |
| interaction | PEER HumanPPI | 26,319 / 234 / 180 | binary accuracy |
| structure | TAPE Contact | 25,299 / 224 / 40 | backbone freeze、contact headのみ学習 |

Metal Ion Binding と DeepLoc 以外は TAPE、PEER、FLIP 等の公式splitを使い、元benchmark側で identity clustering/filtering が行われている。DeepLocは既に30% identityでclusterされ、元training setの20%をvalidationにした。全modelで AlphaFoldDBに構造がないproteinをtrain/testから除くため、元benchmark全体への性能ではない。

事前学習corpusとtest proteinの重複除去やdata cutoffは明示されない。labelを事前学習に使ったわけではないが、test proteinの配列・予測構造を見ている可能性があり、完全な unseen-protein 評価とは区別したい。

## 主要結果

| 課題 | ESM-2 650M | SaProt 650M | 差 |
|---|---:|---:|---:|
| ClinVar AUC | 0.862 | 0.909 | +0.047 |
| ProteinGym Spearman ρ（MSAなし） | 0.475 | 0.478 | +0.003 |
| Thermostability Spearman ρ | 0.680 | 0.724 | +0.044 |
| HumanPPI accuracy | 76.67% | 86.41% | +9.74 points |
| Metal Ion Binding accuracy | 71.56% | 75.75% | +4.19 points |
| EC Fmax | 0.868 | 0.882 | +0.014 |
| GO-MF / BP / CC Fmax | 0.670 / 0.473 / 0.470 | 0.682 / 0.486 / 0.479 | +0.012 / +0.013 / +0.009 |
| DeepLoc subcellular / binary | 82.09% / 91.96% | 85.57% / 93.55% | +3.48 / +1.59 points |

ProteinGymに MSA retrievalを加えると SaProtは0.489で、ESM-2 0.479、MIF-ST 0.480を上回る。一方、MSAなしでの ESM-2差は0.003に留まる。「構造を足せば常に大幅改善」とは言えない。

構造情報の保持を測る contact prediction では、long-range P@L が ESM-2の35.33に対し48.14、P@L/5が52.11に対し74.32だった。全構造tokenを隠した SaProtは long-range P@L 22.15まで低下した。ESMFold構造を使う版は多くの課題で ESM-2と同等以上だが、AF2版より弱く、構造sourceの品質が効く。

SaProt単体が全列で最高ではない。GearNetとのensembleはGO等で単体を上回る。論文の強い根拠は、同じarchitecture・規模の sequence-only ESM-2 と比べ、SA vocabularyが広い課題で改善を与えた点にある。

## 重要なFigure/Table

- Figure 1: SA vocabulary、mask戦略、ESM/BERTへの入力を示す中心図。21 × 21 = 441 tokens。新規性が巨大networkではなくtokenizationにあると分かる。Markdownではリンクのみ。出典: https://www.biorxiv.org/content/10.1101/2023.10.01.560349v5.full.pdf
- Table 1 / Table 2: zero-shot 2課題と supervised 8 datasetの主要比較。ClinVar 0.909、ProteinGym 0.478、HumanPPI 86.41%だが、改善幅は課題依存。必要な列だけを上の表に再構成。出典: https://proceedings.iclr.cc/paper_files/paper/2024/file/1c42513b8895ab11fbbb5b7e8e6b6b02-Paper-Conference.pdf
- Figure 2 / Table 8: AF2座標を直接使う予備モデルがAF2 lossだけを下げ、PDBと下流課題へ汎化しにくい現象。35M条件のClinVar AUCは Evoformer-inspired 0.589、MIF 0.638、SaProt 0.754。予測構造artifactへの警告として重要。出典は上記ICLR PDF。
- Markdown内での扱い: 画像は保存せず、要約表とリンクのみ。
- ライセンス確認: bioRxiv v5 は `No reuse allowed without permission`。ICLR proceedings側のFigure再利用条件は未確認。

## Code / License / Weights

- コード公開: あり
- GitHub URL: https://github.com/westlake-repl/SaProt
- 実装種別: 公式 / 著者実装
- GitHub確認: 確認済み
- リポジトリのライセンス: MIT
- ライセンス確認元: https://github.com/westlake-repl/SaProt/blob/main/LICENSE
- Hugging Face URL:
  - https://huggingface.co/westlake-repl/SaProt_35M_AF2
  - https://huggingface.co/westlake-repl/SaProt_650M_AF2
  - https://huggingface.co/westlake-repl/SaProt_650M_PDB
- Hugging Face種別: model。各model cardのライセンスはMIT
- モデル重み・チェックポイント公開: あり。Hugging Face形式とESM形式を収録
- データセット公開: あり
- 事前学習data: https://huggingface.co/datasets/westlake-repl/AF2_UniRef50
- downstream data: https://drive.google.com/drive/folders/11dNGqPYfLE3M-Mbh4U7IQpuHxJpuRr4g
- 再現性メモ: training、fine-tuning、zero-shot評価scriptとdataが公開される。一方、再学習は64基A100で約3か月と重く、READMEは公開fine-tuning configが論文設定と同一でないと明記する。downstream Google Driveの個別licenseは未確認。UniRef/AlphaFoldDB由来dataの条件も用途別に確認が必要。

## 抗体研究・創薬への意味

SaProtは、抗体残基に「局所三次元環境」を加えたembeddingを作れる。framework内部、CDR loop、表面、coreを区別できるため、少量の抗体labelで熱安定性、発現性、凝集傾向、変異効果を学習するencoder候補になる。同じ置換でも埋没部と露出部で影響が異なる状況に向く。

ただし、全variantにwild-type構造を共用するため、変異でCDR conformationや抗原界面が変わる効果は直接表現しない。Foldseek 3Diは単一chain内の局所幾何であり、抗体–抗原interface、VH–VL pairing、epitope/paratope complementarity、結合自由エネルギーの専用表現でもない。HumanPPIの改善を抗体affinityの証拠にはできない。

実務上は、抗体専用配列モデル、complex構造特徴、developability descriptor、実験labelと組み合わせるのが現実的である。生成候補の事前ranking、または発現・安定性dataへのfine-tuningが入口になる。評価splitはsequence identityだけでなく、clonotype、antigen、epitope単位で切る必要がある。

## 限界と注意点

著者が述べる限界:

- 性能は Foldseek の構造alphabet精度に依存する。
- 650Mは計算制約による規模で、scaling上限を検証していない。
- complex予測やstructure-constrained generationは提案に留まり未検証。

追加の注意点:

- 抗体専用課題、affinity、CDR設計、wet-lab検証はない。
- AFDB構造がないproteinを除外し、構造availabilityによるselection biasがある。
- wild-type構造を全variantに共用し、mutation-induced conformational changeを扱わない。
- ProteinGymでのESM-2からの改善は0.003と小さい。
- pretraining/test重複除去とdata cutoffが不明。HumanPPI testは180例、contact testは40例で、主要表にrun間分散やconfidence intervalがない。
- 低pLDDTをmaskするため、柔軟なCDR loopや天然変性領域では構造情報の利点が減る。
- max length 1,024で、full multi-chain antibody complex向け設計ではない。
- 論文のlearning-rate記載は「4e-4へ上げた後5e-4へlower」と数値上整合しない。再現時はconfigとcommitを固定したい。
- preprint Figureは転載不可。コード/modelのMITと、論文・由来dataの利用条件は分けて確認する。

## この論文を読む上での前提知識

- Protein language model: アミノ酸配列を自己教師あり学習し、残基/タンパク質embeddingを得るモデル。
- MLM: 入力の一部を隠し、文脈から元tokenを予測する学習。SaProtでは構造を文脈として残基を当てる。
- AlphaFold2 / AlphaFoldDB: 配列からの構造予測modelと大規模公開database。
- Foldseek 3Di: 残基と三次元最近傍との幾何関係を20状態で表す構造alphabet。
- pLDDT: AF2の残基単位confidence。SaProtは70未満の構造tokenを未知扱いにする。
- Zero-shot mutation prediction: 対象の実験labelで学習せず、PLMの尤度差から変異を順位付けする。
- Sequence-identity split: 近縁配列をtrain/testに跨がせない分割。抗体ではclonotypeやantigen分割も必要。

## 今日この1報を選んだ理由

今回はユーザー指定の論文である。抗体専用生成モデルだけでなく、そのencoderやfitness predictorを支える汎用protein foundation modelを理解するうえで基盤性が高い。構造をgraphとして直接扱わず語彙へ変換する明快な設計と、AF2 artifactへ適合する予備実験の両方が、予測構造を抗体AIへ投入するときの教材になる。

## 読む優先度

High。構造情報をPLMへ統合する代表的な基盤論文で、code、weights、pretraining/downstream dataが公開されている。抗体への直接証拠はないが、抗体変異効果・安定性modelのencoderを選ぶために、強みと構造依存の限界を理解する価値が高い。

## 自分用メモ

- 抗体/VHH datasetで ESM-2、SaProt、抗体専用PLMを同一split・同一headで比較する。
- CDR-H3、framework、paratope別にstructure tokenの寄与を分解する。
- wild-type構造共用とvariant再予測を比較する。
- complexではinter-chain contact/interface featureを追加する。
- 実装は `SaProt_650M_AF2`、Foldseek、pLDDT maskから始める。PDB中心なら `SaProt_650M_PDB` も比較する。
- 関連: [[SaprotHub]], [[ProstT5]], [[ProteinGym]], [[Foldseek]], [[ESM-2]]

## 関連キーワード

- SaProt
- protein language model
- structure-aware vocabulary
- Foldseek / 3Di token
- AlphaFoldDB
- ESM-2
- zero-shot mutation effect
- ProteinGym
- antibody engineering
- affinity maturation
- developability

## 検索ログ

- 確認先: 指定bioRxiv v5 full HTML/PDF、bioRxiv Details API、ICLR 2024 proceedings、公式GitHub、Hugging Face、downstream dataのGoogle Drive。
- 主な検索: `SaProt ICLR 2024`、`SaProt GitHub official`、`site:huggingface.co/westlake-repl SaProt 650M`、`westlake-repl AF2_UniRef50 license`。
- PDF確認: bioRxiv v5とICLR版の各23ページ、本文とAppendix A–Gを確認。独立Supplementaryは見つからず、付録はPDF内。
- Figure/Table確認:
  - https://www.biorxiv.org/content/10.1101/2023.10.01.560349v5.full.pdf
  - https://proceedings.iclr.cc/paper_files/paper/2024/file/1c42513b8895ab11fbbb5b7e8e6b6b02-Paper-Conference.pdf
- Figure license: bioRxiv APIとPDF bannerで `cc_no` / `No reuse allowed without permission` を確認。画像は保存しない。
- GitHub: https://github.com/westlake-repl/SaProt
- LICENSE: https://github.com/westlake-repl/SaProt/blob/main/LICENSE
- Hugging Face:
  - https://huggingface.co/westlake-repl/SaProt_650M_AF2
  - https://huggingface.co/westlake-repl/SaProt_650M_PDB
  - https://huggingface.co/datasets/westlake-repl/AF2_UniRef50
- その他data: https://drive.google.com/drive/folders/11dNGqPYfLE3M-Mbh4U7IQpuHxJpuRr4g
- 採用理由: ユーザー指定。配列と構造を統合する基盤PLMとして抗体AIへの接続価値が高い。
- 新着/基盤balance: 2024年の基盤論文として、直近の抗体生成論文を支える表現学習の土台を補う。
- 後続情報: 1.3B版とColabSaprot/SaprotHubも公開済みだが、指定された原論文の650M実験とは分けた。
