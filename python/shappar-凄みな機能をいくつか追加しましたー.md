## 凄みな機能をいくつか追加しましたー (2020/3/7)

### 開発中のリポジトリ [Shappar](https://github.com/Hirochon/Shappar)

### やったこと！！
1. ページネーション機能の実装！！！
    - 詳細説明
    1. Twi○terをイメージした時に、フォローしているツイートを表示するわけだけど、全て表示はしていないよね！？
    2. もちろんShapparでもその機能を実装する必要があるわけです(｡･_･｡)
    3. そこでページネーションを実装するわけですが...パッケージインストールしても実装できない！？！？
    4. 僕のDjangoのプログラムが複雑にモデルが組み合わさっていて、無理みたい。自作していきます！笑
    5. 思いつくまで色々過程はあったのですが、端折ってこんなプログラムコードになりました。
    - `queryset = Post.objects.all().order_by('-created_at')[:10]`
    6. 投稿を全て取得するのですが、作成の逆順に取得するもの。そしてスライス使って10件制限をかけています。
    7. Django公式ドキュメントが『こんなことできるよ！』とこっそり言ってました。見つけるのには苦労したでな。笑
    8. さて取得件数はこれで良いとして、次の10件を取得するときはどうするでしょうか！！！？？？
    9. 次に取得したい10件の投稿より一個前の投稿IDを使うことによって、作成日時でフィルタをかけた訳です！まあこんな感じ↓

    ```python:views.py
    if 'pid' in request.GET:
        post_basis = Post.objects.get(id=request.GET['pid'])
        queryset = Post.objects.filter(created_at__lt=post_basis.created_at).order_by('-created_at')[:10]
    else:
        queryset = Post.objects.all().order_by('-created_at')[:10]
    ```

    10. 我ながら綺麗にかけたと思ってます！！
        - クエリパラメータにより取得した投稿IDを使って、特定のクエリセットを取得しやす！
        - クエリセットの作成日時より下のフィルタをかけつつ、作成日時を逆順にして10件だけ取得です。笑
        - `pid`→投稿ID(post_idの略)
    11. ページネーションの完成どす！
        - ※更新する直前の投稿IDを取得できるようにフロントが返してくれてます。
2. 投票状況更新機能を実装！
    1. やはり今現在投票結果がどうなっているのか木になりますよね！
    2. Twi○terでは更新できな...
    3. Shapparではできるようにします！！！(他社との差別化w)
    4. といっても、投稿一覧を投稿一見に変更して返すだけ。笑
3. 検索機能の実装！！！
    1. SNSといえば、おきまりの検索フィルターですよねw
    2. クエリパラメータをエンドポイントに設定してもらって、フィルタして実装です！
    3. ただし検索結果にもページネーションをつけるので、少し難易度上がりました。笑
    4. 結果的に検索、ページネーション含めこんなプログラムに。(後日に書いてるので、変更したりして、若干実際とは違うかも)

    ```python:views.py
    if 'q' in request.GET:
        if 'pid' in request.GET:
            post_basis = Post.objects.get(id=request.GET['pid'])
            queryset = Post.objects.filter(created_at__lt=post_basis.created_at, question__contains=request.GET['q']).order_by('-created_at')[:10]
        else:
            queryset = Post.objects.filter(question__contains=request.GET['q']).order_by('-created_at')[:10]
    elif 'pid' in request.GET:
        post_basis = Post.objects.get(id=request.GET['pid'])
        queryset = Post.objects.filter(created_at__lt=post_basis.created_at).order_by('-created_at')[:10]
    else:
        queryset = Post.objects.all().order_by('-created_at')[:10]
    ```

    5. クールなメソッドチェーンがございます。笑
4. 投票時にも結果が表示できるように、レスポンスにOptionも返すように変更！
