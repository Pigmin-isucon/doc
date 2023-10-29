# Isuconでやることまとめ

## 自分のPC

### ~/.ssh/configに追記
```
HOST {接続名(isuconなど)}
    HostName {グローバルip}
    User {ユーザー名(たぶんisucon)}
    IdentifyFile ~/.ssh/id_ed25519.pub
```

### 公開鍵を持っていく
```
ssh-copy-id -i ~/.ssh/id_ed25519.pub isucon@{グローバルip}
```


## サーバー側

### Taskfileのインストール
```
sudo sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin
curl -O https://raw.githubusercontent.com/Pigmin-isucon/doc/main/Taskfile.yml
```

### setup
Taskfileの中の変数を設定してから実行
```
task setup
```

### gitの設定
```
task ssh
```
出力された公開鍵をコピーしてgithubのリポジトリに置いておく
```
cd {.gitを置くdir}
git remote add origin {remote url}
```
サイズの大きいファイルは.gitignoreに設定して、pushする

### logの準備
nginxでalpを使う準備  
/etc/nginx/nginx.confにこれを書く  
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

mysqlでpt-query-digestを使う準備
```
task slow-on
```
やめるときは`task slow-off`

### その他
```
apt upgrade
```
やっておくといいかも  
goのバージョンを最新にする: https://go.dev/doc/install

## やること
- slow-queryを見てindexを貼る
- alpの結果を見て改善するエンドポイントを決める
    - 単純に遅い、リクエスト数が多い
    - pprofで具体的な個所を見つける
    - pprofで引っかからないならfgprofを使う
- データ数が少ないならオンメモリキャッシュを考える
- 異常なほど遅いなら別サーバーで処理する
- 画像はDBに保存しない
    - nginxのtry_filesを使う
- initializeでできることを考える

