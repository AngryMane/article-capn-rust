<!--
title:   Sphinxでrecommonmarkを使用する際の問題と対策
tags:    Markdown,Python,Python3,Sphinx,documentation
id:      6c1a5c8847f62bad7897
private: false
-->
# 概要
recommonmarkを使うと、Sphinxは全てのドキュメントを毎回ビルドし直します
そもそも当該エクステンションはdeprecated[^1]なエクステンションなので、myst-parserを使う方法を調べました

# 目次
1. 全てのドキュメントを毎回ビルドし直す問題
1. myst-parserの使用方法
    1. myst-parserをインストールする
    1. conf.pyを編集する
    1. makeする

# 全てのドキュメントを毎回ビルドし直す問題
recommonmarkを使用すると、全てのドキュメントが毎回再生成される現象が発生します。[^2]
recommonmarkの公式ドキュメントがconf.pyに以下の★の記述を追加することを推奨していることが原因です

``` conf.py
# for Sphinx-1.4 or newer
extensions = ['recommonmark']

# for Sphinx-1.3
from recommonmark.parser import CommonMarkParser

source_parsers = {
    '.md': CommonMarkParser,   ★ ①
}

source_suffix = ['.rst', '.md']
```

``` conf.py
# At top on conf.py (with other import statements)
import recommonmark
from recommonmark.transform import AutoStructify

# At the bottom of conf.py
def setup(app):
    app.add_config_value('recommonmark_config', {
            'url_resolver': lambda url: github_doc_root + url,　　★ ②
            'auto_toc_tree_section': 'Contents',
            }, True)
    app.add_transform(AutoStructify)
```

①/②の設定はSphinxのキャッシュ(environment.pickle)に保存することができません。[^3]
このため、Sphinxはビルドの度に「*conf.pyが変化した*」と判断し、全てのドキュメントをビルドしなおします。

この問題を回避するために、myst-parserを使用することにしました

# myst-parserの使用方法
myst-parserに乗り換えることで問題を回避できました。使用方法は以下の通り。

### myst-parserをインストールする

``` bash
$ python3 -m pip install myst-parser
```

### conf.pyを編集する

``` conf.py
extensions = ['myst_parser']
```

### makeする
` bash
$ make html
`


[^1]: https://github.com/readthedocs/recommonmark
[^2]: https://github.com/readthedocs/recommonmark/issues/197
[^3]: https://github.com/sphinx-doc/sphinx/blob/a55816deb546828db1383f2875be815bf805f858/sphinx/config.py#L46