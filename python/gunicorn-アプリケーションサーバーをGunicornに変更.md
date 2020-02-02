## アプリケーションサーバーをGunicornに変更しました。(2020/2/3)
### リポジトリ: [Shappar](https://github.com/Hirochon/Shappar)

### そろそろDjango公式が非奨励の標準装備アプリケーションサーバーから移行したかった。
#### 1. とりま『Gunicorn Django』というキーワードでQiita記事を10記事くらい漁る。

やはり1記事だと信用できないので、ヒットしたいくつかの記事を読みます。その結果→知識の定着&技術へのきちんとした理解へ近づける。

この記事でアプリケーションサーバーとWebサーバの関係性が図示されていて分かりやすかった。↓

https://qiita.com/3244/items/c83d9449f0286707b2fb#gunicorn--django--aurora-mysql

#### 2. Gunicornがサーバーを起動する際の架け橋になっていたことを知る。

正直言うと、Nginxの実装に伴ってGunicornの存在も知ったので、Gunicornがそもそも何者なのか知らなかった。笑

上の記事に書いていることや他の記事を見て以下のことに気がついた。
    - pip installするだけで使えるようになる。
    - サーバーの起動に`python3 manage.py runserver 0.0.0.0:8000`ではなく`gunicorn config.wsgi -b 0.0.0.0:8000`といったコマンドを打っている。

#### 3. 『実装するのめっちゃ簡単なやつやん』と独り言をつぶやく

もちろんDocker環境でしか開発しないことが染み付いてる僕は速攻pypiでバージョンを調べ`requirements.txt`と`docker-compose.yml`にこう書き加えた。

```text:requirements.txt
gunicorn==20.0.4
```

```yaml:docker-compose.yml
command: gunicorn config.wsgi -b 0.0.0.0:8000
```

軽く起動コマンドの解説をする。
- `config.wsgi`でstartproject時に生成される`wsgi.py`のディレクトリを参照している。
- `-b 0.0.0.0:8000`でrunserverと同様でポートとホストを指定している。


#### 4. 成功

もう`python3 manage.py runserver`特有の`Quit the server with CONTROL-C.`の文がサーバー起動時に見れないと思うと悲しい。
