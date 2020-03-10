## Shapparの機能改善と面接の毎日が始まったよ (2020/3/9)

### 本日やったこと
1. ひたすらに面接対策！
    - まずは企業様のバリューやミッションを書き出す。
    - 企業様と過去に話した内容を元に大事にしていることを書き出す。
    - 自分の今まで積みかせねてきたもの(人生で大切にしていること、身につけた技術など)を書き出す。
    - 企業様が求めていそうなことをまとめて、🔺と重なっていることをドンドン繋げていく
    - 準備万端！あとは自身を持って、臨むだけ！
2. いざ面接！
    - まとめていたことはあまり使わなかった。笑
    - 面接官の方がすごく気さくに話しかけてくれて、また丁寧に話を聞いてくれてシンプルに嬉しかった。笑
    - 自分が今まで学んできた知識について、僕をいちエンジニアとして会話をして頂けて、楽しいひと時を過ごせた。笑
    - 終わってみて、『ここの回答こうすればよかったなぁ〜』とか色々と思い当たるけど、全力を出し切った気がする。
3. 友達とメンタル調整w
    - 今日あったことを報告しあいつつ、『仕方ないよね』とか『今はもうやるしかない』とか、次につながるようにメンタルを保護w
4. 同時にShapparについての相談も
    - 友達が色々とフロントの実装を頑張ってくれて、すごく見やすく使いやすく修正してくれた。
    - それをS3,ECR,ECS使ってデプロイして、実際に使ってみて、バグとか修正点とかを見つけた。
    - 僕もバックエンド側でバグを見つけたので、Issueに貼っつけて、改善に取り組む↓
5. Shapparにて`selected_num`を正しくレスポンスできないバグを発見。
    - そもそも`selected_num`の取得の原理は、アクセスしてきた【ユーザのUUID】と【optionのID】がPollテーブルに保存されているきなかで判断するプログラムだった。
    - そしてバグの原因は↓のプログラムにあった。

    ```python:views.py
    for data in datas['options']:
        flag = Poll.objects.filter(user_id=pk,option_id=data['id'])
        if len(flag) > 0:
            datas['selected_num'] = Option.objects.get(id=flag[0].option_id).select_num
        else:
            datas['selected_num'] = -1
        del data['id']
        del data['share_id']
        total += data['votes']
    ```

    - 上記のプログラムの場合、フィルタを使って`selected_num`を検出できても、後で投稿されていない【optionID】が見つかると、-1で上書きされていた。
    - なのでシンプルに以下のようにif文を用いて、辞書型datasに要素が無い場合のみ`selected_num`の値に-1を代入するように変更。

    ```python:views.py
    for data in datas['options']:
        flag = Poll.objects.filter(user_id=pk,option_id=data['id'])
        if len(flag) > 0:
            datas['selected_num'] = Option.objects.get(id=flag[0].option_id).select_num
        else:
            if not 'selected_num' in datas:
                datas['selected_num'] = -1
        del data['id']
        del data['share_id']
        total += data['votes']
    ```

6. トップのエンドポイントをフロントに変更した。
    - Shapparはユーザーにとって、静的ファイルでアプリを使っているので、Django側のサーバーで動いているものを配信する必要はなかった。
7. ローカルと本番との環境変更をしやすくした。
    - ローカル環境と本番環境で変更するファイルが3つもあったので、切り替えがしにくいと感じた。
    - 故にあるファイルのコメントアウトを付けて外すだけで、環境の切り替えを行えるように変更しました。
    - そして生まれたファイルが`manage_local.py`と`wsgi_local.py`でした。笑