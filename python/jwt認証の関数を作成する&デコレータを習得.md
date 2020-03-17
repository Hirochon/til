## jwt認証の関数を作成に取り掛かる&デコレータを習得 (2020/3/17)

## 今日やったこと
1. 就活関係で色々やるw
2. 面接挑むに向けていろいろたいさーく
3. 面接に挑む！
    - 面接官の方が、かねてからずっとしたかった髪色&髪型だったので、良いなぁと思ったw
4. 適性検査的なのを受ける！
5. 認証について遂に本格的にやりはじめる！
6. その前にちょっとPythonの学習(デコレーターとは？)
    - ソースコードを書き換えずに既存の関数に変更を加えたい事がある。
    - よく知られているのは、引数として何が渡されたか見るためのデバック文の追加だ。
    - **デコレータ**は、入力として関数をひとつ取り、別の関数を返す関数である。
    - 次のようなトリックたちを使って、デコレートしていく。
        - `*args`と`**kwargs`
        - 関数内関数
        - 引数としての関数
    - また次のような関数`document_it()`を作成した。
        - 関数名と引数の値を表示する
        - その引数を渡して関数を実行する
        - 結果を表示する
        - 実際に使うために変更後の関数を返す
    - コードは次のようになる
        ```python:decorator.py
        def document_it(func):
            def new_function(*args, **kwargs):
                print('Running function:', func.__name__)
                print('Positional arguments:', args)
                print('Keyword arguments:', kwargs)
                result = func(*args, **kwargs)
                print('Result:', result)
                return result
            return new_function
        ```
    
    - また試しに次の関数も定義
        ```python:decorator.py
        def add_ints(a,b):
            return a + b
        ```
    
    - 一旦実行するとこう。
        ```python:zikkou.py
        add_ints(3,5)
        # 8
        ```

    - 手動で関数内関数を使うとこう
        ```python:zikkou.py
        cooler_add_ints = document_it(add_ints)
        cooler_add_ints(3, 5)
        # Running function: add_ints
        # Positional arguments: (3, 5)
        # Keyword arguments: {}
        # Result: 8
        # 8
        ```
    
    - ここまでの感想
        1. まずね。`__name__`とか知らんかったよね。
        2. ずうううううううううううううううーーーーーーーーーっっっっっっっと不思議だった`*args`と`**kwargs`の正体をつきとめましたw
        3. `*args`は引数をタプルとして維持してるやつ
        4. `**kwargs`は`key1=1`みたいに引数で指定すると、辞書化してくれるやつ
    
    - まあいちいち代入とか、関数定義しなくても良いよ〜ってのが**デコレータ**！！
    ```python:zikken.py
    @document_it
    def add_ints(a,b):
        return a + b
    ```

7. DjangoでJWT認証使っている方がいらっしゃった。 →　[[Django]Rest Framework JWTでのパーミッション - Qiita](https://qiita.com/karintou/items/a68d1a86f3b195dcc3c1)

    ```python:permission.py
    from functools import wraps
    from django.http import JsonResponse
    from django.contrib.auth.models import AnonymousUser
    from rest_framework import status
    from rest_framework.exceptions import ValidationError
    from rest_framework_jwt.serializers import VerifyJSONWebTokenSerializer

    def check_user_auth(method, auth):
        def decorator(view_func):

            @wraps(view_func)   # functools.wrapsをインポートして、view_funcを変数の関数的な感じで定義する
            def _decorator(request, *arg, **kwargs):
                if isinstance(method, list):
                    methods = method
                else:
                    methods = [method]

                # unrestricted method
                if request.method not in methods:
                    return view_func(request, *arg, **kwargs)

                user = _get_user_from_token(request)

                # anonymous user
                if not hasattr(user, 'authorization'):
                    return JsonResponse({
                        'auth': False
                    }, status=status.HTTP_401_UNAUTHORIZED)

                if isinstance(auth, list):
                    auths = auth
                else:
                    auths = [auth]

                if user.authorization.auth in auths:
                    return view_func(request, *arg, **kwargs)

                # does not have permission
                return JsonResponse({
                        'auth': False
                    }, status=status.HTTP_401_UNAUTHORIZED)

            return _decorator
        return decorator

    def _get_user_from_token(request):
        token = request.META.get('HTTP_AUTHORIZATION', ' ').split(' ')[1]
        try:
            valid_data = VerifyJSONWebTokenSerializer().validate({'token': token})
            return valid_data['user']
        except ValidationError:
            return AnonymousUser()

    ```

    - 上記ファイルを参考にしたらいいかもだけど、使ってるパッケージが違うことに気付く。笑