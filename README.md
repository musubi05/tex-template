# tex-template
texで文章を書くとき用のテンプレートディレクトリ

## Environment

- TeX Live
- Ruby        (for Rake)
- ImageMagick (for convert raster image files to PDF)
- Inkscape    (for convert vector image files to PDF)
- Pandoc      (for convert markdown to tex)

## 使用方法
``` bash
$ rake
```
でビルドとプレビューが行われる.

``` bash
$ rake clear
```
で中間ファイルを削除する.

``` bash
$ rake clobber
```
で全ての生成されたファイルが削除される.

## 構成
```
% tree --charset ascii
.
|-- Rakefile  % ビルド用Rakefile
|-- dist      % 生成されたpdfファイルが置かれる場所
|-- etc       % ビルド関係の設定ファイル
|-- root.tex  % コンパイルのルートファイル
|-- src       % texファイル置き場 (markdownファイルがあればpandocでtexに変換される)
|-- bib       % bibファイル置き場
`-- sty       % styファイル置き場
```

## License
WTFPL
