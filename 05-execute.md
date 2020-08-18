# 5. 解析の実行

サンプルの解析として、テキストファイルから、
特定の単語の数を数えるという簡単な解析を行います。

## 5.1. 設定ファイルの準備

準備する設定ファイルで重要なのは以下の２点です。

- 入力データがおいてあるS3上の場所
- 解析結果を置く、S3上の場所

設定ファイルの雛形を用意しましたので、それをもとに設定ファイルの作り方を説明していきます。

### 5.1.1. 設定ファイルの雛形

雛形はこのようになります。

カンマ区切りです。
１行目の構文は少し特殊です。これについては後述します。
２行目以降に、独立して実行できる解析を書いていきます。
今回の説明では、１つだけ解析する場合の例です。

カラムの数は、３です。

```csv
--output-recursive OUTDIR,--input-recursive INPUTDATADIR,--input-recursive TOOLSDIR
s3://test-bucket/result/testrun1,s3://test-bucket/GAJ-sample-directory,s3://test-bucket/tools
```

それぞれのカラムの意味は次のようになります。

- `--output-recursive OUTDIR`
  - 解析結果を格納するS3上の場所を指定します。
    - 解析の結果、解析環境の`OUTDIR`の中に出力されるものが、S3上に保存されます。ディレクトリもそのまま保存されます。
  - このまま実行すると解析結果が `s3://test-bucket/result/testrun1` に保存されます。
  - この部分を解析ごとに変更してください。変更方法は後述
    - 例、 `s3://test-bucket/result/20200707-1`
- `--input-recursive INPUTDATADIR`
  - 入力ファイルを置いている場所を指定します。
  - 現在の設定ですと入力データが以下の場所にあることになります。
    - `s3://test-bucket/GAJ-sample-directory`
  - 解析を実行したいデータがはいっているディレクトリを指定してください。変更方法は後述
- `--input-recursive TOOLSDIR`
  - 解析スクリプトを指定します

### 5.1.2. 解析結果のS3上での出力先を指定

解析を実行したあとの結果を出力するS3上の場所を指定します。

このとき、バケットが存在していれば、フォルダが実際に存在していなくても、問題ありません。
バケットとは、`s3://test-bucket/folderA`というのものであれば、`test-bucket`の部分がバケットにあたります。

解析結果を以下の場所に書き込みたいとします。

```text
s3://test-bucket/result/20200707-1
```

この場合、
CSVファイルは、以下のように変更します。
変更場所は、CSVファイルの２行目の最初のカラム、`s3://test-bucket/result/20200707-1`の部分です。

```csv
--output-recursive OUTDIR,--input-recursive INPUTDATADIR,--input-recursive TOOLSDIR
s3://test-bucket/result/20200707-1,s3://test-bucket/samples/inputGAJ-sample-directory,s3://test-bucket/tools
```

### 5.1.3. 入力データがあるディレクトリを指定する

4章でアップロードした入力データのS3上の場所を、設定ファイルに書き込みます
解析を行いたい入力データがの場所にあるとします。

```text
s3://test-bucket/samples/input/20200707
```

この場合、
CSVファイルは、以下のように変更します。
変更場所は、CSVファイルの2行目の2番目のカラムです。

```csv
--output-recursive OUTDIR,--input-recursive INPUTDATADIR,--input-recursive TOOLSDIR
s3://test-bucket/result/20200707-1,s3://test-bucket/samples/input/20200707,s3://test-bucket/samples
```

### 5.1.4. スクリプトファイルを指定する

実行したい解析スクリプトのディレクトリが以下の場所にあるとします。

```text
s3://test-bucket/samples/
```

この場合、
CSVファイルは、以下のように変更します。
変更場所は、CSVファイルの２行目の3番目のカラム、`s3://test-bucket/samples`の部分です。

