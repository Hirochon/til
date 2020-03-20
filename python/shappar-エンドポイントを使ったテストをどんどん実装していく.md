## shappar-エンドポイントを使ったテストをどんどん実装していく (2020/3/20)

### リポジトリ [Shappar](https://github.com/Hirochon/Shappar)

### 今日の積み上げ
1. Makefileをテスト用に書き加える
    - `docker-compose run --rm shappar-back python3 manage.py test --settings=config.settings.local`
    - テストの範囲を決めれるけど、とりあえず今は全てのテストで。
2. はじめて〜〜〜のーテスット！
    - 初めてのテスト作成で仕様あんまわかんなかったけど．．．
    - 使ってるうちに慣れてきた！細かく設定し始めた！
    - pkも設定できたり、認証もできたり、DRFはやっぱりすごいと感じ始めてきた！！
        ```python:auth.py
        token = str(RefreshToken.for_user(self.user1).access_token)
        self.client.credentials(HTTP_AUTHORIZATION='JWT ' + token)
        ```
3. 投稿時のテストを作成！
    - やっぱり初めはテストをすごく感じられるPOSTメソッドのみのAPIViewクラスのテスト書いてくで！
    - 今までエンドポイントやリクエスト、レスポンス、バリデーションを作成してきた集大成をやってみるたいで、楽しいw
    - 途中で`for i option in enumerate(options)`使って、リストに代入とか考えたけど、appendメソッド使って代入する形を採用した
    - まあ結局『ココレスポンスイラナイ...』とか思って、消したんだけどね！
    - ここでは5つテスト作った！(正常:2methods,異常:3methods)
    - プルリクはコチラ → [PullRequest #148](https://github.com/Hirochon/Shappar/pull/148)
4. マイページに関するテストを作成！
    - お次にPATCH/GETメソッドを使ってるMypageAPIクラスでし！
    - 下のようにメソッドの外で定義して．．．
    - `TARGET_URL_WITH_PK = '/api/v1/users/{}/'`
    - 下のように`{}`の中身を設定することができる！！
    - `response = self.client.patch(self.TARGET_URL_WITH_PK.format('user1'), params, format='json')`
    - ここでは6つテスト作った！(正常:2methods,異常:4methods)
    - プルリクはコチラ → [PullRequest #151](https://github.com/Hirochon/Shappar/pull/151)
5. CircleCIに関していくつか発見
    - まずこの前から環境変数を設定してたけど、Pythonで参照できることを存じていなかった！！w
    - 環境変数を使ってClodFrontのURLを設定いたしました！
    - あと設定ファイルを替えているので、CircleCIで`STATIC_URL`が名前に使われることを思いだした！
6. お疲れ様です．．．