## Shapparの細かな機能の実装と修正(2020/2/12)

### リポジトリ: [Shappar](https://github.com/Hirochon/Shappar)

### 今日の積み上げ
1. JWT(Json Web Token:『ジョット』と読む)認証の使い方について詳しくなった。
    - 認証情報を含んだJSON形式のデータをHTTPヘッダで送信できるようにエンコードしたトークン。
    - JWT認証を行うためのアプリはインストール済みなので、エンドポイントの設定を行ってあげるだけで実装完了。
    - Vueにてログインの際にAPIでPOSTして、トークンをlocalstorageに保存→認証という形を取ることがわかった。
    - `views.py`にて`permission_class = [IsAuthenticatedOrReadOnly]`を設定するかしないかでログイン必須かどうかを決める。
2. migrationsが反映されないバグを修正。→ボリュームを使うことによって、ファイルが更新された時に反映できるようにしました。
3. 元々マイページにてレスポンスするjsonの形式が計画していたものと異なっていた。→シリアライザを変更することで実装完了。

```python:serializers.py
"""Before"""

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

```python:serializers.py
"""After"""

from rest_framework import serializers
from django.contrib.auth import get_user_model
from .models import Mypage

class MypageSerializer(serializers.ModelSerializer):
    """マイページ用シリアライザ"""

    user_id = serializers.ReadOnlyField(source='user.id')
    username = serializers.CharField(source='user.username')
    usernonamae = serializers.CharField(source='user.usernonamae')
    iconimage = serializers.ImageField(source='user.iconimage')

    class Meta:
        model = Mypage
        fields = ['user_id','username','usernonamae','introduction','iconimage','homeimage']
```

4. Djangoのバージョンを2.2.10へバージョンアップ
あげれる時に上げておこう精神です！

5. 静的ファイルを読み込むように変更
    1. DjangoがAPIサーバー、VueCLI3がフロントで動いてて、`npm run build`を行った際にトップディレクトリのtemplatesにHTMLを吐き出すように設定しているのだが、何故かCloudFrontが配信されなかった。
    2. Djangoがcssやjsを読み込めていなかったため、読み込めるようにHTMLを修正したところ解決した。