```csv
--output-recursive OUTDIR,--input-recursive INPUTDATADIR,--input-recursive TOOLSDIR
s3://test-bucket/result/20200707-1,s3://test-bucket/samples/input/20200707,s3://test-bucket/samples
```

## 5.2. 解析開始するマシンへログイン

解析を行うマシンへログインをします。ログインする際には、2章で作った鍵を使います。

```text
ssh -i newfile_for_GAJ アカウント名@xxx.xxx.xxx.xxx
```

## 5.3. 解析を行うときに指定するパラメータの確認

### 5.3.1. リージョンの確認

実際に解析の実行を行うAWSリージョンを指定します。
`test-bucket`は、東京リージョンにあるので、 `ap-northeast-1` を指定します。
異なるリージョンを指定すると、AWS内であっても、料金が発生してしまいます。

### 5.3.2. マシンサイズの確認

解析の実行に適切なサイズを指定します。
今回は、`c5.2xlarge` を使います。
この仮想マシンのスペックは以下のようになります。（2020/07/03現在）

```text
c5.2xlarge

vCPU: 8
RAM: 16GB
```

### 5.3.3. スクリプトの準備

以下のようなスクリプトの雛形があります。
変更箇所は、以下の部分です。
具体的には、2箇所あります。

入力データは以下にあるとしています。

```text
s3://test-bucket/samples/input/20200707
```

この場所にあるデータは、解析マシン上では
`${INPUTDATADIR}`という環境変数で指定する場所に、ディレクトリ構造ごとダウンロードされます。
ですので

```text
s3://test-bucket/samples/input/20200707/sampledata.txt
```

というファイルは、解析マシン上では
`${INPUTDATADIR}/sampledata.txt`
という形で指定することができます。

ですので、今回は変更の必要はありませんが、実際の解析では、上記のルールに従い、以下の部分をスクリプト例での該当箇所を、変更してください。

- `--input` で指定しているデータ

```bash
# !/bin/bash
# word-grep-wc.sh

set -e -u

echo "=== Parameter Check ==="
set
echo "======================="

cwltool --outdir ${OUTDIR} ${TOOLSDIR}/Workflows/word-grep-wc.cwl \
    --file_to_search ${INPUTDATADIR}/sampledata.txt \
    --pattern hello
```

### 5.3.4. 設定ファイルの準備

以下のような設定ファイルを準備します。

```word-grep-wc.csv
--output-recursive OUTDIR,--input-recursive INPUTDATADIR,--input-recursive TOOLSDIR
s3://test-bucket/result/20200707-1,s3://test-bucket/samples/input/20200707,s3://test-bucket/samples
```

### 5.3.5. 解析実行前の準備

解析を実行したあと、以下のようなことが起こると解析がとまってしまいます。

- そのままパソコンを閉じる
- そのままターミナルソフトを閉じる
- ネットワークが切れる

上記のようなことがおこっても、解析を止めないようにするには

- `nohup 解析コマンド &` この形で、解析コマンドを実行する
- 解析コマンドを実行するマシンで、screen, tmux, byobu などを使い、そのなかで、解析コマンドを実行する

おすすめは、後者の `screen` , `tmux` , `byobu` などを使うですが、
お使いの環境に合わせてお選びください。

注意点は、どちらも解析コマンドを実行するマシン上での作業ということです。
解析をコマンドを実行する前のパソコンで、上記の操作をおこなっても、
ネットワークがきれたときなどで、解析が止まってしまいます。

### 5.3.6. 解析の実行

解析を実行するのに必要なものがそろいましたので、実際に実行します。
解析コマンドは、AWSのマシン上から実行します。
実行する際に指定するのは主に以下の３つです

- --script
  - 作成したスクリプト
    - 例では、./word-grep-wc.sh
- --tasks
  - 設定ファイル
    - 例では、./word-grep-wc.csv
- --aws-ec2-instance-type
  - インスタンスタイプ
    - 例では、`c5.2xlarge`を指定

