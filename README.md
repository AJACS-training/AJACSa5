# AJACSa柏5

[統合データベース講習会AJACSadvanced(AJACSa)柏5 告知ページ](http://bit.ly/AJACSa5/)

## 開催概要
大学共同利用機関法人 情報・システム研究機構 ライフサイエンス統合データベースセンター(DBCLS)では、科学技術振興機構(JST) バイオサイエンスデータベースセンター(NBDC)が主催している初心者向けの統合データベース講習会AJACSを補完する形で、中上級者向けにAJACSadvanced講習会を実施しています。

今回、非モデル生物における転写配列アッセンブリデータベース(Transcriptome Shotgun Assembly: TSA)を利用し、転写単位ごとの遺伝子発現を定量化する方法を学ぶハンズオン講習会を開催します。

- 日時：2018年8月10日(金) 午後1時30分〜午後5時30分　
- 会場：大学共同利用機関法人 情報・システム研究機構 データサイエンス共同利用基盤施設 [ライフサイエンス統合データベースセンター](https://dbcls.rois.ac.jp/) （[〒277-0871 千葉県柏市若柴178-4-4 東京大学柏の葉キャンパス 駅前サテライト6階](https://dbcls.rois.ac.jp/access.html)） 
- 参加対象者 
	- 次世代シーケンサーDRY解析教本Level1を一通りマスターした程度のUNIXのコマンドラインが使える方
	- Mac所有者もしくはLinuxが使える方(講習はMacで。各自コンピュータを持ち込んでください。貸出はありません)
- プログラム
	- 13:30-17:30「非モデル生物における転写配列アッセンブリデータベース(Transcriptome Shotgut Assembly: TSA)を利用し、転写単位ごとの遺伝子発現を定量化する方法」  坊農 秀雅  (情報・システム研究機構ライフサイエンス統合データベースセンター)

## Sequence Read Archive (SRA)

いわゆる「次世代」シークエンサーから出てきた配列データの一次アーカイブ。
DDBJ/EBI/NCBIの3つによって運営されている国際塩基配列データベース共同研究[International Nucleotide Sequence Database Collaboration (INSDC)](http://www.insdc.org/)によるデータベースの一つ。

- [DDBJのSRAウェブサイト](https://www.ddbj.nig.ac.jp/dra/index.html)
- [NCBIのSRAウェブサイト](https://www.ncbi.nlm.nih.gov/sra)
- [EBIのENAウェブサイト](http://www.ebi.ac.uk/ena)

日本からだとDDBJのそれが最寄り。

### 【課題1】[DDBJ Search](http://sra.dbcls.jp/)を使って、SRAから興味深いデータを検索
1. 気になるFASTQデータをテキスト検索、SRA(DDBJ)からダウンロードします(例: DRR118520)
2. 出て来た**RUN**の所の*sra*のリンクを「リンクのアドレスをコピー」して、例えば`curl`コマンドで取得します
`curl -O ftp://ftp.ddbj.nig.ac.jp/ddbj_database/dra/sra/ByExp/sra/DRX/DRX111/DRX111587/DRR118520/DRR118520.sra`
3. ダウンロードしてきたファイルサイズがオリジナルのそれと同じならOK。そうでない場合は再取得

## Transcriptome Shotgun Assembly (TSA)

TSAは、INSDCによって維持されている、アッセンブルされたcDNA配列のデータベース。
TSAに関してもSRA同様、DDBJ/EBI/NCBIで同じデータが維持されている。
- [DDBJのTSAウェブサイト](https://www.ddbj.nig.ac.jp/ddbj/tsa.html)
- [NCBIのTSAウェブサイト](https://www.ncbi.nlm.nih.gov/genbank/tsa/)


### 【課題2】NCBIの[TSA Browser](https://www.ncbi.nlm.nih.gov/Traces/wgs/?view=TSA)を使って、TSAから興味深いデータを検索
- 気になるTSAデータをNCBIで検索
	- 左の'Taxonomic Groups’からや、'Term'から直接
- アッセンブルされた配列データ(FASTA形式)をダウンロード
	- 'Download'タブから
	- Sequence Read ArchiveのどのRUN ID（`SRR`,`ERR`, `DRR`のいずれかから始まる）からアッセンブルされたものか、に注意



## SRAとTSAを組み合わせた発現定量解析

- すでにアッセンブルされた配列セットはある 
- それを使ってそれの元となった（実は元でなくてもいい）ラン（複数可）のFASTQファイルを使って発現定量する 
- Trinity付属プログラムの[```align_and_estimate_abundance.pl```](https://github.com/trinityrnaseq/trinityrnaseq/wiki/Trinity-Transcript-Quantification) を使うと、転写量を見積もって定量できる


### 【課題3】アッセンブル済みのcDNA配列セットを再利用して発現定量

#### ツールの準備

まずAnaconda([miniconda](https://conda.io/miniconda.html))とBiocondaのインストール [【参考ブログ】](https://bonohu.wordpress.com/2017/07/08/bioconda/)

```
% conda config --add channels defaults
% conda config --add channels conda-forge
% conda config --add channels bioconda
```

- SRA形式ファイルをFASTQ形式に変換する
	- `fastq-dump`するのに必要なsra-toolsをインストール
```% conda install sra-tools```
	- fastq-dumpの実行例（paired-endの場合）
```% fastq-dump DRR118520.sra --split-files```
- 発現定量に必要なツールのインストール
	- samtools
```% conda install samtools```
	- kallisto
```% conda install kallisto```
	- `align_and_estimate_abundance.pl`は、Trinityパッケージに含まれているので、TrinityをGitHubから取ってくる。その際、以下の例ではホームディレクトリ以下の`Downloads`以下に取ってくるようにしてある。


```
% cd 
% cd Downloads 
% git clone https://github.com/trinityrnaseq/trinityrnaseq
```

#### 発現定量スクリプトの実行

以下のスクリプトを`align_and_estimate_abundance.sh`として保存　 [【参考ブログ】](https://bonohu.github.io/align-and-estimate-abundance.html)

```
#!/bin/sh
thre=4
transcript=IACV01.1.fsa_nt.gz
left=DRR118520_1_val_1.fq.gz
right=DRR118520_2_val_2.fq.gz
# parameters to run above
time perl /Users/bono/Downloads/trinityrnaseq/util/align_and_estimate_abundance.pl \
--thread_count $thre \
--transcripts $transcript \
--seqType fq \
--left  $left \
--right $right \
--est_method kallisto \
--kallisto_add_opts "-t $thre" \
--prep_reference --output_dir kallisto_out
```

以下のコマンドでスクリプトを実行

```% sh align_and_estimate_abundance.sh ```

#### 発現定量結果の閲覧

`% less kallisto_out/abundance.tsv`

### 【発展1】アッセンブル済みのcDNA配列セットのさらなる再利用

#### Transdecoderでタンパク質コード配列を予測する

```
% conda install transdecoder
% TransDecoder.LongOrfs -t IACV01.1.fsa_nt.gz
% TransDecoder.Predict -t IACV01.1.fsa_nt.gz
```

`IACV01.1.fsa_nt.transdecoder.pep` というファイル名で予測されたタンパク質配列が得られる。

#### 予測タンパク質配列をDBにしてlocalBLAST

- 統合TVのLocal BLAST編を参考に
  - [Local BLAST の使い方〜導入・準備編(MacOSX版)〜 2017](http://doi.org/10.7875/togotv.2017.031) 
  - [Local BLAST の使い方 〜検索実行・オプション〜 (MacOSX版) 2017](http://doi.org/10.7875/togotv.2017.045) 
- 興味ある遺伝子配列をqueryに、取ってきたデータセットに配列類似性の高い遺伝子がないか、検索してみましょう
