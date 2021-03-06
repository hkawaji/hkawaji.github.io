---
layout: post
title:  "核酸医薬ヌシネルセンの塩基配列検索 (テクニカル・ノート)"
tag: [japanese]
---

核酸医薬ヌシネルセンが相補鎖を構成し得る核酸を、ヒトゲノム塩基配列の中から検索することを試みます。

***( 注 2019.01.12 bowtie_queryEditに、[とんでもない初歩的な間違いが含まれていた旨の指摘をいただきました](https://twitter.com/meso_cacase/status/1083677108159692800)。いかに残念な間違いだったか、念のために(?)[残しておきます](https://github.com/hkawaji/hkawaji.github.io/blob/5c4c82ca9c1961f0d98c8371694ad2ce60100ce2/_posts/2019-01-10-TCACTTTCATAATGCTGG.md) )。***

但し書き:

* これは包括的なサーベイではありません。個人的に「これぐらいは確認しておくべきだろうな」と思いつく範囲のツールだけを採り上げています。
* ここに記載のパラメータは最適である保証はありません。私自身はどのツールの開発者でもありませんので。
* 「相補鎖を構成し得る」として、核酸同士のミスマッチあるいはギャップの合計つまり編集距離(edit distance)が2以内、とテクニカルに定義しています。これは、後にヌシネルセンの[PMDAの審査資料](http://www.pmda.go.jp/drugs/2017/P20170731001/630499000_22900AMX00587_A100_2.pdf) と照らし合わせることを意図したものです。(実際の評価では、評価項目に応じて適切な基準を選定すべきです）
* ヒトゲノム・リファレンス配列(GRCh38)を検索対象と設定します (実際の評価では、評価項目に応じて適切な検索対象を選定すべきです）



TL;DR (まずは結果を)
---
リファレンス・ゲノム内で見つかったヒットの数と、[PMDAの審査資料](http://www.pmda.go.jp/drugs/2017/P20170731001/630499000_22900AMX00587_A100_2.pdf) 17ページで言及されている遺伝子の領域にヒットしたかをまとめると、下記の表のようになります。

```
BNC2      blast     bowtie    bowtie2   bowtie_queryEdit  gggenome  last  vmatch
EFCAB6    blast     -         bowtie2   bowtie_queryEdit  gggenome  last  vmatch
FIGLA     blast     bowtie    -         bowtie_queryEdit  gggenome  last  vmatch
FOXP1     blast     bowtie    bowtie2   bowtie_queryEdit  gggenome  last  vmatch
FREM2     -         bowtie    bowtie2   bowtie_queryEdit  gggenome  last  vmatch
MECOM     blast     bowtie    bowtie2   bowtie_queryEdit  gggenome  last  vmatch
MSR1      blast     bowtie    bowtie2   bowtie_queryEdit  gggenome  last  vmatch
RPGRIP1L  blast     bowtie    -         bowtie_queryEdit  gggenome  last  vmatch
RSF1      blast     -         bowtie2   bowtie_queryEdit  gggenome  last  vmatch
STAT4     blast     bowtie    bowtie2   bowtie_queryEdit  gggenome  last  vmatch
TRHDE     blast     bowtie    bowtie2   bowtie_queryEdit  gggenome  last  vmatch
ULK2      blast     -         -         bowtie_queryEdit  gggenome  last  vmatch
hg38ヒット数  453    231       394       774               774       660   774
```

それぞれのヒット位置については、ゲノム軸上で閲覧できるようにしてあります ( [BNC2](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr9:16409503-16870706) / [EFCAB6](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr22:43625363-43628441) / [FIGLA](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr2:70777310-70790643) / [FOXP1](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr3:70952817-71583993) / [FREM2](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr13:38687129-38887131) / [MECOM](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr3:169084761-169663470) / [MSR1](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr8:16109431-16192715) / [RPGRIP1L](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr16:53598153-53703889) / [RSF1](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr11:77659996-77821017) / [STAT4](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr2:191029576-191151590) / [TRHDE](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr12:72272683-72670757) / [ULK2](http://genome-asia.ucsc.edu/cgi-bin/hgTracks?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen&position=chr17:19770830-19867936)。[テーブル・ブラウザ](http://genome-asia.ucsc.edu/cgi-bin/hgTables?hgS_doOtherUser=submit&hgS_otherUserName=Kawaji&hgS_otherUserSessionName=190112%2DNushinerusen) を経由して、ヒット位置をダウンロードすることもできます )。



blast, bowtie, gggenome, last, vmatchはいずれもアラインメント・ツールの名称です(詳細などは次節参照)。bowtie_queryEditは、クエリ配列に1塩基、2塩基の挿入・欠失を **あらかじめ入れた上で** bowtieを用い検索した結果です(bowtieはギャップを含むアラインメントを見つけられるようには設計されていないため)。



但し書き:

* 審査資料で “TRDHE” と言及されていた遺伝子ですが、そのような名前の遺伝子は [EntrezGene](https://www.ncbi.nlm.nih.gov/gene/) でも [HGNC](http://genenames.org) データベースでも見当たりませんでした。代わりに “TRHDE” と考えて(ややこしいですね)、結果を示しています。
* 実は  bwa, hisat2もトライしてみましたが、coverageがあまりに少ない結果しか得られませんでしたので、ここからは省いています。


ツールについて
---
* GGGenome - 文字列検索エンジン・データベースが一体となったオンラインツール。
	- [https://gggenome.dbcls.jp/](https://gggenome.dbcls.jp/)
	- 検索エンジン部分を除く、インターフェース部分のソースコードは[github](https://github.com/meso-cacase/GGGenome) が公開されています
* BLAST - 1990年から開発されている、説明不要(?)な配列検索ツール。
	- [https://blast.ncbi.nlm.nih.gov/](https://blast.ncbi.nlm.nih.gov/)
	- Camacho C, Coulouris G, Avagyan V, Ma N, Papadopoulos J, Bealer K, Madden TL. BLAST+: architecture and applications. BMC Bioinformatics. 2009 Dec 15;10:421. [PMID:20003500](https://www.ncbi.nlm.nih.gov/pubmed/20003500)
* Bowtie - 次世代シーケンサー、特に短鎖配列のゲノムにマッピングの時に使われることが多い。ギャップを含むアラインメントができない。
	- [http://bowtie-bio.sourceforge.net/bowtie/](http://bowtie-bio.sourceforge.net/bowtie/)
	- Langmead B, Trapnell C, Pop M, Salzberg SL. Ultrafast and memory-efficient alignment of short DNA sequences to the human genome. Genome Biol. 2009;10(3):R25 [PMID:19261174](https://www.ncbi.nlm.nih.gov/pubmed/19261174)
* Bowtie2 - 次世代シーケンサー、少しだけ長い配列のゲノムにマッピングの時に使われることが多い。ギャップ有りアラインメントができる。
	- [http://bowtie-bio.sourceforge.net/bowtie2/](http://bowtie-bio.sourceforge.net/bowtie2/)
	- Langmead B, Salzberg SL. Fast gapped-read alignment with Bowtie 2. Nat Methods. 2012 Mar 4;9(4):357-9. [PMID:22388286](https://www.ncbi.nlm.nih.gov/pubmed/22388286)
* LAST - 設定次第でとてもいろいろなことができるツール。
	- [http://last.cbrc.jp/](http://last.cbrc.jp/)
	- Kiełbasa SM, Wan R, Sato K, Horton P, Frith MC. Adaptive seeds tame genomic sequence comparison. Genome Res. 2011 Mar;21(3):487-93. [PMID:21209072](https://www.ncbi.nlm.nih.gov/pubmed/22388286)
	- [他にも論文多数](http://last.cbrc.jp/doc/last-papers.html)
* VMATCH - 繰り返し配列をしっかりとハンドリングできるツール。
	- [http://www.vmatch.de/](http://www.vmatch.de/)
	- M.I. Abouelhoda, S. Kurtz, and E. Ohlebusch. The enhanced suffix array and its applications to genome analysis. In Proceedings of the Second Workshop on Algorithms in Bioinformatics, pages 449–463. Lecture Notes in Computer Science 2452, Springer-Verlag, 2002.
	- 以前は利用に(簡単な)ライセンスが必要でしたが、現在はバイナリを自由にダウンロードし、利用することができます。





塩基配列検索
---
* __検索対象の塩基配列データベース(ゲノム):__ UCSC Genome Browser Databaseにおいて配布されている ヒトゲノムのリファレンス[配列GRCh38/hg38](http://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/) を対象とします (単純化のため、ハプロタイプ配列; ランダム配列を除いています)。
* __検索する塩基配列:__ [ヌシネルセンに関するPMDAの審査結果報告書](http://www.pmda.go.jp/drugs/2017/P20170731001/630499000_22900AMX00587_A100_2.pdf)より、塩基配列 "TCACTTTCATAATGCTGG" を検索配列とします(下記のquery.fa は、この配列のみを含むファイルです)
* それぞれのツールの他、curl, awk, samtools, bedtools を利用して、編集距離2以下のヒットを検索します。コード・ブロックの中身は、塩基配列検索を実行し[BED形式](https://genome.ucsc.edu/FAQ/FAQformat.html)で出力するシェル・スクリプトです。


#### GGGenome

```sh
curl https://gggenome.dbcls.jp/hg38/2/TCACTTTCATAATGCTGG.bed \
| grep -v '_'  \
| grep -v track 
```
さすがに便利です。そして、早い。

#### blast (2.7.1+)

```sh
blastn  -db hg38.fa -query query.fa -outfmt "17 SR SQ" -word_size 4 -strand both -evalue 100000 \
| samtools calmd - hg38.fa \
| awk 'BEGIN{OFS="\t"}{
  if (match($0,"^@")){
    print
  }else{
    edit_dist = 0;
    if ( match($0,"NM:i:[0-9]+") ) { edit_dist += substr( $0, RSTART + 5, RLENGTH - 5) }
    if ( match($6, "^[0-9]+H"  ) ) { edit_dist += substr( $6, RSTART, RLENGTH - 1) }
    if ( match($6, "[0-9]+H$"  ) ) { edit_dist += substr( $6, RSTART, RLENGTH - 1) }
    $5 = edit_dist # put edit_dist as score
    $1 = $1 ";edit_dist:"edit_dist ";CIGAR:" $6
    if ($5 <= 2){ print }
  } }' \
| samtools view -hu - \
| bamToBed
```
やっていることは、次のとおりです。

1. word sizeとして設定可能な最も小さな値で配列検索し、結果をSAMフォーマットで出力 (後でフィルタすることを見越し、evalueをとても大きくしてあります)
2. samtoolsを使い edit distanceを再計算した上で、hard clipをedit distanceの計算にいれる
3. edit distanceが2以下のもののみを選択し、出力


（久しぶりに使ってみたところ、なんとsamフォーマットでアラインメントが出力できるようになっていて、びっくりしました。blastもがんばっていますね。ただしmakeblastdb の際に -parse_seqids をつけないと、染色体名がきちんと認識されないので注意を。）


#### bowtie (1.2.2)

```sh
bowtie --maxbts 100000 -f --all -v 2 --sam hg38  query.fa \
| samtools view -hb - \
| bamToBed
```
ミスマッチを2個まで許し(-v 2)、みつかったものを全部出力(--all)しています (ins/delが探せない以外は、悪くない)。


#### bowtie2 (2.3.4.3)

```sh
bowtie2 --all -x hg38 -D 40 -R 3 -N 0 -L 6 -i S,1,0 --gbar 1 --score-min G,-5,-5 -f query.fa \
| grep -e @ -e NM:i:0 -e NM:i:1 -e NM:i:2 \
| samtools view -hb - \
| bamToBed
```
seed長を6(-L 6)にして、1bpごとにスライディングさせています(-i S,1,0)。低いアラインメントスコアでもとにかく出力するように (--score-min G,-5,-5)。出力結果から、NMタグを使ってedit distance2以下のものだけを選択しています (期待した程は、見つけてくれなかったですね)。


#### bowtie_queryEdit (bowtie は上記と同じ 1.2.2)

```sh
query=">query\nTCACTTTCATAATGCTGG"

delOne ()
{
  awk '{
  if ( $1 ~ "^>" ) { orig_name = substr($1,2); next}
  len = length($1);
  for (i=0;i<len;i++)
  {
    name = orig_name ";" substr($1,1,i) "_" substr($1,i+2)
    str = substr($1,1,i) substr($1,i+2)
    print ">" name "\n" str
  }
  }'
}

insOne ()
{
  awk '{
  if ( $1 ~ "^>" ) { orig_name = substr($1,2); next}
  split( "A T G C" , nuc, " " )
  len = length($1);
  for (n in nuc)
  { 
    for (i=0;i<=len;i++)
    {
      name = orig_name ";" substr($1,1,i) nuc[n] substr($1,i+1)
      str = substr($1,1,i) nuc[n] substr($1,i+1)
      print ">" name "\n" str
    }
  }
  }'
}

indelOne ()
{
  echo $1 | insOne
  echo $1 | delOne
}

indelTwo ()
{
  echo $1 | insOne | insOne
  echo $1 | delOne | delOne
  echo $1 | delOne | insOne
}

echo $query     | bowtie --maxbts 100000 -f --all -v 2 --sam hg38 - > zero.sam
indelOne $query | bowtie --maxbts 100000 -f --all -v 1 --sam hg38 - > one.sam
indelTwo $query | bowtie --maxbts 100000 -f --all -v 0 --sam hg38 - > two.sam

samtools merge - zero.sam one.sam two.sam \
| bamToBed
```
無理矢理やなぁ...という力技(brute force)ですが、この程度の配列長とins/del数であれば、あっという間に終わります。下手に凝ったツールやパラメータを探したりするよりもよっぽど直球かもしれません(配列長がもっと長くなれば、bowtie2やlastとかが真価を発揮するのでしょうし)。ins/del無しのときは編集距離2以内の結果すべてを出力 (--all -v 2)、ins/delが1個のときは編集距離1以内(--all -v 1)、ins/delが2個のときは編集距離0の結果すべてを出力 (--all -v 0)しています。


#### last (941)

```sh
lastal -L6 -m0 -d0 -y0 -a0 -x 2 -e 14 hg38-m1R00 query.fa \
| maf-convert sam /dev/stdin \
| tr '=' 'M' \
| samtools view -h -T hg38.fa -  \
| samtools calmd - hg38.fa \
| awk 'BEGIN{OFS="\t"}{
  if (match($0,"^@")){
    print
  }else{
    edit_dist = 0;
    if ( match($0,"NM:i:[0-9]+") ) { edit_dist += substr( $0, RSTART + 5, RLENGTH - 5) }
    if ( match($6, "^[0-9]+H"  ) ) { edit_dist += substr( $6, RSTART, RLENGTH - 1) }
    if ( match($6, "[0-9]+H$"  ) ) { edit_dist += substr( $6, RSTART, RLENGTH - 1) }
    $5 = edit_dist # put edit_dist as score
    $1 = $1 ";edit_dist:"edit_dist ";CIGAR:" $6
    if ($5 <= 2){ print }
  } }' \
| samtools view -hu - \
| bamToBed
```

lastはとてもいろいろできるツールですが、[こちらのドキュメント](http://last.cbrc.jp/doc/tag-seeds.txt)を参考にしてパラメータを設定しました("3. Guaranteeing to align reads with up to N mismatches or indels")。インデックスにはlastdbを用い、可能な限り多くのseedをとり(-m1)、繰り返し配列をマスクしない(-R00)ように構築しました。編集距離については上記blastと同様、検索終了後にsamtoolsを使って計算しました。（でも、もっと設定を工夫すればきちんと見つかるかも。。。時間のあるときに、開発者の方に聞いてみたいと思います）


#### vmatch (2.3.0)

```sh
vmatch -d -p -e 2  -showdesc 10 -q query.fa  -complete hg38.fa \
| grep -v ^# \
| awk 'BEGIN{OFS="\t"}{
    left_len=$1; left_seq=$2; left_start=$3;
    strand = "+"; if ( $4 == "P") {strand = "-"}
    right_len=$5; right_seq=$6; right_start=$7;
    dist=$8; eval=$9; score=$10;
    name = right_seq ",dist:"dist ",eval:" eval
    print left_seq, left_start, left_start + left_len, name, score, strand
  }'
```
編集距離が2以下のもの(-e 2)、というオプションを指定しています (さすがvmatch、の結果でしたね)。
