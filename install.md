
# install

* ここで言うインストールとは Docker Desktop のこと
* Windows 環境では *そもそも Hyper-V を利用するため Windows が Pro 以上でなくてはならない*
* Mac 版は普通に入ったが、最初に起動して認証通さないとコマンドのパスが通らない様子

* [Install Docker Desktop on Windows \| Docker Documentation](https://docs.docker.com/docker-for-windows/install/)
* [Docker Hub](https://hub.docker.com/) ※初回はサインアップが必要

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








