# 3. IAM ユーザのデータをセットアップ

こちらから送らせていただいた以下の4つのファイルがあることをご確認ください
（これらのファイルは例です。`fromGAJ.csv`の部分が異なる場合があります。）

- fromGAJ.csv.enc
- key.bin.enc
- fromGAJ.csv.sha256
- key.bin.sha256

以下の２つのファイルは、暗号化されているファイルです。
このあと復元していきます。

- fromGAJ.csv.enc
- key.bin.enc

また、次の2つのファイルは、それぞれのファイルの暗号化前に取得した、sha256の値が書かれており、前述の暗号化されている2つのファイルが、正しく復元できたかを確認するために使います。（後述）

- fromGAJ.csv.sha256
- key.bin.sha256

## 3.1. key.bin.enc の復元

こちらから送らせていただいた鍵ファイル `key.bin.enc` を、ご自身の鍵を使って、復元します
以下のコマンドを実行します。

```console
openssl rsautl -decrypt -inkey newfile_for_GAJ -in key.bin.enc -out key.bin
```

このコマンドを実行しますと、パスワード入力を促す文言が表示されますので、 `newfile_for_GAJ` を作るときに入力したパスワードを入力してください。

正常に完了すると、以下のファイルができあがります

- key.bin

### 3.1.1. Macにて自分の秘密鍵のパスワードをいれてエラーがでるとき

エラーがないときはこの項を読み飛ばしていただいて問題ありません。

この項は、お使いのパソコンが Macの場合に表示されるかもしれないエラーに関する説明です。

```text
User interface error
unable to load Private Key
```

や。正しいパスワードを入力しているのに、

```text
bad password read
```

が、でてきiterm2をお使いの場合は、
iterm2を一度完全に終了させてもう１度起動するか、Macの標準のターミナルで入力すると、復元できるときがあります。

## 3.2. key.binが正しく復元できたかの確認

このファイルの、sha256の値が、こちらから送信させていただいた値と同じかどうかご確認ください。

例

```console
$ shasum -c key.bin.sha256
key.bin: OK
```

## 3.3. fromGAJ.csv.encの復元

 さきほど、できあがった、 `key.bin` を使って、`fromGAJ.csv.enc`の復元を行います。

具体的には以下のコマンドを実行します。

```console
openssl enc -d -aes-256-cbc -in fromGAJ.csv.enc -out fromGAJ.csv -pass file:./key.bin
```

正常に完了すると以下のファイルができあがります。

- fromGAJ.csv

## 3.4. fromGAJ.csvが正しく復元できたかの確認

このファイルの、sha256の値が、こちらから送信させていただいた値と同じかどうかご確認ください。

例、正しく実行できた場合の表示、１行目を入力後、エンターキーを押すと、２行目が表示される。

```console
$ shasum -c fromGAJ.csv.sha256
fromGAJ.csv: OK
```

例、間違った内容になってしまった場合

```console
$ shasum -c fromGAJ.csv.sha256
fromGAJ.csv: FAILED
shasum: WARNING: 1 computed checksum did NOT match
```

## 3.5. AWSのcredentialファイルのセットアップ

AWSを利用するための設定を行います。

すでに、AWSをお使いの場合は、こちらから送信させていただいた、credentialの追加作業をお願いいたします。
それらに従っていただいてかまいません。

ここからは、新規に設定する方法で説明いたします。

参考情報、AWS公式のドキュメント

- [設定ファイルと認証情報ファイルの設定 \- AWS Command Line Interface](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-files.html)

### 3.5.1. お使いのパソコン上のディレクトリの作成

以下のコマンドでディレクトリを作ります

mkdir ~/.aws.GAJ

#### 3.5.1.1. config ファイルの設定

さきほど作成したディレクトリ ~/.aws.GAJ に config というファイルを作ります。
内容は次の２行です。

```text
[default]
region = ap-northeast-1
```

#### 3.5.1.2. credentials ファイルの設定

さきほど作成したディレクトリ ~/.aws.GAJ に credentials というファイルを作ります。
内容は次の3行です。

```text
[default]
aws_access_key_id = fromGAJ.csvの中にあるアクセスキー
aws_secret_access_key = fromGAJ.csvの中にあるシークレットアクセスキー
```

### 3.5.2. AWS上のマシンでの設定

#### 3.5.2.1. AWS上のマシンへsshへログイン

こちらからお伝えしたマシンにssh でログインします。
`newfile_for_GAJ` は、「２章、鍵のセットアップ」で作ったものを指定します。
`ユーザ名`と、ホストにあたる `XXX.XXX.XXX.XXX` は、こちらからお伝えしたものです。

```console
ssh -i newfile_for_GAJ ユーザ名@XXX.XXX.XXX.XXX
```

#### 3.5.2.2. AWS上のマシンでの設定ファイルの配置

今ログインしたAWS上の計算機にも、同じように設定ファイルconfigとcredentialsを配置します。
ただし、ディレクトリが、~/.aws になります。

- mkdir ~/.aws
  - ディレクトリを作成
- ~/.aws/config の作成
  - 3.5.1.1で作ったものと同じ内容を書き込む
- ~/.aws/credentials の作成
  - 3.5.1.2で作ったものと同じ内容を書き込む

### 3.6. 動作確認

TODO： 動作確認について、なにか書く。
