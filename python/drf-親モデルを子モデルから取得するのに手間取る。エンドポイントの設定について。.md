## 親モデルを子モデルから取得するのに手間取る。エンドポイントの設定について。(2月1日)
### リポジトリ: [Shappar](https://github.com/Hirochon/Shappar)
### 実装したい内容: マイページ表示機能にて、必要なjsonをRESTfullなGETレスポンスする。
- ユーザごとにマイページを表示させるわけだけど、まずリレーションシップの子側は親モデルをどうやって取って来るかわからんかった。笑
1. 右の公式ドキュメントにて深さをいじれることを知った。(depth=1)→　https://www.django-rest-framework.org/api-guide/serializers/#specifying-nested-serialization
2. 右のサイトでシリアライザを２つ指定できることを知った。→　http://racchai.hatenablog.com/entry/2016/04/12/Django_REST_framework_%E8%B6%85%E5%85%A5%E9%96%80
3. シリアライザを２つ指定しつつ、深さを決めてやることによって、リレーションシップにて持ってきた奴を表示可能にできるとなんとなく予期する。
4. 結果成功し、以下のような`serializers.py`になった。

```python:serializers.py
from rest_framework import serializers
from django.contrib.auth import get_user_model
from .models import Mypage

class Account_MypageSerializer(serializers.ModelSerializer):
    class Meta:
        model = get_user_model()
        fields = ['id','username','usernonamae','iconimage']

class MypageSerializer(serializers.ModelSerializer):
    user = Account_MypageSerializer()
    class Meta:
        model = Mypage
        fields = ['user','introduction','homeimage']
        depth = 1
```

---------------------------------------------------------------------------------------------------------------------------------------------------------
- 初めにDBに収納されているIDについて勘違いをしていた。
5. モデルを呼び出した時に出力される『jsonデータのIDを親モデルのUUIDと共通にしたい！！』(困惑)とか思ってた。概念間違っていますね。笑
6. views.pyでエンドポイントによる引数(UUID)でデータを取得できるようにしようと方向転換した。
7. 実装したが、URLからデータを取得するとなると、使いにくすぎると共同で開発している友達とIsuueにて話し合って、UUIDからユーザIDに変更することを決定。
8. views.pyにて一度get_user_modelにて引数pkをuserIDにしたい時、objects.get(username=pk)によってpkでユーザを指定すると以下のようなコードとなった。

```python:views.py
from django.shortcuts import get_object_or_404
from rest_framework import views, status
from rest_framework.response import Response

from django.contrib.auth import get_user_model
from .models import Mypage
from .serializers import MypageSerializer

class MypageAPIView(views.APIView):
    """マイページ用APIクラス"""

    def get(self, request, pk, *args, **kwargs):
        """マイページモデルをGETメソッド"""

        """エンドポイントをユーザIDによってマイページを取得する場合"""
        user = get_user_model().objects.get(username=pk)
        mypage = get_object_or_404(Mypage, user_id=user.id)

        # """エンドポイントをUUIDによってマイページを取得する場合"""
        # mypage = get_object_or_404(Mypage, user_id=pk)

        serializer = MypageSerializer(instance=mypage)
        return Response(serializer.data, status.HTTP_200_OK)
```

まあ最終的にはUUIDを代入するわけだけど、get_user_modelを一度噛ませることによって、エンドポイントをuserID(username)によって実装できる。