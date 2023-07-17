<!--
title:   Sketchのデータ構造(#1 目標と調査方法)
tags:    SketchPlugin,sketch,sketchtool
id:      da08748aee423fbc1177
private: true
-->
# はじめに
この記事はSketchのデータ構造を理解するために勉強したログです。
以下の記事に分かれています。

- \#1 目標と調査方法(<-)
- \#2 モデルの概要とデータ構造
- \#3 モデルの詳細と継承関係

# 参考資料
* [公式ドキュメント](https://www.sketch.com/docs/)
* [公式ドキュメント(API)](https://developer.sketch.com/reference/api/)
* [公式ドキュメント(CLI)](https://developer.sketch.com/cli/)
* [公式ドキュメント(レイヤのbool演算について)](https://www.sketch.com/docs/shapes/boolean-operations/)
* [SketchAPI(github)](https://github.com/sketch-hq/SketchAPI)
* [javascriptのクラス名取得方法](https://vividcode.hatenablog.com/entry/js/property-names)

# 目標
ほんとはSketch->QMLのConverterを作りたかった。(公式でもあるけど：[参考](https://doc.qt.io/qtdesignstudio/qtbridge-sketch-using.html))
時間がなかったため、とりあえずデータ構造だけ調べることに,,

# 調査方法

# 事件たち
### Console.appってなんだよ

### アプリどうやって起動すんだ

### Symbol Masterってなんだよ

### クラス名がおかしい
sketchのクラスが持つプロパティを調査するため、以下の方法でクラス名とプロパティを取得した
javascriptを初めて書くので(文法すら知らない)、[javascriptのクラス名取得方法](#参考資料)をパクった。

```javascript
var layers = sketch.Document.selectedLayers()
var o = layers[0]
while ( o ) {
    propNames = propNames.concat( Object.getOwnPropertyNames( o ) );
    o = Object.getPrototypeOf( o );
}
```

実行すると、eとかtみたいなあまりにシンプルすぎるクラス名が出てくるんだが...。
なんすかこれ。

### PrototypeにRectangleクラスがねえ
どうやら生成時に使うだけで、内部のデータはShapeクラスで持つらしい

### ドキュメントに記述の継承関係とgithubのコードの継承関係が違う
ShapeクラスとかGroupを継承してるし...。

### レイヤのindexってなんだよ
どうやら同じ親を持つレイヤに対して、割り振っているindexらしい。
つまり、異なる親を持つレイヤであれば同じindexになることがありうる。同じ親を持つレイヤ同士で同じindexはありえない。
多分イテレータで回すときは、indexで***<font color="Red">降順</font>***に参照する。
絵で↑に来るレイヤを後から追加するからでしょうね。これ図で説明したほうがいいな。

### CLIツールが動かねえ
Macをいちいち開くの面倒なので、ssh経由でCLIでsketchのPluginを動かしたくなった
CLIの公式ドキュメントに沿ってsketchtoolでやってみたけど、まともに動かない

```
シェルコード
```

エラーログにValid licenseが必要とか出るので、もしかして試用版だと使えないのか？と思い、sketchのライセンスを購入して(＄99)試したけどダメ。
クソみてえなツールだな。仕方ないのでSketchをいちいち起動してプラグインを実行することに...

### ShapeクラスとShapePathクラスがあるんだが
ShapeクラスはGroupクラスを継承してる。
SketchをGUIで動かしてもShapeクラスはできないんだが、これどういうクラスだ？
⇒[公式ドキュメント(レイヤのbool演算について)](#参考資料)参照

### Jsonファイルとデータ構造が違うじゃん
SketchのAPIやHeaderを調査した結果と実際のJsonファイルのデータ構造が違うんだが
→