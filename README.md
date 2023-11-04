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
curl -O https://raw.githubusercontent.com/Pigmin-isucon/doc/main/Taskfile.yml
```

### setup

Taskfile の中の変数を設定してから実行

```
task setup
```

### git の設定

```
task keygen
```

出力された公開鍵をコピーして github のリポジトリに置いておく

```
cd {.gitを置くdir}
git remote add origin {remote url}
```

サイズの大きいファイルは.gitignore に設定して、push する

### log の準備

nginx で alp を使う準備  
/etc/nginx/nginx.conf にこれを書く  
書いたら`systemctl nginx reload`

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

```
task slow-on
```

やめるときは`task slow-off`

### その他

- `task restart`でgoのアプリがrestartされるようにする

```
apt upgrade
```

やっておくといいかも  
go のバージョンを最新にする: https://go.dev/doc/install

## やること

- slow-query を見て index を貼る
- alp の結果を見て改善するエンドポイントを決める
  - 単純に遅い、リクエスト数が多い
  - pprof で具体的な個所を見つける
  - pprof で引っかからないなら fgprof を使う
- データ数が少ないならオンメモリキャッシュを考える
- 異常なほど遅いなら別サーバーで処理する
- 画像は DB に保存しない
  - nginx の try_files を使う
- initialize でできることを考える
