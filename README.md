# Isucon でやることまとめ

## 自分の PC

### ~/.ssh/config に追記

```
HOST {接続名(isuconなど)}
    HostName {グローバルip}
    User {ユーザー名(たぶんisucon)}
    IdentityFile ~/.ssh/id_ed25519
```

### 公開鍵を持っていく

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub isucon@{グローバルip}
```

## サーバー側

### Taskfile のインストール

```
sudo sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin
```

```
curl -O https://raw.githubusercontent.com/Pigmin-isucon/doc/main/Taskfile.yml
```

### git の設定

```
task git-config
```

```
task keygen
```

出力された公開鍵をコピーして github のリポジトリに置いておく

```
cd ~
git init
git remote add origin {remote url}
```

サイズの大きいファイルは .gitignore に設定して push する

### setup (ここから先は並行で準備できる)

Taskfile の中の変数を設定してから実行. 多分手元に Taskfile を持ってきて手元で設定したほうがいい.  
ubuntu 環境であることを仮定しているので, alpine とかだったらコマンドも書き直す必要あり

```
task setup
```

- `task restart`で go のサービスが restart されるように & `task log`で go のログが見れるようにしておく
- alp を使う部分で`-m`を設定し, url をグループ化する.

### log の準備

nginx で alp を使う準備  
~/configにcpしてlocalで編集したほうがいい  
コピーしたやつを手元で書き直して、それをサーバーにアップしたらこれを元のディレクトリにコピーし直して編集を反映させる
/etc/nginx/nginx.conf にこれを書く  
書いたら`sudo nginx -t`でエラーが出ないことを確認する

```
    log_format ltsv "time:$time_local"
                    "\thost:$remote_addr"
                    "\tforwardedfor:$http_x_forwarded_for"
                    "\treq:$request"
                    "\tstatus:$status"
                    "\tmethod:$request_method"
                    "\turi:$request_uri"
                    "\tsize:$body_bytes_sent"
                    "\treferer:$http_referer"
                    "\tua:$http_user_agent"
                    "\treqtime:$request_time"
                    "\tcache:$upstream_http_x_cache"
                    "\truntime:$upstream_http_x_runtime"
                    "\tapptime:$upstream_response_time"
                    "\tvhost:$host";

    access_log /var/log/nginx/access.log ltsv;
```

mysql で pt-query-digest を使う準備  
/etc/mysql/mysql.conf.d/mysqld.cnf を書き換える(mariadb だと場所が違う)
場所はhttps://www.banana-juice.com/tech/articles/mysql/my-cnf このやり方で調べる

```
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_queyr_time = 0
```

`task slow-on`でも一応できるが, restart するとリセットされる

### go のアップデート(~~動作確認はしてない~~動かない)

go のバージョンが最新じゃないなら最新にする. `go.mod` でエラーが出るかもなので `go mod tidy`を忘れない

```
task go-update
```

### pprof, fgprof の準備

```go
import (
  ..
  _ "net/http/pprof"
  ..
)

func main() {
	http.DefaultServeMux.Handle("/debug/fgprof", fgprof.Handler())
	go func() {
		log.Println(http.ListenAndServe(":6060", nil))
	}()
}
```

## やること

- slow-query を見て index を貼る
- alp の結果を見て改善するエンドポイントを決める
  - 単純に遅い、リクエスト数が多い
  - pprof で具体的な個所を見つける
  - pprof で引っかからないなら fgprof を使う
- unix domain socket を使うと速い
  - https://pleiades.io/help/datagrip/how-to-connect-to-mysql-with-unix-sockets.html#step-1-locate-a-unix-socket-file
- データ数が少ないならオンメモリキャッシュを考える
- 異常なほど遅いなら別サーバーで処理する
- 画像は DB に保存しない
  - nginx の try_files を使う
- initialize でできることを考える

### 初期実装のデータベースについて

参考実装では2つのデータベースがあります。

- isupipe アプリケーションが利用するデータベース
- isudns PowerDNSのゾーン情報を格納するデータベース

### isupipe データベースのスキーマについて

isupipe データベースのスキーマは初期実装に含まれています

- webapp/sql/initdb.d/00_create_database.sql データベースおよびユーザの作成
- webapp/sql/initdb.d/10_schema.sql isupipe データベースのスキーマ

isupipe データベースを初期化するにはデータベースを `DROP DATABASE isupipe` および `CREATE DATABASE isupipe` で再作成し、

```sh
$ cat webapp/sql/initdb.d/10_schema.sql | sudo mysql isupipe
```

としたのち、データの初期化を行なってください。
