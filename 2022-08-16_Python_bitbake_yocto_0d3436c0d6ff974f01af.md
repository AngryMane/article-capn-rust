<!--
title:   Bitbakeをpythonから使おう
tags:    Python,bitbake,yocto
id:      0d3436c0d6ff974f01af
private: false
-->
# はじめに
yoctoを使ってカスタムしたlinuxをビルドをビルドする仕事してます。
(僕は最近全然仕事してないけど)
この仕事では、例えば以下のようなリクエストが来ます。

* 特定のbbclass(core-image.bbclassとか)を継承したファイルをリストアップしてほしい
* すべてのパッケージのある変数(例えばPE,PV,PR)をリストアップしてほしい
* 特定のパッケージ(例えばgcc)にあたってるbbappendファイルをリストアップしてほしい

面倒くさくない？ 僕はgrepしたくないし、時間かかるからビルドもしたくないんだよ...


# Bitbakeをpythonから使おう

面倒臭いので、なんとかする方法を調査しましょう。
まず、bitbakeコマンドによるビルドの仕組みがざっくりこんな感じ。

![bitbake.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/185272/a6ed90f5-051a-2bde-6001-b80a3c34316f.png)

このTinfoilがすごい複雑なので使いづらいんですね。実際のコードは[ここ](https://git.yoctoproject.org/poky/tree/bitbake/lib/bb/tinfoil.py?h=kirkstone)
一方、server_client間のIFは[ここ](https://git.yoctoproject.org/poky/tree/bitbake/lib/bb/command.py?h=kirkstone)にある程度きれいに定義されています。
これらの事情を考慮すると、↓のようにすればよさそうですね

![bitbake.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/185272/cac48d75-0429-14b8-5fbd-33d454b669e5.png)

このbbclientをpythonで作ればlanguage serverとかからも使いやすくて良さそう。

### 作ったもの

上述の経緯で作ったのが↓のbbclientです。

https://pypi.org/project/bbclient/

ドキュメントも用意しました。英語で書いてるので変なところあったら教えてほしいです...

* [IFの仕様](https://angrymane.github.io/bbclient/bbclient.html)
* [使い方](https://angrymane.github.io/bbclient/usecase.html)

### 何ができるのか
IFが沢山あるので、全部は紹介できないんですが、とりあえず記事の冒頭で述べたユースケースを実装してみます。
まずはbbclientのインスタンスの初期化の方法を↓に示します。

```
import bbclient

# bbclientのインスタンスを初期化
project_path: str = "/PATH/TO/POKY"
init_command: str = ". oe-init-build-env"
client: BBClient = BBClient(project_path, init_command)

# サーバを立ち上げて接続
server_adder: str = "localhost" # 今はまだlocalhostしかサポートできてないです
server_port: int = 8081
client.start_server(server_adder, server_port)

# サーバからのイベントを全部受けとるように設定
ui_handler: int = client.get_uihandler_num()
client.set_event_mask(ui_handler, logging.DEBUG, {}, ["*"])

# レシピをパースしてキャッシュを生成する
ret = client.parse_files()
client.wait_done_async()
```

このclientを使ってそれぞれ以下のようにユースケースを実装できます

> 特定のbbclass(core-image.bbclassとか)を継承しているレシピファイルをリストアップしてほしい

```
ret: List[GetRecipeInheritsResult] = client.get_recipe_inherits()
recipes_inherit = [i for i in ret if "/PATH/TO/core-image.bbclass" in i.inherit_file_paths]
for i in recipes_inherit:
    print(i.recipe_file_path)
```

> すべてのパッケージのある変数(例えばPE,PV,PR)をリストアップしてほしい

```
ret: List[GetRecipesResult] = client.get_recipes()
for i in ret:
    data_store_index: int = client.parse_recipe_file(i.recipe_files)
    pe: Any = client.data_store_connector_cmd(data_store_index, "getVar", "PE")
    pv: Any = client.data_store_connector_cmd(data_store_index, "getVar", "PE")
    pr: Any = client.data_store_connector_cmd(data_store_index, "getVar", "PE")
    print(pe, pv, pr)
```

> 特定のパッケージ(例えばgcc)にあたってるbbappendファイルをリストアップしてほしい

```
ret: List[str] = client.find_best_provider("gcc")
target_recipe_file_path: str = ret[3]
ret: List[str] = client.get_file_appends(target_recipe_file_path)
print(ret)
```

他にもこのclientからイメージをビルドしたり、依存関係のtaskdependsファイルを生成したりできます。

# 最後に
よかったら使ってね。