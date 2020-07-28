# 4. 入力データをアップロード

実際に解析を行いたいファイルをAWSのS3にアップロードする際のコマンドライン操作について述べます。

公式のドキュメントは以下になります。

- [AWS CLI での高レベル \(s3\) コマンドの使用 \- AWS Command Line Interface](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-services-s3-commands.html)

AWSコマンドラインツールがすでにインストールしてあれば、それをお使いいただくことが可能です。

## 4.1. コマンド(aws-GAJ)の説明および、エイリアスの設定

この節では、dockerと、AWSが公式に提供している、docker イメージを使って説明します。AWSコマンドラインツールが使える状況でありましたら、この節は飛ばしていただいて問題ありません。

awsコマンドがない、インストールができないといった場合、dockerを使うことで解消することができます。使用するdockerコンテナは、AWSが提供しているものです。

```console
docker run --rm -ti -v ~/.aws.GAJ:/root/.aws -v $(pwd):/aws -w /aws amazon/aws-cli:2.0.22
```

毎回全部打ち込むのは、操作ミスにもつながるので、以下のようにエイリアス `aws-GAJ` を設定をすることで、簡単に実行できるようになります。
よく使う場合は、`.bashrc` などお使いのシェルの設定ファイルに書いておくことで、お使いのパソコンでいつでも利用できるようになります。

```console
alias aws-GAJ='docker run --rm -ti -v ~/.aws.GAJ:/root/.aws -v $(pwd):/aws -w /aws amazon/aws-cli:2.0.22'
```

またこれ以降では、`aws-GAJ`というコマンドを使うことで説明しますが、
すでに、`aws`コマンドがある場合はcredentialsを正しく設定した上で、`aws-GAJ`の部分を`aws`に置き換えていただければ同じ動作をいたします。

## 4.2. 現在のフォルダ構造を確認

以下のコマンドで、フォルダ構造を確認できます。
この例の場合、すでに、manabuというフォルダが存在している例です。

```console
aws-GAJ s3 ls s3://test-bucket
```

出力例

```text
                           PRE manabu/
```

なにもない場合は、何も表示されません。

```console
aws-GAJ s3 ls s3://no-such-a-test-bucket
```

出力例

`なし`

## 4.3. フォルダごとアップロードする

まず、`ご自身のアカウント名-sample-data`
というディレクトリをご自身のパソコンに作成してください。

このディレクトリの中に、いくつかファイルを置きます。

このディレクトリを s3 にアップロードするには、以下のコマンドを実行します。

```console
aws-GAJ s3 cp --recursive manabu-sample-data s3://test-bucket/manabu-sample-directory-cp
```

出力例

```text
upload: manabu-sample-data/xyz.txt to s3://test-bucket/manabu-sample-directory-cp/xyz.txt
upload: manabu-sample-data/abc.txt to s3://test-bucket/manabu-sample-directory-cp/abc.txt
upload: manabu-sample-data/manabu.txt to s3://test-bucket/manabu-sample-directory-cp/manabu.txt
```

### 4.3.1. アップロードしたファイルの確認

アップロードしたファイルを確認するには以下のコマンドを使います

```console
aws-GAJ s3 ls s3://test-bucket/manabu-sample-directory-cp/
```

出力例

```text
2020-06-25 05:08:20         17 abc.txt
2020-06-25 05:08:20         59 manabu.txt
2020-06-25 05:08:20         24 xyz.txt
```

内容を確認したいフォルダの最後に`/`(スラッシュ)がないと
以下のように、フォルダが存在しているということだけがわかる表示になります。

```console
aws-GAJ s3 ls s3://test-bucket/manabu-sample-directory-cp
```

出力例

```text
                           PRE manabu-sample-directory-cp/
```

## 4.4. ファイルを消す

さきほどアップロードしたサンプルを１つけしてみます。
今回は、abc.txtを消してみます。

```console
aws-GAJ s3 rm s3://test-bucket/manabu-sample-directory-cp/abc.txt
```

出力例

```text
delete: s3://test-bucket/manabu-sample-directory-cp/abc.txt
```

### 4.4.1. ファイルが消えたことを確認する

abc.txtがなくなったことが確認できます。

aws-GAJ s3 ls s3://test-bucket/manabu-sample-directory-cp/
2020-06-25 05:08:20         59 manabu.txt
2020-06-25 05:08:20         24 xyz.txt

## 4.5. フォルダを消す

以下のコマンドでディレクトリを消すことができます

```console
aws-GAJ s3 rm --recursive s3://test-bucket/manabu-sample-directory-cp/
```

出力例

