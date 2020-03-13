## CircleCIをどんどん自分のものにしていくぅ (2020/3/11)

### なにをしているか
1. 早起きしました！w
    - 昨晩早寝したため、必然的に６時起きしました〜
    - 朝マック行きつつコンディション最高！
2. 今日は昨日たてた大手さんにエントリーする前の目標をこなしてく！
    1. CircleCIでPythonを使った簡単なプログラムをビルドしてみる！
        - 先駆者の方から`config.yml`をパクりつつ、全て自分で`config.yml`を書いてみる！
        - `executors`や`commands`,`jobs`,`steps`,`workfolws`,`requires`なども使って、可読性や便利な機能の経験を高めていきたい！
    2. CircleCIで環境構築を成功させる
        - PostgreSQLとの連携
        - パッケージのインストール
        - マイグレーション系
    3. 実際にShapparに搭載していく！
    4. 自動テスト機能を実装！
        - ECR/ECSの設定が上手く行くかなぁ〜ていう感じ
    5. Shapparのテストをちゃんと書く！
        - バリデーションをしっかり決める
        - ちなみにテスト初書きなので時間かかる予定!w
        - allauthパッケージのテストはスルーかなぁ
    6. CircleCIによる自動テスト機能を追加！
        - まあこれは項目に追加するだけなので楽な予定。
3. 以上のタスクをどこにも詰まらずいけたら、今日と明日ぐらいで...おわ...
    - **まあ未来のことを考えても仕方ないので、とりあえず今がんばります！！**
4. とりあえず①を完了させる！
    - 初めてということもあって、めっちゃtypoしたw
    - `executors`を`exectors`て書たり、`ciecleci`とか書いてエラー出たのは秘密！
    - **『何をわざわざこんなプログラムをCircleCIで実装しようとしてんだろ』**とか笑いながら書いてたw
    - 『はろはろだよ〜』と『Very Good!!』てワークフローで直列に出力するCircleCIのリポジトリはコチラw → [CircleCI-Python](https://github.com/Hirochon/circleci-python)
5. 謎いところでハマる...
    - 書き方がまずかったです。

    ```yaml:docker-compose.yml
    version: "3"
    services: 
        circleci-db:
            image: postgres:12.1
            environment:
                POSTGRES_USER: POSTGRES_USER
                POSTGRES_PASSWORD: POSTGRES_PASSWORD
            ports:
                - "5432:5432"
            volumes:
                - "dbdata:/var/lib/postgresql/data/"
    ```

    - `environment`がやばいです。

    ```yaml:docker-compose.yml
    version: "3"
    services: 
        circleci-db:
            image: postgres:12.1
            environment:
                - POSTGRES_USER
                - POSTGRES_PASSWORD
            ports:
                - "5432:5432"
            volumes:
                - "dbdata:/var/lib/postgresql/data/"
    ```

    - 普段docker-composeを使いまわしてるツケがきました。ちゃんと書くようにします。
6. CircleCIのDjangoの環境構築に成功！
    - 専用の設定ファイルの準備をしなくちゃいけない
    - キャッシュを利用したり、キャッシュを読み込んだり、権限を変更したりと、やることが多かった。
    - コマンド部分はすんなり通ってよかった！
7. この時点である会社さんの企業調べを始める！！
8. その企業さんとオンライン面談(面接?)！