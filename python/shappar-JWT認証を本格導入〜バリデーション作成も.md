## shappar-JWT認証を本格導入〜バリデーション作成も (2020/3/18)

### リポジトリ [Shappar](https://github.com/Hirochon/Shappar)

### 今日の積み上げ
1. 超大手ベンチャーの書類選考通ってヤッホイ
2. JWTの日！！！
    - まずは`JWT`についてちゃんと調べる
    - 実際に用いているパッケージ(`simple-jwt`)についてちゃんと調べる
    - パッケージ自体に`views.py`とかでユーザ情報を取得できる手段はなさそう...
    - そして`[IsAuthenticated]`でログインユーザーのみの制限をかけることができる。
3. びっくり仰天！？それでリクエストユーザー判断できるんか〜〜い！！
    - 普通の`allauth`で使っていたリクエストユーザーを取得する方法が使えちゃったw
    - ひらめいて、試しにやってみたらいけて、簡単すぎぃぃぃぃぃぃって叫んだよね。
4. `get_object_or_404`の優秀さ
    - クエリセットを取得する際にDBに存在しなかった瞬間404レスポンスする神モジュールを発見
5. 新たに他ユーザによる不正アクセス防止のために関数を作成
    - ↓これみたいな
    ```python:views.py
    def Response_unauthorized():
        return Response({"detail":"権限がありません"},status.HTTP_401_UNAUTHORIZED)

    class sample(views.APIView):
        def patch(self,request,*args,**kwargs)
        if request.user.username != pk:
            return Response_unauthorized()
    ```

6. JWTが本当に機能しているかどうか実験！
    - コチラのサイトで色々と試す → https://jwt.io/
    - 例えばトークンデコードしてみたり
    - デコードした情報を改変して、それをエンコードしたトークンをCurlしてみたり　→　しっかりブロックされました！！
7. Shapparの機能を全てログインユーザーにしか扱えないようにする！
8. 本番環境とローカル環境で権限の設定を切り替えができるように変更
    - ここでグローバルに認証を聞かせる方法を学びつつ...→　[Permissions - Django REST framework](https://www.django-rest-framework.org/api-guide/permissions/)
9. ここからひたすらバリデーション作りです！！(エラーレスポンス考えたり...)
    - ↓こちらのIssueでどれだけやったかは見れると思います！！(ちゅかれたー)
    - [Issue](https://github.com/Hirochon/Shappar/issues/139)
10. 就活の日程調整しつつ、ねおち〜〜。