<!--
title:   githubで最もやべー関数を発掘する
tags:    C++,Java,JavaScript,Python,Swift
id:      2be9016f82653c8cc07f
private: false
-->
## はじめに
先日、職場で「自分が 改修したor 書いちゃった いちばんやべー関数」ネタで盛り上がりました。
みんないろいろ話してくれましたが、やっぱり僕の書いた「コマンドパターンのメインループ関数(1500行)」の圧勝でした。
なんであんなコード書いたんだろ。

そこで、今日は僕の傷ついたプライド癒すべくgithubから「世界でいちばんやべー関数」を発掘します。
つまり、「俺が書いた関数よりやべー関数に会いに行く」

## 結論
* マジでやべー関数は次の2つ
    * **「opencvリポジトリの[cv::agast_cornerScore\<AgastFeatureDetector::AGAST_7_12s>](https://github.com/opencv/opencv/blob/master/modules/features2d/src/agast_score.cpp#L3383)関数」(複雑度1868)**
    * **「SuiteCRMリポジトリの[OpenTag](https://raw.githubusercontent.com/salesagility/SuiteCRM/master/modules/AOS_PDF_Templates/PDF_Lib/mpdf.php)関数」(複雑度1509)**
* 言語毎の傾向に着目すると...
  * **javascriptにはやべー関数が多い**
  * **python/java/swift/rubyはきれいな関数が多い**

## 方法
### やべー関数ってなんだよ
ここでは、以下の手続きでやべー関数を洗い出します。
1. 「循環的複雑度が高い関数」を洗い出す。
　　　循環的複雑度は[lizard](https://github.com/terryyin/lizard)を使って算出しました。
2.  本当にやべー関数に絞り込む
　　　中身を見て、テストコードとか自動生成コードを除外する。

### 対象リポジトリの決定
[github-trending-api](https://github.com/huchenme/github-trending-api)を使い、
以下の言語の最近1カ月の人気プロジェクトを抜き出して、やべー関数を探します。

* C
* C++
* Python
* Java
* JavaScript
* Swift
* Ruby
* PHP
* Scala
* Golang
* Lua

## コード
テストとか全くしてねぇ
[cycro](https://github.com/AngryMane/cycro)

## 結果

#### 1.「循環的複雑度が高い関数」を洗い出す
コードの中身とか考えず、複雑度が高い関数Top10は以下の通り。

|No| 複雑度 | 関数名 | プロジェクト名 |言語名|
|:-----------------:|:-----------------:|:-----------------:|:------------------:|:------------------:|
|1|5505|[jo](https://raw.githubusercontent.com/nodejs/node/master/deps/v8/tools/profviz/gnuplot-4.6.3-emscripten.js)|[node](https://github.com/nodejs/node)|javascript|
|2|2013|[matchIcon](https://github.com/GitSquared/edex-ui/blob/master/src/assets/misc/file-icons-match.js)|[edex-ui](https://github.com/GitSquared/edex-ui)|javascript|
|3|2001|[foo](https://github.com/llvm/llvm-project/blob/master/clang/test/Sema/many-logical-ops.c)|[llvm-project](https://github.com/llvm/llvm-project)|cpp|
|4|1947|[\*global*](https://raw.githubusercontent.com/nodejs/node/master/deps/v8/tools/profviz/gnuplot-4.6.3-emscripten.js)|[node](https://github.com/nodejs/node)|javascript|
|5|1868|[cv::agast_cornerScore\<AgastFeatureDetector::AGAST_7_12s>](https://github.com/opencv/opencv/blob/master/modules/features2d/src/agast_score.cpp#L3383)|[opencv](https://github.com/opencv/opencv)|cpp|
|6|1647|[int](https://github.com/kubernetes/kubernetes/blob/master/vendor/gonum.org/v1/gonum/graph/formats/dot/internal/lexer/transitiontable.go)|[kubernetes](https://github.com/kubernetes/kubernetes)|go|
|7|1532|[foo](https://github.com/llvm/llvm-project/blob/master/clang/test/OpenMP/nesting_of_regions.cpp#L9)|[llvm-project](https://github.com/llvm/llvm-project)|cpp|
|8|1509|[OpenTag](https://raw.githubusercontent.com/salesagility/SuiteCRM/master/modules/AOS_PDF_Templates/PDF_Lib/mpdf.php)|[SuiteCRM](https://github.com/salesagility/SuiteCRM)|php|
|9|1504|[foo](https://github.com/llvm/llvm-project/blob/master/clang/test/OpenMP/nesting_of_regions.cpp#L8965)|[llvm-project](https://github.com/llvm/llvm-project)|cpp|
|10|1453|[iT](https://raw.githubusercontent.com/nodejs/node/master/deps/v8/tools/profviz/gnuplot-4.6.3-emscripten.js)|[node](https://github.com/nodejs/node)|javascript|難読化してるため|

#### 2. 本当にやべー関数に絞り込む
コードの中身を実際に見てみると...

|No| 複雑度 | 関数名 | プロジェクト名 |言語名|備考|
|:-----------------:|:-----------------:|:-----------------:|:------------------:|:------------------:|:------------------:|
|1|5505|[jo](https://raw.githubusercontent.com/nodejs/node/master/deps/v8/tools/profviz/gnuplot-4.6.3-emscripten.js)|[node](https://github.com/nodejs/node)|javascript|難読化してるため|
|2|2013|[matchIcon](https://github.com/GitSquared/edex-ui/blob/master/src/assets/misc/file-icons-match.js)|[edex-ui](https://github.com/GitSquared/edex-ui)|javascript|if文を2000個並べてるだけ|
|3|2001|[foo](https://github.com/llvm/llvm-project/blob/master/clang/test/Sema/many-logical-ops.c)|[llvm-project](https://github.com/llvm/llvm-project)|cpp|テストのために&&を2000個並べてるだけ|
|4|1947|[\*global*](https://raw.githubusercontent.com/nodejs/node/master/deps/v8/tools/profviz/gnuplot-4.6.3-emscripten.js)|[node](https://github.com/nodejs/node)|javascript|難読化してるため|
|5|1868|[cv::agast_cornerScore\<AgastFeatureDetector::AGAST_7_12s>](https://github.com/opencv/opencv/blob/master/modules/features2d/src/agast_score.cpp#L3383)|[opencv](https://github.com/opencv/opencv)|cpp|**こいつはやばい**|
|6|1647|[int](https://github.com/kubernetes/kubernetes/blob/master/vendor/gonum.org/v1/gonum/graph/formats/dot/internal/lexer/transitiontable.go)|[kubernetes](https://github.com/kubernetes/kubernetes)|go|パーサジェネレータで自動生成したコード|
|7|1532|[foo](https://github.com/llvm/llvm-project/blob/master/clang/test/OpenMP/nesting_of_regions.cpp#L9)|[llvm-project](https://github.com/llvm/llvm-project)|cpp|テスト用コード|
|8|1509|[OpenTag](https://raw.githubusercontent.com/salesagility/SuiteCRM/master/modules/AOS_PDF_Templates/PDF_Lib/mpdf.php)|[SuiteCRM](https://github.com/salesagility/SuiteCRM)|php|**こいつはやばい**|
|9|1504|[foo](https://github.com/llvm/llvm-project/blob/master/clang/test/OpenMP/nesting_of_regions.cpp#L8965)|[llvm-project](https://github.com/llvm/llvm-project)|cpp|テスト用コード|
|10|1453|[iT](https://raw.githubusercontent.com/nodejs/node/master/deps/v8/tools/profviz/gnuplot-4.6.3-emscripten.js)|[node](https://github.com/nodejs/node)|javascript|難読化してるため|

#### 結論
備考欄記述の通り、以下の二つが本当にやべー関数。
実際のコードは君の目で確かめてくれ！(長すぎて張れない)

|No| 複雑度 | 関数名 | プロジェクト名 |言語名|備考|
|:-----------------:|:-----------------:|:-----------------:|:------------------:|:------------------:|:------------------:|
|5|1868|[cv::agast_cornerScore\<AgastFeatureDetector::AGAST_7_12s>](https://github.com/opencv/opencv/blob/master/modules/features2d/src/agast_score.cpp#L3383)|[opencv](https://github.com/opencv/opencv)|cpp|**こいつはやばい**|
|8|1509|[OpenTag](https://raw.githubusercontent.com/salesagility/SuiteCRM/master/modules/AOS_PDF_Templates/PDF_Lib/mpdf.php)|[SuiteCRM](https://github.com/salesagility/SuiteCRM)|php|**こいつはやばい**|


### 言語間の複雑度の比較
言語毎に複雑度の分布をプロットしてみた。

##### 外れ値を含むバイオリンプロット
![alt](http://s.kota2.net/1573379989.png)

##### 外れ値を含まないバイオリンプロット
言語毎に平均値±2*標準偏差の範囲外の値を除外すると、以下の通り。
![alt](http://s.kota2.net/1573379990.png)


## 最後に
なんか間違ってても勘弁して...