```text
delete: s3://test-bucket/manabu-sample-directory-cp/xyz.txt
delete: s3://test-bucket/manabu-sample-directory-cp/manabu.txt
```

### 4.5.1. フォルダが消えたことを確認する

以下のコマンドで、ファイルが消えたことを確認できます。

```console
aws-GAJ s3 ls s3://test-bucket/
```

出力例

```text
                           PRE manabu/
```

## 4.6. S3上にフォルダだけを作るコマンドは現在ありません

aws コマンドラインツールに、S3上にフォルダだけを作るコマンドはありません。

## 4.7. ファイルを単独でアップロード

１つのファイルをアップロードしてみます。
`ご自身のアカウント名.txt` というファイルの中身に、なにかテキストを書いてください。
特別何もなければ、現在の日付と、時間をかいてみるとよいとおもいます。

次にこのファイルをアップロードします。
アップロード先のディレクトリは、存在していなくてもかまいません。
指定したものが作られます。

```console
aws-GAJ s3 cp manabu.txt s3://test-bucket/sample/directory/manabu.txt
```

出力例

```text
upload: ./manabu.txt to s3://test-bucket/sample/directory/manabu.txt
```

注意点
アップロード先で、ファイル名を省略すると、最後のフォルダ名のつもりのファイルができあがってしまいます。

```console
$ aws-GAJ s3 cp manabu.txt s3://test-bucket/sample/directory
upload: ./manabu.txt to s3://test-bucket/sample/directory
$ aws-GAJ s3 ls s3://test-bucket/sample/directory
2020-06-25 05:22:46         66 directory
```

### 4.7.1. アップロードされたことを確認する

ファイルが単独でアップロードされたことを確認します。

```console
aws-GAJ s3 ls s3://test-bucket/sample/directory/
```

出力例

```text
2020-06-25 05:24:56         66 manabu.txt
```

## 4.8. ローカルのディレクトリをS3と同期する

ローカルディレクトリをS3と同期するには
`s3 sync` コマンドを使います。
以下の例では、ローカルの `manabu-sample-data` というディレクトリを、s3の`synctest`というフォルダとしてコピーします。

```console
aws-GAJ s3 sync manabu-sample-data s3://test-bucket/synctest
```

出力例

```text
upload: manabu-sample-data/abc.txt to s3://test-bucket/synctest/abc.txt
upload: manabu-sample-data/manabu.txt to s3://test-bucket/synctest/manabu.txt
upload: manabu-sample-data/xyz.txt to s3://test-bucket/synctest/xyz.txt
```

### 4.8.1. 同期されていることの確認

実際に同期されたことを確認します。

```console
aws-GAJ s3 ls s3://test-bucket/synctest/
```

出力例

```text
2020-06-25 05:33:19         17 abc.txt
2020-06-25 05:33:19         59 manabu.txt
2020-06-25 05:33:19         24 xyz.txt
```

## 4.9. ローカルのディレクトリにファイルを追加して、再度同期をする

今度は、新しいファイルを追加して、再度同期を行います。
そのためにまず、`manabu-sample-data/def.txt`
を作成します。

再度同期します

```console
aws-GAJ s3 sync manabu-sample-data s3://test-bucket/synctest
```

出力例、ファイルdef.txtだけがアップロードされていることがわかる

```text
upload: manabu-sample-data/def.txt to s3://test-bucket/synctest/def.txt
```

### 4.9.1. 追加されたことを確認する

追加されたことを確認します。

```console
aws-GAJ s3 ls s3://test-bucket/synctest/
```

出力例

```text
2020-06-25 05:33:19         17 abc.txt
2020-06-25 05:40:25         16 def.txt
2020-06-25 05:33:19         59 manabu.txt
2020-06-25 05:33:19         24 xyz.txt
```

## 4.10. ローカルのディレクトリにファイルを削除して、再度同期をする

ローカルの`abc.txt`を削除して、再度同期します。
まずは、ローカルのファイルを消します。

例、実際に行った操作

```console
$ rm manabu-sample-data/abc.txt
$ ls manabu-sample-data
def.txt    manabu.txt xyz.txt
```

ローカルで削除したものを、S3上でも削除するには
`--delete` オプションをつけます。

```console
aws-GAJ s3 sync --delete manabu-sample-data s3://test-bucket/synctest
```

出力例

```text
delete: s3://test-bucket/synctest/abc.txt
```

### 4.10.1. 削除されたことの確認

削除されたことを確認します

```console
aws-GAJ s3 ls s3://test-bucket/synctest/
```

出力例

```text
2020-06-25 05:40:25         16 def.txt
2020-06-25 05:33:19         59 manabu.txt
2020-06-25 05:33:19         24 xyz.txt
```