```console
hotsub run   \
 --script ./word-grep-wc.sh \
 --tasks ./word-grep-wc.csv \
 --image hotsub/c4cwl \
 --aws-ec2-instance-type c5.2xlarge \
 --aws-region ap-northeast-1 \
 --log-dir /tmp \
 --verbose
```

### 5.3.7. 解析が正しく行えたかの確認

解析が正しく終了したかの確認はコマンドのステータスで確認します。
具体的には以下のようになります。

```console
echo $?
```

- 0ならばただしく解析できた
- それ以外なら、なにか問題があり、解析が正しく行えていない可能性が高い。

## 5.4. 異なるパラメータ設定で解析を同時に行う

異なるパラメータ設定で、同時に解析を行うことも可能です。
リファレンスデータなど、各解析で同じデータを扱う場合そのデータを、共有ディレクトリとしてマウントすることが可能です。

### 5.4.1. 解析するためのパラメータを記述するファイルを準備

解析プログラムがいくつかの解析に関するパラメータを受けるとき、これらのパラメータについて、パラメータの数だけ計算用の仮想マシンをたてて、同時に解析を行う事が可能です。

今回は、5個のパラメータ(`hello`, `world`, `one`, `two`, `three`)について、それぞれ別々のS3のフォルダに結果を格納するようにする例を紹介します。

以下のようなファイルを作ります。今回は ./word-grep-wc.5words.csv という名前のファイルにします。

```csv
--env WORD, --output-recursive OUTDIR,--input-recursive INPUTDATADIR,--input-recursive TOOLSDIR
hello,s3://test-bucket/result/20200707-1,s3://test-bucket/samples/input/20200707,s3://test-bucket/samples
world,s3://test-bucket/result/20200707-2,s3://test-bucket/samples/input/20200707,s3://test-bucket/samples
one,s3://test-bucket/result/20200707-3,s3://test-bucket/samples/input/20200707,s3://test-bucket/samples
two,s3://test-bucket/result/20200707-4,s3://test-bucket/samples/input/20200707,s3://test-bucket/samples
three,s3://test-bucket/result/20200707-5,s3://test-bucket/samples/input/20200707,s3://test-bucket/samples
```

このようにすることで、各計算マシンの解析スクリプトは、解析用のパラメータを`WORD`という環境変数で受け取ります。また出力するべきS3の場所もそれぞれのマシンで、`OUTDIR`という環境変数で受け取ります。

### 5.4.2. 解析を行うスクリプト

解析スクリプト `word-grep-wc.sh`は以下のようになります。
`OUTDIR`, `TOOLSDIR`,`INPUTDATADIR`, `WORD`が、パラメータファイルで指定した部分です。

前のスクリプトと似ていますが、`--pattern` に渡す値が固定値からパラメータに変更になっています。

```bash
# !/bin/bash
# word-grep-wc.sh

set -e -u

echo "=== Parameter Check ==="
set
echo "======================="

cwltool --outdir ${OUTDIR} ${TOOLSDIR}/Workflows/word-grep-wc.cwl \
    --file_to_search ${INPUTDATADIR}/sampledata.txt \
    --pattern ${WORD}
```

### 5.4.3. 解析を実行する

以下のようにして実行します。
`--script`, `--tasks` がそれぞれ、スクリプトファイルと、パラメータファイルになります。

```console
hotsub run   \
 --script ./word-grep-wc.sh \
 --tasks ./word-grep-wc.5words.csv \
 --image hotsub/c4cwl \
 --aws-ec2-instance-type c5.2xlarge \
 --aws-region ap-northeast-1 \
 --log-dir /tmp \
 --verbose
```

実行をおこなったあと、以前と同じ方法で、正しく解析が行えたかをご確認ください。

#### 5.4.3.1 解析の実行が失敗するときの例

grepを例にしましたが、grepしたときに、対象ファイル内にない文字列を渡した場合、
エラーになることがあります。
