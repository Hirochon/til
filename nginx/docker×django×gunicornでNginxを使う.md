## docker×django×gunicornでNginxを使う (2020/2/8)(2020/2/9:前半)

### 今までの流れ
- Nginxを使うということで、[アプリケーションサーバーをGunicornに変更](https://github.com/Hirochon/til/blob/master/python/gunicorn-%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E3%82%92Gunicorn%E3%81%AB%E5%A4%89%E6%9B%B4.md)しました。
- DockerにてNginxを使用するにはボリュームの知識が必要不可欠ということで、勉強して、勢い余ってQiita記事化してしまう。笑

### 今回でNginxでの配信を絶対完成させたいところ (2020/2/8)
1. とりま全ての元となった[DjangoをDocker Composeでupしよう！ - Qiita](https://qiita.com/kyhei_0727/items/e0eb4cfa46d71258f1be)を見ながら、nginx.confについて解析していく。
2. [Deploying Gunicorn - Gunicorn 20.0.4 documentation](https://docs.gunicorn.org/en/stable/deploy.html)にてGunicorn×Nginxの設定テンプレートを見る。
3. [nginxについてまとめ(設定編) - Qiita](https://qiita.com/morrr/items/7c97f0d2e46f7a8ec967)と[(コピペ)nginx最大パフォーマンスを出すための基本設定](https://gist.github.com/koudaiii/386eb55a29b1adc19c5e)を見つつ、どれが正解なのかわからなくなってきたが、慎重に一つ一つ読む。
4. 設定全てが一つの記事に載っている訳ではないので、[nginxの基本設定を改めてちゃんと調べてみた - Qiita](https://qiita.com/hclo/items/35f00b266506a707447e)も参考にする。
5. どこもかしこもdocker-composeに無名ボリュームを作成してコンテナに保存してるみたいだが、永続化する意味がわからないので、個別にDockerfileを作成してCOPYを使うことにした。その際に[Docker公式Docs](https://docs.docker.com/compose/compose-file/#dockerfile)を参考にして、contextとdockerfike設定を使って、Dockerfileをディレクトリ内に2つ用意した。
6. Nginxの最新版を[DockerHub](https://hub.docker.com/_/nginx)にて確認
7. nginx.confの設定を完了→だけどコンソールでエラーが出てNginxで配信できていないっぽい。(原因:[CORSエラー](https://developer.mozilla.org/ja/docs/Web/HTTP/CORS/Errors/CORSMissingAllowOrigin))
8. 原因を追求し始める。(たしか５時間くらい足掻いて、寝落ちる)

---

### 力尽きた後なので、モチベ最悪ながらも昼過ぎ辺りから改善を始める (2020/2/9:前半)
1. 昨日の時点で色々試した。(覚えてる限りの昨日のlog)
    - 一番思い浮かぶS3側でとりあえず`Access-Control-Allow-Origin`を`*`に設定して全アクセス受け入れる→×
    - ※🔺のときCORSがいじれなくてAWSCLIからCORSのjsonファイルを送ったりした(笑)
    - とりま[AWSのCloudFrontに関しての公式サポート](https://aws.amazon.com/jp/premiumsupport/knowledge-center/no-access-control-allow-origin-error/)でぽいの見つけて実装→×
    - なんかNginx側のポートでアクセスした時にこっちのheaderにも`Access-Control-Allow-Origin`を書かなあかんのかな...(?)とか思ってheaderをイジる→×
    - ググって出てきたのをひたすら実装→×寝落ち
2. 昼過ぎからベッドに入りながら、最終手段について冷静に考え始める→とにかくエラーを消そう！
    - つながってるのCloudFront側だよなぁ...とか、CORSでアクセス可能なんちゃうん...とか思いながら、S3のCloudFront側からしかアクセスできないバケットポリシーを`*`に変更する
    - →◎!?!?!?
    - staticをGETするだけだからいっか。今はこのままで。と思った。
3. 40xと50xのエラー吐くファイルを自作して置いておく。
4. 『同ディレクトリにDockerfile２つあるの嫌だな...』とか思って、Nginx用のDockerfileをNginxのディレクトリ下に配置換え