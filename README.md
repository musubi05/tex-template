# tex-template
texで文章を書くとき用のテンプレートディレクトリ

## 使用方法
``` bash
$ rake
```
でビルドとプレビューが行われる.

## 構成
```
% tree
.
├── Rakefile
├── dist      % 生成されたpdfファイルが置かれる場所
├── etc       % ビルド関係の設定ファイル
├── fig       % 画像ファイルを置いておく場所 (rake時に自動でpdf化 + xbb生成)
├── root.tex  % コンパイルのルートファイル
├── src       % texファイル置き場. (markdownファイルがあればpandocでtexに変換される)
└── sty       % styファイル置き場.
```
