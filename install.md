
# install

* ここで言うインストールとは Docker Desktop のこと
* Windows 環境では *そもそも Hyper-V を利用するため Windows が Pro 以上でなくてはならない*
* コントロールパネルから Windows の機能の有効化で Hyper-V を有効にして再起動しておく
* それからインストーラーを入れる
* Mac 版は普通に入ったが、最初に起動して認証通さないとコマンドのパスが通らない様子

* [Install Docker Desktop on Windows \| Docker Documentation](https://docs.docker.com/docker-for-windows/install/)
* [Docker Hub](https://hub.docker.com/) ※初回はサインアップが必要


# 事前の準備や環境構築

## setup for kubenetes

* Mac: 設定から kubernetes タブを開いて有効にする
* Mac: 完了するとメニュー上でグリーンのアイコンが出るようになる

* `pod.yaml` を作成して中身をコピペする
* コマンドを実行する `kubectl apply -f pod.yaml`
* 稼働している状態は ` kubectl get pods` で確認できる

```
NAME   READY   STATUS              RESTARTS   AGE
demo   0/1     ContainerCreating   0          7s
```

* ログが取れることを確認する `kubectl logs demo`
* 最後に全てをなかったことに `kubectl delete -f pod.yaml`
* これを行うと `get pods` しても何も得られない状態になる

## setup for swarm

* 最初に swarm を初期化する（よくわかってない） `docker swarm init`
* 次に `docker service create --name demo alpine:3.5 ping 8.8.8.8`
* `docker service ps demo` で生成されたサービスの情報を見ることができる

```
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
 463j2s3y4b5o        demo.1              alpine:3.5          docker-desktop      Running             Running 8 seconds ago
```

* `docker service logs demo` これでログを見ることができる
* 作成したサービスを止める（消す？） `docker service rm demo`

ここまでの手順を終えた段階の公式サイトの一文を引用すると……

```
この時点で、開発マシンにDocker Desktopをインストールし、KuberentesおよびSwarmで単純なコンテナー化されたワークロードを実行できることを確認しました。
```

とのこと。


## Windows + Git-Bash の設定

Git のインストールパス以下にある `etc/profile.d/aliases.sh` を、管理者権限で上書きする。

```
	for name in node ipython php php5 psql python2.7 docker
```

上記のように、末尾に Docker を追加する。これを行わないと winpty を使って起動してね的なエラーが出てしまい、毎回 `winpty docker ...` というようにコマンドの最初に winpty を書かないといけなくなる。


## パスがおかしいよって話になった場合

ひとまず遭遇したのは `/bin/bash` が、なぜか Git の `usr/bin/bash` になってしまう現象で、これを回避するには `//bin/bash` のようにスラッシュを二重にする。


# いろいろやってみる

## イメージの pull

考え方としては、まず最初にイメージを pull して Docker Hub（Git でいう GitHub 的なパブリックなホスティングサービス）から任意のイメージを落としてくる。

```
docker image pull centos:6.9
```

このときの `:6.9` の部分が *タグ* と呼ばれる。省略した場合は自動的に `:latest` というタグが付与されているものとしてみなされる。

ちなみに、一度ここで pull しているからといって、後述のコンテナの実行時に正しいタグを指定していないと、latest を落としてこようとすることになるので注意。


## コンテナの実行

イメージが手元に落ちてきたら（これは実際には省略しても大丈夫っぽい）、コンテナを実行する。

```
docker container run -it --name 名前 centos:6.9 //bin/bash
```

`-i` で標準入力を取り、 `-t` でターミナルを紐付ける（つまりこっちに制御を回す）という意味になる。

名前をつけておかないと、ハッシュみたいなようわからん感じになるので `--name` で名前をつけておいたほうがいい。

ログイン後は `ip addr` などでコンテナ内からネットワークがどう見えているのかを確認でき、さらに `ping -c 3 172.17.1.1` などとすることで、ホスト OS に対してネットワークが疎通できるかどうかを確認できる。

このとき `ip addr` の出力結果の `inet` の項目を見て、その `172.xx.1.1` に対して ping を飛ばしてみればよい。


## コンテナの停止

上記のようにコンテナを対話形式で実行している場合は、単にコンテナ内から `exit` するとコンテナが停止する。

停止したかどうかを確認するには `docker container ls -a` の出力結果を見てみるとよい。

STATUS が `Exited` という状態になっていれば、完全に停止していると考えられる。


## コンテナのコミット（イメージを作る操作に相当）

コンテナを再利用できるように *イメージ化* を行う。

このとき `docker container ls -a` 等で表示される CONTAINER ID（ハッシュ値）が必要になる。

```
docker container commit コンテナID タグ名
```

上記でタグ名に指定したものが、そのイメージのタグになる。つまり `docker image ls -a` 等でローカルのイメージ一覧を表示したときに、ここで指定したタグが表示される。

ここまでで……

* コンテナのスナップショットである Docker イメージのダウンロード
* イメージを元にしたコンテナの起動
* コンテナの停止
* コンテナのイメージ化

までができたことになる。


## コンテナの削除

`docker container rm 名前（もしくは、コンテナ ID でも可）`





