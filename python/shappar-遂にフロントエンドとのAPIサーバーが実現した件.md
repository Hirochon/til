## 遂にフロントエンドとのAPIサーバーが実現した件(2020/2/13)

### リポジトリ: [Shappar](https://github.com/Hirochon/Shappar)

### 具体的に何をしてたか
1. 友達と初めてコワーキングスペースに行って、Shapparを開発していた。(※バレンタインデー前日ということでチョコを頂きました。笑)
2. staticフォルダの中身を変更した時に、`collectstatic`を用いれば、通常はS3のストレージに保存されるのだが、されなかったことを発見...
3. どうやらstaticフォルダにマウントを取っていなかった影響で、コンテナ内に変更が反映されず、`collectstatic`時も変更しないままS3に送ってたみたい。
4. という訳で、`docker-compose`で行ったBefore→Afterを載せる。

```yaml:docker-compose.yml
version: "3"
services: 
    ## 〜中略〜 ## 
    nginx:
        build: "./nginx/"
        ports:
            - "80:80"
        volumes:
            - "staticdata:/opt/apps/static/"
        depends_on: 
            - back
volumes:
    dbdata:
    staticdata:
```

```yaml:docker-compose.yml
version: "3"
services: 
    ## 〜中略〜 ## 
    nginx:
        build: "./nginx/"
        ports:
            - "80:80"
        volumes:
            - "./static/:/opt/apps/static/"
            - "staticdata:/opt/apps/static/"
        depends_on: 
            - back
volumes:
    dbdata:
    staticdata:
```

5. 反映されないのも、今見てみれば当然。
    - 言い訳としてShappar_backコンテナにて、Shappar全デイレクトリをコンテナ内の`code`に`COPY`していたので、staticもコピーされていたと思っていた。
    - だけど実際は永続化しているstaticdataは`opt/apps/static`下に関してコンテナの永続化を行っているので、`code`は関係なかった。
6. `accounts`ディレクトリの`migrations`下を消去した。
    - 初期ユーザーのテーブルのカラムを色々と変えたので、いっそ`migrations`ディレクトリを全消去して、DBを書き換えるほうが早かったので、初期化した。
7. DjangoのJWT認証のパッケージである`djoser`の機能の一つ。フロントでのログイン中のユーザ取得機能を追加。至って簡単`path('auth/', include('djoser.urls')),`を追加するだけ。