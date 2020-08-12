# 1. 必要なもの

- AWS cli または、docker

解析を開始するための計算機をAWS上に確保してあります。
そのマシンに、SSHではいることができれば、解析が可能です。解析に必要なコマンドはインストール済みです。

解析用のデータをS3にあげることと、解析結果をS3からダウンロードするときに、手元のパソコンにも、AWSコマンドがあると便利です。
もし、AWSコマンドをいれることができない場合は、Dockerを使うことで解決できます。


# 1.1 Install AWS cli
AWS Command Line Interface (AWS CLI) のバージョン 2 をインストールする方法について、
[AWS CLI バージョン 2 のインストール](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2.html) にまとめられています。
オペレーティングシステムとして、Linux，macOS上，Windowsがサポートされています（2020-08-12現在）。
Dockerを用いる場合も紹介されています。
- [公式 AWS CLI バージョン 2 Docker イメージの使用](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-docker.html)
- [Linux での AWS CLI バージョン 2 のインストール](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-linux.html)
- [macOS での AWS CLI バージョン 2 のインストール](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-mac.html)
- [Windows への AWS CLI バージョン 2 のインストール](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-windows.html)

# 1.2 Install Docker
[Docker公式サイト](https://docs.docker.com/get-docker/)からインストールパッケージをダウンロードし、インストールしてください。
Linux，macOS，Windowsがサポートされています（2020-08-12現在）。

