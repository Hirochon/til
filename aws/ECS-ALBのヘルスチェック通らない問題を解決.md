## ECS-ALBのヘルスチェック通らない問題を解決 (2020/2/29)

### DevOpsを実現させるため、ECSを使いこなしたい！！
#### 就活の苦しい時期で久々(3日ぶり)の勉強かな...？
#### 面談や面接の予定が一切なくて、余裕がある休日にしっかりススメルゾ。

### 今日何をしたか？
1. とりあえず今までやってたことの復習
    - ECRにイメージをpushする方法は理解。(ログイン,Dockerfileのtag付けビルド,ECRに向けたtag付け,push)
    - ECSを使う際の手順
        1. タスク定義(AMIイメージ定義とかコンテナ(ポート,環境変数,ボリューム,CloudWatch(**設定しておくのはログとかバグ見つけるために必須**),順番)とかボリュームの関連付け)
        2. VPC周りの定義
        3. クラスター作成(VPC,Subnetに沿ったインスタンスの生成,数も指定)
        4. ロードバランサー作成(ACMを用いた証明書発行で常時SSL化,インスタンスの指定,ヘルスチェックの設定とか)
        5. Route53のAレコードにロードバランサーを登録
        6. クラスターでサービス作成(タスク指定,ロードバランサーとそれ用のコンテナ指定,AutoScaling指定,※まだ理解していないけどクラスター間の通信もここで設定？)
    - **詰まる**ヘルスチェックがクリアできねえええええ！！！！
