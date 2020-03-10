## ECS-Nginx×Dockerで上手くDjangoサーバーを立ち上げ(2020/3/4)

### すみません...後後後日談なので詳しい流れを覚えてません！笑

### ついにECSを使ったデプロイで、コンテナ間の通信に成功しました！

### やっとこと
1. 確か前日にNginxのupstreamが何故`docker-compose.yml`で定義したサービスと連携してるのか疑問に思った。
2. ECSはコンテナを扱うサービスであって、`docker-compose`を扱うものではないと前々から思っていた。
    - つまりDockerfileのみでNginx→Djangoサーバーでつながる何かを見つけたらどうにかなるんじゃないか！？！？！？
3. 🔺のことを思って、早速Dockerfileのみで立ち上げてやろうと思って、やってみるが**Djangoサーバーすら立ち上がらない**。
    - `docker run --rm ecs-django python manage.py runserver 0.0.0.0:8000`を使うのは`docker-compose`で使ってたコマンドから察してたけど、どうすんだ？
    - ぐぐる！！
4. いまさらDocker×Djangoの入門記事に絞って探す！(※上級者は楽だから、すーーーーぐdocker-compose使うため笑)
    - でもDjangoを`docker-compose`で立ち上げる方法はわかるのに、基本となる`Dockerfile`で立ち上げ方がわからないのはたしかに嫌ですね笑
    - ↓のコマンドが右の記事が見つかる→[Docker で Django 開発環境を構築する](https://leben.mobi/blog/django_docker/python/)
        `docker run --rm -v "$PWD":/code/ -p 8000:8000 my-django-app python manage.py runserver 0.0.0.0:8000`
    - `-p`でポート設定してるのはすぐ理解。
    - -vでマウント取ってるのは分かるけど...**PWD**とはなんぞや！！笑
    - 右の記事でカレントディレクトリの絶対参照を指していたのが分かる。『こんなのあんのやぁ...』という感じw　→　[Volume相対パス指定でもdocker runがしたい！ - Qiita](https://qiita.com/KentoDodo/items/24117025924d64998110)
    - という訳でちょっと改造した下コマンドでDjangoのサーバーの阿智上げに成功しました。
        `docker run --rm -v $PWD:/code/ --name ecs-django -p 8000:8000 ecs-django python manage.py runserver 0.0.0.0:8000`
5. はい。Nginxも立ち上げてみたけど、ちゅながりません。はぁ...
6. 次々に壁にぶつかるから疲れつつ、ばんかい！！！
    - Nginxでジャストなことをしている方を発見！
    - [docker上のnginxから、別のコンテナのwebへリバースプロキシさせる - Qiita](https://qiita.com/74th/items/3545366f5f66eb70ff85)
    - `--link`がいるんですね...。そして以下コマンドの解を導き出す！
    - `docker run -p 80:80 --link ecs-django:ecs-django --name ecs-nginx ecs-nginx`
7. なるほど。Dockerコマンドでここまでの事をして、やっっっっっっっっっっとNginxのupstreamがサービスとして見つけてくれるんですね！！
8. 『`docker-compose.yml`優秀だなぁ...』とか思いつつ、『なんでECSでこれ使えへんのやろ...』とか言ってると、それをクリアできる様な、学習コストゴリ高そうな`ecs-cli`を見つける。
    - 一般的に使われてなさそうなので、タスクキルw
9. まとめると『④⑥でやってたことをECSで実現すれば良い！！』ということなので、ECSでの設定方法探しの旅に出かける。
10. 本当にめちゃめちゃQiQiった(Qiita検索エンジンで調べたw)。
    - どうやらECSをコンソールで扱ってる方が少ないみたい。
    - みーーーんなすーーーぐTerraform使うんだもん。笑
    - 仕方なくTerraformで設定してるパラメータを読むことに...笑
    - なんか意外と読めたので、ようやく良い記事を見つけました！！linkを設定しとる！！
    - [AWS ECSでサービス運用するために必要（そう）なこと - Qiita](https://qiita.com/uchihara/items/8647c94f7af607a9a236)
11. 早速コンソールのECSでlinkを設定できるとこを...
    - 探しまくる！
    - 見つかる！
    - タスク定義！
12. 繋がったあああああああああああああああああああああああああああああああああああああああああああああああああああああ！！！！！！！！！！！！！！！！！！！！！
13. **【喜び】・【達成感】・【感動】→割とマジで本気で叫びました。**
14. 以上AWS ECSを使ったNginx→Djangoのデプロイに成功した話でした。