# pugとsassをgulpで構築する為のリポジトリです

## 概要や注意点など
- 今回使ったプラグイン以外にも様々なものがあるので、案件などによって随時調べたり、変更すると良い↓参考
https://qiita.com/oreo3@github/items/0f037e7409be02336cb9
https://qiita.com/hbsnow/items/8eb7009b3ed716bc48a4#gulp-cached-gulp-changed
- 新しくgulpのプラグインを追加するときは、以下のブラックリストに入っていないか確認し、入っていた場合は、注意する。理由を調べるなど。また、同じようなプラグインがいくつかある場合があるので、追加する際は調べる。
https://github.com/gulpjs/plugins/blob/master/src/blackList.json
- gulpfile.jsの.pipe(cache('html'))の部分のコメントアウトを外す（処理を通す）と、変更を加えたファイルだけがコンパイルされるので、コンパイルがとても早くなる。
ただし、include元（_header.pugなど）のファイルを変更しても、ブラウザに反映されない(includeやmixinなどのファイルに変更が加えたタイミングでは、どのファイルに変更を渡すか伝わらないため)。include元のpugファイルの変更をブラウザに反映したくば、include先（index.pugなど）のpugファイルに何か変更を加えたタイミングでinclude元の変更分が反映されるので、include先に何か変更を加えてみる。
この解決策はいくつかあるが、上手く行かず…。
- gulp-sass-globという、sassのimportでワイルトカードを利用可能にするプラグインがあるが、上手くいかないので、そのうち解消したい。。。参考サイト → https://www.nxworld.net/services-resource/gulp-plugin-gulp-sass-glob.html

-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --

## 基本的なコマンド・スクリプト
- gulp
  ルートディレクトリでこのコマンドを打つと、ローカルサーバーが立ち上がり、ブラウザのタブが新たに開かれ、dist/index.htmlが表示される。表示されない場合はブラウザリロードする。ファイルの監視も行われる。中止したい場合は`control + c`コマンド
- gulp build
  ビルドコマンド。
- gulp html|styles|javascript|imagemin...
  それぞれに定義されたタスクを叩く。

-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --

## 基本的なディレクトリ・ファイル構成
- dist
出力ファイル。基本的にこのディレクトリ以下をサーバーにUPする。基本的にこのディレクトリ内を操作しない。

- src
実際にいじるファイル群。

- gulpfile.js
実際にgulpにさせるタスクの設定を書くファイル。

- package.json, package-lock.json
gulpのプラグインのバージョン管理。

- cacheClear.sh
うまくコンパイル出来ない時に使うシェル。
gulpにキャッシュが溜まった時、pugやsassがうまくコンパイルされない時があるので、このシェルを実行し、リセットする。それでも上手くいかない場合は調べる。

- releaseInit.sh
本番環境適用時に実行するシェル。

-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --

## gulpにやらせるタスク一覧
### 共通
- 各言語に記述エラーがあった場合、コンソールに知らせてくれる & ポップアップ通知をしてくれる
- 各ファイルを変更時にブラウザを自動リロード
- 各ファイルをgulpにキャッシュさせて、差分や変更ファイルのみをコンパイル・圧縮。ただし、pugはinclude先のファイルを感知できないので、初期時はoff。
- ローカルサーバーの立ち上げ
- 各ファイルの圧縮

### HTML関連
- pugのHTMLへのコンパイル
- CSSとJSのインライン化

### Sass関連
- sassのコンパイル
- ベンダープレフィックスの自動付与とブラウザ毎に違う記述の仕方を自動変換・追記。細かな設定はgulpfile.jsで管理
- sassのソースマップの書き出し
- sassをgulpにキャッシュさせて、更新したファイルのみコンパイル

### Javascript関連
- 複数のJSファイルを一つに統合

### 画像関連
- jpg,png,gif,svgの圧縮（圧縮率の指定などはgulpfile.jsで制御）
- jpeg,jpgはプログレッシブjpgに変換。

-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --

## リリース時に行う（本番環境に適用時にする）設定やビルド設定について。
### 上記ではそれぞれタスクを書いているが、初期設定では以下のようになっている
- 本番環境ビルド以外では基本的にHTML・CSS・JSは圧縮しない。
- 本番環境ビルド以外ではCSS・JSのインライン化はしない。インライン化させると、CSSなどのファイル変更時にブラウザリロードさせる機能と併用出来ない為。
- 検証環境がある場合、ogタグの設定は検証環境のURLを参照させる。
- 本番環境ビルド以外ではhtmlのhead内に、metaタグnoindexを設置（Googleにインデックスさせないことにより、検索しても出てこないようにする設定）

