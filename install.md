
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

## コンテナのバックグラウンド起動

`docker container run -itd --name 名前 イメージ //bin/bash`

`-d` オプションでバックグラウンド起動。

## バックグラウンド起動中のコンテナに入る

`docker container attach コンテナ名`

## バックグラウンド起動中のコンテナの停止

`docker container stop コンテナ名`

## 停止中のコンテナを起動させる

`docker container start コンテナ名`

run しても同じだけど、そのままバックグラウンド起動になるってことかな。

## イメージのほうを削除する場合

`docker image rm イメージ名`

ただしこのとき、このイメージを元にしたコンテナが存在している場合は、削除することはできない。

`-f` オプションの付与で強制的にイメージだけを削除することはできる。


# Apache を使う

考え方としては、まず CentOS のイメージからコンテナを起動し、そこに Apache をインストールする。

```
$ docker container run -it --name apache-test centos:latest //bin/bash
[root@]# yum install httpd iproute
```

ウェブサーバのルートに移動して、HTML を作成しておく。

```
[root@]# cd ./var/www/html/
[root@]# echo "Systemd in a container." > index.html
[root@]# exit
```

これでインストールそのものは終わった状態になるので、一度ホスト OS に戻ってきて、このコンテナをイメージ化する。

また、イメージ化したあとのコンテナはもう不要になるので削除しておく。

```
$ docker container commit apache-test centos:apache
$ docker container rm apache-test
```

次に、CentOS を各種サービス有効な状態で起動する。これには `sbin/init` を使った起動を行う。（つまりコマンドラインには入れない）

```
$ docker container run \
-it \
--tmpfs //tmp \
--tmpfs //run \
-v //sys/fs/cgroup://sys/fs/cgroup:ro \
--stop-signal SIGRTMIN+3 \
--name apache-test \
-h apache-test \
centos:apache \
//sbin/init
```

ここでもやはり、すべてのパスに対して Windows では二重のスラッシュが必要な点に注意。

ここで起動がうまく行った場合でも、 `/sbin/init` で起動しているからコマンドラインが現れないので、別のプロセスからこのコンテナに対して `/bin/bash` で入り直すということを行う。

別プロセスから以下のように exec を使ってシェルを起動する。

```
$ docker container exec -it apache-test //bin/bash
```

これで、起動中のコンテナに入ることができた状態（ `docker container attach` したのと同じ状態）なので、ここで apache を起動できる。

起動には `systemctl` コマンドを利用する。

```
[root@]# systemctl start httpd
[root@]# systemctl status httpd
[root@]# ps -ef
```

コンテナが起動した際に、自動的にサービスとして httpd が起動するように設定するには以下のようにする。

```
[root@]# systemctl enable httpd
```

ここまで来たら、コンテナ内で自動的にサーバが起動する設定まで完了したことになるので、このコンテナからイメージを作成して、コンテナを削除する。

```
$ docker container commit apache-test centos:apache-systemd-httpd
$ docker container rm -f apache-test
```

このコンテナの強制削除を行った時点で、最初に sbin/init で起動したプロセスの制御がホスト OS 側に戻ってくる。

新しく作成したイメージからコンテナを起動し、httpd が自動起動しているかをチェックする。

また、このときポートフォワーディングの設定も同時に行っておくことで、ホスト OS 側から、コンテナをローカルホストのように参照できる。

```
$ docker container run \
-it \
--tmpfs //tmp \
--tmpfs //run \
-v //sys/fs/cgroup://sys/fs/cgroup:ro \
--stop-signal SIGRTMIN+3 \
--name apache-test \
-h apache-test \
--publish 8080:80 \
centos:apache-systemd-httpd \
//sbin/init
```

ポートフォワーディングを行っていない場合、別プロセスでコンテナ内に exec してから `ip addr` 等を行って表示される inet の物理アドレスにアクセスしてもつながらなかった。（ネットワーク周りはよくわかってない）

尚、一度起動したコンテナは、明示的に削除しないと重複したときに怒られてしまう。

これを回避するには `--rm` オプションを付与する。これにより、コンテナが停止すると同時に自動的に削除されるようになる。


# ホスト OS 側のディレクトリをコンテナ化して別のコンテナに提供する

bind mount という機能を使って、ホスト OS のディレクトリをコンテナ化し、それを別のコンテナに対して提供している形を作ることができる。

`docker container run` の引数に `--mount` を指定して行う。

```
docker container run --mount type=bind,src=/ホストのパス,dst=/コンテナ内
```

上記が基本の形で、Windows の場合は Users の下の階層を当てて置くのが順当っぽい。これを実行すると Docker Desktop がアクセスを許可していいんですか的なことを言ってくるので許可する。

```
# HOME を基点にする場合
$ docker container run -it --rm --name mount-test --mount type=bind,src=$HOME/mount-test,dst=/root/testdir centos:latest //bin/bash
# カレントディレクトリを基点にする場合
$ docker container run -it --rm --name mount-test --mount type=bind,src="$(pwd)",dst=/root/testdir centos:latest //bin/bash
```

## ホスト OS 上のボリュームをマウントする

Mac の場合 `/var/lib/docker/volumes/` は存在しない。これは VM 上で Docker が動いているからである。

ver18.03 以降の Docker for mac では `~/Library/Containers/com.docker.docker/Data/vms/0` の場所にボリュームが存在するが、ボリュームなのでそのままではファイルは見えない。

ボリュームをマウントする場合は、ボリュームに名前をつけてマウントする。 `docker volume ls` 等で一覧を見ることができる。

```
docker container run -it --rm --name volume-test --mount type=volume,src=ボリューム名,dst=/root/testdir centps:latest /bin/bash
```

恐らく、ボリュームを利用する場合と直接ホスト OS のディレクトリをマウントする方法の大きな違いは、ボリュームであればコンテナ同士でこれを参照しあえるというところだと思われるけど確証なし。


# データ専用の用途で利用できるコンテナ busybox

そもそも `-v` オプションを利用すると、ボリュームをマウントできる。

その前提で次のコマンドを見ると……

```
docker container run -it -v /data0001 --name data01 busybox /bin/bash
```

busybox を利用して `/data0001` をマウントしている、という意味に読み解くことができる。

この `data0001` は他のコンテナから `--volumes-from` でマウント可能。

# ネットワーク

Docker のコンテナ同士での通信などは、Docker 内でのネットワークを構築して行う。

```
# 一覧表示
docker network ls
```

ネットワークもコンテナと同様に、名前をつけて作成しておき、これをコンテナの実行時に割り当てるよう設定できる。

```
docker network create --subnet 172.17.0.0/16 --attachable ネットワーク名
```

たとえば上記のネットワークを利用して、IP を固定した状態でコンテナを起動するとしたら以下のような感じ。

```
docker container run --network ネットワーク名 --ip 172.17.0.99 centos:latest
```

この辺は正直具体例と一緒に勉強しないとようわからん。

# リソースの分配

Docker ではコンテナごとに CPU やメモリ、ストレージ等を個別にどの程度利用していいか制限できる。

この辺は現段階では不要な知識と判断。







