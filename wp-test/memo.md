
## 起動コマンド

```
docker-compose -f ./docker-compose.yml up -d
```

## yml 内のハイフン

yml ファイルの中にハイフンが出てくるところとそうでないところがある。

このハイフンはどうやら配列であることを意味するプレフィックスのようで、引数の種類によっては配列で指定がなされていないといけない場所があるらしく、それでハイフンがついているところとついていないところがある。



## mysql が起動しない

docker-compose.yml に非同期的な処理についての記載が無いと失敗することがある。

```
# 一番最後のコマンド部分
    db:
        image: mariadb:latest
        env_file:
            - ./mysql.env
        networks:
            mynet:
                ipv4_address: 172.20.0.2
        volumes:
            - mysql-vol:/var/lib/mysql
        command:
            'mysqld --innodb-flush-method=fsync'

```

一度、起動に失敗している場合でも、関連ファイルがマウントしたディレクトリに作成されていることがあるので、消してみるとうまくいくことがある。