### 本番環境適用時・サイトリリース時にはreleaseInit.shを実行する。
`sh releaseInit.sh`
内容としては、
1. 念の為キャッシュを削除する（distとnode_modulesを削除、node_modulesを再インストール。）
2. pugとsassのリリースフラグをtrueに書き換え。
3. cssとjsをインライン化させるかどうかを選択。インライン化させる場合はpugとsassのインライン化フラグをtrue。
4. html,css,js,画像などのビルドを実行
5. pugとsassのリリースフラグをfalseに戻す。インライン化フラグがtrueの場合はfalseに戻す。
6. cssとjsをインライン化する場合は不要なdistの中のディレクトリを削除。


-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --

## Pugについて
### ディレクトリ構成
- data
  このファイルはサイトの設定や共通項目を格納しておくファイルです。サイトのタイトルや、ドメインなどを管理します。
- includes以下
  各共通パーツなどを定義し、呼び出します。
- layouts以下
  includes以下などをどのように呼び出すかなどを定義します。
  汎用的な設定に初期設定でしていますが、サイトやプロジェクトに合わせて増やします。
- pages以下
  実際に出力されるファイルを定義します。ページを増やすごとにpugファイルを増やします。このディレクトリ以下はディレクトリ構造もそのまま保持してdist以下に出力されます。

## faviconとアイコンについて
/includes/meta.pug に記載のfaviconとアイコンの設定は以下のURLを参考。基本的にPCとSPのファビコンやホーム画面、お気に入りアイコンなど様々なアイコンをサポート。
180px×180pxと64px×64pxのfavicon.icoとapple-touch-icon.pngを（名前は揃える）img直下に保存。画像の作り方は以下のURLのようなジェネレーターで作成するか、デザイナーに作成依頼。
- https://mamewaza.com/support/blog/favicon2017.html
- https://www.icoconverter.com/

### 参考
今回参考にしたURLなど
#### ディレクトリ構成などを参考にした
https://qiita.com/garakuta/items/c83548c74e45838e3fe0
#### 公式ドキュメント
https://pugjs.org/api/getting-started.html

## Sassについて
### ディレクトリ構成
- all
  全てのscssを読み込む為のファイル。
- config
  scssの設定や、共通変数などを定義する。
- variables
  変数を定義する為のファイル
- reset
  いわゆるリセットCSS。
- base
  サイト全体で基盤となるscssを定義する。
  ファイル内の記述が多くなってしまう場合はファイル分割などをする。
- library/functions以下
  サイト全体で使える汎用的なsass関数を定義する。
- library/mixins以下
  サイト全体で使える汎用的なmixinを定義する。もしファイルを増やす場合は、base.scssに読み込み設定を追記する。
- library/projectMixins以下
  そのプロジェクト（サイト内）で独自に使うmixinを定義
- components
  パーツ毎に共通で使えるスタイルを記述する。ボタンやメニューやモーダルなど。
- modules以下
  BEM記法でいうBlockを定義する。ヘッダーやフッターなど
- pages
  実際に出力されるファイル群。上記までの組み合わせで実現出来ないページ毎のスタイルを当てていく。

## JSについて
src/jsディレクトリ以下のファイルは全てdist/js以下にscript.jsとして出力される。
初期時はダミーファイルが３つあり、ビルドすることで全てdist/js以下に一つのファイルにまとまって出力がされていることが確認できるようになっている。

## 画像について
jpg,jpeg,png,gif,svgファイルは圧縮される。
icoファイルに関してはdist/img以下にコピーはされるが、圧縮はされない。
上記ファイル以外は監視もコピーもしないので、追加でサポートしたい拡張子があればgulpfile.jsで定義する。
初期時はsrc/img以下にダミーファイルが用意してあり、ビルドすることでdist/img以下に圧縮された画像ファイルを確認できる。

## その他ファイルについて
フォントなど、プロジェクト独自で使うファイル群はsrc/other以下に入れるようにする。
gulpfileではdistディレクトリにコピーされるタスクのみ記述してある。

-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --

## ページを増やす(HTMLを増やす)際の手順
1. `src/pug/pages`以下に増やしたい分のpugファイルを作成。pugの記述の仕方はサンプルファイルを参照。
2. 同じく`src/pug/data.pug`に増やした分のファイルの設定を`var settings`内に例を見ながら記述。
3. `src/scss/pages`以下に1で作ったファイルと同じファイル名でscssファイルを作成（もしディレクトリを`src/pug/pages`に作成していた場合は、同じように作成）