2. 過去実験から出た結果を一旦整理と、どこから間違ってるのか確認作業を始める。
    - とりま理想的なコンテナの配置をQiitaとかチュートリアルみながら[やってみる](https://github.com/Hirochon/til/blob/master/aws/ECS-%E6%A6%82%E5%BF%B5%E3%82%92%E7%90%86%E8%A7%A3%E3%81%97%E3%81%A6%E3%81%93%E3%83%BC%E3%81%9C%E7%B7%A8.md)→動かない。
    - Goのアプリだったらヘルスチェックは何故か通った。(見てなかった気がする。通ってないかも。でもアプリが動いたことに[めっちゃ歓喜してたw](https://github.com/Hirochon/til/blob/master/aws/ECS-%E6%A6%82%E5%BF%B5%E8%A7%A3%E8%AA%AC&Go%E3%81%AE%E7%B0%A1%E5%8D%98%E3%81%AA%E3%82%A2%E3%83%97%E3%83%AA%E3%81%8B%E3%82%89%E3%82%BF%E3%82%B9%E3%82%AF%E5%AE%9A%E7%BE%A9.md))
    - コンテナ上でDjango初期装備サーバーを動かしても、インスタンス上では動くけど、ヘルスチェックは通らない。(※これはTILに書いてないけど26〜28のどこかでやってた。)
        - 『じゃあいいじゃん』みたいな事を思う方がいるかもだけど、ALBが機能しないので、`ALBと連携してる常時SSL化したドメインでアプリ開けない`し、`ECSのサービス止まるし`で**問題山積み**。
3. 他に誰か詰まっていないかを僕の独自脈を使って捜索！！ → **同じような方を発見！**
    - どうやら、ECSを使ったアプリは一般的に`Nginxコンテナの80番ポートで受けて、そこからバックエンドやAPIサーバーに通信する`ケースが多いらしい。→じゃあもうこうじゃないと動かないのかなと仮結論。
    - じゃあ僕のECSのタスク定義で間違っているのは以下の点であることが判明する。
        1. Djangoの初期装備サーバーで直に80ポートで受けて、コンテナポートを8000で受けるようなことをしてた。
        2. つまりNginxは使ってるけど、静的ファイルの配信メインで、バックエンド,APIサーバーにさばくようなことはしていない。
    - 次に見えてきたToDoは`WebサーバーNginxを使ってDjangoの初期サーバーを起動させること`。
4. 早速Nginxをロードバランサーの受け手としてタスクを定義。Djangoはいつもどおり。
    - ちょっと文字ばかりで気持ち悪いので、なんとなくDjangoを起動する際に設定すべきコマンドを載せときます。笑

    ```text:command.txt
    python,manage.py,runserver,0.0.0.0:8000
    ```

    - そうなんです。タスク定義でコマンドを設定する際は、空白ではなく`カンマ`で文字の分け目を判別するのです。
    - **結局ヘルスチェックOKもらえませんでした。**
5. よって問題はそれ以前であると仮定しました。NginxをDocker上で立ち上げてヘルスチェックをいただくことから始めます。
    - あれまてよ。そういえばDockerでNginxを単体で動かしたことねえw。まずはローカルで起動させるところから。
    - 右の記事を参考にして、`docker run -d -p 80:80 --name nginx-server ecs-nginx`こんなコマンドでとりま立ち上げてみる。[docker の nginx イメージの設定ファイルを眺めながら、独自ページを表示します。 - Qiita](https://qiita.com/mochizukikotaro/items/1d28888b78f401227ed6)
    - 無理でした笑。初期設定のnginx.confでサーバーを立ち上げてないので、自分でlocationを設定しなあかんことに気づき、右記事を参考にしつつ表示成功！[nginxでlocationが上手くいかない｜teratail](https://teratail.com/questions/114971)
    - これでローカルでは成功したので、ECSでやってみる。
        1. 再びECSの全設定を一から作り直す。
        2. なんとヘルスチェックが通る！！
        3. ECS使って以来はじめてヘルスチェック通った！！！成功やあああ！！！(相当歓喜)
6. 成功したので、当然Nginx→Djangoという形をタスク定義して、ECSで動かしてみる。
    - `CloudWatch`みたところ、あれ？何このエラー。`no live upstreams while connecting to upstream`。
    - どうやらupstreamが生きてないみたいですw
    - てかまずローカルで試すべきだった。
7. ローカルでやってみる。→案の定エラー。
    -　じゃあ今までどうやってたか。こうやってた。

    ```conf:nginx.conf
    upstream back_server {
        server back:8000 fail_timeout=0;
    }
    ```

    - docker-composeで定義したコンテナの名前とポートを使ってた。
    - ぐぐってると、`Gunicorn`で`systemd`をつかって`upstream`書いてる記事が多くて、これ使わなあかんのかーと。
    - 『また壁にぶち当たったかぁ。』と思って、発狂。
8. 風呂入って気分スッキリしに行く。
9. 知らぬ間に寝落ち。
10. そーいえば、かねてから`make migrations`,`make migrate`,を打つのが面倒くさい改善してと要望が来ていたので、`docker-compose up`する際の実行内容を書き換えました。
    - ちなみに右の記事を参考にしたよ！[[DockerCompose]Commandを複数行実行する方法 | エンジニアの眠れない夜](https://sleepless-se.net/2018/06/02/dockercomposecommand%E3%82%92%E8%A4%87%E6%95%B0%E8%A1%8C%E5%AE%9F%E8%A1%8C%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95/)

    ```yaml:docker-compose.yml
    version: "3"
    services: 
        back:
            build: "./"
            ports: 
                - "8000:8000"
            volumes:
                - "./:/code/"                                       # プロジェクトのルートディレクトリからマウントを取る
                - "staticdata:/code/static/"                        # volumesを用いたnginxとのコンテナ間のリソース共有
            # 本番環境
            # command: >
            # bash -c "python3 manage.py makemigrations &&
            # python3 manage.py migrate &&
            # gunicorn config.wsgi -b 0.0.0.0:8000"
            
            # テスト環境
            command: >
                bash -c "python3 manage.py makemigrations &&
                python3 manage.py migrate &&
                python3 manage.py runserver 0.0.0.0:8000"
            depends_on: 
                - db                                                # 2. docker-compose up時にdbから立ち上げる

        #     〜　以下略　〜     #
    ```

## てか翌日談なので、作業してる傍らで書いてるわけではなく、詳細に何してたかはかききれてないよ！